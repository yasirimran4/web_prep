# Chapter 9: Asynchronous Programming in C# and ASP.NET Core

**Critical chapter.** Async was not named in Lab 01–03 PDFs; it is included because your master prompt lists it as studied and it is standard for ASP.NET Core + ADO.NET. **Verify with your teacher whether hand-written async is on the exam.**

## 9.1 Synchronous vs asynchronous

**Sync:** Thread waits idle while DB responds.  
**Async:** While waiting for I/O (database, file, network), the thread can serve other requests.

**Intuition:** Sync = cashier stands frozen waiting for card machine. Async = cashier helps next customer while machine processes.

**Roman Urdu:** Async ka matlab CPU ko bekaar khada na rakhna jab tak database jawab de — wait ke dauran doosri request handle ho sakti hai.

Async does **not** magically speed up heavy CPU math; it helps **I/O waits**.

---

## 9.2 `async`, `await`, `Task`, `Task<T>`

| Keyword / type | Meaning |
|----------------|---------|
| `async` | Marks method as asynchronous; allows `await` inside |
| `await` | Wait for a Task to finish **without blocking the thread** the same way `.Result` does |
| `Task` | Represents an async operation with **no return value** |
| `Task<T>` | Async operation that eventually returns a **T** |

```csharp
// Returns nothing useful to caller except "work finished"
public async Task SaveAsync()
{
    await cmd.ExecuteNonQueryAsync();
}

// Returns List<Car> when done
public async Task<List<Car>> GetAvailableAsync()
{
    // ... await reader ...
    return list;
}
```

### `void` vs `Task` vs `Task<T>`

| Return type | Use |
|-------------|-----|
| `async void` | **Avoid** (except event handlers) — exceptions hard to catch |
| `async Task` | Async method with no result |
| `async Task<T>` | Async method that returns `T` |

---

## 9.3 What `await` does (before / after)

```csharp
public async Task<IActionResult> Index()
{
    // BEFORE await: still on request context
    var cars = await _repo.GetAvailableAsync();
    // AFTER await: continuation runs when DB finished
    return View(cars);
}
```

While awaiting, ASP.NET can use the thread for other requests.

---

## 9.4 Side-by-side: sync vs async MVC

**Synchronous:**

```csharp
public IActionResult Index()
{
    var cars = _repository.GetAll();
    return View(cars);
}
```

**Asynchronous:**

```csharp
public async Task<IActionResult> Index()
{
    var cars = await _repository.GetAllAsync();
    return View(cars);
}
```

Every layer that awaits must itself be async (propagate async all the way).

---

## 9.5 Full stack trace (interface → repository → ADO.NET → controller → view)

```csharp
// 1) Interface
public interface ICarRepository
{
    Task<List<Car>> GetAvailableAsync();
    Task<bool> BookAsync(int id);
}

// 2) Repository with real async ADO.NET
public class CarRepository : ICarRepository
{
    private readonly string _cs;
    public CarRepository(string cs) => _cs = cs;

    public async Task<List<Car>> GetAvailableAsync()
    {
        var list = new List<Car>();
        const string sql = "SELECT Id, Name, PricePerDay FROM Cars WHERE IsBooked = 0";
        await using var conn = new SqlConnection(_cs);
        await using var cmd = new SqlCommand(sql, conn);
        await conn.OpenAsync();
        await using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            list.Add(new Car
            {
                Id = reader.GetInt32(0),
                Name = reader.GetString(1),
                PricePerDay = reader.GetDecimal(2)
            });
        }
        return list;
    }

    public async Task<bool> BookAsync(int id)
    {
        const string sql = @"UPDATE Cars SET IsBooked = 1
                             WHERE Id = @id AND IsBooked = 0";
        await using var conn = new SqlConnection(_cs);
        await using var cmd = new SqlCommand(sql, conn);
        cmd.Parameters.AddWithValue("@id", id);
        await conn.OpenAsync();
        int rows = await cmd.ExecuteNonQueryAsync();
        return rows == 1;
    }
}

// 3) Controller
public class CarsController : Controller
{
    private readonly ICarRepository _repo;
    public CarsController(ICarRepository repo) => _repo = repo;

    public async Task<IActionResult> Index()
    {
        List<Car> cars = await _repo.GetAvailableAsync();
        return View(cars);
    }

    [HttpPost]
    public async Task<IActionResult> Book(int id)
    {
        bool ok = await _repo.BookAsync(id);
        if (!ok) return RedirectToAction(nameof(Index));
        return RedirectToAction(nameof(Booked));
    }

    public async Task<IActionResult> Booked()
    {
        // similar GetBookedAsync...
        return View(await _repo.GetAvailableAsync()); // replace with GetBookedAsync in real code
    }
}
```

### Where `async` / `await` are required

| Layer | Must be async? | Why |
|-------|----------------|-----|
| ADO.NET call | Yes (`OpenAsync`, etc.) | Actual I/O |
| Repository method | Yes | awaits DB |
| Interface signature | Yes (`Task<...>`) | match implementation |
| Controller action | Yes | awaits repository |
| View | No | just displays list |

**Wrong:** mark controller `async` but call sync `GetAll()` with no await — gains nothing.  
**Wrong:** `async` method with no `await` — compiler warning; still sync work.

---

## 9.6 Exception handling

```csharp
public async Task<List<Car>> GetAvailableAsync()
{
    try
    {
        // await DB
        return list;
    }
    catch (SqlException ex)
    {
        // log ex
        throw; // or return empty / rethrow domain exception
    }
}
```

`await` unwraps exceptions from the Task into normal `catch`.

---

## 9.7 Avoid `.Result` and `.Wait()` in web apps

```csharp
// BAD — can deadlock / block thread pool
var cars = _repo.GetAvailableAsync().Result;
_repo.GetAvailableAsync().Wait();
```

Prefer `await` all the way up.

---

## 9.8 Concurrency vs parallelism vs async I/O (short)

| Term | Idea |
|------|------|
| Async I/O | Don’t block waiting for DB/network |
| Concurrency | Many tasks in progress (interleaved) |
| Parallelism | Multiple CPU cores compute together |

Web DB calls → **async I/O**, not “run SQL on 8 cores.”

---

## 9.9 Exam traps

1. Returning `List<Car>` from method declared `Task<List<Car>>` without Task — use `return list` inside `async` method (compiler wraps) or `return Task.FromResult(list)`.  
2. Interface sync but implementation async — signatures must match.  
3. Forgetting `await` → you pass a Task object to the view by mistake.  
4. `async void` action methods.  
5. Mixing: controller async, repository sync only — incomplete story.

**Exam answer:** `async`/`await` with `Task`/`Task<T>` let web actions wait for database I/O without blocking threads; asynchronous methods should be propagated from data access through the repository to the controller.

**Practice:** Convert Lab 02 `GetAllProducts` to `GetAllProductsAsync` and show the controller signature change.

\newpage
