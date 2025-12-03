# 📖 QUICK REFERENCE GUIDE
# **Tài liệu tham khảo nhanh ASP.NET Core**

---

## 📌 Common Commands

### .NET CLI

```powershell
# Tạo project mới
dotnet new webapi -n MyAPI
dotnet new mvc -n MyMVC

# Restore packages
dotnet restore

# Build project
dotnet build

# Run project
dotnet run

# Watch mode (auto reload)
dotnet watch run

# Add package
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

### Entity Framework Core

```powershell
# Install EF tools
dotnet tool install --global dotnet-ef

# Add migration
dotnet ef migrations add MigrationName

# Update database
dotnet ef database update

# Remove last migration
dotnet ef migrations remove

# Drop database
dotnet ef database drop

# Generate SQL script
dotnet ef migrations script
```

---

## 📌 Project Structure Best Practices

```
EShopAPI/
├── Controllers/       ← API Endpoints
├── Models/           ← Entity classes
├── DTOs/             ← Data Transfer Objects
├── Services/         ← Business logic
├── Repositories/     ← Data access
├── Data/             ← DbContext
├── Middleware/       ← Custom middleware
├── Helpers/          ← Utilities
└── Program.cs        ← Configuration
```

---

## 📌 Common Code Patterns

### Controller Template

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;
    
    public ProductsController(IProductService service)
    {
        _service = service;
    }
    
    [HttpGet]
    public async Task<ActionResult<List<Product>>> GetAll()
    {
        var items = await _service.GetAllAsync();
        return Ok(items);
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<Product>> GetById(int id)
    {
        var item = await _service.GetByIdAsync(id);
        if (item == null) return NotFound();
        return Ok(item);
    }
    
    [HttpPost]
    public async Task<ActionResult<Product>> Create(CreateRequest request)
    {
        var item = await _service.CreateAsync(request);
        return CreatedAtAction(nameof(GetById), new { id = item.Id }, item);
    }
    
    [HttpPut("{id}")]
    public async Task<ActionResult<Product>> Update(int id, UpdateRequest request)
    {
        var item = await _service.UpdateAsync(id, request);
        return Ok(item);
    }
    
    [HttpDelete("{id}")]
    public async Task<ActionResult> Delete(int id)
    {
        await _service.DeleteAsync(id);
        return NoContent();
    }
}
```

### Service Template

```csharp
public interface IProductService
{
    Task<List<Product>> GetAllAsync();
    Task<Product?> GetByIdAsync(int id);
    Task<Product> CreateAsync(CreateRequest request);
    Task<Product> UpdateAsync(int id, UpdateRequest request);
    Task DeleteAsync(int id);
}

public class ProductService : IProductService
{
    private readonly IProductRepository _repo;
    
    public ProductService(IProductRepository repo)
    {
        _repo = repo;
    }
    
    public async Task<List<Product>> GetAllAsync()
    {
        return await _repo.GetAllAsync();
    }
    
    // ... implement other methods
}
```

### Repository Template

```csharp
public interface IRepository<T> where T : class
{
    Task<List<T>> GetAllAsync();
    Task<T?> GetByIdAsync(int id);
    Task<T> CreateAsync(T entity);
    Task<T> UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

public class Repository<T> : IRepository<T> where T : class
{
    protected readonly AppDbContext _context;
    protected readonly DbSet<T> _dbSet;
    
    public Repository(AppDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }
    
    public virtual async Task<List<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }
    
    // ... implement other methods
}
```

---

## 📌 Common Packages

```xml
<!-- Web API -->
<PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="8.0.0" />
<PackageReference Include="Swashbuckle.AspNetCore" Version="6.5.0" />

<!-- Entity Framework Core -->
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="8.0.0" />

<!-- Authentication -->
<PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="8.0.0" />
<PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="8.0.0" />

<!-- Utilities -->
<PackageReference Include="AutoMapper" Version="12.0.1" />
<PackageReference Include="FluentValidation.AspNetCore" Version="11.3.0" />
<PackageReference Include="Serilog.AspNetCore" Version="8.0.0" />
```

---

## 📌 Common HTTP Status Codes

