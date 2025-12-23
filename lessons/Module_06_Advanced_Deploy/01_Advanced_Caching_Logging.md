# 🟧 CHƯƠNG 12
# **ADVANCED TOPICS: CACHING, LOGGING, ERROR HANDLING & DEPLOYMENT**

Chương này tập hợp các kỹ thuật nâng cao giúp ứng dụng chạy nhanh, ổn định và dễ maintain trong môi trường production.

---

# 🎯 MỤC TIÊU HỌC TẬP

Sau chương này, bạn sẽ:

- Implement In-Memory Caching để tăng performance
- Sử dụng Serilog cho logging chuyên nghiệp
- Xử lý lỗi tập trung với Global Exception Handler
- Configure Health Checks
- Deploy lên IIS và Azure
- Best practices cho Production

---

# 1. **IN-MEMORY CACHING**

## 1.1. Tại sao cần Caching?

### 🏠 Ví dụ đời sống: Tủ lạnh

- **Không cache**: Mỗi lần nấu ăn phải ra chợ mua  
- **Có cache**: Mua sẵn để trong tủ lạnh, nấu ngay khi cần

```csharp
// ❌ Không cache: Query DB mỗi request
public async Task<List<Category>> GetCategories()
{
    return await _db.Categories.ToListAsync(); // Chậm!
}

// ✅ Có cache: Query DB 1 lần, sau đó lấy từ cache
public async Task<List<Category>> GetCategories()
{
    var cacheKey = "categories";
    
    if (!_cache.TryGetValue(cacheKey, out List<Category> categories))
    {
        categories = await _db.Categories.ToListAsync();
        _cache.Set(cacheKey, categories, TimeSpan.FromMinutes(10));
    }
    
    return categories;
}
```

---

## 1.2. Setup Caching

```csharp
// Program.cs
builder.Services.AddMemoryCache();

// Controller
public class CategoriesController : Controller
{
    private readonly IMemoryCache _cache;
    private readonly AppDbContext _db;
    
    public CategoriesController(IMemoryCache cache, AppDbContext db)
    {
        _cache = cache;
        _db = db;
    }
}
```

---

## 1.3. Cache Operations

### Set Cache

```csharp
public async Task<List<Product>> GetProducts()
{
    var cacheKey = "all_products";
    
    if (!_cache.TryGetValue(cacheKey, out List<Product> products))
    {
        // Cache miss → Query DB
        products = await _db.Products.Include(p => p.Category).ToListAsync();
        
        // Lưu vào cache với options
        var cacheOptions = new MemoryCacheEntryOptions()
            .SetAbsoluteExpiration(TimeSpan.FromMinutes(10)) // Hết hạn sau 10 phút
            .SetSlidingExpiration(TimeSpan.FromMinutes(2))   // Gia hạn nếu được access trong 2 phút
            .SetPriority(CacheItemPriority.Normal);
        
        _cache.Set(cacheKey, products, cacheOptions);
    }
    
    return products;
}
```

---

### Remove Cache

```csharp
public async Task<Product> CreateProduct(Product product)
{
    await _db.Products.AddAsync(product);
    await _db.SaveChangesAsync();
    
    // Xóa cache vì data đã thay đổi
    _cache.Remove("all_products");
    
    return product;
}
```

---

### Cache with different keys

```csharp
public async Task<List<Product>> GetProductsByCategory(int categoryId)
{
    var cacheKey = $"products_category_{categoryId}";
    
    if (!_cache.TryGetValue(cacheKey, out List<Product> products))
    {
        products = await _db.Products
            .Where(p => p.CategoryId == categoryId)
            .ToListAsync();
        
        _cache.Set(cacheKey, products, TimeSpan.FromMinutes(5));
    }
    
    return products;
}
```

---

## 1.4. Cache Service Pattern

```csharp
// Services/CacheService.cs
public interface ICacheService
{
    T? Get<T>(string key);
    void Set<T>(string key, T value, TimeSpan? expiration = null);
    void Remove(string key);
    void RemoveByPattern(string pattern);
}

public class CacheService : ICacheService
{
    private readonly IMemoryCache _cache;
    
    public CacheService(IMemoryCache cache)
    {
        _cache = cache;
    }
    
    public T? Get<T>(string key)
    {
        return _cache.TryGetValue(key, out T value) ? value : default;
    }
    
    public void Set<T>(string key, T value, TimeSpan? expiration = null)
    {
        var options = new MemoryCacheEntryOptions();
        
        if (expiration.HasValue)
            options.SetAbsoluteExpiration(expiration.Value);
        else
            options.SetAbsoluteExpiration(TimeSpan.FromMinutes(10)); // Default
        
        _cache.Set(key, value, options);
    }
    
    public void Remove(string key)
    {
        _cache.Remove(key);
    }
    
    public void RemoveByPattern(string pattern)
    {
        // In-memory cache không support pattern matching
        // Cần dùng Redis nếu cần feature này
    }
}

// Program.cs
builder.Services.AddSingleton<ICacheService, CacheService>();
```

