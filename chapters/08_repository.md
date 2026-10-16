# Chapter 8: Repository Pattern and Application Architecture

## 8.1 Your DAL *is* the repository idea

**Lab wording:** `ProductDAL` — Data Access Layer.  
**Industry wording:** Repository — same job: **all database code in one place**, UI/controllers never open `SqlConnection`.

**HW 4 reminder:** That assignment forbids repositories/interfaces and uses in-memory lists. This chapter is for architecture understanding beyond HW 4.

---

## 8.2 Why not put SQL in controllers?

| Problem if SQL in controller | Benefit of DAL/Repository |
|------------------------------|---------------------------|
| Duplicated connection code | One place to fix bugs |
| Hard to test | Can mock interface later |
| Fat controllers | Thin actions: call method, return View |
| Mix HTML concerns with SQL | Clear separation |

---

## 8.3 Model vs repository vs service vs controller

| Layer | Job | Car Rental example |
|-------|-----|--------------------|
| Model | Data shape | `Car` |
| Repository/DAL | CRUD SQL | `CarRepository` / `CarDAL` |
| Service (optional) | Business rules | “Cannot book if already booked” |
| Controller | HTTP in/out | `CarsController` |
| View | HTML | `Index.cshtml` |

Beginner exams often merge service into controller + repository.

---

## 8.4 Folder structure (Car Rental + ADO.NET)

```
CarRental/
  Models/Car.cs
  Data/ICarRepository.cs      # optional interface
  Data/CarRepository.cs       # ADO.NET implementation
  Controllers/CarsController.cs
  Views/...
  Program.cs
```

---

## 8.5 Interface + implementation

```csharp
// Data/ICarRepository.cs
public interface ICarRepository
{
    List<Car> GetAvailable();
    List<Car> GetBooked();
    bool Book(int id);
}

// Data/CarRepository.cs
public class CarRepository : ICarRepository
{
    private readonly string _cs;
    public CarRepository(string connectionString) => _cs = connectionString;

    public List<Car> GetAvailable()
    {
        // SELECT ... WHERE IsBooked = 0  with SqlDataReader
        // map to List<Car>
        throw new NotImplementedException("Write full SQL in exam practice");
    }

    public List<Car> GetBooked() { /* IsBooked = 1 */ throw new NotImplementedException(); }

    public bool Book(int id)
    {
        // UPDATE Cars SET IsBooked = 1 WHERE Id=@id AND IsBooked = 0
        // return rowsAffected == 1
        throw new NotImplementedException();
    }
}
```

**Controller calling repository:**

```csharp
public class CarsController : Controller
{
    private readonly ICarRepository _repo;
    public CarsController(ICarRepository repo) => _repo = repo;

    public IActionResult Index() => View(_repo.GetAvailable());

    [HttpPost]
    public IActionResult Book(int id)
    {
        if (!_repo.Book(id))
            return RedirectToAction(nameof(Index));
        return RedirectToAction(nameof(Booked));
    }

    public IActionResult Booked() => View(_repo.GetBooked());
}
```

**Data flow:** Browser → Controller → Repository → SQL Server → Repository maps rows → Controller → View → Browser.

---

## 8.6 Advantages and limitations

**Advantages:** separation of concerns, reuse, easier testing with fakes, cleaner controllers.  
**Limitations:** extra types/files; overkill for tiny HW 4; bad repositories that leak SQL details still hurt.

**Exam answer:** A repository (or DAL) centralizes data access so controllers request data through methods instead of embedding SQL and connection code.

\newpage
