# AI Symptom Checker — Study & Presentation Guide

This guide is designed to help you explain the **AI Symptom Checker** module of the MediCare Smart Healthcare System to your college professors or peers. It covers the architecture, the AI integration, and the fallback safety mechanism.

---

## 1. Module Overview
The AI Symptom Checker is a smart diagnostic assistant built into the patient dashboard. It allows patients to type their symptoms in plain English (e.g., *"I have severe head pain and dizziness"*) and instantly receive a structured medical recommendation.

**Key Features to Highlight in your Presentation:**
- **Natural Language Processing:** Understands plain text symptoms.
- **Structured JSON Responses:** Forces the AI to output machine-readable data (Condition, Specialist, Department, Urgency, Advice).
- **Zero-Config AI:** Uses a free, keyless AI provider (Pollinations AI) to avoid API key management.
- **High Reliability:** Implements a graceful "Fallback Engine" so the system never crashes if the AI goes down.
- **Smart Routing:** Generates dynamic UI buttons to immediately book an appointment with the recommended specialist.

---

## 2. Technical Architecture & Data Flow

You can explain the system as a 3-tier architecture:

### A. Frontend (Presentation Layer)
- **File:** `src/main/resources/templates/symptom-checker.html`
- **Action:** The patient submits a form. JavaScript intercepts the submission, attaches the user's JWT (JSON Web Token) for security, and sends a `POST` request to the backend.
- **Rendering:** Once the backend replies, the frontend dynamically builds a Result Card. It color-codes the urgency (e.g., Red for Emergency) and creates a dynamic booking link (e.g., `/doctors?dept=Neurology`).

### B. Controller (Routing & Validation Layer)
- **File:** `AiController.java`
- **Endpoint:** `POST /ai/analyze`
- **Action:** Acts as a gatekeeper. It validates the input to ensure the patient actually typed something and that the text isn't too long (preventing malicious payloads or API abuse). If valid, it passes the data to the Service layer.

### C. Service (Business Logic & AI Layer)
- **File:** `AiSymptomService.java`
- **Action:** This is the brain of the module. It orchestrates the HTTP call to the external AI model and handles any failures.

---

## 3. How the AI Integration Works (Prompt Engineering)

The most impressive part to explain is how the backend forces a conversational AI to act like an API. 

1. **The Prompt:** The Java backend wraps the patient's symptoms in a strict set of instructions (Prompt Engineering). 
2. **The Request:** It sends a REST request to `https://text.pollinations.ai/openai` using Spring's `RestTemplate`.
3. **The Enforcement:** The prompt strictly commands the AI: *"Respond ONLY with a valid JSON object (no markdown, no explanation)"*.
4. **The Parsing:** When the AI replies, the Java code uses Jackson (`ObjectMapper`) to parse the AI's JSON text into a Java `Map`.

**Example of the JSON Structure expected from the AI:**
```json
{
  "condition": "Migraine",
  "specialization": "Neurologist",
  "department": "Neurology",
  "urgency": "High",
  "advice": "Rest in a quiet, dark room. Avoid screens."
}
```

---

## 4. The Fallback Engine (Fault Tolerance)

Professors love **Fault Tolerance**—which means the system doesn't crash when something goes wrong. You should heavily emphasize this!

**The Problem:** External APIs can time out, go offline, or hallucinate bad data.
**The Solution:** The `AiSymptomService` wraps the AI call in a `try-catch` block. 

If the AI fails for *any reason*, the system silently catches the error and routes the patient's symptoms to the **Keyword Fallback Engine**.

**How the Fallback Engine Works:**
1. It maintains a hardcoded list of 11 medical `Rules` (e.g., Cardiology, Neurology).
2. Each rule has a list of keywords (e.g., `"chest pain"`, `"headache"`, `"head pain"`).
3. The engine scans the patient's input, counts how many keywords match each rule, and picks the rule with the highest score.
4. It returns the exact same Java `Map` structure as the AI, but tags it with `"source": "keyword"` instead of `"source": "ai"`.

Because the data structure is identical, the frontend doesn't even know the AI failed! It just renders the card seamlessly, ensuring the patient always gets a recommendation.

---

## 5. Summary for your Presentation
If you are asked to summarize the AI Symptom checker in 3 sentences, use this:

> *"Our AI Symptom Checker uses a Spring Boot backend to securely pass patient symptoms to an external Large Language Model via REST API. We use prompt engineering to force the AI to return structured JSON data, which our frontend dynamically renders into actionable booking links. To ensure high availability, we implemented a fault-tolerant keyword fallback engine that automatically takes over if the external AI service ever goes offline."*
