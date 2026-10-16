# Chapter 14: Full Integrated Car Rental Reference

Two tracks are shown on purpose:

1. **HW 4 track (confirmed assignment):** in-memory lists, no EF/DB/repos/interfaces.  
2. **Architecture track (exam expansion):** ADO.NET repository + optional DI/async — **not** what HW 4 allowed, but connects Chapters 7–10.

Mark configuration you must supply yourself with `YOUR_*`.

---

## 14.1 Folder structure (architecture track)

```
CarRental/
  Program.cs
  appsettings.json          # YOUR connection string here
  Controllers/CarsController.cs
  Models/Car.cs
  ViewModels/AvailableCarsViewModel.cs
  Data/ICarRepository.cs
  Data/CarRepository.cs
  Views/
    _ViewStart.cshtml
    _ViewImports.cshtml
    Shared/_Layout.cshtml
    Shared/_AdminLayout.cshtml
    Cars/Index.cshtml
    Cars/Booked.cshtml
    Cars/Create.cshtml
  wwwroot/
```

---

## 14.2 Model

```csharp
// Models/Car.cs
using System.ComponentModel.DataAnnotations;

namespace CarRental.Models
{
    public class Car
    {
        public int Id { get; set; }

        [Required, StringLength(100)]
        public string Name { get; set; } = string.Empty;

        [Range(1, 1000000)]
        public decimal PricePerDay { get; set; }

        public bool IsBooked { get; set; }
    }
}
```

---

## 14.3 HW 4 controller (in-memory — persistent only until restart)

```csharp
public class CarsController : Controller
{
    private static List<Car> availableCars = new()
    {
        new Car { Id = 1, Name = "Alto", PricePerDay = 3000m },
        new Car { Id = 2, Name = "Corolla", PricePerDay = 6000m },
        new Car { Id = 3, Name = "Civic", PricePerDay = 8000m }
    };
    private static List<Car> bookedCars = new();

    [HttpGet]
    public IActionResult Index() => View(availableCars);

    [HttpPost]
    public IActionResult Book(int id)
    {
        var car = availableCars.FirstOrDefault(c => c.Id == id);
        if (car == null) return RedirectToAction(nameof(Index));
        availableCars.Remove(car);
        bookedCars.Add(car);
        return RedirectToAction(nameof(Booked));
    }

    [HttpGet]
    public IActionResult Booked() => View(bookedCars);
}
```

**This does not permanently save to a database.** Restart clears bookings.

---

## 14.4 SQL schema (architecture track — you create DB)

```sql
-- Run on YOUR SQL Server instance
CREATE TABLE Cars (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Name VARCHAR(100) NOT NULL,
    PricePerDay DECIMAL(10,2) NOT NULL,
    IsBooked BIT NOT NULL DEFAULT 0
);

INSERT INTO Cars (Name, PricePerDay, IsBooked) VALUES
('Alto', 3000, 0), ('Corolla', 6000, 0), ('Civic', 8000, 0);
```

`appsettings.json` (placeholder):

```json
{
  "ConnectionStrings": {
    "Default": "Server=YOUR_SERVER;Database=CarRentalDB;User Id=YOUR_USER;Password=YOUR_PASSWORD;TrustServerCertificate=True;"
  }
}
```

---

## 14.5 Repository (ADO.NET + async)

