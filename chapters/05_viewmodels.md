# Chapter 5: ViewModels and Multiple Objects

> **Evidence note:** HW 4 only passes `List<Car>`. ViewModels are standard MVC exam material and match topics you listed as studied with ChatGPT; confirm with your teacher if they appear on the paper.

## 5.1 The problem

A view may need:

- list of available cars  
- total car count  
- page heading  
- a success message flag  

Razor allows **only one** `@model` directive. You cannot write:

```cshtml
@model List<Car>
@model string   @* ERROR — not allowed *@
```

## 5.2 Solution: ViewModel

**Definition:** A **ViewModel** is a class designed for one screen. It combines several properties the view needs into **one** object.

**Domain model vs ViewModel:**

| Domain model (`Car`) | ViewModel (`AvailableCarsViewModel`) |
|----------------------|--------------------------------------|
| Mirrors business/DB entity | Mirrors **page** needs |
| Reused across app | Often one view (or a few) |
| `Id`, `Name`, `PricePerDay` | `Cars`, `TotalCount`, `Heading` |

---

## 5.3 Create a ViewModel

```csharp
// ViewModels/AvailableCarsViewModel.cs
using CarRental.Models;

namespace CarRental.ViewModels
{
    public class AvailableCarsViewModel
    {
        public string Heading { get; set; } = "Available Cars";
        public int TotalCount { get; set; }
        public List<Car> Cars { get; set; } = new();
    }
}
```

## 5.4 Controller passes ViewModel

```csharp
[HttpGet]
public IActionResult Index()
{
    var vm = new AvailableCarsViewModel
    {
        Heading = "Available Cars",
        Cars = availableCars,
        TotalCount = availableCars.Count
    };
    return View(vm);
}
```

## 5.5 View uses ViewModel

```cshtml
@model CarRental.ViewModels.AvailableCarsViewModel

<h2>@Model.Heading</h2>
<p>Total: @Model.TotalCount</p>

@if (Model.Cars.Count == 0)
{
    <div class="alert alert-info">No cars available.</div>
}
else
{
    @foreach (var car in Model.Cars)
    {
        <div>@car.Name — PKR @car.PricePerDay</div>
    }
}
```

---

## 5.6 ViewModel vs ViewBag / ViewData

| Approach | Type-safe? | Refactor-friendly? | Exam clarity |
|----------|------------|--------------------|--------------|
| ViewModel | Yes | Excellent | Best |
| ViewBag | Weak | Poor | OK for demos |
| ViewData | Cast needed | Poor | Avoid for multi-data |

**Intuition:** ViewModel = ek dabba jisme page ki sari cheezen packed hain.

---

## 5.7 Forms and ViewModels

When a form needs extra fields (customer name + car id), make a form ViewModel:

```csharp
public class BookCarViewModel
{
    public int CarId { get; set; }
    [Required]
    public string CustomerName { get; set; } = "";
}
```

(HW 4 does not require customer details — keep that distinction clear.)

---

## Exam answer

A ViewModel is a presentation-focused class that packages all data one view needs into a single object so the view can stay strongly typed with one `@model` directive.

**Practice:** Design a ViewModel for Booked page with `Cars`, `Heading`, and `HasBookings` bool.

\newpage
