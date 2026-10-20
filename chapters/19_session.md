# Chapter 19: Session State in ASP.NET Core MVC

## 19.1 Cookies vs session (detail)

| | Cookie | Session |
|-|--------|---------|
| Main data location | Browser | **Server** (by default in-memory cache in teaching apps) |
| What browser usually keeps | The full small value **or** only a **session id** cookie | Session **id** cookie (not the whole cart object) |
| Size | Small (~4KB) | Larger server-side payloads OK (within reason) |
| Tampering | Easy for user to edit values | Harder to edit server data; id still sensitive |
| App restart (in-memory) | Cookie may remain | **In-memory session data is lost** |
| Multi-server | Cookie still on client | In-memory session **not shared** across servers without sticky sessions / distributed cache |

**Clarify:** Session data is generally **server-side**. The browser stores a **session identifier**, not the complete session dictionary.

---

## 19.2 How session works

```
1. First request that uses session → server creates session id
2. Response Set-Cookie: .AspNetCore.Session=...
3. Next requests send that cookie
4. Server loads session data for that id from its store
5. Idle timeout expires → session data discarded
```

---

## 19.3 Configure session in `Program.cs`

**Order matters:** `UseSession()` must run **before** endpoints that need session, and after routing is configured appropriately. Typical MVC order:

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();

// 1) In-memory distributed cache (required for default session store)
builder.Services.AddDistributedMemoryCache();

// 2) Session services
builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(20); // idle expiry
    options.Cookie.HttpOnly = true;
    options.Cookie.IsEssential = true;
    options.Cookie.Name = ".WebShop.Session";
});

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
}

app.UseStaticFiles();
app.UseRouting();

app.UseSession(); // MUST be before MapControllerRoute / endpoints that use session

app.UseAuthorization(); // if you also use auth later: UseAuthentication before UseAuthorization

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

### Line-by-line

- `AddDistributedMemoryCache` — provides the storage provider session needs.  
- `AddSession` — registers session feature + cookie options.  
- `IdleTimeout` — if user idle 20 minutes, session expires.  
- `UseSession` — enables reading/writing `HttpContext.Session` for the request.

**Without these:** `HttpContext.Session` throws or is unavailable.

---

## 19.4 Reading and writing values

**File:** `Controllers/SessionDemoController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;

namespace WebShop.Controllers
{
    public class SessionDemoController : Controller
    {
        [HttpGet]
        public IActionResult Set()
        {
            HttpContext.Session.SetString("UserName", "Ali");
            HttpContext.Session.SetInt32("VisitCount", 1);
            return Content("Session values set. Visit /SessionDemo/Get");
        }

        [HttpGet]
        public IActionResult Get()
        {
            string? name = HttpContext.Session.GetString("UserName");
            int? visits = HttpContext.Session.GetInt32("VisitCount");

            if (name == null)
                return Content("No UserName in session (new/expired/missing).");

            return Content($"Name={name}, Visits={visits}");
        }

        [HttpGet]
        public IActionResult Bump()
        {
            int count = HttpContext.Session.GetInt32("VisitCount") ?? 0;
            count++;
            HttpContext.Session.SetInt32("VisitCount", count);
            return Content($"VisitCount is now {count}");
        }

        [HttpGet]
        public IActionResult Clear()
        {
            HttpContext.Session.Clear();
            return Content("Session cleared.");
        }
    }
}
```

### Important APIs

```csharp
HttpContext.Session.SetString(key, value);
HttpContext.Session.GetString(key);      // null if missing

HttpContext.Session.SetInt32(key, value);
HttpContext.Session.GetInt32(key);       // int? null if missing
```

There is **no** built-in `SetObject` — store objects via JSON (next section / cart chapter).

---

## 19.5 Storing objects with JSON in session

```csharp
using System.Text.Json;

public static class SessionExtensions
{
    public static void SetObject<T>(this ISession session, string key, T value)
    {
        session.SetString(key, JsonSerializer.Serialize(value));
    }

    public static T? GetObject<T>(this ISession session, string key)
    {
        string? json = session.GetString(key);
        return json == null ? default : JsonSerializer.Deserialize<T>(json);
    }
}
```

(Full helper used again in Chapter 20.)

---

## 19.6 App restart and multi-server limits

| Scenario | In-memory session (`AddDistributedMemoryCache`) |
|----------|--------------------------------------------------|
| App pool recycle / `dotnet run` restart | Session data **gone** |
| Two servers behind load balancer | User may hit another server → session **missing** unless sticky sessions or Redis/SQL distributed cache |

Cookies for preferences may still exist; do not confuse them with session payload.

---

## 19.7 Exam practice

**Q1.** Where is session *data* stored vs the session *id*?  
**Q2.** Write `Program.cs` lines for memory cache + session + `UseSession`.  
**Q3.** Why might GetString return null after 25 minutes?  
**Q4.** Cookie vs session for a 20-item cart?

### Answer key

1. Data on server; id usually in browser cookie.  
2. See section 19.3.  
3. IdleTimeout expired (if 20 min).  
4. Session (or DB) preferred — cookies too small/insecure for cart authority.

\newpage