```csharp
public interface ICarRepository
{
    Task<List<Car>> GetAvailableAsync();
    Task<List<Car>> GetBookedAsync();
    Task<bool> BookAsync(int id);
    Task AddAsync(Car car);
}

public class CarRepository : ICarRepository
{
    private readonly string _cs;
    public CarRepository(IConfiguration config)
    {
        _cs = config.GetConnectionString("Default")
              ?? throw new InvalidOperationException("Set ConnectionStrings:Default");
    }

    public async Task<List<Car>> GetAvailableAsync()
    {
        return await QueryAsync("SELECT Id, Name, PricePerDay, IsBooked FROM Cars WHERE IsBooked = 0");
    }

    public async Task<List<Car>> GetBookedAsync()
    {
        return await QueryAsync("SELECT Id, Name, PricePerDay, IsBooked FROM Cars WHERE IsBooked = 1");
    }

    private async Task<List<Car>> QueryAsync(string sql)
    {
        var list = new List<Car>();
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
                PricePerDay = reader.GetDecimal(2),
                IsBooked = reader.GetBoolean(3)
            });
        }
        return list;
    }

    public async Task<bool> BookAsync(int id)
    {
        const string sql = "UPDATE Cars SET IsBooked = 1 WHERE Id=@id AND IsBooked = 0";
        await using var conn = new SqlConnection(_cs);
        await using var cmd = new SqlCommand(sql, conn);
        cmd.Parameters.AddWithValue("@id", id);
        await conn.OpenAsync();
        return await cmd.ExecuteNonQueryAsync() == 1;
    }

    public async Task AddAsync(Car car)
    {
        const string sql = "INSERT INTO Cars (Name, PricePerDay, IsBooked) VALUES (@n, @p, 0)";
        await using var conn = new SqlConnection(_cs);
        await using var cmd = new SqlCommand(sql, conn);
        cmd.Parameters.AddWithValue("@n", car.Name);
        cmd.Parameters.AddWithValue("@p", car.PricePerDay);
        await conn.OpenAsync();
        await cmd.ExecuteNonQueryAsync();
    }
}
```

---

## 14.6 DI registration + controller

```csharp
// Program.cs
builder.Services.AddControllersWithViews();
builder.Services.AddScoped<ICarRepository, CarRepository>();
```

```csharp
public class CarsController : Controller
{
    private readonly ICarRepository _repo;
    public CarsController(ICarRepository repo) => _repo = repo;

    [HttpGet]
    public async Task<IActionResult> Index()
    {
        var cars = await _repo.GetAvailableAsync();
        var vm = new AvailableCarsViewModel
        {
            Heading = "Available Cars",
            Cars = cars,
            TotalCount = cars.Count
        };
        return View(vm);
    }

    [HttpPost, ValidateAntiForgeryToken]
    public async Task<IActionResult> Book(int id)
    {
        if (!await _repo.BookAsync(id))
            return RedirectToAction(nameof(Index));
        return RedirectToAction(nameof(Booked));
    }

    [HttpGet]
    public async Task<IActionResult> Booked()
        => View(await _repo.GetBookedAsync());

    [HttpGet]
    public IActionResult Create() => View(new Car());

    [HttpPost, ValidateAntiForgeryToken]
    public async Task<IActionResult> Create(Car car)
    {
        if (!ModelState.IsValid) return View(car);
        await _repo.AddAsync(car);
        return RedirectToAction(nameof(Index));
    }
}
```

---

## 14.7 Views + layout

Use Chapter 11 `_Layout` (dark navbar, `@RenderBody()`, footer, Bootstrap 5.3).  
Index: Chapter 5 ViewModel + Chapter 12 cards.  
Booked: Chapter 4 table.  
Create: Chapter 6 Tag Helper form.

**Admin layout example:**

```cshtml
@{ Layout = "_AdminLayout"; }
```

**No layout:**

```cshtml
@{ Layout = null; }
```

**Page script section:**

```cshtml
@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

---

## 14.8 Request flow (architecture track)

```
GET /Cars/Index
  → CarsController.Index (async)
  → ICarRepository.GetAvailableAsync
  → SQL Server SELECT
  → AvailableCarsViewModel
  → Views/Cars/Index.cshtml inside _Layout
  → HTML cards

POST /Cars/Book id=2
  → BookAsync UPDATE
  → 302 Redirect Booked
GET /Cars/Booked
  → table of booked cars
```

---

## 14.9 Conceptual vs production

| Example | Persistent? |
|---------|-------------|
| HW 4 static lists | No — RAM only |
| ADO.NET repository | Yes — SQL Server |
| Missing real connection string | Will not run until you configure `YOUR_*` |

Do not present in-memory demos as if they permanently save bookings.

\newpage
