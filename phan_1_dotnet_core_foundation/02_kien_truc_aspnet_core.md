# 🟦 CHƯƠNG 02
# **KIẾN TRÚC ASP.NET CORE**

ASP.NET Core là framework hiện đại để xây dựng ứng dụng web và API. Chương này giúp bạn hiểu kiến trúc cốt lõi của ASP.NET Core — nền tảng cho mọi ứng dụng Backend.

---

# 🎯 MỤC TIÊU HỌC TẬP

Sau chương này, bạn sẽ:

- Hiểu kiến trúc ASP.NET Core: Pipeline, Middleware, DI
- Nắm vững **Dependency Injection (DI)** — trái tim của ASP.NET Core
- Biết cách cấu hình ứng dụng với `appsettings.json`
- Hiểu **Middleware Pipeline** — cách request được xử lý
- Tạo được API đầu tiên với Minimal API và Controller
- Hiểu sự khác biệt giữa Development và Production environment

---

# 1. **TỔNG QUAN ASP.NET CORE**

## 1.1. ASP.NET Core là gì?

**ASP.NET Core** là framework **mã nguồn mở, đa nền tảng** để xây dựng:
- Web API (RESTful services)
- MVC Web Applications
- Blazor (SPA)
- gRPC services

### 🏠 Ví dụ đời sống

ASP.NET Core giống như **khung xương của một tòa nhà**:
- **Foundation** = .NET Runtime
- **Khung xương** = ASP.NET Core (Pipeline, DI, Configuration)
- **Nội thất** = Controllers, Services (code của bạn)

---

## 1.2. Tại sao chọn ASP.NET Core?

✅ **Cross-platform** — Chạy trên Windows, Linux, macOS  
✅ **High performance** — Nhanh nhất trong các framework .NET  
✅ **Dependency Injection built-in** — Không cần thư viện thứ 3  
✅ **Modern** — Cloud-ready, containerization (Docker)  
✅ **Open source** — Miễn phí, cộng đồng lớn  

---

# 2. **CẤU TRÚC DỰ ÁN ASP.NET CORE**

## 2.1. Tạo dự án mới

```powershell
# Tạo Web API
dotnet new webapi -n MyFirstAPI
cd MyFirstAPI
dotnet run
```

Mở trình duyệt: `https://localhost:5001/weatherforecast`

---

## 2.2. Cấu trúc thư mục

```
MyFirstAPI/
├── Controllers/           ← API Controllers
│   └── WeatherForecastController.cs
├── Properties/
│   └── launchSettings.json    ← Development settings
├── appsettings.json          ← Configuration
├── appsettings.Development.json
├── Program.cs                ← Entry point (Quan trọng!)
├── MyFirstAPI.csproj         ← Project file
└── WeatherForecast.cs        ← Model
```

---

## 2.3. Program.cs — Trái tim của ứng dụng

### .NET 6+ (Minimal Hosting Model)

```csharp
// Program.cs - .NET 6+
var builder = WebApplication.CreateBuilder(args);

// ========== CONFIGURATION STAGE ==========
// 1. Add services to the container (DI Container)
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// ========== MIDDLEWARE PIPELINE STAGE ==========
// 2. Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

// 3. Run the application
app.Run();
```

### 🎒 Ví dụ đời sống

`Program.cs` giống như **bản thiết kế xây nhà**:
- **Configuration Stage** = Chuẩn bị vật liệu (xi măng, gạch, sắt)
- **Middleware Pipeline** = Quy trình xây (móng → tường → mái)
- **app.Run()** = Khởi công xây dựng

---

# 3. **DEPENDENCY INJECTION (DI) — QUAN TRỌNG NHẤT**

## 3.1. DI là gì?

**Dependency Injection** là pattern giúp:
- Giảm phụ thuộc giữa các class
- Code dễ test
- Dễ thay đổi implementation

### ❌ Không dùng DI (Bad)

```csharp
public class ProductService
{
    private readonly AppDbContext _db;
    
    public ProductService()
    {
        // ❌ Tạo dependency trực tiếp = Hard coupling
        _db = new AppDbContext();
    }
    
    public List<Product> GetProducts()
    {
        return _db.Products.ToList();
    }
}

// Vấn đề:
// 1. Khó test (không thể mock AppDbContext)
// 2. Khó thay đổi (nếu cần dùng DB khác phải sửa code)
// 3. Tạo nhiều instance không cần thiết
```

