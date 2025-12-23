# 📘 CHƯƠNG 03
# **ASP.NET CORE CONVENTIONS - QUY TẮC MẶC ĐỊNH**

> **Convention over Configuration** = Quy ước thay vì cấu hình. ASP.NET Core có nhiều quy tắc mặc định giúp bạn code ít hơn, nhanh hơn.

---

## 🎯 MỤC TIÊU HỌC TẬP

Sau chương này, bạn sẽ:

- ✅ Hiểu **Convention over Configuration** là gì
- ✅ Nắm vững **Controller Naming Conventions**
- ✅ Hiểu **Action Naming Conventions**
- ✅ Biết **View Location Conventions**
- ✅ Hiểu **Routing Conventions**
- ✅ Nắm **Model Binding Conventions**
- ✅ Biết **Dependency Injection Conventions**
- ✅ Hiểu **Configuration Conventions**
- ✅ Áp dụng conventions để code nhanh hơn

---

## 📖 1. CONVENTION OVER CONFIGURATION LÀ GÌ?

### 1.1. Khái niệm

**Convention over Configuration (CoC)** = Quy ước thay vì cấu hình

**Ý tưởng:** Thay vì phải cấu hình mọi thứ, framework tự động hiểu dựa trên quy ước đặt tên.

### 🏠 Ví dụ đời sống

**Không có Convention:**
```
Bạn: "Tôi muốn tìm file HomeController"
Hệ thống: "File đó ở đâu? Tên gì? Extension gì?"
Bạn: "Ở Controllers/HomeController.cs"
Hệ thống: "OK, tìm thấy!"
```

**Có Convention:**
```
Bạn: "Tôi muốn HomeController"
Hệ thống: "Tự động tìm trong Controllers/HomeController.cs" ✅
```

### 1.2. Lợi ích

✅ **Code ít hơn:** Không cần cấu hình nhiều  
✅ **Dễ hiểu:** Quy ước rõ ràng, nhất quán  
✅ **Nhanh hơn:** Framework tự động làm việc  
✅ **Ít lỗi:** Tuân theo quy ước = ít lỗi  

### 1.3. Nhược điểm

❌ **Phải nhớ quy ước:** Nếu không biết → lỗi  
❌ **Ít linh hoạt:** Muốn custom phải override  

---

## 🎮 2. CONTROLLER NAMING CONVENTIONS

### 2.1. Quy tắc đặt tên Controller

**Quy tắc:** Tên class phải kết thúc bằng `Controller`

```csharp
// ✅ ĐÚNG - Có hậu tố Controller
public class HomeController : Controller { }
public class ProductsController : Controller { }
public class UserManagementController : Controller { }

// ❌ SAI - Thiếu hậu tố Controller
public class Home : Controller { } // ❌ Routing không tìm thấy!
public class Products : Controller { } // ❌
```

### 2.2. Routing tự động loại bỏ "Controller"

**Convention:** Routing tự động bỏ `Controller` khi tạo URL

```csharp
// Controller: HomeController
// URL tự động: /Home (không phải /HomeController)

public class HomeController : Controller
{
    public IActionResult Index() { return View(); }
}

// URL: /Home/Index
// Routing tìm: HomeController.Index()
```

**Ví dụ cụ thể:**

| Controller Class | URL Segment | Giải thích |
|-----------------|-------------|------------|
| `HomeController` | `/Home` | Bỏ "Controller" |
| `ProductsController` | `/Products` | Bỏ "Controller" |
| `UserManagementController` | `/UserManagement` | Bỏ "Controller" |
| `AdminDashboardController` | `/AdminDashboard` | Bỏ "Controller" |

### 2.3. Ví dụ minh họa

```csharp
// Controllers/ProductsController.cs
public class ProductsController : Controller
{
    public IActionResult Index()
    {
        return View();
    }
}

// Routing tự động:
// URL: /Products/Index
// → Tìm: ProductsController.Index()
// → View: Views/Products/Index.cshtml
```

**Test:**

```csharp
// Program.cs
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// User truy cập: /Products
// Routing parse:
//   - controller = "Products"
//   - action = "Index" (mặc định)
// → Tìm: ProductsController.Index() ✅
```

### 2.4. Exception: Controller không kế thừa Controller

**Quy tắc:** Nếu class không kế thừa `Controller`, phải có attribute `[Controller]`

