# MediCare Smart Healthcare System — Study Guide

## Project Overview
A Spring Boot web application for managing users, patients, doctors, appointments, medical records, and notifications. Uses JWT authentication, Spring Data JPA, Thymeleaf templates, and a small client-side static UI.

## Purpose
This project demonstrates a basic hospital/clinic management system covering common backend responsibilities: authentication, role-based access, CRUD for domain objects, appointment lifecycle, and medical record management.

## Tech Stack
- Java 17+
- Spring Boot (Web, Security, Data JPA)
- Hibernate / Jakarta Persistence
- Lombok
- Thymeleaf templates
- Maven

## Quick Start
1. Ensure Java 17+ and Maven are installed.
2. Configure DB in `src/main/resources/application.properties` (embedded H2 is common for dev).
3. Build and run:

```bash
mvn clean package
mvn spring-boot:run
# or run the packaged JAR
java -jar target/*.jar
```

4. Default admin account is created at startup from properties `app.admin.username` and `app.admin.password`.

## Important Files
- Application entry: src/main/java/com/medicare/project/Application.java
- Security: src/main/java/com/medicare/project/configuration/SecurityConfig.java
- Data seeder: src/main/java/com/medicare/project/configuration/DataInitializer.java
- Auth controller: src/main/java/com/medicare/project/controller/AuthController.java
- Patient controller: src/main/java/com/medicare/project/controller/PatientController.java
- Doctor controller: src/main/java/com/medicare/project/controller/DoctorController.java
- Appointment controller: src/main/java/com/medicare/project/controller/AppointmentController.java
- Medical records: src/main/java/com/medicare/project/controller/MedicalRecordController.java
- Entities: src/main/java/com/medicare/project/entity/
- Repositories: src/main/java/com/medicare/project/repository/
- Templates: src/main/resources/templates/
- Static: src/main/resources/static/

## Data Model (core entities)
- `User` — authentication account; has many-to-many `Role`.
- `Role` — ROLE_ADMIN, ROLE_DOCTOR, ROLE_PATIENT, ROLE_STAFF.
- `Patient` — patient domain profile; optional one-to-one link to `User`.
- `Doctor` — doctor domain profile; optional one-to-one link to `User`.
- `Appointment` — links `Patient` and `Doctor`; fields: date, timeSlot, status (PENDING/CONFIRMED/COMPLETED/CANCELLED).
- `MedicalRecord` — links to `Patient`, `Doctor`, optional `Appointment`.

## Authentication & Authorization
- JWT-based stateless authentication.
- `POST /auth/login` authenticates via `AuthenticationManager` and returns a JWT.
- Client must send `Authorization: Bearer <token>` for protected endpoints.
- `SecurityConfig` exposes public endpoints for login/register and static pages; protects API routes under `/patient/**`, `/doctor/**`, `/appointment/**`, `/medical-record/**`, etc.
- Role-based route restrictions exist for certain actions (e.g., confirming/completing appointments requires DOCTOR or ADMIN).

## Main API Overview
- Auth
  - `POST /auth/login` — login, returns token, username, role, fullName.
  - `POST /auth/register` — registers a user and creates a `Patient` profile.
  - `POST /auth/forgot-password` — generates a temporary password and returns it (no email delivery in current code).
  - `POST /auth/change-password` — change password for authenticated user.
  - `GET /auth/me` — returns authenticated user info.
- Patient
  - `GET /patient/me`, `PUT /patient/me` — view/update own profile.
  - `GET /patient`, `GET /patient/{id}`, `DELETE /patient/{id}` — admin operations.
- Doctor
  - `GET /doctor`, `GET /doctor/available`, `GET /doctor/specialization/{spec}`, `GET /doctor/{id}`
  - `POST /doctor` — admin adds doctor (creates `User` and returns temp credentials).
  - `DELETE /doctor/{id}` — removes doctor and linked user.
- Appointment
  - `GET /appointment/slots?doctorId=&date=` — shows booked slots and all slots.
  - `POST /appointment/book` — patient books a slot with validations.
  - `GET /appointment/my` — returns appointments for the current user (patient, doctor, or all for admin).
  - `POST /appointment/confirm/{id}`, `POST /appointment/complete/{id}`, `POST /appointment/cancel/{id}` — lifecycle operations.
- Medical Record
  - `POST /medical-record/complete-visit` — create medical record and mark appointment completed.
  - `POST /medical-record/add` — standalone record creation.
  - `GET /medical-record/my`, `GET /medical-record/patient/{id}`, `GET /medical-record` — fetch records.

## Typical Request Flow — Booking
1. Patient logs in → gets JWT.
2. Client requests available slots (`GET /appointment/slots`).
3. Patient posts booking (`POST /appointment/book`) with JWT.
4. Server validates date/slot/conflict and creates `Appointment` with status `PENDING`.
5. NotificationService is used to notify doctor/admins.
6. Doctor confirms (`POST /appointment/confirm/{id}`) → status `CONFIRMED`.
7. After visit, doctor creates a `MedicalRecord` and may mark the appointment `COMPLETED`.

## Notable Implementation Details & Considerations
- `forgot-password` returns temporary password directly; consider secure reset token + email flow for production.
- Manual cascading deletes in controllers; consider JPA cascade settings or DB FK constraints.
- Controllers often return `Map` responses — stronger API surface uses DTOs and `@Valid`.
- No global exception handler; add `@ControllerAdvice` for consistent API errors.
- Date/time uses `LocalDate` and string parsing — consider timezone implications for real deployments.
- Role checks are partly in `SecurityConfig` and partly runtime; consider consistent method-level security (`@PreAuthorize`).

## Testing
- Run tests:

```bash
mvn test
```

- Recommended tests:
  - Integration tests for booking and medical record flows (SpringBootTest + test DB).
  - Unit tests for `NotificationService`, `JwtUtil`, and controller logic.

## Suggested Improvements (homework)
- Implement secure password-reset (token + email) instead of returning temp passwords in responses.
- Replace `Map` responses with typed DTOs and use `@Valid` on request bodies.
- Add global error handling via `@ControllerAdvice`.
- Add OpenAPI/Swagger documentation.
- Add audit fields and configure proper JPA cascade/foreign keys.
- Add unit and integration tests for critical flows.

## Study Exercises
- Exercise 1: Trace the booking flow from UI to DB; list involved controllers, services, and repositories.
- Exercise 2: Implement an email-based password reset: create token entity, send email, and secure reset endpoint.
- Exercise 3: Convert `POST /appointment/book` to accept a typed DTO and validate fields with Bean Validation.
- Exercise 4: Write an integration test: register user → create appointment → doctor confirm → doctor adds medical record.

## Resources
- Spring Boot: https://spring.io/projects/spring-boot
- Spring Security: https://spring.io/guides/gs/securing-web/
- Spring Data JPA / Hibernate docs

---

Created by the code review assistant. For conversion to PDF, see the companion file `PRINT_INSTRUCTIONS.md`.
