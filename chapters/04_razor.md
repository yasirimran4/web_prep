# Chapter 4: Razor Syntax and Data Passing

## 4.1 What is Razor?

**Definition:** Razor is the markup language for MVC views: HTML mixed with C# using `@`. Files use extension `.cshtml`.

**Why:** Build dynamic pages (lists of cars) without separate templating engines.

---

## 4.2 `@model` vs `@Model`

| Syntax | Meaning |
|--------|---------|
| `@model List<Car>` | **Directive** at top: declares the view’s type (compile-time) |
| `@Model` | **Property**: the actual object instance passed from the controller |

```cshtml
@* Views/Cars/Index.cshtml *@
@model List<CarRental.Models.Car>

<h2>Available Cars</h2>
<p>Select a car to book.</p>

@if (Model == null || !Model.Any())
{
    <div class="alert alert-info">No cars available.</div>
}
else
{
    <div class="row">
    @foreach (var car in Model)
    {
        <div class="col-md-4 mb-3">
            <div class="card h-100 shadow-sm">
                <div class="card-body">
                    <h5 class="card-title">@car.Name</h5>
                    <p class="card-text">PKR @car.PricePerDay / day</p>
                    <span class="badge bg-success">Available</span>
                </div>
            </div>
        </div>
    }
    </div>
}
```

Matching controller:

```csharp
public IActionResult Index()
{
    return View(availableCars); // type must match @model
}
```

---

## 4.3 Expressions and code blocks

```cshtml
@* Expression — writes encoded value *@
<p>@car.Name</p>

@* Code block — no automatic output *@
@{
    var title = "Available Cars";
    var count = Model.Count;
}
<h1>@title (@count)</h1>
```

---

## 4.4 Conditionals and loops

```cshtml
@if (Model.Count == 0)
{
    <div class="alert alert-info">Empty</div>
}
else if (Model.Count == 1)
{
    <p>Only one car left.</p>
}
else
{
    <p>Many cars.</p>
}

@for (int i = 0; i < Model.Count; i++)
{
    <li>@Model[i].Name</li>
}
```

---

## 4.5 ViewBag, ViewData, TempData

| Bag | Type | Lifetime | Survives redirect? |
|-----|------|----------|--------------------|
| `ViewBag` | dynamic | Current request | No |
| `ViewData` | dictionary | Current request | No |
| `TempData` | dictionary | Until read (often next request) | Yes (one hop) |

```csharp
public IActionResult Index()
{
    ViewBag.PageTitle = "Available Cars";
    ViewData["Count"] = availableCars.Count;
    return View(availableCars);
}

public IActionResult Book(int id)
{
    TempData["Message"] = "Car booked!";
    return RedirectToAction(nameof(Booked));
}
```

```cshtml
<h1>@ViewBag.PageTitle</h1>
<p>Count: @ViewData["Count"]</p>
@if (TempData["Message"] != null)
{
    <div class="alert alert-success">@TempData["Message"]</div>
}
```

**Prefer strongly typed models / ViewModels** for exam clarity; bags are shortcuts.

---

## 4.6 Passing a single model vs a list

```csharp
return View(car);                 // @model Car
return View(availableCars);       // @model List<Car>  or IEnumerable<Car>
```

**Booked table view (HW 4):**

```cshtml
@model List<CarRental.Models.Car>

<div class="d-flex justify-content-between align-items-center mb-3">
    <h2>Booked Cars</h2>
    <a class="btn btn-secondary" asp-action="Index">Back to Cars</a>
</div>

@if (Model.Count == 0)
{
    <div class="alert alert-info">No bookings yet.</div>
}
else
{
    <div class="table-responsive">
        <table class="table table-striped table-hover">
            <thead class="table-dark">
                <tr>
                    <th>ID</th><th>Car Name</th><th>Price Per Day</th><th>Status</th>
                </tr>
            </thead>
            <tbody>
            @foreach (var car in Model)
            {
                <tr>
                    <td>@car.Id</td>
                    <td>@car.Name</td>
                    <td>@car.PricePerDay</td>
                    <td><span class="badge text-bg-warning">Booked</span></td>
                </tr>
            }
            </tbody>
        </table>
    </div>
}
```

---

## 4.7 Common Razor mistakes

1. Forgetting `@model` → `Model` is untyped/`dynamic` issues.  
2. Writing `@model` twice with different types — **illegal**.  
3. Using `Model.Name` when `@model` is `List<Car>` — lists have no `Name`.  
4. Missing `@` before C# identifiers.  
5. Putting heavy business logic in the view (prefer controller/DAL).

**Exam answer:** `@model` declares the view’s type; `@Model` accesses the instance passed by `return View(data)`.

\newpage