---

# 2. **LOGGING VỚI SERILOG**

## 2.1. Install Serilog

```powershell
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.File
dotnet add package Serilog.Sinks.Console
```

---

## 2.2. Configure Serilog

```csharp
// Program.cs
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// Configure Serilog
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
    .MinimumLevel.Override("Microsoft.Hosting.Lifetime", LogEventLevel.Information)
    .WriteTo.Console()
    .WriteTo.File(
        "logs/eshop-.txt",
        rollingInterval: RollingInterval.Day,
        outputTemplate: "{Timestamp:yyyy-MM-dd HH:mm:ss.fff} [{Level:u3}] {Message:lj}{NewLine}{Exception}")
    .CreateLogger();

builder.Host.UseSerilog();

// ... rest of configuration

var app = builder.Build();

// Log requests
app.UseSerilogRequestLogging();

app.Run();
```

---

## 2.3. Logging trong Code

```csharp
public class ProductsController : Controller
{
    private readonly ILogger<ProductsController> _logger;
    private readonly AppDbContext _db;
    
    public ProductsController(ILogger<ProductsController> logger, AppDbContext db)
    {
        _logger = logger;
        _db = db;
    }
    
    public async Task<IActionResult> Index()
    {
        _logger.LogInformation("Fetching all products");
        
        try
        {
            var products = await _db.Products.ToListAsync();
            
            _logger.LogInformation("Found {Count} products", products.Count);
            
            return View(products);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error fetching products");
            throw;
        }
    }
    
    public async Task<IActionResult> Create(Product product)
    {
        _logger.LogInformation("Creating product: {@Product}", product);
        
        await _db.Products.AddAsync(product);
        await _db.SaveChangesAsync();
        
        _logger.LogInformation("Product created with ID: {ProductId}", product.Id);
        
        return RedirectToAction("Index");
    }
}
```

---

## 2.4. Log Levels

```csharp
_logger.LogTrace("Very detailed information");      // Development only
_logger.LogDebug("Debug information");              // Development
_logger.LogInformation("General information");      // Production
_logger.LogWarning("Warning message");              // Production
_logger.LogError(ex, "Error occurred");             // Production
_logger.LogCritical(ex, "Critical error!");         // Production
```

---

# 3. **GLOBAL EXCEPTION HANDLING**

## 3.1. Exception Middleware

```csharp
// Middleware/GlobalExceptionMiddleware.cs
public class GlobalExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<GlobalExceptionMiddleware> _logger;
    
    public GlobalExceptionMiddleware(
        RequestDelegate next, 
        ILogger<GlobalExceptionMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "An unhandled exception occurred");
            await HandleExceptionAsync(context, ex);
        }
    }
    
    private static Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        context.Response.ContentType = "application/json";
        context.Response.StatusCode = exception switch
        {
            NotFoundException => StatusCodes.Status404NotFound,
            ValidationException => StatusCodes.Status400BadRequest,
            UnauthorizedAccessException => StatusCodes.Status401Unauthorized,
            _ => StatusCodes.Status500InternalServerError
        };
        
        var response = new
        {
            statusCode = context.Response.StatusCode,
            message = exception.Message,
            details = context.Request.Host.Host == "localhost" ? exception.StackTrace : null
        };
        
        return context.Response.WriteAsJsonAsync(response);
    }
}

// Program.cs
app.UseMiddleware<GlobalExceptionMiddleware>();
```

---

## 3.2. Custom Exceptions

```csharp
// Exceptions/NotFoundException.cs
public class NotFoundException : Exception
{
    public NotFoundException(string message) : base(message)
    {
    }
}

// Exceptions/ValidationException.cs
public class ValidationException : Exception
{
    public ValidationException(string message) : base(message)
    {
    }
}

// Usage
public async Task<Product> GetProductByIdAsync(int id)
{
    var product = await _db.Products.FindAsync(id);
    
    if (product == null)
        throw new NotFoundException($"Product with ID {id} not found");
    
    return product;
}
```

