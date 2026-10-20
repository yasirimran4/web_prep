# Web Technologies Exam Preparation Guide

**PUCIT / FCIT — SEF24 — Professor Shuja Ur Rehman Baig**

## Outputs (single complete document)

| File | Description |
|------|-------------|
| [Web_Technologies_Exam_Prep_Guide.pdf](./Web_Technologies_Exam_Prep_Guide.pdf) | Full printable PDF (Part I + Part II) |
| [Web_Technologies_Exam_Prep_Guide.docx](./Web_Technologies_Exam_Prep_Guide.docx) | Editable Word source export |
| [Web_Technologies_Exam_Prep_Guide.md](./Web_Technologies_Exam_Prep_Guide.md) | Combined Markdown |
| `chapters/` | Chapter sources (`00`–`26`) |

## Structure

- **Part I (Ch. 1–15):** C#, HTTP, MVC, Razor, ViewModels, Forms, ADO.NET, Repository, Async, DI, Layouts, Bootstrap, Car Rental, revision  
- **Part II (Ch. 16–26):** State, Cookies, Serialization, Session, Cart, Identity, Signup, Login, Roles, Part II revision  

## Rebuild workflow

```bash
cd Exam_Prep_Guide
cat chapters/*.md > Web_Technologies_Exam_Prep_Guide.md   # use ordered list — see build note
pandoc Web_Technologies_Exam_Prep_Guide.md -o Web_Technologies_Exam_Prep_Guide.docx --toc --toc-depth=3
soffice --headless --convert-to pdf --outdir . Web_Technologies_Exam_Prep_Guide.docx
```

Prefer concatenating chapters in numeric order (`00` … `26`), not raw glob order alone if your shell sorts oddly.