### ✅ Dùng DI (Good)

```csharp
public class ProductService
{
    private readonly AppDbContext _db;
    
    // ✅ Nhận dependency từ constructor
    public ProductService(AppDbContext db)
    {
        _db = db;
    }
    
    public async Task<List<Product>> GetProducts()
    {
        return await _db.Products.ToListAsync();
    }
}

// Program.cs - Đăng ký service
builder.Services.AddDbContext<AppDbContext>();
builder.Services.AddScoped<ProductService>();

// Controller - ASP.NET Core tự động inject
public class ProductsController : ControllerBase
{
    private readonly ProductService _productService;
    
    // ASP.NET Core tự động tạo ProductService và inject vào
    public ProductsController(ProductService productService)
    {
        _productService = productService;
    }
}
```

---

## 3.2. Service Lifetime (Vòng đời service)

ASP.NET Core có 3 loại lifetime:

### 🟢 Transient — Tạo mới mỗi lần inject

```csharp
builder.Services.AddTransient<IEmailService, EmailService>();
```

**Khi nào dùng:**
- Services nhẹ, stateless
- Services được sử dụng ít lần
- Ví dụ: EmailService, LoggerService

**Minh họa:**
```
Request 1:
  ├─ Controller A → EmailService #1
  └─ Controller B → EmailService #2  (khác instance)

Request 2:
  └─ Controller A → EmailService #3  (instance mới)
```

---

### 🟡 Scoped — Tạo mới mỗi request

```csharp
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddDbContext<AppDbContext>(); // Mặc định là Scoped
```

**Khi nào dùng:**
- Services làm việc với Database (DbContext)
- Services cần share state trong 1 request
- **QUAN TRỌNG:** DbContext phải là Scoped

**Minh họa:**
```
Request 1:
  ├─ Controller A → ProductService #1 → DbContext #1
  └─ Repository  → ProductService #1 → DbContext #1  (cùng instance)

Request 2:
  └─ Controller A → ProductService #2 → DbContext #2  (instance mới)
```

---

### 🔴 Singleton — Chỉ tạo 1 instance duy nhất

```csharp
builder.Services.AddSingleton<IConfiguration>(configuration);
```

**Khi nào dùng:**
- Services nặng, khởi tạo lâu
- Services không có state hoặc state shared toàn app
- Ví dụ: Configuration, Logger, Cache

**Minh họa:**
```
Request 1:
  ├─ Controller A → CacheService #1

Request 2:
  ├─ Controller A → CacheService #1  (cùng instance)
  └─ Controller B → CacheService #1  (cùng instance)
```

---

## 3.3. Ví dụ thực tế: E-Shop

```csharp
// Program.cs

var builder = WebApplication.CreateBuilder(args);

// ========== ĐĂNG KÝ SERVICES ==========

// 1. DbContext (Scoped)
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// 2. Repository Pattern (Scoped)
builder.Services.AddScoped<IProductRepository, ProductRepository>();
builder.Services.AddScoped<ICategoryRepository, CategoryRepository>();

// 3. Business Services (Scoped)
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddScoped<IOrderService, OrderService>();

// 4. Utility Services (Transient)
builder.Services.AddTransient<IEmailService, EmailService>();
builder.Services.AddTransient<IPdfGenerator, PdfGenerator>();

// 5. Caching (Singleton)
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();

// 6. Configuration (Singleton - built-in)
// IConfiguration đã được đăng ký tự động

builder.Services.AddControllers();

var app = builder.Build();
// ...
```

---

## 3.4. Interface vs Concrete Class

### ✅ Best Practice: Dùng Interface

```csharp
// Interface
public interface IProductService
{
    Task<List<Product>> GetProductsAsync();
    Task<Product?> GetProductByIdAsync(int id);
    Task<Product> CreateProductAsync(Product product);
}

// Implementation
public class ProductService : IProductService
{
    private readonly AppDbContext _db;
    
    public ProductService(AppDbContext db)
    {
        _db = db;
    }
    
    public async Task<List<Product>> GetProductsAsync()
    {
        return await _db.Products.ToListAsync();
    }
    
    // ... other methods
}

// Đăng ký
builder.Services.AddScoped<IProductService, ProductService>();

// Sử dụng
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService; // Interface, not concrete
    
    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }
}
```