---

# 4. **HEALTH CHECKS**

```csharp
// Program.cs
builder.Services.AddHealthChecks()
    .AddDbContextCheck<AppDbContext>()
    .AddCheck("API", () => HealthCheckResult.Healthy("API is running"));

var app = builder.Build();

// Endpoint: /health
app.MapHealthChecks("/health");

// Endpoint với chi tiết: /health/detail
app.MapHealthChecks("/health/detail", new HealthCheckOptions
{
    ResponseWriter = async (context, report) =>
    {
        context.Response.ContentType = "application/json";
        var result = JsonSerializer.Serialize(new
        {
            status = report.Status.ToString(),
            checks = report.Entries.Select(e => new
            {
                name = e.Key,
                status = e.Value.Status.ToString(),
                description = e.Value.Description,
                duration = e.Value.Duration
            })
        });
        await context.Response.WriteAsync(result);
    }
});
```

---

# 5. **COMPRESSION**

```csharp
// Program.cs
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;
    options.Providers.Add<GzipCompressionProvider>();
});

builder.Services.Configure<GzipCompressionProviderOptions>(options =>
{
    options.Level = System.IO.Compression.CompressionLevel.Fastest;
});

var app = builder.Build();

app.UseResponseCompression();
```

---

# 6. **DEPLOYMENT**

## 6.1. Publish Application

```powershell
# Build Release
dotnet build -c Release

# Publish
dotnet publish -c Release -o ./publish
```

---

## 6.2. appsettings.Production.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=production-server;Database=EShopDb;User Id=sa;Password=YourStrongPassword"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "JwtSettings": {
    "SecretKey": "your-production-secret-key-from-environment-variable"
  }
}
```

---

## 6.3. IIS Deployment

1. **Install .NET Hosting Bundle** trên server
2. **Tạo Application Pool** (No Managed Code)
3. **Publish** ứng dụng
4. **Copy** files vào `C:\inetpub\wwwroot\eshop`
5. **Tạo Website** trong IIS
6. **Configure** Connection String

```xml
<!-- web.config tự động generate -->
<configuration>
  <system.webServer>
    <handlers>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" />
    </handlers>
    <aspNetCore processPath="dotnet" 
                arguments=".\EShopAPI.dll" 
                stdoutLogEnabled="true" 
                hostingModel="inprocess" />
  </system.webServer>
</configuration>
```

---

## 6.4. Azure App Service Deployment

```powershell
# Login Azure CLI
az login

# Create Resource Group
az group create --name eshop-rg --location eastus

# Create App Service Plan
az appservice plan create --name eshop-plan --resource-group eshop-rg --sku B1

# Create Web App
az webapp create --name eshop-api --resource-group eshop-rg --plan eshop-plan

# Deploy
az webapp deployment source config-zip `
  --resource-group eshop-rg `
  --name eshop-api `
  --src ./publish.zip
```

---

## 6.5. Docker Deployment

```dockerfile
# Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["EShopAPI.csproj", "./"]
RUN dotnet restore "EShopAPI.csproj"
COPY . .
RUN dotnet build "EShopAPI.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "EShopAPI.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "EShopAPI.dll"]
```

```powershell
# Build image
docker build -t eshop-api:latest .

# Run container
docker run -d -p 8080:80 --name eshop-api eshop-api:latest
```

---

# 7. **BEST PRACTICES CHECKLIST**

## 🟢 Security

- [ ] Không hardcode connection strings, secrets
- [ ] Sử dụng Environment Variables cho production
- [ ] Enable HTTPS
- [ ] Validate tất cả inputs
- [ ] Implement rate limiting
- [ ] Sử dụng JWT với expiration time
- [ ] Hash passwords với BCrypt/Identity

---

## 🟢 Performance

- [ ] Sử dụng Async/Await
- [ ] Implement caching cho data ít thay đổi
- [ ] Pagination cho danh sách lớn
- [ ] Eager loading (Include) thay vì N+1 queries
- [ ] Index các cột thường query
- [ ] Enable response compression

---

## 🟢 Logging & Monitoring

- [ ] Log tất cả errors
- [ ] Log important events (login, create order)
- [ ] Không log sensitive data (password, credit card)
- [ ] Configure different log levels cho dev/prod
- [ ] Implement health checks
- [ ] Monitor performance metrics

---

## 🟢 Code Quality

