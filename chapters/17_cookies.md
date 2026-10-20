# Chapter 17: Cookies in ASP.NET Core MVC

## 17.1 What a cookie is

**Simple definition:** A **cookie** is a small name/value piece of data that the **server asks the browser to store**, and that the **browser sends back** on later matching requests.

**Where stored?** In the **browser** (client), not as an ordinary file on your web server’s disk for each preference.

**Intuition:** The server gives the browser a tiny sticker. Next visit, the browser shows the sticker so the server can recognise preferences or a session id.

---

## 17.2 Complete cookie lifecycle

```
1. Browser → GET /Account/SetName
2. Server action runs Response.Cookies.Append("UserName", "Ali", options)
3. HTTP Response includes header: Set-Cookie: UserName=Ali; path=/; ...
4. Browser stores cookie UserName=Ali
5. Later Browser → GET /Account/Hello
   Request includes header: Cookie: UserName=Ali
6. Server reads Request.Cookies["UserName"] → "Ali"
7. To delete: Response.Cookies.Delete("UserName") → Set-Cookie with expired date
```

**Critical distinction:**

| API | Direction | Meaning |
|-----|-----------|---------|
| `Response.Cookies.Append` / `Delete` | **Server → Browser** | Instruct browser to create/update/delete |
| `Request.Cookies["name"]` | **Browser → Server** | Read what browser sent on **this** request |

You cannot “read” a cookie you just appended on the **same** response’s `Request.Cookies` in a reliable teaching sense for a brand-new cookie — the browser sends it on the **next** request.

---

## 17.3 Complete controller example

**File:** `Controllers/CookieDemoController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;

namespace WebShop.Controllers
{
    public class CookieDemoController : Controller
    {
        private const string CookieName = "UserName";

        // 1) Create / set cookie
        [HttpGet]
        public IActionResult SetName(string name = "Ali")
        {
            var options = new CookieOptions
            {
                Expires = DateTimeOffset.UtcNow.AddDays(7),
                HttpOnly = true,          // not readable by JavaScript (helps vs XSS theft)
                Secure = false,           // set true in production WITH HTTPS
                SameSite = SameSiteMode.Lax,
                IsEssential = true,       // if using cookie consent middleware
                Path = "/"
            };

            Response.Cookies.Append(CookieName, name, options);
            return Content($"Cookie '{CookieName}' set to '{name}'. Now open /CookieDemo/ReadName");
        }

        // 2) Read cookie on another request
        [HttpGet]
        public IActionResult ReadName()
        {
            string? value = Request.Cookies[CookieName];
            if (string.IsNullOrEmpty(value))
                return Content("Cookie missing or expired.");

            return Content($"Hello, {value}! (read from Request.Cookies)");
        }

        // 3) Update cookie (Append again with same name)
        [HttpGet]
        public IActionResult UpdateName(string name = "Sara")
        {
            Response.Cookies.Append(CookieName, name, new CookieOptions
            {
                Expires = DateTimeOffset.UtcNow.AddDays(7),
                HttpOnly = true,
                SameSite = SameSiteMode.Lax,
                Path = "/"
            });
            return Content($"Updated cookie to '{name}'.");
        }

        // 4) Delete cookie
        [HttpGet]
        public IActionResult DeleteName()
        {
            Response.Cookies.Delete(CookieName);
            // Or: Response.Cookies.Delete(CookieName, new CookieOptions { Path = "/" });
            return Content("Cookie deleted (browser will drop it).");
        }

        // 5) Handle missing / expired
        [HttpGet]
        public IActionResult SafeRead()
        {
            if (!Request.Cookies.TryGetValue(CookieName, out string? value)
                || string.IsNullOrWhiteSpace(value))
            {
                return RedirectToAction(nameof(SetName));
            }
            return View("Greeting", model: value);
        }
    }
}
```

**Optional view** `Views/CookieDemo/Greeting.cshtml`:

```cshtml
@model string
<h2>Welcome back, @Model</h2>
```

### Line-by-line highlights

- `CookieOptions.Expires` — persistent cookie until that time; omit Expires for a session cookie (browser session).  
- `HttpOnly = true` — JavaScript cannot read it via `document.cookie` (reduces XSS cookie theft). **Not encryption.**  
- `Secure = true` — browser sends cookie only over **HTTPS**.  
- `SameSite` — controls cross-site sending (`Strict` / `Lax` / `None`).  
- `Path` / `Domain` — scope which URLs receive the cookie.

---

## 17.4 Expected output

1. Visit `/CookieDemo/SetName?name=Ali` → text says cookie set.  
2. Visit `/CookieDemo/ReadName` → `Hello, Ali!`  
3. Visit `/CookieDemo/UpdateName?name=Sara` then Read → `Hello, Sara!`  
4. Visit `/CookieDemo/DeleteName` then Read → `Cookie missing or expired.`

---

## 17.5 Cookie security limitations

1. Users can edit cookies in DevTools — **do not trust cookie values for prices or roles**.  
2. Serialization ≠ encryption (next chapter).  
3. `HttpOnly` ≠ confidential from the user; user can still see it in browser tools.  
4. Large data / secrets / passwords → **never** in ordinary cookies.  
5. Prefer session/DB for carts and authoritative business data.

---

## 17.6 Comparison: Request vs Response cookies

| | Request | Response |
|-|---------|----------|
| Who fills it? | Browser | Your server code |
| Typical API | `Request.Cookies["x"]` | `Response.Cookies.Append` / `Delete` |
| Meaning | “What I already have” | “Please store/change this” |

---

## 17.7 Exam practice

**Q1.** Where is a cookie stored?  
**Q2.** Write code to create an HttpOnly cookie named `Theme` = `dark` for 30 days.  
**Q3.** Find the error: reading `Request.Cookies["Theme"]` in the same action immediately after `Append` and expecting it always for a brand-new cookie.  
**Q4.** Difference between deleting a cookie and letting it expire.

### Answer key

1. In the browser (client).  
2. `Response.Cookies.Append("Theme","dark", new CookieOptions { Expires = DateTimeOffset.UtcNow.AddDays(30), HttpOnly = true });`  
3. New cookie is sent by browser on **subsequent** requests.  
4. Delete sends instruction to remove now; expiry removes automatically later.

\newpage