**Lợi ích:**
- Dễ test (có thể mock Interface)
- Dễ thay đổi implementation
- Loose coupling

---

# 4. **CONFIGURATION — CẤU HÌNH ỨNG DỤNG**

## 4.1. appsettings.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=EShopDb;Trusted_Connection=True;TrustServerCertificate=True"
  },
  "JwtSettings": {
    "SecretKey": "your-super-secret-key-min-32-chars",
    "Issuer": "EShopAPI",
    "Audience": "EShopClient",
    "ExpiryInMinutes": 60
  },
  "EmailSettings": {
    "SmtpServer": "smtp.gmail.com",
    "SmtpPort": 587,
    "SenderEmail": "noreply@eshop.com",
    "SenderName": "E-Shop"
  }
}
```

---

## 4.2. appsettings.Development.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.AspNetCore": "Information"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=EShopDb_Dev;Trusted_Connection=True"
  }
}
```

**Ghi chú:** File Development sẽ override file gốc khi chạy ở Development environment.

---

## 4.3. Đọc Configuration

### Cách 1: Inject IConfiguration

```csharp
public class EmailService
{
    private readonly IConfiguration _config;
    
    public EmailService(IConfiguration config)
    {
        _config = config;
    }
    
    public void SendEmail(string to, string subject, string body)
    {
        var smtpServer = _config["EmailSettings:SmtpServer"];
        var smtpPort = _config.GetValue<int>("EmailSettings:SmtpPort");
        var senderEmail = _config["EmailSettings:SenderEmail"];
        
        // Send email logic...
    }
}
```

### Cách 2: Options Pattern (Recommended)

```csharp
// 1. Tạo class cấu hình
public class JwtSettings
{
    public string SecretKey { get; set; } = string.Empty;
    public string Issuer { get; set; } = string.Empty;
    public string Audience { get; set; } = string.Empty;
    public int ExpiryInMinutes { get; set; }
}

// 2. Đăng ký trong Program.cs
builder.Services.Configure<JwtSettings>(
    builder.Configuration.GetSection("JwtSettings"));

// 3. Sử dụng
public class TokenService
{
    private readonly JwtSettings _jwtSettings;
    
    public TokenService(IOptions<JwtSettings> jwtSettings)
    {
        _jwtSettings = jwtSettings.Value;
    }
    
    public string GenerateToken(User user)
    {
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_jwtSettings.SecretKey));
        // ... generate token
    }
}
```

---

## 4.4. Connection String

```csharp
// Program.cs
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
```

---

## 4.5. Environment Variables

```csharp
// Đọc từ Environment Variable
var secretKey = Environment.GetEnvironmentVariable("JWT_SECRET_KEY");

// Hoặc từ Configuration (tự động đọc từ env var)
var secretKey = builder.Configuration["JwtSettings:SecretKey"];
```

**Production:** Không lưu secret vào appsettings.json, dùng Environment Variables hoặc Azure Key Vault.

---

# 5. **MIDDLEWARE PIPELINE**

## 5.1. Middleware là gì?

**Middleware** là các component xử lý request/response theo pipeline.

### 🏠 Ví dụ đời sống

Middleware giống như **quy trình kiểm tra an ninh sân bay**:
1. Check-in → Middleware 1
2. Kiểm tra hộ chiếu → Middleware 2
3. Kiểm tra hành lý → Middleware 3
4. Lên máy bay → Endpoint (Controller)

Nếu fail ở bước nào → Dừng pipeline, trả về response ngay.

---

## 5.2. Luồng Request Pipeline

```
Request
  ↓
[Middleware 1] ──→ [Middleware 2] ──→ [Middleware 3] ──→ [Endpoint]
  ↑                    ↑                    ↑                 ↓
  └────────────────────┴────────────────────┴─────────────── Response
```

---

## 5.3. Built-in Middleware

