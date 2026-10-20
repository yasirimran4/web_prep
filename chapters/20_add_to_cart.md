# Chapter 20: Add-to-Cart Logic in ASP.NET Core MVC

This chapter combines **HTTP statelessness**, **session**, **JSON serialization**, **models**, **controllers**, **views**, and **forms**.

## 20.1 Teaching store products

| Product ID | Product | Price |
|------------|---------|------:|
| 1 | Mouse | PKR 1,500 |
| 2 | Keyboard | PKR 3,000 |
| 3 | Headphones | PKR 4,000 |

**Teaching assumption:** product catalogue is an **in-memory list**.  
**Real apps:** load products and **authoritative prices** from the database; validate stock/quantity; recalculate totals on the server at checkout. Never trust a price posted from the browser.

**Why not store the whole cart in a cookie?** Size limits, easy tampering of quantities/prices, and awkward updates. **Session** (or DB for logged-in carts) is more suitable for teaching and many real apps.

---

## 20.2 Project files

```text
Models/Product.cs
Models/CartItem.cs
Extensions/SessionExtensions.cs
Controllers/CartController.cs
Views/Cart/Index.cshtml
Program.cs   (session enabled — Chapter 19)
```

Namespace throughout: `WebShop`.

---

## 20.3 Models

**`Models/Product.cs`**

```csharp
namespace WebShop.Models
{
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public decimal Price { get; set; }
    }
}
```

**`Models/CartItem.cs`**

```csharp
namespace WebShop.Models
{
    public class CartItem
    {
        public int ProductId { get; set; }
        public string ProductName { get; set; } = string.Empty;
        public decimal UnitPrice { get; set; }
        public int Quantity { get; set; }

        public decimal Subtotal => UnitPrice * Quantity;
    }
}
```

---

## 20.4 Session helper

**`Extensions/SessionExtensions.cs`**

```csharp
using System.Text.Json;
using Microsoft.AspNetCore.Http;

namespace WebShop.Extensions
{
    public static class SessionExtensions
    {
        public static void SetObject<T>(this ISession session, string key, T value)
        {
            session.SetString(key, JsonSerializer.Serialize(value));
        }

        public static T? GetObject<T>(this ISession session, string key)
        {
            string? json = session.GetString(key);
            if (string.IsNullOrWhiteSpace(json))
                return default;
            return JsonSerializer.Deserialize<T>(json);
        }
    }
}
```

---

## 20.5 Cart controller (complete)

**`Controllers/CartController.cs`**

```csharp
using Microsoft.AspNetCore.Mvc;
using WebShop.Extensions;
using WebShop.Models;

namespace WebShop.Controllers
{
    public class CartController : Controller
    {
        private const string CartKey = "ShoppingCart";

        // Teaching catalogue — replace with DB in production
        private static readonly List<Product> Products = new()
        {
            new Product { Id = 1, Name = "Mouse", Price = 1500m },
            new Product { Id = 2, Name = "Keyboard", Price = 3000m },
            new Product { Id = 3, Name = "Headphones", Price = 4000m }
        };

        private List<CartItem> GetCart()
        {
            return HttpContext.Session.GetObject<List<CartItem>>(CartKey)
                   ?? new List<CartItem>();
        }

        private void SaveCart(List<CartItem> cart)
        {
            HttpContext.Session.SetObject(CartKey, cart);
        }

        [HttpGet]
        public IActionResult Index()
        {
            List<CartItem> cart = GetCart();
            ViewBag.Products = Products; // for Add buttons on same page (teaching)
            ViewBag.Total = cart.Sum(i => i.Subtotal);
            return View(cart);
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public IActionResult Add(int productId)
        {
            Product? product = Products.FirstOrDefault(p => p.Id == productId);
            if (product == null)
                return RedirectToAction(nameof(Index));

            List<CartItem> cart = GetCart();
            CartItem? existing = cart.FirstOrDefault(i => i.ProductId == productId);

            if (existing != null)
            {
                existing.Quantity++; // already in cart → increase quantity
            }
            else
            {
                cart.Add(new CartItem
                {
                    ProductId = product.Id,
                    ProductName = product.Name,
                    UnitPrice = product.Price, // from server list, not from form
                    Quantity = 1
                });
            }

            SaveCart(cart);
            return RedirectToAction(nameof(Index)); // PRG pattern
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public IActionResult Remove(int productId)
        {
            List<CartItem> cart = GetCart();
            CartItem? item = cart.FirstOrDefault(i => i.ProductId == productId);
            if (item != null)
            {
                cart.Remove(item);
                SaveCart(cart);
            }
            return RedirectToAction(nameof(Index));
        }
    }
}
```

