# Chapter 11: Razor Layouts, Shared Files, and Sections

**High priority — HW 4 Task 3.**

## 11.1 Why layouts?

**Definition:** A **layout** is a shared HTML shell (navbar, footer, CSS/JS) that wraps individual views.

**Why underscore?** Convention: `_Layout.cshtml`, `_ViewStart.cshtml` — files starting with `_` are partial/shared helpers, not directly browsed as pages. This is mostly a **naming convention**, not magic by itself; `_ViewStart` / `_ViewImports` have **special framework behaviour**.

---

## 11.2 Special files

| File | Role |
|------|------|
| `Views/Shared/_Layout.cshtml` | Master HTML template |
| `Views/_ViewStart.cshtml` | Sets default `Layout` for views in folder tree |
| `Views/_ViewImports.cshtml` | Shared `@using` and tag helpers |
| Partial e.g. `_CarCard.cshtml` | Reusable fragment |

```cshtml
@* Views/_ViewStart.cshtml *@
@{
    Layout = "_Layout";
}
```

```cshtml
@* Views/_ViewImports.cshtml *@
@using CarRental.Models
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

---

## 11.3 `@RenderBody()` — HW 4 Car Rental layout

```cshtml
@* Views/Shared/_Layout.cshtml *@
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>@ViewData["Title"] - Car Rental</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
          rel="stylesheet" />
    @await RenderSectionAsync("Styles", required: false)
</head>
<body>
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
  <div class="container">
    <a class="navbar-brand" asp-controller="Cars" asp-action="Index">Car Rental</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse"
            data-bs-target="#navMain" aria-controls="navMain"
            aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navMain">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item">
          <a class="nav-link" asp-controller="Cars" asp-action="Index">Available Cars</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" asp-controller="Cars" asp-action="Booked">Booked Cars</a>
        </li>
      </ul>
    </div>
  </div>
</nav>

<main class="container my-4">
    @RenderBody()
</main>

<footer class="container border-top py-3 text-center">
    All rights reserved
</footer>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
@await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

**How it wraps a view:** Razor executes Index.cshtml content and injects it where `@RenderBody()` sits.

**Navbar tip:** With `navbar-dark`, do **not** force dark text classes on links (`text-dark`) — they become invisible.

---

## 11.4 Sections

**Definition:** A **section** is optional/required content a view defines and the layout renders in a named slot.

```cshtml
@* In layout *@
@await RenderSectionAsync("Scripts", required: false)
```

```cshtml
@* In Views/Cars/Index.cshtml *@
@section Scripts {
    <script>
        console.log("Index page script");
    </script>
}

@section Styles {
    <style>.card { border-radius: 0.5rem; }</style>
}
```

| API | Role |
|-----|------|
| `@RenderBody()` | Main view content |
| `@RenderSectionAsync("Name")` | Named extra content |

**Rule:** Section is **defined in the view**, **rendered in the layout**. If `required: true` and view omits section → error.

---

## 11.5 Multiple layouts / no layout

**Admin layout:** `Views/Shared/_AdminLayout.cshtml` — different nav.

```cshtml
@{
    Layout = "_AdminLayout";
}
```

**Standalone (no layout):**

```cshtml
@{
    Layout = null;
}
```

---

## 11.6 Partials

```cshtml
@await Html.PartialAsync("_CarCard", car)
```

Partials usually start with `_`. They do not run a full layout cycle by themselves.

---

## 11.7 Bootstrap CSS/JS placement

- **CSS** in `<head>` of layout (once)  
- **JS bundle** before `</body>` (once)  
- Page-specific scripts in `@section Scripts`  

HW 4: load Bootstrap **once in the layout**, not in every view.

---

## 11.8 Common errors

1. Forgetting `_ViewStart` → views without chrome.  
2. Typo in `data-bs-target` vs collapse `id` → toggle broken.  
3. Defining `@section Scripts` but layout has no `RenderSectionAsync`.  
4. Putting Bootstrap JS only on one page — collapse fails elsewhere.  
5. Conflicting text colour on dark navbar.

**Exam answer:** `_Layout.cshtml` provides shared page chrome; `@RenderBody()` inserts the view; `_ViewStart` sets the default layout; sections inject page-specific CSS/JS into slots in the layout.

\newpage