```csharp
var app = builder.Build();

// 1. Exception Handling (phải đặt đầu tiên)
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage(); // Hiển thị lỗi chi tiết
}
else
{
    app.UseExceptionHandler("/Error"); // Trang lỗi custom
    app.UseHsts(); // HTTP Strict Transport Security
}

// 2. HTTPS Redirection
app.UseHttpsRedirection(); // HTTP → HTTPS

// 3. Static Files (nếu có)
app.UseStaticFiles(); // Phục vụ file tĩnh (CSS, JS, images)

// 4. Routing
app.UseRouting(); // Xác định route

// 5. CORS (nếu cần)
app.UseCors("AllowAll");

// 6. Authentication (xác thực - ai bạn là?)
app.UseAuthentication();

// 7. Authorization (phân quyền - bạn được làm gì?)
app.UseAuthorization();

// 8. Endpoints
app.MapControllers(); // Map đến Controllers

app.Run();
```

**THỨ TỰ QUAN TRỌNG!** Sai thứ tự = lỗi khó debug.

---

## 5.4. Custom Middleware

### Ví dụ: Request Logging Middleware

```csharp
// RequestLoggingMiddleware.cs
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;
    
    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        // Before: Log request
        _logger.LogInformation($"Request: {context.Request.Method} {context.Request.Path}");
        
        // Call next middleware
        await _next(context);
        
        // After: Log response
        _logger.LogInformation($"Response: {context.Response.StatusCode}");
    }
}

// Extension method
public static class RequestLoggingMiddlewareExtensions
{
    public static IApplicationBuilder UseRequestLogging(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<RequestLoggingMiddleware>();
    }
}

// Program.cs
app.UseRequestLogging(); // Sử dụng
```

---

### Ví dụ: API Key Authentication Middleware

```csharp
public class ApiKeyMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IConfiguration _config;
    
    public ApiKeyMiddleware(RequestDelegate next, IConfiguration config)
    {
        _next = next;
        _config = config;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        // Lấy API Key từ header
        if (!context.Request.Headers.TryGetValue("X-API-Key", out var apiKey))
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("API Key is missing");
            return; // Dừng pipeline
        }
        
        // Validate API Key
        var validApiKey = _config["ApiKey"];
        if (apiKey != validApiKey)
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("Invalid API Key");
            return;
        }
        
        // API Key hợp lệ → Tiếp tục pipeline
        await _next(context);
    }
}
```

---

# 6. **MINIMAL API vs CONTROLLER-BASED API**

## 6.1. Minimal API (.NET 6+)

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Define endpoints directly
app.MapGet("/", () => "Hello World!");

app.MapGet("/products", async (AppDbContext db) =>
{
    return await db.Products.ToListAsync();
});

app.MapGet("/products/{id}", async (int id, AppDbContext db) =>
{
    var product = await db.Products.FindAsync(id);
    return product is not null ? Results.Ok(product) : Results.NotFound();
});

app.MapPost("/products", async (Product product, AppDbContext db) =>
{
    db.Products.Add(product);
    await db.SaveChangesAsync();
    return Results.Created($"/products/{product.Id}", product);
});

app.Run();
```

**Ưu điểm:**
- Ngắn gọn
- Phù hợp API đơn giản

**Nhược điểm:**
- Khó tổ chức khi API lớn
- Thiếu cấu trúc

---

## 6.2. Controller-based API (Recommended)

```csharp
// Controllers/ProductsController.cs
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly AppDbContext _db;
    
    public ProductsController(AppDbContext db)
    {
        _db = db;
    }
    
    // GET: api/products
    [HttpGet]
    public async Task<ActionResult<List<Product>>> GetProducts()
    {
        var products = await _db.Products.ToListAsync();
        return Ok(products);
    }
    
    // GET: api/products/5
    [HttpGet("{id}")]
    public async Task<ActionResult<Product>> GetProduct(int id)
    {
        var product = await _db.Products.FindAsync(id);
        
        if (product == null)
            return NotFound();
            
        return Ok(product);
    }
    
    // POST: api/products
    [HttpPost]
    public async Task<ActionResult<Product>> CreateProduct(Product product)
    {
        _db.Products.Add(product);
        await _db.SaveChangesAsync();
        
        return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
    }
}
```

**Ưu điểm:**
- Tổ chức tốt
- Dễ maintain
- Hỗ trợ nhiều features (Filter, Authorization, Model Binding, ...)

---

# 7. **ENVIRONMENT — DEVELOPMENT vs PRODUCTION**

## 7.1. Kiểm tra Environment

```csharp
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    // Development only
    app.UseSwagger();
    app.UseSwaggerUI();
    app.UseDeveloperExceptionPage();
}
else
{
    // Production only
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}
```

---

## 7.2. Cấu hình theo Environment

```
appsettings.json               ← Base config
appsettings.Development.json   ← Dev overrides
appsettings.Production.json    ← Prod overrides
```

ASP.NET Core tự động merge theo thứ tự:
1. appsettings.json
2. appsettings.{Environment}.json

---

## 7.3. Đặt Environment

### Development (mặc định)

```powershell
# PowerShell
$env:ASPNETCORE_ENVIRONMENT = "Development"
dotnet run
```

### Production

```powershell
$env:ASPNETCORE_ENVIRONMENT = "Production"
dotnet run
```

---

# 8. **BÀI TẬP THỰC HÀNH**

## 📝 Bài 1: Tạo API đầu tiên

Tạo API quản lý Categories với các endpoint:
- GET /api/categories
- GET /api/categories/{id}
- POST /api/categories

```csharp
public class Category
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
}
```

<details>
<summary>💡 Đáp án</summary>

```csharp
[ApiController]
[Route("api/[controller]")]
public class CategoriesController : ControllerBase
{
    private static List<Category> _categories = new()
    {
        new Category { Id = 1, Name = "Electronics", Description = "Electronic devices" },
        new Category { Id = 2, Name = "Books", Description = "Books and magazines" }
    };
    
