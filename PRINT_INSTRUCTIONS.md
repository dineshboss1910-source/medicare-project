# Convert STUDY_GUIDE.md to printable formats

## Convert to PDF using Pandoc
1. Install Pandoc (https://pandoc.org/installing.html) and a PDF engine (e.g., wkhtmltopdf, or LaTeX like TeX Live).
2. Run:

```bash
pandoc STUDY_GUIDE.md -o STUDY_GUIDE.pdf
```

If using LaTeX for higher-quality PDF:

```bash
pandoc STUDY_GUIDE.md -o STUDY_GUIDE.pdf --pdf-engine=xelatex
```

## Convert to PDF using VS Code
1. Open `STUDY_GUIDE.md` in VS Code.
2. Install the "Markdown PDF" extension.
3. Use the extension command `Markdown PDF: Export (pdf)`.

## Convert to printable HTML
```bash
pandoc STUDY_GUIDE.md -s -o STUDY_GUIDE.html
```

## Quick tip (Windows PowerShell)
```powershell
# Generate PDF with pandoc
pandoc .\STUDY_GUIDE.md -o .\STUDY_GUIDE.pdf --pdf-engine=xelatex
```
