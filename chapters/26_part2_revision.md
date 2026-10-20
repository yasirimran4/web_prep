# Chapter 26: Integrated Revision — State, Session, Cart, Identity

## 26.1 Big comparison table

| Topic | Core idea | Typical storage |
|-------|-----------|-----------------|
| HTTP statelessness | Each request independent | N/A (protocol) |
| Cookie | Browser stores name/value | Client |
| Cookie + JSON | Object ↔ string in cookie | Client (small, non-secret) |
| Session | Server dictionary + id cookie | Server data + client id |
| Serialization | Object ↔ JSON string | N/A (technique) |
| Authentication | Who are you? | Auth cookie + user DB |
| Authorization | What can you access? | Roles/policies checked each request |
| Identity | Membership framework | EF Identity tables + auth cookie |

---

## 26.2 Cookie request-flow diagram

```
Browser                     Server
   |                          |
   | GET /CookieDemo/SetName  |
   |------------------------->|
   |   Set-Cookie: UserName   |
   |<-------------------------|
   | (stores cookie)          |
   |                          |
   | GET /CookieDemo/ReadName |
   | Cookie: UserName=Ali     |
   |------------------------->|
   |   "Hello, Ali" HTML/text |
   |<-------------------------|
```

---

## 26.3 Shopping-cart request-flow diagram

```
POST /Cart/Add (productId=1) + SessionId cookie
        |
        v
CartController.Add
  load List<CartItem> from Session (JSON)
  lookup Product price on server
  qty++ or insert new item
  save List to Session
        |
        v
302 → GET /Cart/Index
  read Session cart → table + total
```

---

## 26.4 Login + role authorization flow

```
POST /Account/Login
  PasswordSignInAsync
     | fail → show errors
     | ok  → auth cookie
           → Redirect
GET /Admin/Dashboard  [Authorize(Roles="Admin")]
  UseAuthentication → User principal
  UseAuthorization → role check
     | has Admin → View
     | else → AccessDenied
```

---

## 26.5 Essential APIs (Part II cheat sheet)

```csharp
// Cookies
Response.Cookies.Append("k", "v", new CookieOptions { ... });
Request.Cookies["k"];
Response.Cookies.Delete("k");

// JSON
JsonSerializer.Serialize(obj);
JsonSerializer.Deserialize<T>(json);

// Session setup
builder.Services.AddDistributedMemoryCache();
builder.Services.AddSession(...);
app.UseSession();
HttpContext.Session.SetString / GetString / SetInt32 / GetInt32

// Identity
await _userManager.CreateAsync(user, password);
await _signInManager.PasswordSignInAsync(...);
await _signInManager.SignOutAsync();
await _roleManager.RoleExistsAsync / CreateAsync
await _userManager.AddToRoleAsync(user, "Customer");

// AuthZ
[Authorize]
[Authorize(Roles = "Admin")]
[Authorize(Roles = "Admin,Manager")] // OR

app.UseAuthentication();
app.UseAuthorization();
```

Async: `public async Task<IActionResult> ...` + `await` — no `.Result`.

---

## 26.6 Common mistakes and fixes

| Mistake | Fix |
|---------|-----|
| Trust cookie price | Price from server/DB only |
| Session without `UseSession` | Add services + middleware |
| `static` cart for all users | Per-user session/DB |
| Password in cookie/plain DB column | `CreateAsync` / Identity hash |
| `UseAuthorization` before `UseAuthentication` | Swap order |
| Blind `Redirect(returnUrl)` | `Url.IsLocalUrl` |
| Roles from open registration form | Assign `Customer` only in code |
| Think HttpOnly = encryption | It only blocks JS access |
| Think serialize = encrypt | JSON is readable text |
| `.Result` on Identity calls | Use `await` |

---

## 26.7 Likely theory questions

1. Why is HTTP called stateful or stateless? (Answer: **stateless**.)  
2. Request vs Response cookies.  
3. Cookie vs session.  
4. Authentication vs authorization.  
5. What does Identity store instead of plain passwords?  
6. Meaning of `Roles = "Admin,Manager"`.

---

## 26.8 Likely paper coding questions