```csharp
// ✅ ĐÚNG - Kế thừa Controller
public class HomeController : Controller { }

// ✅ ĐÚNG - Có [Controller] attribute
[Controller]
public class Home : ControllerBase { }

// ❌ SAI - Không có cả hai
public class Home : ControllerBase { } // ❌ Routing không tìm thấy!
```

---

## 🎯 3. ACTION NAMING CONVENTIONS

### 3.1. Quy tắc đặt tên Action

**Quy tắc:** Action = public method trả về `IActionResult` (hoặc `Task<IActionResult>`)

```csharp
public class HomeController : Controller
{
    // ✅ ĐÚNG - Public method, trả về IActionResult
    public IActionResult Index()
    {
        return View();
    }

    // ✅ ĐÚNG - Async action
    public async Task<IActionResult> GetDataAsync()
    {
        return View();
    }

    // ❌ SAI - Private method (không phải action)
    private IActionResult Helper()
    {
        return View(); // ❌ Routing không tìm thấy!
    }

    // ❌ SAI - Không trả về IActionResult
    public void DoSomething() { } // ❌ Không phải action
}
```

### 3.2. Action Method Signature

**Các kiểu trả về hợp lệ:**

```csharp
public class ProductsController : Controller
{
    // ✅ IActionResult (linh hoạt nhất)
    public IActionResult Index()
    {
        return View();
        // Hoặc: return Json(...), return Redirect(...), etc.
    }

    // ✅ ActionResult<T> (type-safe)
    public ActionResult<Product> GetProduct(int id)
    {
        var product = GetProductById(id);
        return Ok(product); // Trả về Product
    }

    // ✅ Task<IActionResult> (async)
    public async Task<IActionResult> GetProductsAsync()
    {
        var products = await _db.Products.ToListAsync();
        return View(products);
    }

    // ✅ Specific types (ViewResult, JsonResult, etc.)
    public ViewResult Index()
    {
        return View();
    }
}
```

### 3.3. Action không được gọi (Non-Actions)

**Quy tắc:** Method có `[NonAction]` attribute → không phải action

```csharp
public class ProductsController : Controller
{
    // ✅ Action - Routing tìm thấy
    public IActionResult Index()
    {
        return View();
    }

    // ❌ Non-Action - Routing KHÔNG tìm thấy
    [NonAction]
    public IActionResult Helper()
    {
        return View(); // Không thể truy cập qua URL
    }

    // ✅ Helper method (private) - Không phải action
    private void LogInfo(string message)
    {
        _logger.LogInformation(message);
    }
}
```

**Khi nào dùng `[NonAction]`?**

```csharp
public class ProductsController : Controller
{
    // Method này public nhưng không muốn là action
    [NonAction]
    public IActionResult ValidateProduct(Product product)
    {
        // Logic validation
        if (string.IsNullOrEmpty(product.Name))
            return BadRequest("Name is required");
        
        return Ok();
    }

    // Action sử dụng helper method
    [HttpPost]
    public IActionResult Create(Product product)
    {
        var validationResult = ValidateProduct(product); // Gọi helper
        if (validationResult is BadRequestObjectResult)
            return validationResult;
        
        // Create product...
        return RedirectToAction(nameof(Index));
    }
}
```

### 3.4. Action Naming trong Routing

**Convention:** Action name = Method name

```csharp
public class ProductsController : Controller
{
    // Method: Index
    // URL: /Products/Index hoặc /Products
    public IActionResult Index() { }

    // Method: Details
    // URL: /Products/Details
    public IActionResult Details(int id) { }

    // Method: Create
    // URL: /Products/Create
    public IActionResult Create() { }
}
```

**Ví dụ minh họa:**

| Method Name | URL (Conventional) | Giải thích |
|-------------|---------------------|------------|
| `Index()` | `/Products` hoặc `/Products/Index` | Index là mặc định |
| `Details(int id)` | `/Products/Details/1` | Có parameter |
| `Create()` | `/Products/Create` | Action name = method name |
| `Edit(int id)` | `/Products/Edit/1` | Action name = method name |

---

## 📁 4. VIEW LOCATION CONVENTIONS

### 4.1. Quy tắc tìm View

**Convention:** View Engine tìm View theo pattern:

```
Views/{Controller}/{Action}.cshtml
```

