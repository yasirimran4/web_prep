# Chapter 24: Login, Logout, and Authentication Cookies

## 24.1 LoginViewModel

**`ViewModels/LoginViewModel.cs`**

```csharp
using System.ComponentModel.DataAnnotations;

namespace WebShop.ViewModels
{
    public class LoginViewModel
    {
        [Required]
        [EmailAddress]
        public string Email { get; set; } = string.Empty;

        [Required]
        [DataType(DataType.Password)]
        public string Password { get; set; } = string.Empty;

        [Display(Name = "Remember me")]
        public bool RememberMe { get; set; }

        public string? ReturnUrl { get; set; }
    }
}
```

---

## 24.2 Login and logout actions

Add to **`Controllers/AccountController.cs`** (same DI as Chapter 23):

```csharp
[HttpGet]
[AllowAnonymous]
public IActionResult Login(string? returnUrl = null)
{
    return View(new LoginViewModel { ReturnUrl = returnUrl });
}

[HttpPost]
[AllowAnonymous]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Login(LoginViewModel model)
{
    if (!ModelState.IsValid)
        return View(model);

    var result = await _signInManager.PasswordSignInAsync(
        model.Email,
        model.Password,
        isPersistent: model.RememberMe,
        lockoutOnFailure: false);

    if (result.Succeeded)
    {
        if (!string.IsNullOrEmpty(model.ReturnUrl) && Url.IsLocalUrl(model.ReturnUrl))
            return Redirect(model.ReturnUrl);

        return RedirectToAction("Index", "Home");
    }

    ModelState.AddModelError(string.Empty, "Invalid login attempt.");
    return View(model);
}

[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Logout()
{
    await _signInManager.SignOutAsync();
    return RedirectToAction("Index", "Home");
}

[HttpGet]
[AllowAnonymous]
public IActionResult AccessDenied()
{
    return View();
}
```

### API meanings

| Call | Meaning |
|------|---------|
| `PasswordSignInAsync` | Verify password hash; issue auth cookie on success |
| `isPersistent: true` | Longer-lived auth cookie (“Remember me”) |
| `SignOutAsync` | Remove auth cookie / sign-out |
| `Url.IsLocalUrl` | Prevent open-redirect attacks on `returnUrl` |

Again: action is `async Task<IActionResult>` and uses `await`.

---

## 24.3 Login view

**`Views/Account/Login.cshtml`**

```cshtml
@model WebShop.ViewModels.LoginViewModel
@{
    ViewData["Title"] = "Login";
}

<h2>Login</h2>
<div asp-validation-summary="ModelOnly" class="text-danger"></div>

<form asp-action="Login" method="post">
    @Html.AntiForgeryToken()
    <input type="hidden" asp-for="ReturnUrl" />

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

    <div class="form-check mb-3">
        <input asp-for="RememberMe" class="form-check-input" />
        <label asp-for="RememberMe" class="form-check-label"></label>
    </div>

    <button type="submit" class="btn btn-primary">Log in</button>
</form>
```

**Logout button** (e.g. in `_Layout.cshtml` when signed in):

```cshtml
@using Microsoft.AspNetCore.Identity
@inject SignInManager<IdentityUser> SignInManager

@if (SignInManager.IsSignedIn(User))
{
    <form asp-controller="Account" asp-action="Logout" method="post" class="d-inline">
        @Html.AntiForgeryToken()
        <button type="submit" class="btn btn-link nav-link">Logout (@User.Identity?.Name)</button>
    </form>
}
else
{
    <a class="nav-link" asp-controller="Account" asp-action="Login">Login</a>
    <a class="nav-link" asp-controller="Account" asp-action="Register">Register</a>
}
```

---

## 24.4 Successful vs unsuccessful login flow

**Success**

```
POST Login → ModelState OK → PasswordSignInAsync succeeds
  → auth cookie in Response → Redirect Home/returnUrl
Next GET → UseAuthentication reads cookie → User.Identity.IsAuthenticated == true
```

**Failure**

```
Wrong password → result.Succeeded false
  → ModelState error → return View(model)
  → no auth cookie issued
```

**Logout**

```
POST Logout + antiforgery → SignOutAsync → cookie cleared → Home as anonymous
```

---

## 24.5 Why password is not in the auth cookie

Identity stores a **password hash** in the database. At login it verifies the typed password against the hash, then issues a **protected authentication ticket**. Stealing the cookie is still bad (session hijack) — use HTTPS (`Secure` cookies) in production — but the raw password is not sitting in the cookie string.

\newpage
