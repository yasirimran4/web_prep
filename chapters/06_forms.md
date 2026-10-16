# Chapter 6: HTML Forms, Tag Helpers, GET, POST, and Model Binding

**High priority for HW 4 and exams.**

## 6.1 HTML form basics

**Definition:** A **form** collects user input and sends it to the server with GET or POST.

| Element | Role |
|---------|------|
| `form` | Container; `action` = URL; `method` = get/post |
| `label` | Accessible caption for an input |
| `input` | text, number, hidden, submit |
| `select` | dropdown |
| `textarea` | multi-line text |
| submit button | triggers send |

```html
<form action="/Cars/Book" method="post">
  <input type="hidden" name="id" value="2" />
  <button type="submit">Book Car</button>
</form>
```

**Field names ↔ model binding:** `name="id"` binds to action parameter `int id` or property `Id`.

---

## 6.2 GET forms vs POST forms

| | GET form | POST form |
|-|----------|-----------|
| Data in | Query string | Body |
| Use | Search/filter | Create/update/book |
| HW 4 | — | Book button |

---

## 6.3 `[HttpGet]` and `[HttpPost]`

```csharp
[HttpGet]
public IActionResult Create() => View();

[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Create(Car car)
{
    if (!ModelState.IsValid)
        return View(car); // redisplay form with errors

    // save...
    return RedirectToAction(nameof(Index));
}
```

Same action name, different HTTP verb — common pattern.

---

## 6.4 Model binding

**Definition:** ASP.NET Core copies request values into parameters or model properties by **name matching**.

```csharp
// Primitive
public IActionResult Book(int id) { }

// Complex model
public IActionResult Create(Car car) { }
```

Form:

```html
<input name="Name" value="Alto" />
<input name="PricePerDay" value="3000" />
```

binds to `car.Name` and `car.PricePerDay`.

---

## 6.5 Tag Helpers (MVC)

| Tag Helper | Purpose |
|------------|---------|
| `asp-controller` | Sets controller for link/form |
| `asp-action` | Sets action |
| `asp-for` | Binds label/input to model property + generates `name`/`id` |
| `asp-validation-for` | Shows validation message for a property |
| `asp-route-id` | Route value |

```cshtml
@model CarRental.Models.Car

<form asp-controller="Cars" asp-action="Create" method="post">
    <div class="mb-3">
        <label asp-for="Name" class="form-label"></label>
        <input asp-for="Name" class="form-control" />
        <span asp-validation-for="Name" class="text-danger"></span>
    </div>
    <div class="mb-3">
        <label asp-for="PricePerDay" class="form-label"></label>
        <input asp-for="PricePerDay" class="form-control" />
        <span asp-validation-for="PricePerDay" class="text-danger"></span>
    </div>
    <button type="submit" class="btn btn-primary">Save</button>
</form>
```

---

## 6.6 HW 4 book form (complete, with filenames)

**Controller** — `Controllers/CarsController.cs` (excerpt):

```csharp
[HttpPost]
public IActionResult Book(int id)
{
    var car = availableCars.FirstOrDefault(c => c.Id == id);
    if (car == null)
        return RedirectToAction(nameof(Index));

    availableCars.Remove(car);
    bookedCars.Add(car);
    return RedirectToAction(nameof(Booked));
}
```

**View** — `Views/Cars/Index.cshtml` (card with POST):

```cshtml
<form asp-action="Book" method="post">
    <input type="hidden" name="id" value="@car.Id" />
    <button type="submit" class="btn btn-primary w-100">Book Car</button>
</form>
```

### Trace when user books Corolla (Id = 2)

1. Browser shows Index (GET) with three cards.  
2. User clicks Book on Corolla.  
3. POST `/Cars/Book` with body `id=2` (+ antiforgery token if Tag Helpers used).  
4. Model binder sets `id = 2`.  
5. Controller finds Corolla, removes from `availableCars`, adds to `bookedCars`.  
6. Returns **302** `RedirectToAction("Booked")`.  
7. Browser GETs `/Cars/Booked`.  
8. Table shows Corolla with yellow “Booked” badge.

If Id not found: redirect Index, lists unchanged.

---

## 6.7 Validation: `ModelState` and data annotations

```csharp
using System.ComponentModel.DataAnnotations;

public class Car
{
    public int Id { get; set; }

    [Required]
    [StringLength(100)]
    public string Name { get; set; } = "";

    [Range(1, 100000)]
    public decimal PricePerDay { get; set; }
}
```

```csharp
if (!ModelState.IsValid)
    return View(car);
```

**Accepting data ≠ saving data:** validate first; only then insert via DAL.

---

## 6.8 Antiforgery

Tag Helper forms emit a hidden `__RequestVerificationToken`. On POST, add:

```csharp
[ValidateAntiForgeryToken]
```

Protects against Cross-Site Request Forgery.

---

## 6.9 Common mistakes

1. Using GET for Book (unsafe; can be triggered by crawler).  
2. Wrong `name` on input → parameter stays 0/null.  
3. Forgetting redirect after POST → refresh re-submits.  
4. Returning View after successful POST instead of Redirect.  
5. `asp-for` on a view without matching `@model`.

**Exam answer:** Model binding maps request values to action parameters/properties by name; POST actions that change state should redirect (PRG) after success.

\newpage
