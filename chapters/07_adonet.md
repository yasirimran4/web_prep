# Chapter 7: Database Connectivity and SQL (ADO.NET)

**High priority.** Confirmed stack from **Lab 02** and **Lab 03**: **ADO.NET** with **SQL Server**, organized in a **DAL** class. **EF Core was not part of your labs; HW 4 forbids it.**

## 7.1 Relational basics

| Term | Meaning |
|------|---------|
| Table | Collection of rows (e.g. `Products`, `Cars`) |
| Row | One record |
| Column | One field |
| Primary key | Unique id (`ProductId IDENTITY`) |
| Foreign key | Reference to another table (not required in Lab 02) |

**CRUD:** Create (INSERT), Read (SELECT), Update (UPDATE), Delete (DELETE).

Lab 02 table:

```sql
CREATE TABLE Products (
    ProductId INT PRIMARY KEY IDENTITY(1,1),
    Name VARCHAR(100) NOT NULL,
    Price DECIMAL(10,2) NOT NULL,
    QuantityInStock INT NOT NULL,
    Category VARCHAR(50) NOT NULL
);
```

Car-rental equivalent (you supply DB name/credentials yourself):

```sql
CREATE TABLE Cars (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Name VARCHAR(100) NOT NULL,
    PricePerDay DECIMAL(10,2) NOT NULL,
    IsBooked BIT NOT NULL DEFAULT 0
);
```

---

## 7.2 What is ADO.NET?

**Definition:** ADO.NET is .NET’s classic API for talking to relational databases using connections, commands, and readers/adapters.

**Why:** Direct SQL control; matches your course labs exactly.

**Important classes (Lab 02/03):**

| Class | Role |
|-------|------|
| `SqlConnection` | Connection to SQL Server |
| `SqlCommand` | SQL text + parameters |
| `SqlDataReader` | Forward-only connected read |
| `SqlParameter` | Safe parameter value |
| `SqlDataAdapter` | Fill/update `DataTable` (disconnected) |
| `SqlCommandBuilder` | Auto-generate update commands for adapter |
| `DataTable` / `DataRow` | In-memory table/row |

Namespaces: `System.Data`, `System.Data.SqlClient` (or `Microsoft.Data.SqlClient` on modern .NET — same ideas).

---

## 7.3 Connection strings

**Pattern only — put your real server/login in config; do not invent passwords in exam notes as if they were real.**

```
Server=YOUR_SERVER;Database=ProductDB;User Id=YOUR_USER;Password=YOUR_PASSWORD;TrustServerCertificate=True;
```

Or Windows auth style (if teacher uses it):

```
Server=.;Database=ProductDB;Integrated Security=True;TrustServerCertificate=True;
```

Store in a private field on the DAL (Lab 02).

---

## 7.4 Connected model (Lab 02)

**Definition:** Keep a connection open while you execute and read (`SqlDataReader`).

**Flow:** Open connection → Execute command → Read rows → Close/dispose.

