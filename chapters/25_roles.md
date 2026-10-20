# Chapter 25: Role-Based Authorization

## 25.1 What a role is

**Definition:** A **role** is a named group of permissions, e.g. `Admin`, `Customer`, `Manager`. Users can belong to one or more roles. Authorization attributes check those roles.

**Authentication vs role authorization:**

- Logged in = authenticated (we know who you are).  
- `[Authorize(Roles = "Admin")]` = must be authenticated **and** in Admin role.

If logged in but wrong role → **Access Denied** (not the login page), configured via `AccessDeniedPath`.

---

## 25.2 Create roles and assign users

**Example seeder** — call once at startup or from an Admin-only action.

**`Data/IdentitySeeder.cs`**

```csharp
using Microsoft.AspNetCore.Identity;

namespace WebShop.Data
{
    public static class IdentitySeeder
    {
        public static async Task SeedAsync(IServiceProvider services)
        {
            using var scope = services.CreateScope();
            var roleManager = scope.ServiceProvider.GetRequiredService<RoleManager<IdentityRole>>();
            var userManager = scope.ServiceProvider.GetRequiredService<UserManager<IdentityUser>>();

            string[] roles = { "Admin", "Customer", "Manager" };

            foreach (string roleName in roles)
            {
                if (!await roleManager.RoleExistsAsync(roleName))
                {
                    IdentityResult createRole = await roleManager.CreateAsync(new IdentityRole(roleName));
                    // In production, check createRole.Succeeded and log errors
                }
            }

            // Demo admin — change email/password; never ship default passwords
            string adminEmail = "admin@example.com";
            IdentityUser? admin = await userManager.FindByEmailAsync(adminEmail);
            if (admin == null)
            {
                admin = new IdentityUser { UserName = adminEmail, Email = adminEmail };
                IdentityResult createUser = await userManager.CreateAsync(admin, "Admin@12345");
                if (createUser.Succeeded)
                {
                    IdentityResult addRole = await userManager.AddToRoleAsync(admin, "Admin");
                    // Check addRole.Succeeded
                }
            }
        }
    }
}
```

Call after `var app = builder.Build();`:

```csharp
await IdentitySeeder.SeedAsync(app.Services);
```

**Prevent duplicate roles:** always `RoleExistsAsync` before `CreateAsync`.

**Public registration:** after `CreateAsync`, you may `AddToRoleAsync(user, "Customer")` only — **never** allow a form field `Role=Admin` from the client.

---

## 25.3 Protect controllers and actions

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace WebShop.Controllers
{
    public class AdminController : Controller
    {
        // Any authenticated user
        [Authorize]
        public IActionResult Profile() => View();

        // Only Admin
        [Authorize(Roles = "Admin")]
        public IActionResult Dashboard() => View();

        // Admin OR Manager (comma = either role, NOT both required)
        [Authorize(Roles = "Admin,Manager")]
        public IActionResult Reports() => View();
    }

    public class StoreController : Controller
    {
        [Authorize(Roles = "Customer")]
        public IActionResult MyCart() => RedirectToAction("Index", "Cart");
    }
}
```

**Comma meaning:** `Roles = "Admin,Manager"` means user needs **Admin OR Manager**.

For complex rules (Admin AND claim X), prefer **authorization policies** over stacking raw roles.

---

## 25.4 Access denied behaviour

In Chapter 22:

```csharp
options.AccessDeniedPath = "/Account/AccessDenied";
```

**`Views/Account/AccessDenied.cshtml`**

```cshtml
@{
    ViewData["Title"] = "Access denied";
}
<h2>Access denied</h2>
<p>You are signed in but not allowed to view this page.</p>
```

| Situation | Typical result |
|-----------|----------------|
| Anonymous hits `[Authorize]` | Redirect to Login |
| Customer hits `[Authorize(Roles="Admin")]` | Access Denied |

---

## 25.5 Checking results of role operations

```csharp
IdentityResult result = await _userManager.AddToRoleAsync(user, "Customer");
if (!result.Succeeded)
{
    foreach (var e in result.Errors)
        ModelState.AddModelError("", e.Description);
}
```

Same pattern for `CreateAsync` on roles.

---

## 25.6 Policies (brief, real apps)

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("MustBeManager", policy =>
        policy.RequireRole("Manager", "Admin"));
});

// [Authorize(Policy = "MustBeManager")]
```

Use when roles alone become messy.

---

## 25.7 Exam practice

1. Difference authentication vs authorization.  
2. Write attribute allowing Admin or Manager.  
3. Why registration must not accept arbitrary role from the form.  
4. What middleware order is required?

\newpage
