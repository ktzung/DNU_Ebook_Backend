# 🟩 CHƯƠNG 03
# **MVC PATTERN & ROUTING**

ASP.NET Core MVC là pattern phổ biến để xây dựng web applications. Chương này giúp bạn hiểu cách MVC hoạt động và cách định tuyến (routing) requests đến đúng controller.

---

# 🎯 MỤC TIÊU HỌC TẬP

Sau chương này, bạn sẽ:

- Hiểu MVC Pattern: Model, View, Controller
- Tạo Controllers và Actions
- Nắm vững Routing (Conventional và Attribute)
- Hiểu ActionResult và các loại responses
- Truyền dữ liệu từ Controller sang View
- Xây dựng trang web đầu tiên với MVC

---

# 1. **MVC PATTERN LÀ GÌ?**

## 1.1. Khái niệm

**MVC = Model-View-Controller** — Pattern tách biệt logic ứng dụng thành 3 phần:

### 🏠 Ví dụ đời sống: Nhà hàng

```
Model (Món ăn)      ← Dữ liệu, logic nghiệp vụ
   ↕
Controller (Bồi bàn) ← Nhận order, điều phối
   ↕
View (Thực đơn)     ← Giao diện người dùng
```

**Workflow:**
1. Khách hàng xem **View** (thực đơn)
2. Khách gọi món → **Controller** (bồi bàn) nhận order
3. Controller lấy thông tin từ **Model** (món ăn)
4. Controller trả **View** (món ăn) cho khách

---

## 1.2. MVC trong ASP.NET Core

```
User Request
    ↓
Routing → Controller → Model (Database)
              ↓
            View (HTML)
              ↓
         Response
```

---

# 2. **CONTROLLER — ĐIỀU KHIỂN LOGIC**

## 2.1. Tạo Controller đầu tiên

```csharp
// Controllers/HomeController.cs
using Microsoft.AspNetCore.Mvc;

public class HomeController : Controller
{
    // Action Method
    public IActionResult Index()
    {
        return View(); // Trả về Views/Home/Index.cshtml
    }
    
    public IActionResult About()
    {
        return View();
    }
    
    public IActionResult Contact()
    {
        return View();
    }
}
```

**Quy ước:**
- Controller class phải có hậu tố `Controller`
- Kế thừa từ `Controller` base class
- Action = public method trả về `IActionResult`

---

## 2.2. Action Results

### 🟢 ViewResult — Trả về View (HTML)

```csharp
public IActionResult Index()
{
    return View(); // Views/Home/Index.cshtml
}

public IActionResult About()
{
    return View("AboutPage"); // Views/Home/AboutPage.cshtml
}
```

---

### 🟢 JsonResult — Trả về JSON

```csharp
public IActionResult GetProducts()
{
    var products = new[] 
    {
        new { Id = 1, Name = "Laptop" },
        new { Id = 2, Name = "Mouse" }
    };
    return Json(products);
}
```

---

### 🟢 RedirectResult — Chuyển hướng

```csharp
public IActionResult OldPage()
{
    return Redirect("/Home/NewPage");
}

public IActionResult RedirectToAction()
{
    return RedirectToAction("Index", "Home");
}
```

---

### 🟢 ContentResult — Trả về text

```csharp
public IActionResult GetText()
{
    return Content("Hello World!", "text/plain");
}
```

---

### 🟢 NotFoundResult — 404

```csharp
public IActionResult GetProduct(int id)
{
    var product = _db.Products.Find(id);
    if (product == null)
        return NotFound(); // 404
    
    return View(product);
}
```

---

## 2.3. Truyền dữ liệu từ Controller sang View

### Cách 1: Model (Strongly Typed) — ✅ Best Practice

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

public IActionResult Details(int id)
{
    var product = new Product 
    { 
        Id = id, 
        Name = "Laptop", 
        Price = 1500 
    };
    
    return View(product); // Truyền model vào view
}
```

```html
<!-- Views/Products/Details.cshtml -->
@model Product

<h1>@Model.Name</h1>
<p>Price: $@Model.Price</p>
```

---

### Cách 2: ViewBag (Dynamic)

```csharp
public IActionResult Index()
{
    ViewBag.Title = "Home Page";
    ViewBag.Products = new List<string> { "Laptop", "Mouse" };
    return View();
}
```

```html
@{
    var title = ViewBag.Title;
}
<h1>@title</h1>
```

---

### Cách 3: ViewData (Dictionary)

```csharp
public IActionResult Index()
{
    ViewData["Title"] = "Home Page";
    ViewData["Count"] = 10;
    return View();
}
```

```html
<h1>@ViewData["Title"]</h1>
<p>Count: @ViewData["Count"]</p>
```

---

### Cách 4: TempData (Redirect giữ dữ liệu)

```csharp
public IActionResult Create(Product product)
{
    _db.Products.Add(product);
    _db.SaveChanges();
    
    TempData["Message"] = "Product created successfully!";
    return RedirectToAction("Index");
}