| Code | Meaning | When to use |
|------|---------|-------------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Validation error |
| 401 | Unauthorized | Not authenticated |
| 403 | Forbidden | Not authorized |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Duplicate, constraint violation |
| 500 | Internal Server Error | Server error |

---

## 📌 LINQ Quick Reference

```csharp
// Filter
products.Where(p => p.Price > 1000)

// Map/Transform
products.Select(p => new { p.Id, p.Name })

// Sort
products.OrderBy(p => p.Price)
products.OrderByDescending(p => p.Price)

// Pagination
products.Skip((page - 1) * pageSize).Take(pageSize)

// First/Single
products.FirstOrDefault(p => p.Id == 1)

// Aggregate
products.Count()
products.Sum(p => p.Price)
products.Average(p => p.Price)
products.Max(p => p.Price)

// Check
products.Any(p => p.Stock > 0)
products.All(p => p.Price > 0)

// Join
products.Join(categories, 
    p => p.CategoryId, 
    c => c.Id, 
    (p, c) => new { p.Name, c.CategoryName })

// Group
products.GroupBy(p => p.CategoryId)
```

---

## 📌 Dependency Injection Lifetimes

| Lifetime | Behavior | Use for |
|----------|----------|---------|
| **Transient** | New instance every time | Lightweight, stateless services |
| **Scoped** | One per request | DbContext, repositories |
| **Singleton** | One for entire app lifetime | Configuration, cache |

```csharp
// Registration
builder.Services.AddTransient<IEmailService, EmailService>();
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddSingleton<ICacheService, CacheService>();
```

---

## 📌 Connection String Templates

### Local SQL Server (Windows Auth)
```json
"Server=.;Database=MyDb;Trusted_Connection=True;TrustServerCertificate=True"
```

### Local SQL Server (SQL Auth)
```json
"Server=.;Database=MyDb;User Id=sa;Password=YourPassword;TrustServerCertificate=True"
```

### Azure SQL Database
```json
"Server=tcp:myserver.database.windows.net,1433;Database=MyDb;User ID=admin;Password=YourPassword;Encrypt=True"
```

---

## 📌 Common Errors & Solutions

### 1. DbContext null reference
**Error:** Object reference not set to an instance of an object  
**Solution:** Đăng ký DbContext trong Program.cs
```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
```

### 2. Migration fails
**Error:** The migrations configuration type was not found  
**Solution:** Rebuild project, ensure tools installed
```powershell
dotnet ef migrations add InitialCreate --project YourProject
```

### 3. CORS error
**Error:** Access to fetch has been blocked by CORS policy  
**Solution:** Add CORS policy
```csharp
builder.Services.AddCors(options => {
    options.AddPolicy("AllowAll", 
        policy => policy.AllowAnyOrigin().AllowAnyMethod().AllowAnyHeader());
});
app.UseCors("AllowAll");
```

### 4. 401 Unauthorized
**Error:** 401 Unauthorized even with valid token  
**Solution:** Check middleware order
```csharp
app.UseAuthentication(); // BEFORE UseAuthorization
app.UseAuthorization();
```

---

## 📌 Testing với Postman

### Setup Environment
```json
{
  "baseUrl": "https://localhost:5001",
  "token": ""
}
```

### Save Token After Login
```javascript
// Tests tab in Login request
pm.test("Save token", function () {
    var data = pm.response.json();
    pm.environment.set("token", data.token);
});
```

### Use Token in Requests
```
Authorization: Bearer {{token}}
```

---

## 📌 Git Commands for Project

```bash
# Initialize
git init
git add .
git commit -m "Initial commit"

# Common workflow
git status
git add .
git commit -m "Add feature X"
git push origin main

# Branching
git checkout -b feature/new-api
git checkout main
git merge feature/new-api
```

---

## 📌 Deployment Checklist

- [ ] Remove sensitive data from appsettings.json
- [ ] Use Environment Variables for secrets
- [ ] Enable HTTPS
- [ ] Configure CORS properly
- [ ] Add logging (Serilog)
- [ ] Add health checks
- [ ] Configure connection string pooling
- [ ] Enable compression
- [ ] Add rate limiting
- [ ] Document API with Swagger

---

**Tài liệu này sẽ được cập nhật thường xuyên!**
