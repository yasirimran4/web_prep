# Chapter 15: Final Exam Revision

## A. One-page cheat sheet

- **MVC:** Model=data, View=HTML, Controller=request handler  
- **GET** read / **POST** change; after POST → **Redirect** (PRG)  
- `@model T` declares type; `@Model` is instance  
- **ViewModel** packs multiple values into one `@model`  
- **ADO.NET:** `SqlConnection` → `SqlCommand` + parameters → `ExecuteNonQuery` / `ExecuteReader`  
- **DAL/Repository:** no SQL in controllers  
- **SQL injection:** never concatenate user input into SQL  
- **Connected:** DataReader | **Disconnected:** DataAdapter + DataTable  
- **async Task\<IActionResult\>** + `await` repo async methods  
- **DI:** `AddScoped<ICarRepository, CarRepository>()` + constructor injection  
- **Layout:** `@RenderBody()`; sections via `@RenderSectionAsync`  
- **Bootstrap 5:** layout once; cards/tables/navbar/alerts; JS for collapse/modal  

---

## B. Essential syntax collection

```csharp
// Action + view + redirect
public IActionResult Index() => View(model);
public IActionResult Go() => RedirectToAction(nameof(Index));
public async Task<IActionResult> Index() => View(await _repo.GetAvailableAsync());

// Razor
@model List<Car>
@foreach (var c in Model) { <p>@c.Name</p> }

// Form + binding
[HttpPost] public IActionResult Book(int id) { ... }
<input type="hidden" name="id" value="@car.Id" />

// Parameterized SQL
cmd.Parameters.AddWithValue("@id", id);

// DI
builder.Services.AddScoped<ICarRepository, CarRepository>();
public CarsController(ICarRepository repo) => _repo = repo;

// Layout sections
@await RenderSectionAsync("Scripts", required: false)
@section Scripts { <script>...</script> }
```

Bootstrap: `container`, `row`, `col-md-4`, `card`, `table table-striped`, `navbar-dark bg-dark`, `alert alert-info`, `badge bg-success`, `pagination` / `page-item` / `page-link`.

---

## C. Comparison tables

| A | B | Key difference |
|---|---|----------------|
| `IActionResult` | `ViewResult` | Interface vs view-only result |
| ViewBag | ViewData | dynamic vs dictionary; both request-scoped |
| ViewData | TempData | TempData can survive one redirect |
| Model | ViewModel | Domain entity vs page DTO |
| GET | POST | Read vs change state |
| `List<T>` | `IEnumerable<T>` | Concrete mutable list vs enumerable contract |
| Class | Interface | Implementation vs contract |
| IoC | DI | Principle vs injection technique |
| Transient | Scoped | Singleton | Every inject / per request / app lifetime |
| Sync | Async | Block wait vs non-blocking I/O wait |
| `Task` | `Task<T>` | No value vs returns T |
| Layout | Partial | Full shell vs fragment |
| RenderBody | RenderSectionAsync | Main content vs named slot |
| Connected | Disconnected | Reader live connection vs DataTable offline |
| Bootstrap CSS | Bootstrap JS | Look vs interactive components |

---

## D. Mock university exam

### Part I — MCQs (1 mark each)

1. HW 4 booking should use: (a) GET (b) POST (c) PUT only (d) HEAD  
2. Lab 02 data access API: (a) EF Core (b) ADO.NET (c) Dapper (d) Hibernate  
3. Prevent SQL injection: (a) concatenate (b) parameters (c) disable DB (d) ViewBag  
4. `@model` means: (a) instance (b) type declaration (c) layout (d) route  
5. After successful POST Book, best next step: (a) View Index (b) Redirect Booked (c) Content (d) File  
6. `AddScoped` lifetime: (a) one per app (b) one per request (c) every inject (d) never  
7. Disconnected Lab 03 uses: (a) only StreamWriter (b) DataTable+Adapter (c) only MVC (d) Redis  
8. Dark navbar links should avoid: (a) `nav-link` (b) conflicting `text-dark` (c) collapse (d) brand  
9. `ExecuteNonQuery` used for: (a) SELECT lists (b) INSERT/UPDATE/DELETE (c) CSS (d) routing  
10. Async controller signature: (a) `void` (b) `Task<IActionResult>` (c) `Car` (d) `string`

