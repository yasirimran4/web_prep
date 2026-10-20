# Chapter 18: Serialization and Deserialization of Objects in Cookies

## 18.1 Why a C# object cannot be a cookie value

Cookies store **strings** (name/value text). A `Customer` object lives in .NET memory with fields and type info. You cannot write:

```csharp
Response.Cookies.Append("Customer", customer); // INVALID — needs string
```

You must convert the object to a string first (**serialize**), then later convert the string back (**deserialize**).

---

## 18.2 Definitions

| Term | Meaning |
|------|---------|
| **Serialization** | Object → string (often JSON) |
| **Deserialization** | String → object |
| **JSON** | Text format like `{"Id":1,"Name":"Ali"}` — useful, readable, standard |

**Analogy:** Serialization packs a suitcase (object → JSON text). Deserialization unpacks it.

```csharp
using System.Text.Json;

string json = JsonSerializer.Serialize(customer);
Customer? customer = JsonSerializer.Deserialize<Customer>(json);
```

---

## 18.3 Serialization is NOT encryption

| Claim | Truth |
|-------|-------|
| “JSON in cookie is secret” | **False** — anyone can read/edit it in DevTools |
| “HttpOnly encrypts data” | **False** — only blocks JS access |
| “Safe for passwords” | **Never** store passwords/secrets/trusted money values in ordinary cookies |

Cookie size is limited (often ~4KB per cookie). Prefer **session** or **database** for large carts or sensitive data.

---

## 18.4 Complete MVC example

### Model — `Models/Customer.cs`

```csharp
namespace WebShop.Models
{
    public class Customer
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string City { get; set; } = string.Empty;
    }
}
```

### Controller — `Controllers/CustomerCookieController.cs`

```csharp
using System.Text.Json;
using Microsoft.AspNetCore.Mvc;
using WebShop.Models;

namespace WebShop.Controllers
{
    public class CustomerCookieController : Controller
    {
        private const string CookieKey = "CustomerJson";

        [HttpGet]
        public IActionResult Save()
        {
            var customer = new Customer
            {
                Id = 1,
                Name = "Ali Khan",
                City = "Lahore"
            };

            string json = JsonSerializer.Serialize(customer);

            Response.Cookies.Append(CookieKey, json, new CookieOptions
            {
                Expires = DateTimeOffset.UtcNow.AddHours(2),
                HttpOnly = true,
                SameSite = SameSiteMode.Lax,
                Path = "/"
                // Do NOT put passwords here
            });

            return Content("Customer saved to cookie as JSON. Visit /CustomerCookie/Load");
        }

        [HttpGet]
        public IActionResult Load()
        {
            if (!Request.Cookies.TryGetValue(CookieKey, out string? json)
                || string.IsNullOrWhiteSpace(json))
            {
                return Content("No customer cookie found.");
            }

            try
            {
                Customer? customer = JsonSerializer.Deserialize<Customer>(json);
                if (customer == null)
                    return Content("Cookie JSON was empty/invalid.");

                return Content($"Loaded: {customer.Id} — {customer.Name} ({customer.City})");
            }
            catch (JsonException)
            {
                return Content("Invalid JSON in cookie (tampered or corrupt).");
            }
        }

        [HttpGet]
        public IActionResult Clear()
        {
            Response.Cookies.Delete(CookieKey);
            return Content("Customer cookie cleared.");
        }
    }
}
```

### Execution flow

1. `Save` builds `Customer` → `Serialize` → string → `Response.Cookies.Append`.  
2. Browser stores JSON text.  
3. `Load` reads `Request.Cookies` → `Deserialize<Customer>` → use properties.  
4. Missing/invalid JSON → friendly message / try-catch.

### Expected browser text

- After Save: confirmation message.  
- After Load: `Loaded: 1 — Ali Khan (Lahore)`.

---

## 18.5 When to prefer session or database

| Use cookie JSON | Prefer session / DB |
|-----------------|---------------------|
| Small preferences (theme, city label) | Shopping cart, orders |
| Non-sensitive display hints | Prices, stock, roles, passwords |
| Tiny objects | Large lists |

---

## 18.6 Exam practice

**Q1.** Define serialization and deserialization.  
**Q2.** Why not store password in a cookie even if HttpOnly?  
**Q3.** Write Serialize + Append for a `Product` with Id/Name/Price.  
**Q4.** What happens if JSON is edited by the user?

### Answer key

1. Object↔string conversion.  
2. User can still see/edit; not encrypted; credential theft risk.  
3. Similar to Save action above.  
4. Deserialize may throw or produce wrong object — validate; never trust for money/auth.

\newpage