    [HttpGet]
    public ActionResult<List<Category>> GetCategories()
    {
        return Ok(_categories);
    }
    
    [HttpGet("{id}")]
    public ActionResult<Category> GetCategory(int id)
    {
        var category = _categories.FirstOrDefault(c => c.Id == id);
        if (category == null)
            return NotFound();
        return Ok(category);
    }
    
    [HttpPost]
    public ActionResult<Category> CreateCategory(Category category)
    {
        category.Id = _categories.Max(c => c.Id) + 1;
        _categories.Add(category);
        return CreatedAtAction(nameof(GetCategory), new { id = category.Id }, category);
    }
}
```
</details>

---

## 📝 Bài 2: Dependency Injection

Tạo `ICategoryService` interface và implementation, sử dụng DI.

<details>
<summary>💡 Đáp án</summary>

```csharp
// Interface
public interface ICategoryService
{
    List<Category> GetAll();
    Category? GetById(int id);
    Category Create(Category category);
}

// Implementation
public class CategoryService : ICategoryService
{
    private static List<Category> _categories = new()
    {
        new Category { Id = 1, Name = "Electronics", Description = "Electronic devices" }
    };
    
    public List<Category> GetAll() => _categories;
    
    public Category? GetById(int id) => _categories.FirstOrDefault(c => c.Id == id);
    
    public Category Create(Category category)
    {
        category.Id = _categories.Any() ? _categories.Max(c => c.Id) + 1 : 1;
        _categories.Add(category);
        return category;
    }
}

// Program.cs
builder.Services.AddScoped<ICategoryService, CategoryService>();

// Controller
[ApiController]
[Route("api/[controller]")]
public class CategoriesController : ControllerBase
{
    private readonly ICategoryService _categoryService;
    
    public CategoriesController(ICategoryService categoryService)
    {
        _categoryService = categoryService;
    }
    
    [HttpGet]
    public ActionResult<List<Category>> GetCategories()
    {
        return Ok(_categoryService.GetAll());
    }
}
```
</details>

---

## 📝 Bài 3: Custom Middleware

Tạo middleware log thời gian xử lý request.

<details>
<summary>💡 Đáp án</summary>

```csharp
public class PerformanceMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<PerformanceMiddleware> _logger;
    
    public PerformanceMiddleware(RequestDelegate next, ILogger<PerformanceMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var stopwatch = Stopwatch.StartNew();
        
        await _next(context);
        
        stopwatch.Stop();
        var elapsed = stopwatch.ElapsedMilliseconds;
        
        if (elapsed > 1000)
        {
            _logger.LogWarning($"Slow request: {context.Request.Path} took {elapsed}ms");
        }
        else
        {
            _logger.LogInformation($"Request: {context.Request.Path} took {elapsed}ms");
        }
    }
}

// Program.cs
app.UseMiddleware<PerformanceMiddleware>();
```
</details>

---

## ❌ 6. CÁC LỖI THƯỜNG GẶP

### ❌ Lỗi 1: Đăng ký service với lifetime sai

```csharp
// ❌ Vấn đề: DbContext dùng Singleton (sai!)
builder.Services.AddSingleton<AppDbContext>(); // ❌ DbContext không thread-safe