**Ví dụ:**

```csharp
// Controller: HomeController
// Action: Index()
// → Tìm: Views/Home/Index.cshtml

public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View(); // Tự động tìm Views/Home/Index.cshtml
    }
}
```

### 4.2. View Location Paths (Thứ tự tìm)

ASP.NET Core tìm View theo thứ tự:

```
1. Views/{Controller}/{Action}.cshtml
2. Views/Shared/{Action}.cshtml
3. (Nếu không tìm thấy → Lỗi)
```

**Ví dụ minh họa:**

```csharp
// Controller: ProductsController
// Action: Index()

// Thứ tự tìm:
// 1. Views/Products/Index.cshtml ✅ (ưu tiên)
// 2. Views/Shared/Index.cshtml (nếu không có ở bước 1)
```

### 4.3. Chỉ định View cụ thể

**Convention:** Có thể chỉ định View name khác với Action name

```csharp
public class ProductsController : Controller
{
    // Mặc định: Tìm Views/Products/Index.cshtml
    public IActionResult Index()
    {
        return View();
    }

    // Chỉ định View name: Tìm Views/Products/List.cshtml
    public IActionResult Index()
    {
        return View("List"); // ← Chỉ định tên View
    }

    // Chỉ định View trong Shared: Tìm Views/Shared/CustomView.cshtml
    public IActionResult Index()
    {
        return View("~/Views/Shared/CustomView.cshtml"); // ← Đường dẫn đầy đủ
    }

    // Chỉ định View với Model
    public IActionResult Index()
    {
        var products = GetProducts();
        return View("List", products); // View name + Model
    }
}
```

### 4.4. View Location Formats

**Các format View location:**

```csharp
// 1. Relative path (trong cùng Controller folder)
return View("Details"); // Views/Products/Details.cshtml

// 2. Relative path (trong Shared folder)
return View("~/Views/Shared/Error.cshtml");

// 3. Absolute path
return View("/Views/Home/About.cshtml");

// 4. View name với extension
return View("Index.cshtml"); // Không cần .cshtml, nhưng có thể dùng
```

### 4.5. _ViewStart.cshtml Convention

**Convention:** `_ViewStart.cshtml` chạy trước mỗi View

```
Views/
├── _ViewStart.cshtml    ← Chạy trước mỗi View
├── Home/
│   └── Index.cshtml    ← Chạy sau _ViewStart
└── Shared/
    └── _Layout.cshtml
```

**Nội dung _ViewStart.cshtml:**

```csharp
@{
    Layout = "_Layout"; // Set Layout mặc định
}
```

**Luồng render:**

```
1. Controller.Index() gọi View()
   ↓
2. Chạy _ViewStart.cshtml (set Layout)
   ↓
3. Render Views/Home/Index.cshtml
   ↓
4. Wrap vào _Layout.cshtml (@RenderBody())
   ↓
5. HTML cuối cùng
```

### 4.6. _ViewImports.cshtml Convention

**Convention:** `_ViewImports.cshtml` import namespaces cho tất cả Views

**Vị trí:**

```
Views/
├── _ViewImports.cshtml  ← Import namespaces
├── Home/
│   └── Index.cshtml     ← Tự động có các namespaces
```

**Nội dung _ViewImports.cshtml:**

```csharp
@using MyFirstRazorApp.Models
@using MyFirstRazorApp.ViewModels
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

**Lợi ích:**

```csharp
// Không có _ViewImports.cshtml
@model MyFirstRazorApp.Models.Product // Phải viết đầy đủ namespace

