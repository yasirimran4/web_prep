# Chapter 21: Authentication, Authorization, and ASP.NET Core Identity

## 21.1 Three separate ideas

| Concept | Question it answers | Simple meaning |
|---------|---------------------|----------------|
| **Authentication** | Who are you? | Prove identity (login) |
| **Authorization** | What may you do? | Permissions / roles after you are known |
| **ASP.NET Core Identity** | How do we manage users? | Framework for users, password hashes, roles, cookies, account APIs |

**Analogy:** Authentication = showing your student ID at the gate. Authorization = whether that ID allows the admin lab. Identity = the university system that issues IDs and stores hashed credentials.

---

## 21.2 Authentication cookie vs preference cookie

| Ordinary preference cookie | Authentication cookie (Identity) |
|----------------------------|----------------------------------|
| Theme, language, city label | Protected login ticket |
| Often readable meaning | Encrypted/protected by data-protection APIs |
| You invent the value | Created by `SignInManager` after password check |
| Not proof of identity | Establishes `HttpContext.User` |

**Password is never stored in the authentication cookie.** The cookie carries a secure authentication ticket / claims principal reference, not your password text.

---

## 21.3 How a user stays signed in

```
Signup
  → User record created in database
  → Password stored as a secure hash (not plain text)
  → Login
  → Password verification against hash
  → Authentication ticket/cookie issued
  → Subsequent requests send auth cookie
  → Authentication middleware builds ClaimsPrincipal (current user)
  → Authorization middleware checks [Authorize] / roles
```

Between requests, HTTP is still stateless — the **auth cookie** (and Identity store) reconnects identity.

---

## 21.4 Middleware roles

| Middleware | Job |
|------------|-----|
| `UseAuthentication()` | Read auth cookie → set `HttpContext.User` |
| `UseAuthorization()` | Enforce `[Authorize]` / policies / roles |

**Order:** `UseAuthentication()` **before** `UseAuthorization()`.

---

## 21.5 Important Identity types

| Type | Role |
|------|------|
| `UserManager<IdentityUser>` | Create users, hash passwords, add to roles, find by email |
| `SignInManager<IdentityUser>` | Sign in / sign out / password sign-in |
| `RoleManager<IdentityRole>` | Create/check roles |
| User store / `IdentityDbContext` | EF Core tables for users, roles, claims |
| Database | Persistent user/role data |

Async reminder (Part I Chapter 9): Identity methods are async — use `async Task<IActionResult>` and `await`. Do **not** call `.Result` / `.Wait()` on Identity tasks in MVC actions.

---

## 21.6 Exam answers to memorise

- **Authentication** identifies the user.  
- **Authorization** decides access rights.  
- **Identity** is the built-in membership system (users, hashes, roles, auth cookie).

\newpage
