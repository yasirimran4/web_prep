# Chapter 12: Bootstrap 5 Complete Exam Revision

Based on **HW 4** (Bootstrap **5.3**) plus common components for exams. Bootstrap styles and JS behaviour; it does **not** do server pagination, SQL, or booking logic.

## 12.1 Layout and grid

| Class | Meaning |
|-------|---------|
| `container` | Centred fixed-width responsive wrapper |
| `container-fluid` | Full width |
| `row` | Horizontal row of columns |
| `col` | Equal flexible column |
| `col-md-6` | Half width from `md` breakpoint up |
| `col-sm-12` | Full width from `sm` up |

Breakpoints (BS5): `sm` 576px, `md` 768px, `lg` 992px, `xl` 1200px, `xxl` 1400px.

```html
<div class="container my-4">
  <div class="row">
    <div class="col-md-4">Card 1</div>
    <div class="col-md-4">Card 2</div>
    <div class="col-md-4">Card 3</div>
  </div>
</div>
```

**Mistake:** columns without a `row` parent; nesting rows incorrectly.

---

## 12.2 Spacing

| Class | Meaning |
|-------|---------|
| `m-3` | margin all sides |
| `mt-2` / `mb-3` | margin top / bottom |
| `mx-auto` | horizontal centre (block) |
| `p-3` | padding all |
| `py-2` / `px-4` | padding y / x |

**Margin** = outside gap; **padding** = inside gap.

---

## 12.3 Flexbox

```html
<div class="d-flex justify-content-between align-items-center gap-2 flex-wrap">
  <h2>Booked Cars</h2>
  <a class="btn btn-secondary">Back to Cars</a>
</div>
```

| Class | Effect |
|-------|--------|
| `d-flex` | flex container |
| `flex-row` / `flex-column` | direction |
| `justify-content-center` / `between` | main axis |
| `align-items-center` | cross axis |
| `gap-2` | gap between items |
| `flex-wrap` | allow wrap |

---

## 12.4 Typography and colours

`text-center`, `text-start`, `text-end`, `fw-bold`, `fst-italic`, `text-uppercase`, `h1`–`h6`, `display-1`, `lead`, `text-primary`, `text-danger`, `text-muted`, `bg-light`, `bg-dark`, `bg-success`.

---

## 12.5 Display and sizing

`d-none`, `d-block`, `d-md-block`, `w-100`, `h-50`, `border`, `border-primary`, `rounded`, `rounded-circle`.

---

## 12.6 Buttons

```html
<button class="btn btn-primary w-100">Book Car</button>
<a class="btn btn-secondary" href="/Cars/Index">Back</a>
```

Variants: `btn-success`, `btn-danger`, `btn-warning`, `btn-outline-primary`, `btn-sm`, `btn-lg`.

---

## 12.7 Cards (HW 4 available cars)

```html
<div class="card h-100 shadow-sm">
  <div class="card-body d-flex flex-column">
    <h5 class="card-title">Corolla</h5>
    <p class="card-text">PKR 6000 / day</p>
    <span class="badge bg-success mb-2">Available</span>
    <form method="post" action="/Cars/Book" class="mt-auto">
      <input type="hidden" name="id" value="2" />
      <button class="btn btn-primary w-100" type="submit">Book Car</button>
    </form>
  </div>
</div>
```

`h-100` + `mt-auto` keeps buttons aligned in a grid.

---

## 12.8 Tables (HW 4 booked)

```html
<table class="table table-striped table-hover">
  <thead class="table-dark">...</thead>
  <tbody>...</tbody>
</table>
```

Wrap in `table-responsive` for small screens.

---

## 12.9 Forms

```html
<div class="mb-3">
  <label class="form-label">Name</label>
  <input class="form-control" />
</div>
<select class="form-select">...</select>
```

---

## 12.10 Alerts and badges

```html
<div class="alert alert-info">No cars available.</div>
<span class="badge bg-success">Available</span>
<span class="badge text-bg-warning">Booked</span>
```

HW 4: yellow booked badge with dark text → `text-bg-warning` (or `bg-warning text-dark`).

---

## 12.11 Navbar (HW 4)

See Chapter 11 full example. Key pieces:

- `navbar navbar-expand-lg navbar-dark bg-dark`
- Brand left: Car Rental  
- `navbar-toggler` + `data-bs-toggle="collapse"` + `data-bs-target="#navMain"`
- Collapse div **same id** `navMain`
- Needs Bootstrap **JS bundle** for toggle

---

## 12.12 List groups

```html
<ul class="list-group list-group-flush">
  <li class="list-group-item">Alto</li>
  <li class="list-group-item">Civic</li>
</ul>
```

Also: `list-group-numbered`, `list-group-horizontal`.

---

## 12.13 Pagination (UI only)

```html
<ul class="pagination">
  <li class="page-item disabled"><a class="page-link">Prev</a></li>
  <li class="page-item active"><a class="page-link">1</a></li>
  <li class="page-item"><a class="page-link">2</a></li>
  <li class="page-item"><a class="page-link">Next</a></li>
</ul>
```

Server must still skip/take rows — Bootstrap only draws controls.

---

## 12.14 Modals

```html
<button class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#confirmModal">
  Open
</button>
<div class="modal fade" id="confirmModal" tabindex="-1">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">Confirm</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
      </div>
      <div class="modal-body">Book this car?</div>
      <div class="modal-footer">
        <button class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
        <button class="btn btn-primary">Yes</button>
      </div>
    </div>
  </div>
</div>
```

**Needs JS bundle** for `data-bs-toggle` / `dismiss`.

---

## 12.15 Dropdowns

```html
<div class="dropdown">
  <button class="btn btn-secondary dropdown-toggle" data-bs-toggle="dropdown">Menu</button>
  <ul class="dropdown-menu">
    <li><a class="dropdown-item" href="#">Item</a></li>
  </ul>
</div>
```

---

## 12.16 What belongs in layout vs view

| In `_Layout` | In individual views |
|--------------|---------------------|
| Bootstrap CSS + JS | Page heading, cards, tables |
| Navbar, footer | Forms for that page |
| `@RenderBody()` | Optional `@section Scripts` |

**Exam answer:** Bootstrap 5 is a CSS/JS toolkit for responsive layout and components; MVC still owns routing, data, and booking logic.

\newpage
