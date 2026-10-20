# Chapter 23: Signup and Registration

## 23.1 RegisterViewModel

**`ViewModels/RegisterViewModel.cs`**

```csharp
using System.ComponentModel.DataAnnotations;

namespace WebShop.ViewModels
{
    public class RegisterViewModel
    {
        [Required]
        [EmailAddress]
        [Display(Name = "Email")]
        public string Email { get; set; } = string.Empty;

        [Required]
        [MinLength(6)]
        [DataType(DataType.Password)]
        public string Password { get; set; } = string.Empty;

        [Required]
        [DataType(DataType.Password)]
        [Compare(nameof(Password), ErrorMessage = "Passwords do not match.")]
        [Display(Name = "Confirm password")]
        public string ConfirmPassword { get; set; } = string.Empty;
    }
}
```

**Two validation layers:**

1. **ViewModel annotations** + `ModelState.IsValid` (format, required, compare).  
2. **Identity password policy** inside `CreateAsync` (length, digits, etc. from `IdentityOptions`).

Never assign `user.PasswordHash = plainPassword` yourself. Always use Identity APIs so the password is **hashed**.

---

## 23.2 AccountController — register actions

**`Controllers/AccountController.cs`** (register portion)

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using WebShop.ViewModels;

namespace WebShop.Controllers
{
    public class AccountController : Controller
    {
        private readonly UserManager<IdentityUser> _userManager;
        private readonly SignInManager<IdentityUser> _signInManager;

        public AccountController(
            UserManager<IdentityUser> userManager,
            SignInManager<IdentityUser> signInManager)
        {
            _userManager = userManager;
            _signInManager = signInManager;
        }

        [HttpGet]
        [AllowAnonymous]
        public IActionResult Register()
        {
            return View();
        }

        [HttpPost]
        [AllowAnonymous]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Register(RegisterViewModel model)
        {
            if (!ModelState.IsValid)
                return View(model);

            var user = new IdentityUser
            {
                UserName = model.Email,
                Email = model.Email
            };

            // Password hashed inside CreateAsync — never store plain text
            IdentityResult result = await _userManager.CreateAsync(user, model.Password);

            if (result.Succeeded)
            {
                // Optional: sign in immediately after register
                await _signInManager.SignInAsync(user, isPersistent: false);
                return RedirectToAction("Index", "Home");
            }

            foreach (IdentityError error in result.Errors)
            {
                ModelState.AddModelError(string.Empty, error.Description);
            }

            return View(model);
        }
    }
}
```

### Why `async` / `await`?

`CreateAsync` and `SignInAsync` touch the database and auth cookie — they return `Task` / `Task<IdentityResult>`. The action must be:

```csharp
public async Task<IActionResult> Register(...)
```

and **await** those calls. Avoid `.Result`.

---

## 23.3 Registration view

**`Views/Account/Register.cshtml`**

```cshtml
@model WebShop.ViewModels.RegisterViewModel
@{
    ViewData["Title"] = "Register";
}

<h2>Create account</h2>

<div asp-validation-summary="ModelOnly" class="text-danger"></div>

<form asp-action="Register" method="post">
    @Html.AntiForgeryToken()

    <div class="mb-3">
        <label asp-for="Email" class="form-label"></label>
        <input asp-for="Email" class="form-control" />
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>

    <div class="mb-3">
        <label asp-for="Password" class="form-label"></label>
        <input asp-for="Password" class="form-control" />
        <span asp-validation-for="Password" class="text-danger"></span>
    </div>

    <div class="mb-3">
        <label asp-for="ConfirmPassword" class="form-label"></label>
        <input asp-for="ConfirmPassword" class="form-control" />
        <span asp-validation-for="ConfirmPassword" class="text-danger"></span>
    </div>

    <button type="submit" class="btn btn-primary">Register</button>
</form>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

---

## 23.4 Execution flow

1. GET Register → empty form.  
2. POST with email/password → antiforgery OK.  
3. `ModelState` checks annotations.  
4. `CreateAsync` hashes password, inserts user.  
5. Failures (weak password, duplicate email) → Identity errors into `ModelState` → redisplay form.  
6. Success → optional `SignInAsync` → redirect Home (auth cookie set).

---

## 23.5 Exam tips

- Do not insert passwords with raw SQL into Identity tables.  
- Public registration must **not** let users pick `Admin` role (Chapter 25).  
- `[ValidateAntiForgeryToken]` on POST.

\newpage
