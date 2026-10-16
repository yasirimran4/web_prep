# Chapter 2: Web Fundamentals and HTTP

## 2.1 Client–server architecture

**Simple definition:** The **client** (usually a browser) requests resources; the **server** processes the request and sends a response.

**Roles:**

| Piece | Responsibility |
|-------|----------------|
| Browser | UI, sends HTTP, renders HTML/CSS/JS |
| Web server / Kestrel | Receives HTTP, runs ASP.NET Core |
| MVC application | Routing, controllers, views, business rules |
| Database (SQL Server) | Persistent storage (Labs 02–03; not HW 4) |

```
  Browser  --HTTP request-->  ASP.NET Core MVC  --SQL-->  SQL Server
  Browser  <--HTML/redirect--  ASP.NET Core MVC  <--rows-- SQL Server
```

---

## 2.2 HTTP request and response

**Request parts:** method (GET/POST), URL/path, query string, headers, optional body.  
**Response parts:** status code, headers, body (HTML, JSON, empty for redirect).

**Example GET:**

```
GET /Cars/Index HTTP/1.1
Host: localhost:5000
```

**Example POST (book a car):**

```
POST /Cars/Book HTTP/1.1
Content-Type: application/x-www-form-urlencoded

id=2
```

---

## 2.3 GET vs POST

| | GET | POST |
|-|-----|------|
| Purpose | Read / display | Change state (book, save, delete) |
| Data location | Query string / URL | Request body |
| Idempotent? | Should be safe to repeat | Not for bookings |
| Caching | Often cacheable | Usually not |
| HW 4 | `Index`, `Booked` | `Book` |

**Exam answer:** GET retrieves data without changing server state; POST submits data that may change state (for example booking a car).

---

## 2.4 URL, route, query string, body, headers, status codes

- **URL:** `https://localhost:5001/Cars/Booked`
- **Route:** `/Cars/Booked` → controller `Cars`, action `Booked`
- **Query string:** `/Cars/Details?id=3` → `id=3`
- **Body:** form fields on POST
- **Headers:** `Content-Type`, cookies, etc.

| Code | Meaning | Typical MVC use |
|------|---------|-----------------|
| 200 | OK | `return View(...)` success |
| 302 | Found / Redirect | `RedirectToAction(...)` |
| 400 | Bad Request | `BadRequest()` |
| 404 | Not Found | `NotFound()` |
| 500 | Server Error | Unhandled exception |

---

## 2.5 How a browser request reaches ASP.NET Core MVC

```
1. User clicks link or submits form
2. Browser sends HTTP request to Kestrel
3. Middleware pipeline runs (routing, etc.)
4. Routing matches /Cars/Available → CarsController.Available
5. Action runs (may call DAL/repository)
6. Action returns IActionResult (View / Redirect / ...)
7. If View: Razor renders HTML (often inside _Layout)
8. Response HTML returned to browser
9. Browser paints page; Bootstrap CSS/JS style and behave
```

**HW 4 booking flow (Post/Redirect/Get):**

```
GET  /Cars/Index   → show cards
POST /Cars/Book    → move car between lists
302 Redirect to Booked
GET  /Cars/Booked  → show table
```

Refreshing after redirect does **not** re-POST the book action — that is why PRG matters.

---

## 2.6 Basic form / database security (course-aligned)

1. **SQL injection:** never concatenate user input into SQL — Lab 02/03 require parameters.  
2. **Antiforgery:** MVC forms include a hidden token against CSRF (Chapter 6).  
3. **Validate input** before writing to DB.  
4. **Least privilege** connection string (concept; supply your own credentials — never invent secrets in notes).

---

## Chapter 2 practice

**Q1.** Draw the request path for `/Cars/Booked`.  
**Q2.** Why is booking a POST, not a GET?  
**Q3.** What does status 302 mean after `Book`?  
**Q4.** Difference between query string and form body?

### Answer key

1. Browser → server → route to CarsController.Booked → View → HTML.  
2. Booking changes state; GET should be safe/read-only.  
3. Redirect response; browser then GETs Booked.  
4. Query string is in the URL; body is in the POST payload.

\newpage