---

## 20.6 View

**`Views/Cart/Index.cshtml`**

```cshtml
@model List<WebShop.Models.CartItem>
@{
    ViewData["Title"] = "Cart";
    var products = (List<WebShop.Models.Product>)ViewBag.Products;
    decimal total = (decimal)ViewBag.Total;
}

<h2>Products</h2>
<div class="row mb-4">
@foreach (var p in products)
{
    <div class="col-md-4 mb-2">
        <div class="card">
            <div class="card-body">
                <h5>@p.Name</h5>
                <p>PKR @p.Price</p>
                <form asp-controller="Cart" asp-action="Add" method="post">
                    @Html.AntiForgeryToken()
                    <input type="hidden" name="productId" value="@p.Id" />
                    <button type="submit" class="btn btn-primary">Add to Cart</button>
                </form>
            </div>
        </div>
    </div>
}
</div>

<h2>Your Cart</h2>
@if (Model.Count == 0)
{
    <div class="alert alert-info">Cart is empty.</div>
}
else
{
    <table class="table table-striped">
        <thead>
            <tr>
                <th>Product</th><th>Price</th><th>Qty</th><th>Subtotal</th><th></th>
            </tr>
        </thead>
        <tbody>
        @foreach (var item in Model)
        {
            <tr>
                <td>@item.ProductName</td>
                <td>@item.UnitPrice</td>
                <td>@item.Quantity</td>
                <td>@item.Subtotal</td>
                <td>
                    <form asp-action="Remove" method="post">
                        @Html.AntiForgeryToken()
                        <input type="hidden" name="productId" value="@item.ProductId" />
                        <button class="btn btn-sm btn-danger" type="submit">Remove</button>
                    </form>
                </td>
            </tr>
        }
        </tbody>
    </table>
    <p class="fw-bold">Total: PKR @total</p>
}
```

Ensure `_ViewImports.cshtml` has Tag Helpers enabled (Part I Chapter 11).

---

## 20.7 Request → response execution flow

```
User clicks Add on Mouse (Id=1)
  → POST /Cart/Add  (form: productId=1 + antiforgery)
  → Model binder sets productId = 1
  → Controller loads cart from Session (JSON → List<CartItem>)
  → Finds Product 1 in server list (price 1500)
  → If Mouse already in cart: Quantity++
    else: add new CartItem Quantity=1
  → Save cart to Session (List → JSON string)
  → 302 Redirect /Cart/Index
  → GET Index loads cart from Session → HTML table
```

**Relationship to HTTP statelessness:** Each POST/GET is a new request. The cart survives only because **session** (server data + session id cookie) reconnects requests for the same browser session.

---

## 20.8 Dry run — two clicks on Mouse

| Step | Cart in session after action |
|------|------------------------------|
| Start | `[]` |
| 1st Add Mouse | `[{Mouse, 1500, Qty 1, Sub 1500}]` Total 1500 |
| 2nd Add Mouse | `[{Mouse, 1500, Qty 2, Sub 3000}]` Total 3000 |

Browser does **not** need to send the whole cart each time — only the session cookie id + `productId`.

---

## 20.9 Common mistakes

1. Trusting `price` from a hidden form field.  
2. Forgetting `UseSession` / `AddSession`.  
3. Using `static List<CartItem>` for all users.  
4. Not redirecting after POST (refresh re-adds).  
5. Storing cart only in a huge cookie.

---

## 20.10 Exam questions

1. Explain how the cart persists across requests.  
2. Write `Add` logic that increments quantity.  
3. Why recalculate price from server catalogue?

\newpage