- [ ] Sử dụng Dependency Injection
- [ ] Repository Pattern cho data access
- [ ] Service Layer cho business logic
- [ ] DTO/ViewModel thay vì Entity
- [ ] Unit tests cho critical logic
- [ ] Code review trước khi merge

---

## 🟢 Deployment

- [ ] Separate configs cho dev/staging/prod
- [ ] Automated CI/CD pipeline
- [ ] Database migrations
- [ ] Backup strategy
- [ ] Rollback plan
- [ ] Load testing

---

# 8. **BÀI TẬP THỰC HÀNH**

## 📝 Bài 1: Implement Caching

Thêm caching vào `ProductService.GetAllAsync()`.

<details>
<summary>💡 Đáp án</summary>

```csharp
public async Task<List<Product>> GetAllAsync()
{
    var cacheKey = "all_products";
    
    if (!_cache.TryGetValue(cacheKey, out List<Product> products))
    {
        products = await _db.Products
            .Include(p => p.Category)
            .ToListAsync();
        
        _cache.Set(cacheKey, products, TimeSpan.FromMinutes(10));
        _logger.LogInformation("Products cached");
    }
    else
    {
        _logger.LogInformation("Products from cache");
    }
    
    return products;
}
```
</details>

---

## 📝 Bài 2: Custom Exception

Tạo `InsufficientStockException` và throw khi stock không đủ.

<details>
<summary>💡 Đáp án</summary>

```csharp
public class InsufficientStockException : Exception
{
    public InsufficientStockException(string message) : base(message)
    {
    }
}

// Usage
public async Task CreateOrderAsync(CreateOrderRequest request)
{
    foreach (var item in request.Items)
    {
        var product = await _db.Products.FindAsync(item.ProductId);
        if (product.Stock < item.Quantity)
        {
            throw new InsufficientStockException(
                $"Product {product.Name} has only {product.Stock} in stock");
        }
    }
}
```
</details>

---

## ❌ 4. CÁC LỖI THƯỜNG GẶP

### ❌ Lỗi 1: Cache không được invalidate

```csharp
// ❌ Vấn đề: Cache cũ sau khi update
public async Task UpdateProductAsync(int id, Product product)
{
    _context.Products.Update(product);
    await _context.SaveChangesAsync();
    // Quên xóa cache → Cache vẫn có dữ liệu cũ
}

// ✅ Giải pháp: Invalidate cache sau khi update
public async Task UpdateProductAsync(int id, Product product)
{
    _context.Products.Update(product);
    await _context.SaveChangesAsync();
    
    _cache.Remove($"product_{id}"); // ✅ Xóa cache
    _cache.Remove("products_list"); // ✅ Xóa list cache
}
```

**🔍 Giải thích:** Cache phải được invalidate khi dữ liệu thay đổi, nếu không sẽ trả về dữ liệu cũ.

---

### ❌ Lỗi 2: Log quá nhiều thông tin nhạy cảm

```csharp
// ❌ Vấn đề: Log password, credit card
_logger.LogInformation("User login: {Email}, Password: {Password}", 
    email, password); // ❌ Nguy hiểm!

// ✅ Giải pháp: Không log thông tin nhạy cảm
_logger.LogInformation("User login attempt: {Email}", email); // ✅
```

**🔍 Giải thích:** Không bao giờ log password, credit card, tokens. Chỉ log thông tin cần thiết.

---

### ❌ Lỗi 3: Exception không được handle

```csharp
// ❌ Vấn đề: Exception leak ra client
[HttpGet("{id}")]
public async Task<ActionResult<Product>> GetProduct(int id)
{
    var product = await _service.GetProductAsync(id); // Có thể throw
    return Ok(product);
}

// ✅ Giải pháp: Global exception handler
public class GlobalExceptionMiddleware
{
    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unhandled exception");
            await HandleExceptionAsync(context, ex);
        }
    }
}
```

**🔍 Giải thích:** Exception phải được handle để không leak thông tin nhạy cảm ra client.

---

## 🎯 5. CASE STUDY / VÍ DỤ THỰC TẾ

### Case Study 1: Production-Ready Application

**Yêu cầu:** Cấu hình ứng dụng sẵn sàng cho production với caching, logging, error handling.