// Có _ViewImports.cshtml
@model Product // Chỉ cần tên class
```

---

## 🗺️ 5. ROUTING CONVENTIONS

### 5.1. Conventional Routing Pattern

**Convention mặc định trong Program.cs:**

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Giải thích pattern:**

```
{controller=Home}/{action=Index}/{id?}
│              │  │           │  │   │
│              │  │           │  │   └─ Optional (có thể có hoặc không)
│              │  │           │  └───── Parameter name
│              │  │           └──────── Default value
│              │  └─────────────────── Action segment
│              └────────────────────── Default value
└────────────────────────────────────── Controller segment
```

### 5.2. URL Matching Rules

**Quy tắc khớp URL:**

| URL | Controller | Action | Id | Giải thích |
|-----|-----------|--------|-----|------------|
| `/` | Home | Index | null | Mặc định |
| `/Home` | Home | Index | null | Controller mặc định, action mặc định |
| `/Home/Index` | Home | Index | null | Đầy đủ |
| `/Products` | Products | Index | null | Action mặc định = Index |
| `/Products/Details` | Products | Details | null | Không có id |
| `/Products/Details/1` | Products | Details | 1 | Có id |
| `/Products/Details/1/extra` | ❌ | ❌ | ❌ | Không khớp (quá nhiều segments) |

### 5.3. Route Constraints Convention

**Convention:** Constraints validate parameters

```csharp
// Program.cs
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id:int?}"); // ← id phải là int
```

**Các constraints phổ biến:**

```csharp
{id:int}           // Phải là số nguyên
{id:guid}          // Phải là GUID
{id:min(1)}        // Tối thiểu là 1
{id:max(100)}      // Tối đa là 100
{id:range(1,100)}  // Từ 1 đến 100
{name:alpha}       // Chỉ chữ cái
{name:required}    // Bắt buộc
{name:length(5)}   // Độ dài = 5
{name:length(5,10)} // Độ dài từ 5 đến 10
```

**Ví dụ:**

```csharp
// URL hợp lệ: /Products/Details/1
// URL không hợp lệ: /Products/Details/abc → 404

public IActionResult Details(int id) // id phải là int
{
    // ...
}
```

### 5.4. Attribute Routing Conventions

**Convention:** Attribute routing override conventional routing

```csharp
[Route("san-pham")] // Route prefix
public class ProductsController : Controller
{
    // URL: /san-pham
    [Route("")]
    [Route("danh-sach")]
    public IActionResult Index() { }

    // URL: /san-pham/chi-tiet/1
    [Route("chi-tiet/{id:int}")]
    public IActionResult Details(int id) { }
}
```

**Quy tắc:**

1. **Route prefix:** `[Route("prefix")]` trên Controller
2. **Route trên Action:** Kết hợp với prefix
3. **Override:** Attribute routing có ưu tiên cao hơn conventional

---

## 🔗 6. MODEL BINDING CONVENTIONS

### 6.1. Model Binding Sources (Thứ tự tìm)

**Convention:** ASP.NET Core tìm dữ liệu theo thứ tự:

```
1. Form values (POST body)
2. Route values (/Products/Details/1)
3. Query string (?id=1&name=test)
4. Request headers (ít dùng)
```

**Ví dụ:**

```csharp
// URL: /Products/Details/1?name=Laptop
public IActionResult Details(int id, string name)
{
    // id = 1 (từ route)
    // name = "Laptop" (từ query string)
}
```

### 6.2. Parameter Binding Conventions

**Convention:** Tên parameter phải khớp với key trong request

```csharp
// URL: /Products/Details/1
// Route: {controller}/{action}/{id}
public IActionResult Details(int id) // ← id khớp với route parameter
{
    // id = 1
}

// Form POST: name="Laptop"&price=1000
[HttpPost]
public IActionResult Create(string name, decimal price) // ← Tên khớp với form field
{
    // name = "Laptop"
    // price = 1000
}
```

### 6.3. Complex Model Binding

**Convention:** Bind object từ form/JSON

```csharp
// Form POST:
// name=Laptop&price=1000&stock=10

[HttpPost]
public IActionResult Create(Product product) // ← Tự động bind
{
    // product.Name = "Laptop"
    // product.Price = 1000
    // product.Stock = 10
}
```

**Model:**

```csharp
public class Product
{
    public string Name { get; set; }    // Bind từ "name"
    public decimal Price { get; set; }   // Bind từ "price"
    public int Stock { get; set; }       // Bind từ "stock"
}
```

### 6.4. [FromBody], [FromForm], [FromRoute], [FromQuery]

**Convention:** Chỉ định nguồn dữ liệu cụ thể

```csharp
// Từ Route
public IActionResult Details([FromRoute] int id) { }

// Từ Query String
public IActionResult Search([FromQuery] string keyword) { }

// Từ Form
[HttpPost]
public IActionResult Create([FromForm] Product product) { }

// Từ Body (JSON)
[HttpPost]
public IActionResult Create([FromBody] Product product) { }
```

**Ví dụ:**

```csharp
// URL: /Products/Search?keyword=laptop
public IActionResult Search([FromQuery] string keyword)
{
    // keyword = "laptop" (từ query string)
}

