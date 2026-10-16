# Chapter 13: Other Topics from Course History

## 13.1 Lab 01 — Streams and file handling (confirmed)

**Byte stream:** `FileStream` + `Encoding.UTF8.GetBytes` / decode buffer → `books_byte.dat`.  
**Character stream:** `StreamWriter` / `StreamReader.ReadLine()` → `books_char.txt`.

**Difference (exam):** Byte streams work with raw bytes; character streams work with text and encodings more conveniently. Both can store the same book records; APIs differ.

Menu: Add Book, View via Byte, View via Character, Exit.

---

## 13.2 .NET / CLR

See Chapter 1. Console labs compile to managed assemblies; MVC apps run under ASP.NET Core on the CLR/Kestrel.

---

## 13.3 ADO.NET summary pointer

Chapter 7 is the full treatment. Remember: **DAL only**, parameterized SQL, connected vs disconnected.

---

## 13.4 SQL Server (confirmed) vs others

Lab 02 names **SQL Server** (`SqlConnection`). PostgreSQL / MySQL not evidenced in your lab files.

---

## 13.5 MVC folder structure

Chapter 3. HW 4 Car Rental is the practical MVC piece.

---

## 13.6 ASP.NET Core Web APIs

**Not evidenced** in Labs 01–03 or HW 4. Place on **syllabus verification list**.

---

## 13.7 Security

- SQL injection → parameterized queries (Labs 02–03)  
- Antiforgery on POST forms (MVC)  
- Validate input before DB writes  

---

## 13.8 Transactions / Unit of Work

Transactions: brief in Chapter 7. **Unit of Work:** not evidenced — verify.

---

## 13.9 Testing, deployment

Not in provided lab PDFs — verify.

---

## 13.10 lecture6 stubs

`Student.cs` / `StudentDAL.cs` match the Lab 02 pattern (model + DAL). Files were empty in the workspace — treat as unfinished practice, not alternate syllabus.

---

## Syllabus verification checklist (ask teacher)

- [ ] Is async/await on the written exam?  
- [ ] Are DI lifetimes examinable?  
- [ ] ViewModels required?  
- [ ] EF Core ever introduced later?  
- [ ] Web API?  
- [ ] Auth?  
- [ ] Only ADO.NET + MVC + Bootstrap?

\newpage