```csharp
// Program.cs - Production configuration
var builder = WebApplication.CreateBuilder(args);

// 1. Caching
builder.Services.AddMemoryCache();
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
});

// 2. Logging với Serilog
builder.Host.UseSerilog((context, config) =>
{
    config.ReadFrom.Configuration(context.Configuration)
          .Enrich.FromLogContext()
          .WriteTo.Console()
          .WriteTo.File("logs/app-.txt", rollingInterval: RollingInterval.Day)
          .WriteTo.Seq("http://localhost:5341");
});

// 3. Health Checks
builder.Services.AddHealthChecks()
    .AddDbContextCheck<AppDbContext>()
    .AddSqlServer(builder.Configuration.GetConnectionString("DefaultConnection"))
    .AddRedis(builder.Configuration.GetConnectionString("Redis"));

// 4. Global Exception Handler
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();

var app = builder.Build();

// Middleware
app.UseExceptionHandler();
app.UseSerilogRequestLogging();
app.UseHealthChecks("/health");

app.Run();
```

---

## ✅ 6. BEST PRACTICES

### 6.1. Caching Best Practices

```csharp
// ✅ Đúng: Cache key rõ ràng
var cacheKey = $"product_{id}";
var cacheKey = $"products_page_{page}_size_{pageSize}";

// ✅ Đúng: Set expiration
_cache.Set(key, data, TimeSpan.FromMinutes(10));

// ✅ Đúng: Invalidate khi update
_cache.Remove(key);
```

### 6.2. Logging Best Practices

```csharp
// ✅ Đúng: Log levels phù hợp
_logger.LogTrace("Detailed trace");
_logger.LogDebug("Debug info");
_logger.LogInformation("General info");
_logger.LogWarning("Warning");
_logger.LogError(ex, "Error occurred");
_logger.LogCritical(ex, "Critical error");

// ✅ Đúng: Structured logging
_logger.LogInformation("User {UserId} created order {OrderId}", 
    userId, orderId);
```

### 6.3. Error Handling Best Practices

```csharp
// ✅ Đúng: Global exception handler
app.UseExceptionHandler();

// ✅ Đúng: Custom error responses
catch (NotFoundException ex)
{
    return NotFound(new { message = ex.Message });
}
catch (ValidationException ex)
{
    return BadRequest(new { message = ex.Message, errors = ex.Errors });
}
```

---

# 📝 7. QUICK NOTES

### Caching:
- **In-Memory**: `AddMemoryCache()` - Nhanh, mất khi restart
- **Redis**: Distributed cache - Shared giữa nhiều instances
- **Cache Keys**: Rõ ràng, có namespace
- **Expiration**: Set thời gian hợp lý

### Logging:
- **Serilog**: Structured logging
- **Log Levels**: Trace, Debug, Info, Warning, Error, Critical
- **Sinks**: Console, File, Database, Seq, etc.

### Error Handling:
- **Global Exception Handler**: Xử lý tập trung
- **Problem Details**: Standard error format
- **Logging**: Log mọi exception

### Health Checks:
- **Basic**: `/health` endpoint
- **Detailed**: Check DB, Redis, external services
- **UI**: `/health/ready`, `/health/live`

### Best Practices:
- ✅ Cache với expiration
- ✅ Invalidate cache khi update
- ✅ Structured logging
- ✅ Global exception handler
- ✅ Health checks
- ✅ Không log thông tin nhạy cảm

---

# 🧪 8. MINI TEST

1. **Caching có tác dụng gì?**
   - A. Tăng bảo mật
   - B. Giảm số lần query DB
   - C. Tự động backup data

2. **Serilog ghi log vào đâu?**
   - A. Chỉ Console
   - B. Chỉ File
   - C. Console, File, Database (configurable)

3. **Health Checks dùng để làm gì?**
   - A. Kiểm tra code lỗi
   - B. Kiểm tra app có đang chạy
   - C. Test performance

<details>
<summary>💡 Đáp án</summary>

1. **B** - Giảm số lần query DB → Tăng performance
2. **C** - Configurable, có thể ghi nhiều nơi cùng lúc
3. **B** - Kiểm tra app và dependencies (DB, services) có sẵn sàng
</details>

---

# 📌 TÓM TẮT CHƯƠNG

✅ **Caching** giảm DB queries, tăng performance  
✅ **Serilog** logging chuyên nghiệp với nhiều sinks  
✅ **Global Exception Handler** xử lý lỗi tập trung  
✅ **Health Checks** monitor app status  
✅ **Deployment** IIS, Azure, Docker  
✅ **Best Practices** security, performance, logging  

---

**Chúc mừng bạn đã hoàn thành toàn bộ khóa học Backend Development! 🎉**

**Tiếp theo: [Mega Project 01 - E-Shop API →](../mega_projects/01_eshop_console_api.md)**