// POST /Products/Create
// Body: {"name":"Laptop","price":1000}
[HttpPost]
public IActionResult Create([FromBody] Product product)
{
    // product được bind từ JSON body
}
```

---

## 💉 7. DEPENDENCY INJECTION CONVENTIONS

### 7.1. Constructor Injection Convention

**Convention:** DI Container tự động inject dependencies qua constructor

```csharp
public class ProductsController : Controller
{
    private readonly IProductService _productService;
    private readonly ILogger<ProductsController> _logger;

    // ✅ ĐÚNG - Constructor injection
    public ProductsController(
        IProductService productService,
        ILogger<ProductsController> logger)
    {
        _productService = productService;
        _logger = logger;
    }
}
```

**Quy tắc:**

1. **Constructor public:** Phải public
2. **Parameters:** DI Container tự động resolve
3. **Registration:** Service phải được đăng ký trong `Program.cs`

### 7.2. Service Registration Conventions

**Convention:** Đăng ký service với lifetime

```csharp
// Program.cs
builder.Services.AddScoped<IProductService, ProductService>();
//            ↑ Lifetime  ↑ Interface    ↑ Implementation
```

**Lifetimes:**

```csharp
// Transient: Mỗi lần request = instance mới
builder.Services.AddTransient<IProductService, ProductService>();

// Scoped: Mỗi HTTP request = 1 instance (khuyên dùng cho DbContext)
builder.Services.AddScoped<IProductService, ProductService>();

// Singleton: Toàn bộ app = 1 instance
builder.Services.AddSingleton<IProductService, ProductService>();
```

### 7.3. Naming Convention cho Services

**Convention:** Interface = `I{ServiceName}`, Implementation = `{ServiceName}`

```csharp
// ✅ ĐÚNG - Tuân theo convention
public interface IProductService { }
public class ProductService : IProductService { }

// Đăng ký:
builder.Services.AddScoped<IProductService, ProductService>();
```

**Ví dụ:**

```csharp
// Interface
public interface IProductService
{
    Task<List<Product>> GetProductsAsync();
}

// Implementation
public class ProductService : IProductService
{
    public async Task<List<Product>> GetProductsAsync()
    {
        // Implementation
    }
}

// Registration
builder.Services.AddScoped<IProductService, ProductService>();

// Usage trong Controller
public class ProductsController : Controller
{
    private readonly IProductService _productService;

    public ProductsController(IProductService productService)
    {
        _productService = productService; // Tự động inject
    }
}
```

---

## ⚙️ 8. CONFIGURATION CONVENTIONS

### 8.1. appsettings.json Convention

**Convention:** Cấu hình được load tự động từ `appsettings.json`

```
appsettings.json          ← Base configuration
appsettings.Development.json  ← Development overrides
appsettings.Production.json   ← Production overrides
```

**Ví dụ:**

```json
// appsettings.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDb;..."
  }
}
```

**Đọc trong code:**

```csharp
// Program.cs
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");

// Hoặc
var logLevel = builder.Configuration["Logging:LogLevel:Default"];
```

### 8.2. Environment Variables Convention

**Convention:** Environment variables override appsettings.json

```
Priority (cao → thấp):
1. Environment Variables
2. appsettings.{Environment}.json
3. appsettings.json
```

**Ví dụ:**

```bash
# Environment variable
export ConnectionStrings__DefaultConnection="Server=prod;Database=MyDb;..."

# Hoặc trong Windows
set ConnectionStrings__DefaultConnection=Server=prod;Database=MyDb;...
```

### 8.3. Options Pattern Convention

**Convention:** Bind configuration vào strongly-typed class

```csharp
// appsettings.json
{
  "JwtSettings": {
    "SecretKey": "my-secret-key",
    "Issuer": "MyApp",
    "Audience": "MyApp",
    "ExpirationMinutes": 60
  }
}

// Model
public class JwtSettings
{
    public string SecretKey { get; set; } = string.Empty;
    public string Issuer { get; set; } = string.Empty;
    public string Audience { get; set; } = string.Empty;
    public int ExpirationMinutes { get; set; }
}

// Registration
builder.Services.Configure<JwtSettings>(
    builder.Configuration.GetSection("JwtSettings"));