1. Write code to append an HttpOnly cookie for 7 days.  
2. Serialize a `Customer` to cookie and read it back.  
3. Configure session in `Program.cs`.  
4. Write `CartController.Add` with quantity increment.  
5. Write `Register` POST with `CreateAsync`.  
6. Write `Login` POST with `PasswordSignInAsync` and safe `returnUrl`.  
7. Protect an action for Admin only.

---

## 26.9 Short-answer drill

1. Where is session **data** stored?  
2. Name three `CookieOptions` properties.  
3. Which middleware establishes `User`?  
4. Idle timeout means what?

**Answers:** 1) Server (id in browser). 2) Expires, HttpOnly, Secure, SameSite, Path… 3) Authentication. 4) Session expires after idle period.

---

## 26.10 Dry-run exercises

**A.** Cookie `Theme=dark` set; user opens new tab same site — is Theme sent? → **Yes** (matching cookie).  

**B.** In-memory session; restart app; open cart — cart? → **Empty** (session store cleared).  

**C.** User in role Customer opens `[Authorize(Roles="Admin")]` while logged in → **Access Denied**.  

**D.** Add Mouse twice — quantities? → **2** (Chapter 20 dry run).

---

## 26.11 Mock exam (Part II)

### MCQs

1. HTTP is: (a) stateful by default (b) stateless (c) encrypted always (d) a database  
2. `Request.Cookies` reads values from: (a) response only (b) current incoming request (c) SQL (d) ViewBag  
3. Best teaching place for cart list: (a) huge cookie (b) session/DB (c) ViewBag forever (d) CSS  
4. `CreateAsync(user, password)` stores: (a) plain password (b) hash (c) cookie only (d) nothing  
5. Middleware order: (a) Authorization then Authentication (b) Authentication then Authorization  

### Short

6. Define serialization.  
7. Why validate `returnUrl`?  
8. Difference TempData vs Session.

### Coding

9. Session SetString/GetString for `"City"="Lahore"`.  
10. `[Authorize(Roles = "Admin,Manager")]` on `Reports`.  
11. Delete cookie `"UserName"`.

### Integration

12. Trace Add-to-Cart for Keyboard then Remove.  
13. Trace failed login then successful login.

### Answer key (Part II mock)

1. b  2. b  3. b  4. b  5. b  
6. Object → string (often JSON).  
7. Prevent open redirect to evil external sites.  
8. TempData short-lived across one redirect; Session lasts until timeout/clear for that session id.  
9–11. See Chapters 17–19, 25.  
12. Session gains Keyboard qty 1; Remove deletes item; Index shows empty/info.  
13. Fail: no cookie, errors shown; Success: auth cookie, redirect, `User` authenticated.

---

## 26.12 Final Part II checklist

I can explain and write from memory:

- [ ] Why HTTP is stateless (without claiming the browser remembers nothing)  
- [ ] Cookie lifecycle; Request vs Response APIs  
- [ ] `CookieOptions` (Expires, HttpOnly, Secure, SameSite)  
- [ ] JSON serialize/deserialize for cookies/session  
- [ ] Session setup + Set/Get String/Int32  
- [ ] Full Add-to-Cart with quantity++ and Remove  
- [ ] Authentication vs authorization vs Identity  
- [ ] `Program.cs` Identity + Auth middleware order  
- [ ] Register / Login / Logout with `async`/`await`  
- [ ] Role create/assign and `[Authorize(Roles=...)]`  
- [ ] Safe `returnUrl`, antiforgery, no plain-text passwords  

---

## Document completeness (after update)

| Part | Chapters | Status |
|------|----------|--------|
| Part I | 1–15 | Preserved |
| Part II | 16 State | Added |
| | 17 Cookies | Added |
| | 18 Cookie serialization | Added |
| | 19 Session | Added |
| | 20 Add-to-Cart | Added |
| | 21 Identity concepts | Added |
| | 22 Configure Identity | Added |
| | 23 Signup | Added |
| | 24 Login/Logout | Added |
| | 25 Roles | Added |
| | 26 Part II revision | Added |

\newpage

# End of Handbook

You now have **one** complete Web Technologies exam guide: Part I (MVC, ADO.NET, async, Bootstrap, layouts) plus Part II (state, cookies, session, cart, Identity). Practise writing the critical actions on paper. Good luck — Professor Shuja’s Web Technologies (SEF24).