// ✅ Giải pháp: Dùng Scoped cho DbContext
builder.Services.AddDbContext<AppDbContext>(options => 
    options.UseSqlServer(connectionString)); // ✅ Mặc định là Scoped
```

**🔍 Giải thích:** DbContext không thread-safe, phải dùng Scoped (mỗi request 1 instance). Singleton sẽ gây lỗi khi nhiều requests đồng thời.

---

### ❌ Lỗi 2: Circular Dependency

```csharp
// ❌ Vấn đề: Service A phụ thuộc B, B phụ thuộc A
public class ServiceA
{
    public ServiceA(ServiceB b) { }
}

public class ServiceB
{
    public ServiceB(ServiceA a) { } // Circular dependency!
}

// ✅ Giải pháp: Tách interface hoặc refactor
public interface IServiceA { }
public class ServiceA : IServiceA
{
    public ServiceA(IServiceB b) { }
}
```

**🔍 Giải thích:** Circular dependency gây lỗi khi DI container tạo service. Tách interface hoặc refactor để phá vỡ vòng lặp.

---

### ❌ Lỗi 3: Middleware sai thứ tự

```csharp
// ❌ Vấn đề: UseAuthorization trước UseAuthentication
app.UseAuthorization(); // ❌ Phải có authentication trước
app.UseAuthentication();

// ✅ Giải pháp: Đúng thứ tự
app.UseAuthentication(); // ✅ Trước
app.UseAuthorization(); // ✅ Sau
```

**🔍 Giải thích:** Phải authenticate trước khi authorize. Thứ tự middleware rất quan trọng.

---

### ❌ Lỗi 4: Không dispose DbContext

```csharp
// ❌ Vấn đề: Tạo DbContext thủ công, không dispose
public void ProcessData()
{
    var db = new AppDbContext(); // ❌ Không dispose
    var products = db.Products.ToList();
}

// ✅ Giải pháp: Dùng DI, tự động dispose
public class ProductService
{
    private readonly AppDbContext _db;
    public ProductService(AppDbContext db) { _db = db; } // ✅ Tự động dispose
}
```

**🔍 Giải thích:** DbContext cần dispose để giải phóng connection. DI container tự động dispose khi request kết thúc.

---

### ❌ Lỗi 5: Configuration không được load

```csharp
// ❌ Vấn đề: Đọc config trước khi builder.Build()
var builder = WebApplication.CreateBuilder(args);
var connectionString = builder.Configuration["ConnectionStrings:Default"]; // ✅ OK
var app = builder.Build();
var config = app.Configuration["ConnectionStrings:Default"]; // ✅ Cũng OK

// ❌ Nhưng nếu thay đổi config sau Build(), không có hiệu lực
```

**🔍 Giải thích:** Configuration được load khi `CreateBuilder()`, sau đó không thay đổi. Dùng IConfiguration để đọc.

---

### ❌ Lỗi 6: Options Pattern không validate

```csharp
// ❌ Vấn đề: Không validate config
builder.Services.Configure<JwtSettings>(config.GetSection("JwtSettings"));

// ✅ Giải pháp: Validate khi startup
builder.Services.AddOptions<JwtSettings>()
    .Bind(config.GetSection("JwtSettings"))
    .ValidateDataAnnotations()
    .ValidateOnStart(); // ✅ Validate ngay khi start
```

**🔍 Giải thích:** Validate config khi startup để phát hiện lỗi sớm, không đợi đến khi dùng.

---

## 🎯 7. CASE STUDY / VÍ DỤ THỰC TẾ

### Case Study 1: Multi-layer Architecture với DI

**Yêu cầu:** Xây dựng API với Repository Pattern và Service Layer.

```csharp
// 1. Repository Interface
public interface IProductRepository
{
    Task<List<Product>> GetAllAsync();
    Task<Product?> GetByIdAsync(int id);
    Task<Product> CreateAsync(Product product);
}

// 2. Repository Implementation
public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _db;
    
    public ProductRepository(AppDbContext db)
    {
        _db = db;
    }
    
    public async Task<List<Product>> GetAllAsync()
    {
        return await _db.Products.ToListAsync();
    }
    
    public async Task<Product?> GetByIdAsync(int id)
    {
        return await _db.Products.FindAsync(id);
    }
    
    public async Task<Product> CreateAsync(Product product)
    {
        _db.Products.Add(product);
        await _db.SaveChangesAsync();
        return product;
    }
}