public IActionResult Index()
{
    // TempData vẫn còn sau redirect
    var message = TempData["Message"];
    return View();
}
```

---

# 3. **ROUTING — ĐỊNH TUYẾN REQUEST**

## 3.1. URL Structure

```
https://example.com/{controller}/{action}/{id?}
                      ↓           ↓        ↓
                    Home       Index      5
```

---

## 3.2. Conventional Routing (Traditional)

```csharp
// Program.cs
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Ví dụ:**
- `/` → `HomeController.Index()`
- `/Home` → `HomeController.Index()`
- `/Products` → `ProductsController.Index()`
- `/Products/Details` → `ProductsController.Details()`
- `/Products/Details/5` → `ProductsController.Details(5)`

---

## 3.3. Attribute Routing (Modern) — ✅ Recommended

```csharp
[Route("products")]
public class ProductsController : Controller
{
    // GET: /products
    [HttpGet]
    public IActionResult Index()
    {
        return View();
    }
    
    // GET: /products/5
    [HttpGet("{id}")]
    public IActionResult Details(int id)
    {
        return View();
    }
    
    // GET: /products/create
    [HttpGet("create")]
    public IActionResult Create()
    {
        return View();
    }
    
    // POST: /products/create
    [HttpPost("create")]
    public IActionResult Create(Product product)
    {
        // Save to database
        return RedirectToAction("Index");
    }
    
    // GET: /products/5/edit
    [HttpGet("{id}/edit")]
    public IActionResult Edit(int id)
    {
        return View();
    }
}
```

---

## 3.4. Route Constraints

```csharp
// Chỉ chấp nhận số
[HttpGet("{id:int}")]
public IActionResult Details(int id) { }

// Chỉ chấp nhận số > 0
[HttpGet("{id:int:min(1)}")]
public IActionResult Details(int id) { }

// Chỉ chấp nhận GUID
[HttpGet("{id:guid}")]
public IActionResult Details(Guid id) { }

// Regex pattern
[HttpGet("{slug:regex(^[a-z-]+$)}")]
public IActionResult Details(string slug) { }
```

---

## 3.5. Multiple Routes

```csharp
[HttpGet]
[Route("products")]
[Route("items")] // Cả 2 routes đều work
public IActionResult Index()
{
    return View();
}
```

---

# 4. **VÍ DỤ THỰC TẾ: PRODUCTS CONTROLLER**

## 4.1. Setup Models

```csharp
// Models/Product.cs
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public int Stock { get; set; }
}
```

---

## 4.2. Products Controller (Full CRUD)

```csharp
// Controllers/ProductsController.cs
[Route("products")]
public class ProductsController : Controller
{
    private static List<Product> _products = new()
    {
        new Product { Id = 1, Name = "Laptop", Price = 1500, Stock = 10 },
        new Product { Id = 2, Name = "Mouse", Price = 25, Stock = 50 }
    };
    
    // GET: /products
    [HttpGet]
    public IActionResult Index()
    {
        return View(_products);
    }
    
    // GET: /products/5
    [HttpGet("{id}")]
    public IActionResult Details(int id)
    {
        var product = _products.FirstOrDefault(p => p.Id == id);
        if (product == null)
            return NotFound();
        
        return View(product);
    }
    
    // GET: /products/create
    [HttpGet("create")]
    public IActionResult Create()
    {
        return View();
    }
    
    // POST: /products/create
    [HttpPost("create")]
    public IActionResult Create(Product product)
    {
        product.Id = _products.Any() ? _products.Max(p => p.Id) + 1 : 1;
        _products.Add(product);
        
        TempData["SuccessMessage"] = "Product created successfully!";
        return RedirectToAction("Index");
    }
    
    // GET: /products/5/edit
    [HttpGet("{id}/edit")]
    public IActionResult Edit(int id)
    {
        var product = _products.FirstOrDefault(p => p.Id == id);
        if (product == null)
            return NotFound();
        
        return View(product);
    }
    
    // POST: /products/5/edit
    [HttpPost("{id}/edit")]
    public IActionResult Edit(int id, Product updatedProduct)
    {
        var product = _products.FirstOrDefault(p => p.Id == id);
        if (product == null)
            return NotFound();
        
        product.Name = updatedProduct.Name;
        product.Description = updatedProduct.Description;
        product.Price = updatedProduct.Price;
        product.Stock = updatedProduct.Stock;
        
        TempData["SuccessMessage"] = "Product updated successfully!";
        return RedirectToAction("Index");
    }
    
    // POST: /products/5/delete
    [HttpPost("{id}/delete")]
    public IActionResult Delete(int id)
    {
        var product = _products.FirstOrDefault(p => p.Id == id);
        if (product == null)
            return NotFound();
        
        _products.Remove(product);
        
        TempData["SuccessMessage"] = "Product deleted successfully!";
        return RedirectToAction("Index");
    }
}
```

