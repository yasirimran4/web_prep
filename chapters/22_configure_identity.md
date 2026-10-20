# Chapter 22: Configuring ASP.NET Core Identity

## 22.1 Packages and providers

Typical NuGet packages (versions follow your SDK):

- `Microsoft.AspNetCore.Identity.EntityFrameworkCore`
- `Microsoft.EntityFrameworkCore.SqlServer` **or** `Npgsql.EntityFrameworkCore.PostgreSQL`
- `Microsoft.EntityFrameworkCore.Tools` (migrations)

**Lab history note:** Earlier labs used **ADO.NET + SQL Server**. Identity scaffolding commonly uses **EF Core** for its user store. That does **not** erase your ADO.NET knowledge — Identity’s membership tables are a separate concern. HW 4 still forbade EF for that car-rental exercise.

---

## 22.2 ApplicationDbContext

**`Data/ApplicationDbContext.cs`**

```csharp
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

namespace WebShop.Data
{
    public class ApplicationDbContext : IdentityDbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options)
        {
        }
    }
}
```

`IdentityDbContext` (or `IdentityDbContext<IdentityUser>`) already includes user/role tables.

---

## 22.3 Connection string placeholders

**`appsettings.json`** — supply **your** values; do not invent real passwords in notes:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=YOUR_SERVER;Database=WebShopIdentity;User Id=YOUR_USER;Password=YOUR_PASSWORD;TrustServerCertificate=True;"
  }
}
```

**PostgreSQL variant:**

```json
"DefaultConnection": "Host=YOUR_HOST;Port=5432;Database=WebShopIdentity;Username=YOUR_USER;Password=YOUR_PASSWORD"
```

---

## 22.4 Register services in `Program.cs`

**SQL Server:**

```csharp
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using WebShop.Data;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddIdentity<IdentityUser, IdentityRole>(options =>
    {
        options.Password.RequiredLength = 6;
        options.Password.RequireNonAlphanumeric = false;
        options.SignIn.RequireConfirmedAccount = false; // teaching default
    })
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();

builder.Services.ConfigureApplicationCookie(options =>
{
    options.LoginPath = "/Account/Login";
    options.AccessDeniedPath = "/Account/AccessDenied";
});

builder.Services.AddControllersWithViews();
// If you still use session for cart:
builder.Services.AddDistributedMemoryCache();
builder.Services.AddSession();

var app = builder.Build();

app.UseStaticFiles();
app.UseRouting();

app.UseSession();
app.UseAuthentication(); // before authorization
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

**PostgreSQL change only the DbContext line:**

```csharp
options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection"));
```

---

## 22.5 Migrations and schema

In Rider / terminal (project folder):

```bash
dotnet ef migrations add InitialIdentity
dotnet ef database update
```

This creates Identity tables (`AspNetUsers`, `AspNetRoles`, …). You must have EF tools installed and a valid connection string.

---

## 22.6 Custom MVC pages vs Identity UI scaffolding

| Approach | Meaning |
|----------|---------|
| **Custom MVC** (this guide) | You write `AccountController`, ViewModels, Razor views |
| **Identity UI / scaffold** | Visual Studio can add prebuilt Register/Login Razor Class Library pages |

Both use the same `UserManager` / `SignInManager` under the hood. Exams often want **custom** controller actions you can write by hand.

---

## 22.7 Middleware order checklist

```
UseRouting → UseSession (if any) → UseAuthentication → UseAuthorization → MapControllers
```

Wrong order → user appears anonymous, or `[Authorize]` misbehaves.

\newpage
