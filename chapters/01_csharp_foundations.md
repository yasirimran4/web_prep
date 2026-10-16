# Chapter 1: C# and .NET Foundations Relevant to Web Development

## 1.1 Class, object, instance, and interface

**Simple definition:** A **class** is a blueprint. An **object** (also called an **instance**) is a live copy of that blueprint in memory. An **interface** is a contract that says *what* methods a type must provide, without saying *how*.

**Why needed:** Web apps need models (`Car`, `Product`), data helpers (`ProductDAL`), and later services. Clear types keep exam code readable.

**Intuition (Roman Urdu tip):** Class = ghar ka map. Object = actual ghar. Interface = rule book: “door aur window hona zaroori hai,” bina yeh bataye ke wood se banao ya steel se.

**Syntax:**

```csharp
public class Car
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal PricePerDay { get; set; }
}

public interface ICarRepository
{
    List<Car> GetAvailable();
}
```

**Exam answer:** A class defines structure and behaviour; an object is a concrete instance of a class; an interface defines a contract that implementing classes must fulfil.

---

## 1.2 Properties, fields, methods, constructors, access modifiers

**Lab evidence:** Lab 02 requires private fields with public get/set properties, parameterized + parameterless constructors, and `ToString()`.

```csharp
// Models/Car.cs  (mirrors Lab 02 Product style)
public class Car
{
    private int _id;
    private string _name = string.Empty;
    private decimal _pricePerDay;

    public int Id
    {
        get { return _id; }
        set { _id = value; }
    }

    public string Name
    {
        get { return _name; }
        set { _name = value; }
    }

    public decimal PricePerDay
    {
        get { return _pricePerDay; }
        set { _pricePerDay = value; }
    }

    // Parameterless — needed for object initializers and model binding
    public Car() { }

    // Parameterized
    public Car(int id, string name, decimal pricePerDay)
    {
        Id = id;
        Name = name;
        PricePerDay = pricePerDay;
    }

    public override string ToString()
    {
        return $"[{Id}] {Name} — PKR {PricePerDay}/day";
    }
}
```

| Keyword | Meaning |
|---------|---------|
| `public` | Visible everywhere |
| `private` | Only inside this class |
| `protected` | This class + derived classes |
| `get` / `set` | Read / write property accessors |
| `override` | Replace a base-class method (`ToString`) |

**Object initializer (Lab 02):**

```csharp
var c = new Car { Id = 1, Name = "Alto", PricePerDay = 3000m };
```

**Common mistakes:** Forgetting parameterless constructor when using `{ Name = ... }`; making fields public instead of properties (breaks encapsulation requirement in lab).

**Practice:** Write a `Product` class matching Lab 02 columns from memory.

---

## 1.3 Namespaces and `using`

**Definition:** A **namespace** groups related types. A **`using`** directive imports a namespace so you can write short type names.

```csharp
using System.Data.SqlClient;   // or Microsoft.Data.SqlClient

namespace CarRental.Models
{
    public class Car { /* ... */ }
}
```

In ASP.NET Core MVC, folders often mirror namespaces: `Controllers`, `Models`, `ViewModels`.

---

## 1.4 Generics, `List<T>`, and `IEnumerable<T>`

**Definition:** **Generics** let you write type-safe collections like `List<Car>` instead of untyped lists.

| Type | Meaning | Typical use |
|------|---------|-------------|
| `List<T>` | Concrete, resizable list; can Add/Remove/index | DAL return type, in-memory HW 4 lists |
| `IEnumerable<T>` | “Something you can foreach”; read-oriented contract | Method parameters, flexible return type |

```csharp
List<Car> available = new List<Car>();
available.Add(new Car(1, "Alto", 3000m));

IEnumerable<Car> viewData = available; // List implements IEnumerable
foreach (Car car in viewData)
{
    Console.WriteLine(car.Name);
}
```

**Why program to `IEnumerable` sometimes?** Caller only needs to loop; you can later change the concrete type. **Why `List` in DAL?** Lab 02 explicitly returns `List<Product>` — fine for exams.

**Exam answer:** `List<T>` is a concrete collection with Add/Remove; `IEnumerable<T>` is an interface for iterating a sequence. Every `List<T>` is an `IEnumerable<T>`, but not every `IEnumerable<T>` is a `List<T>`.

---

## 1.5 Interface vs concrete class; programming to interfaces

```csharp
public interface IProductDal
{
    List<Product> GetAllProducts();
}

public class ProductDAL : IProductDal
{
    public List<Product> GetAllProducts() { /* ADO.NET ... */ return list; }
}
```

| | Class | Interface |
|-|-------|-----------|
| Has method bodies? | Yes | No (until default interface methods — avoid in exam) |
| Can `new` it? | Yes | No |
| Purpose | Implementation | Contract |

**HW 4 note:** Teacher said *do not use interfaces* for that lab. Still learn the idea — viva and later exams often ask.

---

## 1.6 Inheritance and polymorphism (light)

```csharp
public class Vehicle { public string Name { get; set; } = ""; }
public class Car : Vehicle { public decimal PricePerDay { get; set; } }
```

Polymorphism: treat a `Car` as a `Vehicle` when only shared members are needed. Your labs focus more on **composition** (DAL uses Product) than deep inheritance.

---

## 1.7 Exceptions and basic error handling

```csharp
try
{
    // open connection, execute command
}
catch (SqlException ex)
{
    Console.WriteLine("Database error: " + ex.Message);
}
finally
{
    // dispose / close if not using `using`
}
```

Prefer `using` for connections (auto-dispose):

```csharp
using (SqlConnection conn = new SqlConnection(connectionString))
{
    conn.Open();
}
```

---

## 1.8 Value types vs reference types (exam-relevant)

| Value types | Reference types |
|-------------|-----------------|
| `int`, `decimal`, `bool`, `struct` | `class`, `string`, arrays, `List<T>` |
| Copied by value | Variable holds a reference |

`string` is a reference type but often feels immutable. Models like `Car` are reference types — two variables can point to the same car object.

---

## 1.9 .NET, CLR, managed code, assemblies (overview)

| Term | Short definition |
|------|------------------|
| **.NET** | Platform + libraries for building apps |
| **CLR** | Common Language Runtime — runs managed code, GC, JIT |
| **Managed code** | Code executed by the CLR |
| **Assembly** | Compiled unit (`.dll` or `.exe`) |
| **DLL** | Library assembly reused by apps |
| **Executable** | Entry-point assembly (`OutputType` Exe in console labs) |

Your Lab 02 project is a console **Exe** targeting a .NET SDK (`net9.0` appears in local lecture stubs). ASP.NET Core MVC apps also compile to assemblies hosted by a web server (Kestrel).

---

## Chapter 1 practice set

**Q1.** Define class vs object vs interface in three lines.  
**Q2.** Why does Lab 02 ask for both constructors?  
**Q3.** Difference between `List<Car>` and `IEnumerable<Car>`?  
**Q4.** Write a `Product` class with private fields and public properties.  
**Q5.** Find the error: `IProductDal dal = new IProductDal();`

### Answer key (Ch 1)

1. Class = blueprint; object/instance = living copy; interface = contract of members without implementation.  
2. Parameterized for explicit creation; parameterless for object initializers / binding.  
3. List is concrete and mutable with indexes; IEnumerable only guarantees enumeration.  
4. Mirror Lab 02: ProductId, Name, Price, QuantityInStock, Category.  
5. Cannot instantiate an interface; use `new ProductDAL()`.

\newpage
