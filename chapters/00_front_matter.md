---
title: "Web Technologies Exam Preparation Guide"
subtitle: "PUCIT / FCIT — SEF24 — ASP.NET Core MVC, C#, ADO.NET, Bootstrap 5, State & Identity"
author: "Personalized revision handbook for BS Software Engineering"
date: "October 2026 (revised — Part II: State, Session, Cart, Identity)"
---

\newpage

# Evidence Map and How to Use This Guide

## What this document is

This is a **course revision handbook** for your Web Technologies exam at the University of the Punjab (PUCIT/FCIT), taught by **Professor Shuja Ur Rehman Baig**. It follows your actual labs and homework, uses a consistent **Car Rental** example, and prepares you for both conceptual and hand-written coding questions.

## Honest research note (read this)

**ChatGPT conversation history was not accessible** in this Cursor session. This guide was built from:

| Source | Status | What it confirms |
|--------|--------|------------------|
| `SEF24-Web-Lab-01.pdf` | Confirmed | C# console, byte/character streams, `FileStream`, `StreamWriter`/`StreamReader` |
| `SEF24-Web-Lab-02-Updated.docx` | Confirmed | Classes/encapsulation, **ADO.NET**, SQL Server, `ProductDAL`, CRUD, parameterized SQL |
| `SEF24-Web-Lab-03.docx` | Confirmed | Safe search (anti SQL injection), JSON ser/deser, **disconnected model** (`DataTable`, `SqlDataAdapter`) |
| `web_hw1.pdf` (HW 4) | Confirmed | ASP.NET Core MVC Car Rental, Bootstrap 5.3, layouts, GET/POST, cards/tables; **no EF Core, DB, repos, or interfaces for that HW** |
| `HW_5_OC.pdf` (Lab 5) | Confirmed | Book Store MVC: ViewModels, `_Layout`/`RenderBody`/`RenderSection`, plain HTML forms (no Tag Helpers), in-memory lists |
| `lecture6/` (`Student`, `StudentDAL`) | Partial | Same DAL naming pattern as Lab 02 (files were empty stubs) |
| Prior ChatGPT Web chats | **Not available** | Topics you listed in the master prompt (ViewModels, DI, async, layouts/sections, etc.) are included as **exam prep aligned with your stated learning goals**, clearly labelled |
| Part II chapters (16–26) | Exam prep | State, cookies, session, cart, ASP.NET Core Identity — appended from newly studied topics (verify with teacher) |

**Do not claim EF Core was your taught data-access stack.** Labs 02–03 use **ADO.NET + SQL Server**. HW 4 uses **in-memory lists**. Repository/interfaces/DI appear in later conceptual chapters as patterns that build on your DAL work — verify with your teacher if they are on the written exam.

## Topic classification

### 1. Confirmed topics studied (course materials)

- C# classes, properties, constructors, encapsulation, `ToString()`
- Byte streams vs character streams; file I/O
- ADO.NET connected model: `SqlConnection`, `SqlCommand`, `SqlDataReader`
- Parameterized queries and SQL injection awareness
- Full CRUD through a **DAL** class
- Disconnected model: `DataTable`, `SqlDataAdapter`, `SqlCommandBuilder`
- JSON `Serialize` / `Deserialize`
- ASP.NET Core MVC: models, controllers, strongly typed views
- GET/POST actions, Post/Redirect/Get style booking flow
- Shared `_Layout.cshtml`, Bootstrap 5.3 navbar, cards, tables, alerts, badges
- In-memory `List<Car>` as temporary data store (HW 4)

### 2. Related / mentioned but not deeply evidenced in files

- ViewModels, `ViewBag`/`ViewData`/`TempData` (common MVC; not in HW text)
- Tag Helpers and data annotations (typical MVC form work)
- Dependency Injection and interfaces (HW 4 explicitly forbade interfaces)
- `async`/`await` with database calls (natural extension of ADO.NET)
- Repository pattern (your DAL is the same idea under a different name)

### 3. Syllabus gaps to verify with your teacher

- Entity Framework Core for *business* data (HW 4 forbids EF; Identity user store often uses EF — confirm exam scope)
- Unit of Work
- ASP.NET Core Web APIs
- Deployment and hosting
- Whether async programming is examinable
- Whether DI lifetimes (`Transient`/`Scoped`/`Singleton`) are examinable
- Exact Identity depth on the written paper (Part II Chapters 21–25 prepare you anyway)

### Part II (Chapters 16–26) — newly appended

State management, cookies, JSON in cookies, session, add-to-cart, ASP.NET Core Identity (register/login/logout/roles).

## Exam strategy (teacher-oriented)

1. **Define** in one sentence (exam answer box in each chapter).
2. **Compare** related ideas in a table (GET/POST, List/IEnumerable, connected/disconnected).
3. **Write syntax by hand** — practice without IntelliSense (Rider will not help in the exam hall).
4. **Trace** a request: Browser → Routing → Controller → DAL/Repository → DB → View → HTML.
5. **Know HW 4 rules** separately from “full architecture” patterns: HW 4 = in-memory, no interfaces; later chapters = how production MVC usually grows.

## Consistent example

Throughout this guide we use **Car Rental**:

- Domain model: `Car` with `Id`, `Name`, `PricePerDay`
- Controllers: `CarsController`
- Views: Available cars (cards), Booked cars (table)
- Layout: dark responsive Bootstrap navbar

When we add database chapters, the same `Car` maps to a SQL Server table via ADO.NET — the technology your labs actually taught.

\newpage
