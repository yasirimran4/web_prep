# Chapter 3: ASP.NET Core MVC Architecture

## 3.1 What MVC means

**Simple definition:** **MVC** separates an app into **Model** (data), **View** (HTML UI), and **Controller** (handles requests and chooses responses).

**Why:** Controllers stay thin; views stay for display; models/DAL own data. Easier exam tracing and maintenance.

**Intuition:** Restaurant — Customer (browser) talks to Waiter (controller). Waiter gets food info from Kitchen/stock (model/DAL). Plate presentation is the View.

| Part | Responsibility | HW 4 example |
|------|----------------|--------------|
| Model | Data shape | `Car` |
| View | Razor `.cshtml` | `Views/Cars/Index.cshtml` |
| Controller | Actions | `CarsController` |

---

## 3.2 Typical project folders

```
CarRental/
  Program.cs              # app startup, services, middleware
  Controllers/
    CarsController.cs
  Models/
    Car.cs
  Views/
    _ViewStart.cshtml
    _ViewImports.cshtml
    Shared/
      _Layout.cshtml
    Cars/
      Index.cshtml
      Booked.cshtml
  wwwroot/                # static CSS, JS, images
  appsettings.json        # config (connection strings later)
```

---

## 3.3 Controllers, actions, and naming

```csharp
// Controllers/CarsController.cs
using Microsoft.AspNetCore.Mvc;
using CarRental.Models;

public class CarsController : Controller
{
    // HW 4 style — in-memory (resets on restart)
    private static List<Car> availableCars = new()
    {
        new Car { Id = 1, Name = "Alto", PricePerDay = 3000m },
        new Car { Id = 2, Name = "Corolla", PricePerDay = 6000m },
        new Car { Id = 3, Name = "Civic", PricePerDay = 8000m }
    };
    private static List<Car> bookedCars = new();

    [HttpGet]
    public IActionResult Index()
    {
        return View(availableCars); // Views/Cars/Index.cshtml
    }

    [HttpPost]
    public IActionResult Book(int id)
    {
        Car? car = availableCars.FirstOrDefault(c => c.Id == id);
        if (car == null)
            return RedirectToAction(nameof(Index));

        availableCars.Remove(car);
        bookedCars.Add(car);
        return RedirectToAction(nameof(Booked));
    }

    [HttpGet]
    public IActionResult Booked()
    {
        return View(bookedCars);
    }
}
```

**Conventions:**

- Class name ends with `Controller`
- Action method name maps to view name by default
- Route `/Cars/Index` → `CarsController.Index`

---

## 3.4 How `/Cars/Available` would be processed

If you had an action `Available`:

1. Route values: controller=`Cars`, action=`Available`
2. Framework creates `CarsController` (via DI)
3. Calls `Available()`
4. `return View()` looks for `Views/Cars/Available.cshtml`
5. Renders with layout from `_ViewStart`

---

## 3.5 Return types: `IActionResult` vs `ViewResult`

| Type | Meaning |
|------|---------|
| `IActionResult` | Interface for any action result (view, redirect, content, 404…) |
| `ViewResult` | Specific result that renders a Razor view |

**Exam tip:** Prefer `IActionResult` as the declared return type so one action can `return View()` **or** `RedirectToAction()`.

```csharp
public IActionResult Details(int id)
{
    if (id <= 0) return BadRequest();
    var car = availableCars.FirstOrDefault(c => c.Id == id);
    if (car == null) return NotFound();
    return View(car);           // ViewResult under the hood
}
```

| Method | Effect |
|--------|--------|
| `return View()` | View named like the action |
| `return View("Details")` | Named view `Details.cshtml` |
| `return View(model)` | Strongly typed model to default view |
| `RedirectToAction("Booked")` | 302 to another action |
| `Redirect("/home")` | 302 to raw URL |
| `Content("ok")` | Plain text |
| `NotFound()` | 404 |
| `BadRequest()` | 400 |

**When to return a view vs redirect:**  
- View: show a page (GET).  
- Redirect: after successful POST (PRG), or when car not found → back to Index.

---

## 3.6 Complete request lifecycle diagram

```
Browser
  |  GET /Cars/Index
  v
Kestrel + Middleware
  v
Routing  -->  CarsController.Index()
  v
(optional) DAL / Repository / DB
  v
return View(availableCars)
  v
Razor: Views/Cars/Index.cshtml + Shared/_Layout.cshtml
  v
HTML + Bootstrap CSS/JS
  v
Browser paints "Available Cars"
```

---

## 3.7 `Program.cs` (conceptual)

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
// Later: builder.Services.AddScoped<ICarRepository, CarRepository>();

var app = builder.Build();
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
}
app.UseStaticFiles();
app.UseRouting();
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Cars}/{action=Index}/{id?}");
app.Run();
```

---

## Chapter 3 exam practice

**Short:** What does MVC stand for?  
**Difference:** `IActionResult` vs `ViewResult`.  
**Syntax:** Write `Book` POST that redirects to `Booked`.  
**Trace:** User opens `/Cars/Booked` — which files run?  
**Error:** `return View("Index", bookedCars)` from `Booked` — which view?

### Answer key

- Model-View-Controller.  
- Interface vs specific view result.  
- See code above.  
- Routing → `Booked` action → `Views/Cars/Booked.cshtml` + layout.  
- Still uses `Index.cshtml` because of the name argument; model is booked list.

\newpage