// Usage
public class TokenService
{
    private readonly JwtSettings _jwtSettings;

    public TokenService(IOptions<JwtSettings> jwtSettings)
    {
        _jwtSettings = jwtSettings.Value; // Tự động bind
    }
}
```

---

## 🔧 9. MIDDLEWARE CONVENTIONS

### 9.1. Middleware Order Convention

**Convention:** Thứ tự middleware quan trọng!

```csharp
var app = builder.Build();

// 1. Exception Handling (đầu tiên)
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
}

// 2. HTTPS Redirection
app.UseHttpsRedirection();

// 3. Static Files
app.UseStaticFiles();

// 4. Routing
app.UseRouting();

// 5. Authentication
app.UseAuthentication();

// 6. Authorization
app.UseAuthorization();

// 7. Endpoints
app.MapControllers();
```

**Quy tắc:** Sai thứ tự = lỗi khó debug!

### 9.2. Middleware Naming Convention

**Convention:** Middleware class kết thúc bằng `Middleware`

```csharp
// ✅ ĐÚNG
public class RequestLoggingMiddleware { }
public class PerformanceMiddleware { }

// ❌ SAI (không bắt buộc, nhưng nên theo)
public class RequestLogging { }
```

---

## 📂 10. FILE STRUCTURE CONVENTIONS

### 10.1. Project Structure Convention

**Convention:** Cấu trúc thư mục chuẩn

```
MyApp/
├── Controllers/          ← Controllers
│   └── HomeController.cs
├── Views/               ← Views
│   ├── Home/
│   └── Shared/
├── Models/              ← Domain models
├── Services/            ← Business logic
├── Data/                ← Data access (DbContext)
├── ViewModels/          ← View models
├── wwwroot/             ← Static files
├── Program.cs           ← Entry point
└── appsettings.json    ← Configuration
```

### 10.2. Namespace Convention

**Convention:** Namespace = Project name + Folder path

```csharp
// File: Controllers/ProductsController.cs
namespace MyFirstRazorApp.Controllers
{
    public class ProductsController : Controller { }
}

// File: Models/Product.cs
namespace MyFirstRazorApp.Models
{
    public class Product { }
}

// File: Services/ProductService.cs
namespace MyFirstRazorApp.Services
{
    public class ProductService { }
}
```

---

## 🎯 11. TỔNG HỢP CÁC CONVENTIONS QUAN TRỌNG

### 11.1. Checklist Conventions

| Convention | Quy tắc | Ví dụ |
|------------|---------|-------|
| **Controller** | Kết thúc bằng `Controller` | `HomeController` |
| **Action** | Public method, trả về `IActionResult` | `public IActionResult Index()` |
| **View Location** | `Views/{Controller}/{Action}.cshtml` | `Views/Home/Index.cshtml` |
| **Routing** | `{controller}/{action}/{id?}` | `/Home/Index` |
| **Model Binding** | Tên parameter khớp với key | `id` trong route → `int id` |
| **DI** | Constructor injection | `public Controller(IService service)` |
| **Service** | Interface = `I{Name}`, Class = `{Name}` | `IProductService`, `ProductService` |
| **Configuration** | `appsettings.json` → `appsettings.{Env}.json` | Development overrides |

### 11.2. Ví dụ tổng hợp

**Controller:**

```csharp
// Controllers/ProductsController.cs
namespace MyFirstRazorApp.Controllers
{
    public class ProductsController : Controller // ← Convention: Controller suffix
    {
        private readonly IProductService _productService; // ← Convention: DI

        public ProductsController(IProductService productService) // ← Convention: Constructor injection
        {
            _productService = productService;
        }

        // Convention: Action = public IActionResult
        // Convention: URL = /Products/Index
        public IActionResult Index() // ← Convention: Index là mặc định
        {
            var products = _productService.GetProducts();
            return View(products); // ← Convention: Tìm Views/Products/Index.cshtml
        }

        // Convention: URL = /Products/Details/1
        public IActionResult Details(int id) // ← Convention: id từ route
        {
            var product = _productService.GetProduct(id);
            return View(product); // ← Convention: Tìm Views/Products/Details.cshtml
        }
    }
}
```

**Service:**

```csharp
// Services/IProductService.cs
namespace MyFirstRazorApp.Services
{
    public interface IProductService // ← Convention: I prefix
    {
        List<Product> GetProducts();
        Product GetProduct(int id);
    }
}