```csharp
// ProductDAL.cs — pattern from Lab 02
using System.Data.SqlClient;

public class ProductDAL
{
    private readonly string _connectionString;

    public ProductDAL(string connectionString)
    {
        _connectionString = connectionString;
    }

    private SqlConnection GetConnection()
    {
        return new SqlConnection(_connectionString);
    }

    // CREATE
    public void AddProduct(Product product)
    {
        const string sql = @"INSERT INTO Products (Name, Price, QuantityInStock, Category)
                             VALUES (@Name, @Price, @Qty, @Category)";
        using SqlConnection conn = GetConnection();
        using SqlCommand cmd = new SqlCommand(sql, conn);
        cmd.Parameters.AddWithValue("@Name", product.Name);
        cmd.Parameters.AddWithValue("@Price", product.Price);
        cmd.Parameters.AddWithValue("@Qty", product.QuantityInStock);
        cmd.Parameters.AddWithValue("@Category", product.Category);
        conn.Open();
        cmd.ExecuteNonQuery(); // INSERT/UPDATE/DELETE
    }

    // READ ALL
    public List<Product> GetAllProducts()
    {
        var list = new List<Product>();
        const string sql = "SELECT ProductId, Name, Price, QuantityInStock, Category FROM Products";
        using SqlConnection conn = GetConnection();
        using SqlCommand cmd = new SqlCommand(sql, conn);
        conn.Open();
        using SqlDataReader reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            list.Add(new Product
            {
                ProductId = reader.GetInt32(0),
                Name = reader.GetString(1),
                Price = reader.GetDecimal(2),
                QuantityInStock = reader.GetInt32(3),
                Category = reader.GetString(4)
            });
        }
        return list;
    }

    // READ ONE
    public Product? GetProductById(int productId)
    {
        const string sql = @"SELECT ProductId, Name, Price, QuantityInStock, Category
                             FROM Products WHERE ProductId = @Id";
        using SqlConnection conn = GetConnection();
        using SqlCommand cmd = new SqlCommand(sql, conn);
        cmd.Parameters.AddWithValue("@Id", productId);
        conn.Open();
        using SqlDataReader reader = cmd.ExecuteReader();
        if (!reader.Read()) return null;
        return new Product
        {
            ProductId = reader.GetInt32(0),
            Name = reader.GetString(1),
            Price = reader.GetDecimal(2),
            QuantityInStock = reader.GetInt32(3),
            Category = reader.GetString(4)
        };
    }

    // UPDATE
    public void UpdateProduct(Product product)
    {
        const string sql = @"UPDATE Products
                             SET Name=@Name, Price=@Price, QuantityInStock=@Qty, Category=@Category
                             WHERE ProductId=@Id";
        using SqlConnection conn = GetConnection();
        using SqlCommand cmd = new SqlCommand(sql, conn);
        cmd.Parameters.AddWithValue("@Name", product.Name);
        cmd.Parameters.AddWithValue("@Price", product.Price);
        cmd.Parameters.AddWithValue("@Qty", product.QuantityInStock);
        cmd.Parameters.AddWithValue("@Category", product.Category);
        cmd.Parameters.AddWithValue("@Id", product.ProductId);
        conn.Open();
        cmd.ExecuteNonQuery();
    }

    // DELETE
    public void DeleteProduct(int productId)
    {
        const string sql = "DELETE FROM Products WHERE ProductId=@Id";
        using SqlConnection conn = GetConnection();
        using SqlCommand cmd = new SqlCommand(sql, conn);
        cmd.Parameters.AddWithValue("@Id", productId);
        conn.Open();
        cmd.ExecuteNonQuery();
    }
}
```

### Execute methods

| Method | Returns | Use |
|--------|---------|-----|
| `ExecuteNonQuery` | rows affected | INSERT/UPDATE/DELETE |
| `ExecuteReader` | `SqlDataReader` | SELECT many/one |
| `ExecuteScalar` | first cell | COUNT(*), single value |

### Mapping query → objects

1. Execute reader.  
2. For each row, `new Product { ... }`.  
3. Add to `List<Product>`.  
4. Return list to menu/controller — **SQL stays inside DAL**.

---

## 7.5 Parameterized queries and SQL injection (Lab 02 + 03)

**Bad (vulnerable):**

```csharp
string sql = "SELECT * FROM Products WHERE Name LIKE '%" + keyword + "%'";
```

Attacker types: `' OR 1=1 --` and may see all rows / worse.

**Good (Lab 03 SearchProductsByName):**

```csharp
public List<Product> SearchProductsByName(string keyword)
{
    var list = new List<Product>();
    const string sql = @"SELECT ProductId, Name, Price, QuantityInStock, Category
                         FROM Products WHERE Name LIKE @keyword";
    using SqlConnection conn = GetConnection();
    using SqlCommand cmd = new SqlCommand(sql, conn);
    cmd.Parameters.AddWithValue("@keyword", "%" + keyword + "%");
    // wildcards in VALUE, not concatenated into SQL text
    conn.Open();
    using SqlDataReader reader = cmd.ExecuteReader();
    while (reader.Read())
    {
        list.Add(/* map columns */);
    }
    return list;
}
```