### Part II — Short questions

11. Define MVC.  
12. Difference List vs IEnumerable.  
13. Why DAL/repository?  
14. What does `@RenderBody()` do?  
15. Connected vs disconnected in one sentence each.

### Part III — Comparisons

16. GET vs POST (table).  
17. Model vs ViewModel.  
18. ViewBag vs TempData.

### Part IV — Code tracing

19. Given HW 4 lists with Alto, Corolla, Civic available; user POSTs Book id=2 twice quickly after first redirect — what is in each list after first successful book?

### Part V — Find the error

20. ```csharp
public async Task<IActionResult> Index() {
    var cars = _repo.GetAvailableAsync();
    return View(cars);
}
```
21. ```csharp
string sql = "SELECT * FROM Products WHERE Name LIKE '%" + keyword + "%'";
```

### Part VI — Coding

22. Write `Product` class (Lab 02 fields) with constructors.  
23. Write `AddProduct` with parameterized `SqlCommand`.  
24. Write strongly typed Index view loop for `List<Car>`.  
25. Write `_ViewStart` + navbar brand “Car Rental”.

### Part VII — Integration

26. Design flow: Bootstrap form Create Car → validate → ADO.NET insert → redirect Index. List files touched.  
27. Explain how to convert HW 4 in-memory Book to SQL `UPDATE` via repository without EF Core.

---

### Mock exam answer key

1. b  2. b  3. b  4. b  5. b  6. b  7. b  8. b  9. b  10. b  

11. Separates Model, View, Controller responsibilities.  
12. List is concrete/addable; IEnumerable is for enumeration.  
13. Centralize SQL; keep controllers thin; reuse/test.  
14. Inserts the view’s main HTML into the layout.  
15. Connected: read with open connection (DataReader). Disconnected: fill DataTable, update later via Adapter.  

16–18. See section C tables.  

19. After first book: available = Alto, Civic; booked = Corolla. Second POST with id=2 finds nothing → redirect Index, lists unchanged.  

20. Missing `await` — view receives `Task<List<Car>>` instead of list.  
21. SQL injection risk — use parameter + wildcards in value.  

22–25. See Chapters 1, 7, 4, 11.  

26. `Create.cshtml`, `CarsController.Create` GET/POST, `Car` annotations, `CarRepository.AddAsync`, `Program.cs` DI, `_Layout` Bootstrap form classes, redirect Index.  
27. Replace static lists with `ICarRepository.BookAsync` running parameterized UPDATE; register DI; keep same views.

---

## E. Last-minute checklist

**I can define:** MVC, DAL, parameterized query, ViewModel, layout, model binding, async Task, DI scoped.  

**I can compare:** GET/POST, List/IEnumerable, connected/disconnected, class/interface, IoC/DI, RenderBody/Section.  

**I can trace:** Browser → route → controller → DAL → SQL → view → HTML; PRG booking.  

**I can write from memory:** Car model; CarsController Index/Book/Booked; Razor foreach table/cards; ProductDAL Add/GetAll; `_Layout` navbar; Bootstrap card+table classes.  

**I know HW 4 rules:** in-memory, Bootstrap 5.3, no EF/DB/repos/interfaces for that assignment.  

**I know lab stack:** streams → ADO.NET SQL Server DAL → disconnected/JSON → MVC Car Rental.

---

## Completeness review (against master prompt)

| Required chapter | Present |
|------------------|---------|
| 1 C# foundations | Yes |
| 2 HTTP | Yes |
| 3 MVC | Yes |
| 4 Razor | Yes |
| 5 ViewModels | Yes |
| 6 Forms | Yes |
| 7 Database ADO.NET | Yes (EF marked not taught) |
| 8 Repository | Yes |
| 9 Async | Yes |
| 10 DI | Yes |
| 11 Layouts/sections | Yes |
| 12 Bootstrap | Yes |
| 13 Other / verification | Yes |
| Integrated Car Rental | Yes |
| Cheat sheet / syntax / comparisons / mock / checklist | Yes |

**Evidence honesty:** ChatGPT chat history not available; Labs 01–03 + HW 4 used as primary course evidence.

\newpage

# End of Handbook

Practice writing code on paper. Re-do Lab 02 CRUD and HW 4 views without looking. Good luck — Professor Shuja’s Web Technologies (SEF24).