// 3. Service Layer
public interface IProductService
{
    Task<List<ProductDto>> GetProductsAsync();
    Task<ProductDto?> GetProductAsync(int id);
    Task<ProductDto> CreateProductAsync(CreateProductRequest request);
}

public class ProductService : IProductService
{
    private readonly IProductRepository _repository;
    private readonly IMapper _mapper; // AutoMapper
    
    public ProductService(IProductRepository repository, IMapper mapper)
    {
        _repository = repository;
        _mapper = mapper;
    }
    
    public async Task<List<ProductDto>> GetProductsAsync()
    {
        var products = await _repository.GetAllAsync();
        return _mapper.Map<List<ProductDto>>(products);
    }
    
    public async Task<ProductDto?> GetProductAsync(int id)
    {
        var product = await _repository.GetByIdAsync(id);
        return product == null ? null : _mapper.Map<ProductDto>(product);
    }
    
    public async Task<ProductDto> CreateProductAsync(CreateProductRequest request)
    {
        var product = _mapper.Map<Product>(request);
        var created = await _repository.CreateAsync(product);
        return _mapper.Map<ProductDto>(created);
    }
}

// 4. Controller
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
    public async Task<ActionResult<List<ProductDto>>> GetProducts()
    {
        var products = await _service.GetProductsAsync();
        return Ok(products);
    }
}

// 5. Program.cs - Đăng ký services
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));

builder.Services.AddScoped<IProductRepository, ProductRepository>();
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddAutoMapper(typeof(Program));
```

**Best practices:**
- Tách Repository và Service layer
- Dùng Interface cho loose coupling
- DI tự động inject dependencies
- Scoped lifetime cho DbContext và services

---

### Case Study 2: Custom Middleware cho Request Logging và Error Handling

**Yêu cầu:** Tạo middleware log requests và handle errors globally.

```csharp
// 1. Request Logging Middleware
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;
    
    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var stopwatch = Stopwatch.StartNew();
        var requestPath = context.Request.Path;
        var requestMethod = context.Request.Method;
        
        _logger.LogInformation($"Request: {requestMethod} {requestPath}");
        
        try
        {
            await _next(context);
        }
        finally
        {
            stopwatch.Stop();
            var statusCode = context.Response.StatusCode;
            _logger.LogInformation(
                $"Response: {requestMethod} {requestPath} - {statusCode} - {stopwatch.ElapsedMilliseconds}ms");
        }
    }
}

// 2. Global Exception Handler Middleware
public class GlobalExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<GlobalExceptionMiddleware> _logger;
    
    public GlobalExceptionMiddleware(RequestDelegate next, ILogger<GlobalExceptionMiddleware> logger)
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
            _logger.LogError(ex, "Unhandled exception occurred");
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
            _ => StatusCodes.Status500InternalServerError
        };
        
        var response = new ErrorResponse
        {
            StatusCode = context.Response.StatusCode,
            Message = exception.Message,
            Details = context.RequestServices.GetService<IWebHostEnvironment>()?.IsDevelopment() == true
                ? exception.ToString()
                : null
        };
        
        return context.Response.WriteAsync(JsonSerializer.Serialize(response));
    }
}

// 3. Program.cs
var app = builder.Build();

// Exception handling phải đặt đầu tiên
app.UseMiddleware<GlobalExceptionMiddleware>();
app.UseMiddleware<RequestLoggingMiddleware>();

app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Giải thích:**
- Middleware xử lý request/response
- Global exception handler bắt mọi exception
- Logging middleware track performance
- Thứ tự middleware rất quan trọng

---

## ✅ 8. BEST PRACTICES

### 8.1. DI Best Practices

```csharp
// ✅ Đúng: Dùng Interface
public interface IEmailService { }
public class EmailService : IEmailService { }
builder.Services.AddScoped<IEmailService, EmailService>();

// ✅ Đúng: Lifetime phù hợp
builder.Services.AddSingleton<IConfiguration>(); // Config không đổi
builder.Services.AddScoped<AppDbContext>(); // Mỗi request 1 instance
builder.Services.AddTransient<IValidator>(); // Mỗi lần inject tạo mới

// ❌ Sai: DbContext dùng Singleton
builder.Services.AddSingleton<AppDbContext>(); // ❌ Không thread-safe
```