**Lab comment to remember:** Parameterized commands keep user input **data**, not **executable SQL**.

---

## 7.6 Disconnected model (Lab 03)

**Definition:** Load data into a `DataTable`, close the connection, edit in memory, later push changes with `SqlDataAdapter.Update`.

```csharp
public DataTable LoadProductsTable(out SqlDataAdapter adapter)
{
    adapter = new SqlDataAdapter("SELECT ProductId, Name, Price, QuantityInStock, Category FROM Products",
                                 GetConnection());
    SqlCommandBuilder builder = new SqlCommandBuilder(adapter);
    DataTable table = new DataTable();
    adapter.Fill(table); // opens/closes connection for you
    return table;
}

public void UpdatePriceInTable(DataTable table, SqlDataAdapter adapter, int productId, decimal newPrice)
{
    DataRow[] rows = table.Select("ProductId = " + productId);
    if (rows.Length == 0) return;
    rows[0]["Price"] = newPrice;
    adapter.Update(table); // writes back to SQL Server
}
```

| Connected | Disconnected |
|-----------|--------------|
| `SqlDataReader` while connected | `DataTable` in memory |
| Good for quick read loops | Good for edit sets / UI grids |
| Lab 02 CRUD | Lab 03 price update module |

---

## 7.7 JSON export/import (Lab 03)

```csharp
using System.Text.Json;

public void ExportProductsToJson(string filePath)
{
    List<Product> products = GetAllProducts();
    string json = JsonSerializer.Serialize(products);
    File.WriteAllText(filePath, json);
}

public void ImportProductsFromJson(string filePath)
{
    string json = File.ReadAllText(filePath);
    List<Product>? products = JsonSerializer.Deserialize<List<Product>>(json);
    if (products == null) return;
    foreach (var p in products)
        AddProduct(p); // mind duplicates in real apps
}
```

---

## 7.8 Transactions (conceptual — verify if taught)

```csharp
using SqlConnection conn = GetConnection();
conn.Open();
using SqlTransaction tx = conn.BeginTransaction();
try
{
    // several commands with CommandTransaction = tx
    tx.Commit();
}
catch
{
    tx.Rollback();
    throw;
}
```

**Why:** All succeed or all fail (e.g. book car + payment).

---

## 7.9 Async ADO.NET (extension for web — Chapter 9)

If using async:

- `OpenAsync`
- `ExecuteReaderAsync`
- `ExecuteNonQueryAsync`
- `ExecuteScalarAsync`

Only if your exam expects async — verify. Pattern matches sync names + `Async` suffix + `await`.

---

## 7.10 What was NOT confirmed as taught

| Technology | Status |
|------------|--------|
| ADO.NET + SQL Server | **Confirmed** Labs 02–03 |
| DAL class | **Confirmed** |
| EF Core | **Not in labs; HW 4 forbids** |
| Dapper | **Not evidenced** |
| PostgreSQL | **Not evidenced** (SQL Server in Lab 02) |

---

## 7.11 Common mistakes

1. SQL in the console/controller instead of DAL.  
2. String-concatenating user input into SQL.  
3. Forgetting `Open()`.  
4. Not disposing connections (`using`).  
5. Assuming `ExecuteNonQuery` returns the new identity (use `SCOPE_IDENTITY()` / scalar if needed).  
6. Confusing connected reader with disconnected adapter.

**Exam answer:** ADO.NET uses `SqlConnection` and `SqlCommand` to run SQL; `SqlDataReader` reads connected results; parameterized queries prevent SQL injection; a DAL hides all of this from the UI layer.

\newpage