---

# 5. **ACTION PARAMETERS — NHẬN DỮ LIỆU**

## 5.1. Route Parameters

```csharp
// URL: /products/5
[HttpGet("{id}")]
public IActionResult Details(int id)
{
    // id = 5
}
```

---

## 5.2. Query String

```csharp
// URL: /products?page=2&pageSize=10
[HttpGet]
public IActionResult Index(int page = 1, int pageSize = 10)
{
    // page = 2, pageSize = 10
}
```

---

## 5.3. Form Data

```csharp
// POST với form data
[HttpPost]
public IActionResult Create(Product product)
{
    // ASP.NET Core tự động bind form fields vào product
}
```

---

## 5.4. Multiple Parameters

```csharp
// URL: /products/search?query=laptop&page=1
[HttpGet("search")]
public IActionResult Search(string query, int page = 1)
{
    var results = _products
        .Where(p => p.Name.Contains(query))
        .Skip((page - 1) * 10)
        .Take(10)
        .ToList();
    
    return View(results);
}
```

---

# 6. **AREAS — TỔ CHỨC DỰ ÁN LỚN**

Areas giúp chia ứng dụng thành các modules nhỏ.

```
Areas/
├── Admin/
│   ├── Controllers/
│   │   └── ProductsController.cs
│   └── Views/
│       └── Products/
│           └── Index.cshtml
└── Customer/
    ├── Controllers/
    └── Views/
```

```csharp
// Areas/Admin/Controllers/ProductsController.cs
[Area("Admin")]
[Route("admin/products")]
public class ProductsController : Controller
{
    [HttpGet]
    public IActionResult Index()
    {
        return View();
    }
}

// Program.cs
app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");
```

---

# 7. **BÀI TẬP THỰC HÀNH**

## 📝 Bài 1: Tạo Categories Controller

Tạo `CategoriesController` với các actions:
- Index — Hiển thị danh sách
- Details — Xem chi tiết
- Create — Tạo mới
- Edit — Chỉnh sửa
- Delete — Xóa

<details>
<summary>💡 Đáp án</summary>

```csharp
[Route("categories")]
public class CategoriesController : Controller
{
    private static List<Category> _categories = new()
    {
        new Category { Id = 1, Name = "Electronics" },
        new Category { Id = 2, Name = "Books" }
    };
    
    [HttpGet]
    public IActionResult Index()
    {
        return View(_categories);
    }
    
    [HttpGet("{id}")]
    public IActionResult Details(int id)
    {
        var category = _categories.FirstOrDefault(c => c.Id == id);
        if (category == null) return NotFound();
        return View(category);
    }
    
    [HttpGet("create")]
    public IActionResult Create()
    {
        return View();
    }
    
    [HttpPost("create")]
    public IActionResult Create(Category category)
    {
        category.Id = _categories.Max(c => c.Id) + 1;
        _categories.Add(category);
        return RedirectToAction("Index");
    }
}
```
</details>

---

## 📝 Bài 2: Custom Route

Tạo route: `/shop/item/{id}` trỏ đến `ProductsController.Details(id)`

<details>
<summary>💡 Đáp án</summary>

```csharp
[Route("shop")]
public class ProductsController : Controller
{
    [HttpGet("item/{id}")]
    public IActionResult Details(int id)
    {
        // ...
    }
}
```
</details>

---

# 🧪 MINI TEST

1. **MVC là viết tắt của gì?**
   - A. Model-View-Component
   - B. Model-View-Controller
   - C. Module-View-Controller

2. **Action method phải trả về kiểu gì?**
   - A. string
   - B. IActionResult
   - C. View

3. **Route `/products/5` sẽ map đến?**
   - A. ProductsController.Index(5)
   - B. ProductsController.Details(5)
   - C. Tùy routing configuration

4. **TempData khác ViewBag như thế nào?**
   - A. TempData tồn tại qua redirect
   - B. TempData nhanh hơn
   - C. Không khác gì

<details>
<summary>💡 Đáp án</summary>

1. **B** - Model-View-Controller
2. **B** - IActionResult (hoặc ActionResult<T>)
3. **C** - Tùy routing configuration (thường là Details)
4. **A** - TempData tồn tại qua redirect, ViewBag chỉ trong 1 request
</details>

---

# 📌 TÓM TẮT CHƯƠNG

✅ **MVC Pattern** chia ứng dụng: Model (dữ liệu), View (UI), Controller (logic)  
✅ **Controller** xử lý requests, gọi Model, trả về View  
✅ **Action Results**: View, Json, Redirect, NotFound, Content  
✅ **Routing**: Conventional vs Attribute (prefer Attribute)  
✅ **Truyền dữ liệu**: Model (best), ViewBag, ViewData, TempData  
✅ **Areas** tổ chức dự án lớn  

---

**Chương tiếp theo: [04. Model Binding & Validation →](./04_model_binding_validation.md)**