### 8.2. Configuration Best Practices

```csharp
// ✅ Đúng: Options Pattern
builder.Services.Configure<JwtSettings>(config.GetSection("JwtSettings"));
builder.Services.AddOptions<JwtSettings>()
    .Bind(config.GetSection("JwtSettings"))
    .ValidateDataAnnotations()
    .ValidateOnStart();

// ✅ Đúng: Environment-specific config
if (app.Environment.IsDevelopment())
{
    // Development settings
}
else
{
    // Production settings
}
```

### 8.3. Middleware Best Practices

```csharp
// ✅ Đúng: Thứ tự middleware
app.UseExceptionHandler(); // 1. Exception handling đầu tiên
app.UseHttpsRedirection(); // 2. HTTPS
app.UseStaticFiles(); // 3. Static files
app.UseRouting(); // 4. Routing
app.UseAuthentication(); // 5. Authentication
app.UseAuthorization(); // 6. Authorization
app.MapControllers(); // 7. Endpoints

// ✅ Đúng: Conditional middleware
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
```

### 8.4. Service Registration Best Practices

```csharp
// ✅ Đúng: Extension methods để tổ chức
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddApplicationServices(this IServiceCollection services)
    {
        services.AddScoped<IProductService, ProductService>();
        services.AddScoped<IOrderService, OrderService>();
        return services;
    }
}

// Program.cs
builder.Services.AddApplicationServices();
```

---

# 🧪 9. MINI TEST

1. **Dependency Injection có mấy loại lifetime?**
   - A. 2 (Transient, Singleton)
   - B. 3 (Transient, Scoped, Singleton)
   - C. 4 (Transient, Scoped, Singleton, Request)

2. **DbContext nên dùng lifetime nào?**
   - A. Transient
   - B. Scoped
   - C. Singleton

3. **Middleware nào phải đặt đầu tiên?**
   - A. UseAuthorization
   - B. UseExceptionHandler
   - C. UseRouting

4. **Configuration file nào được load sau cùng?**
   - A. appsettings.json
   - B. appsettings.Development.json
   - C. Cả 2 cùng lúc

<details>
<summary>💡 Đáp án</summary>

1. **B** - 3 (Transient, Scoped, Singleton)
2. **B** - Scoped (mỗi request 1 instance)
3. **B** - UseExceptionHandler (phải bắt exception sớm nhất)
4. **B** - appsettings.Development.json (override appsettings.json)
</details>

---

# 📝 10. QUICK NOTES

### Dependency Injection:
- **Transient**: Tạo mới mỗi lần inject
- **Scoped**: Một instance per request (dùng cho DbContext)
- **Singleton**: Một instance cho toàn app (dùng cho services stateless)

### Configuration:
- `appsettings.json`: Config chung
- `appsettings.{Environment}.json`: Config theo environment
- Options Pattern: Type-safe configuration
- Environment Variables: Override config

### Middleware Pipeline:
- Thứ tự quan trọng!
- Exception handling đầu tiên
- Authentication trước Authorization
- Routing trước Endpoints

### Program.cs Structure:
```csharp
var builder = WebApplication.CreateBuilder(args);
// 1. Configuration
builder.Services.AddControllers();
builder.Services.AddDbContext<AppDbContext>();

var app = builder.Build();
// 2. Middleware Pipeline
app.UseExceptionHandler();
app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### Best Practices:
- ✅ Dùng Interface cho DI
- ✅ Scoped cho DbContext
- ✅ Options Pattern cho config
- ✅ Extension methods để tổ chức services
- ✅ Global exception handler

---

# 📌 11. TÓM TẮT CHƯƠNG

✅ ASP.NET Core là framework hiện đại, đa nền tảng  
✅ **Dependency Injection** là trái tim (3 lifetimes: Transient, Scoped, Singleton)  
✅ **Configuration** dùng appsettings.json và Options Pattern  
✅ **Middleware Pipeline** xử lý request theo thứ tự  
✅ Controller-based API tốt hơn Minimal API cho dự án lớn  
✅ Development vs Production có config khác nhau  

---

**Chương tiếp theo: [03. MVC Pattern & Routing →](../phan_2_mvc_database/03_mvc_pattern.md)**