// Services/ProductService.cs
namespace MyFirstRazorApp.Services
{
    public class ProductService : IProductService // ← Convention: Implementation
    {
        public List<Product> GetProducts() { }
        public Product GetProduct(int id) { }
    }
}
```

**Registration:**

```csharp
// Program.cs
builder.Services.AddScoped<IProductService, ProductService>(); // ← Convention: Scoped lifetime
```

**View:**

```
Views/
└── Products/
    ├── Index.cshtml    ← Convention: Views/{Controller}/{Action}.cshtml
    └── Details.cshtml
```

---

## ❌ 12. CÁC LỖI THƯỜNG GẶP KHI KHÔNG TUÂN THEO CONVENTION

### 12.1. Controller không tìm thấy

```csharp
// ❌ SAI - Thiếu "Controller" suffix
public class Home : Controller { }
// URL: /Home → 404 Not Found

// ✅ ĐÚNG
public class HomeController : Controller { }
// URL: /Home → Tìm thấy HomeController
```

### 12.2. View không tìm thấy

```csharp
// Controller: ProductsController
// Action: Index()

// ❌ SAI - View ở sai vị trí
Views/Product/Index.cshtml  // Thiếu "s" trong Products

// ✅ ĐÚNG
Views/Products/Index.cshtml // Đúng tên Controller (bỏ "Controller")
```

### 12.3. Action không được gọi

```csharp
// ❌ SAI - Private method
private IActionResult Helper() { }
// URL: /Products/Helper → 404

// ✅ ĐÚNG - Public method
public IActionResult Helper() { }
// URL: /Products/Helper → OK
```

### 12.4. Model Binding không hoạt động

```csharp
// URL: /Products/Details/1

// ❌ SAI - Tên parameter không khớp
public IActionResult Details(int productId) // ← Không khớp với route {id}
{
    // productId = 0 (default value)
}

// ✅ ĐÚNG - Tên khớp với route
public IActionResult Details(int id) // ← Khớp với route {id}
{
    // id = 1
}
```

---

## 📝 13. TÓM TẮT

### ✅ Controller Conventions:
- Tên class kết thúc bằng `Controller`
- Kế thừa `Controller` hoặc có `[Controller]`
- Routing tự động bỏ "Controller" khi tạo URL

### ✅ Action Conventions:
- Public method
- Trả về `IActionResult` hoặc `Task<IActionResult>`
- Tên method = Action name trong URL

### ✅ View Conventions:
- Location: `Views/{Controller}/{Action}.cshtml`
- `_ViewStart.cshtml` chạy trước mỗi View
- `_ViewImports.cshtml` import namespaces

### ✅ Routing Conventions:
- Pattern: `{controller=Home}/{action=Index}/{id?}`
- Attribute routing override conventional
- Route constraints validate parameters

### ✅ Model Binding Conventions:
- Thứ tự: Form → Route → Query → Headers
- Tên parameter phải khớp với key
- Complex objects tự động bind

### ✅ DI Conventions:
- Constructor injection
- Interface = `I{Name}`, Class = `{Name}`
- Lifetime: Transient, Scoped, Singleton

### ✅ Configuration Conventions:
- `appsettings.json` → `appsettings.{Env}.json`
- Environment variables có ưu tiên cao nhất
- Options pattern cho strongly-typed config

---

## 🎯 BÀI TẬP THỰC HÀNH

### Bài 1: Tạo Controller theo Convention
- Tạo `BlogController` với các actions: Index, Details, Create
- Đảm bảo tuân theo tất cả conventions
- Test routing với các URL khác nhau

### Bài 2: Model Binding
- Tạo form với các fields: Name, Email, Message
- Bind vào Model trong Controller
- Kiểm tra binding từ Form, Route, Query

### Bài 3: DI Convention
- Tạo `IBlogService` và `BlogService`
- Đăng ký trong `Program.cs`
- Inject vào `BlogController`

---

**Hiểu rõ conventions giúp bạn code nhanh hơn, ít lỗi hơn!** 🚀

---

**Chương tiếp theo: [03. MVC Pattern & Routing →](../phan_2_mvc_database/03_mvc_pattern.md)**

