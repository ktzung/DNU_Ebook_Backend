# 🎯 THỰC HÀNH 01
# **TẠO ỨNG DỤNG WEB RAZOR ĐẦU TIÊN - MVC & ROUTING**

> **Mục tiêu:** Tạo ứng dụng web Razor đơn giản từ đầu, hiểu rõ luồng **Routing → Controller → View** với ASP.NET Core 8

---

## 📋 MỤC TIÊU THỰC HÀNH

Sau bài thực hành này, bạn sẽ:

- ✅ Tạo project ASP.NET Core MVC 8 từ đầu
- ✅ Hiểu **luồng request**: Routing → Controller → View → Response
- ✅ Hiểu cấu trúc thư mục MVC
- ✅ Tạo Controllers và Actions
- ✅ Tạo Views với Razor syntax
- ✅ Hiểu Routing (Conventional và Attribute)
- ✅ Truyền dữ liệu từ Controller sang View
- ✅ Có ứng dụng web chạy được với nhiều trang

---

## 🛠️ CHUẨN BỊ

### Công cụ cần có:
- ✅ Visual Studio 2022 (Community Edition trở lên) hoặc VS Code
- ✅ **.NET 8 SDK** (bắt buộc - phiên bản mới nhất)
- ✅ Trình duyệt web (Chrome, Edge, Firefox)

### Kiến thức cần có:
- ✅ C# cơ bản (class, method, variable)
- ✅ HTML cơ bản
- ✅ Hiểu cơ bản về web (URL, HTTP)

---

## 🔄 LUỒNG REQUEST/RESPONSE TRONG ASP.NET CORE MVC

Trước khi bắt đầu, hãy hiểu luồng hoạt động:

```
┌─────────────────────────────────────────────────────────────┐
│  1. USER GỬI REQUEST                                        │
│     Browser: https://localhost:5001/Home/Index             │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  2. MIDDLEWARE PIPELINE (Program.cs)                       │
│     - HTTPS Redirection                                     │
│     - Static Files                                          │
│     - Routing                                               │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  3. ROUTING (Program.cs)                                   │
│     Pattern: {controller}/{action}/{id?}                    │
│     URL: /Home/Index                                        │
│     → Tìm: HomeController.Index()                          │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  4. CONTROLLER (HomeController.cs)                         │
│     public IActionResult Index()                           │
│     {                                                       │
│         return View(); // Trả về View                      │
│     }                                                       │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  5. VIEW ENGINE (Razor)                                    │
│     Tìm file: Views/Home/Index.cshtml                      │
│     Render HTML với dữ liệu từ Controller                  │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  6. RESPONSE                                               │
│     HTML được gửi về Browser                              │
│     Browser hiển thị trang web                             │
└─────────────────────────────────────────────────────────────┘
```

**Ví dụ cụ thể:**

1. **User click link:** `<a href="/About">About</a>`
2. **Browser gửi request:** `GET /About`
3. **Routing:** `/About` → `AboutController.Index()`
4. **Controller:** `AboutController.Index()` chạy, trả về `View()`
5. **View Engine:** Tìm `Views/About/Index.cshtml`, render HTML
6. **Response:** HTML được gửi về browser
7. **Browser:** Hiển thị trang About

**Chúng ta sẽ học từng bước một, từ dễ đến khó!**

---

## 📦 BƯỚC 1: TẠO PROJECT MỚI (DỄ - BẮT ĐẦU)

### 1.1. Tạo ASP.NET Core MVC Project (.NET 8)

#### Cách 1: Visual Studio 2022

1. **Mở Visual Studio 2022**

2. **Tạo Project mới:**
   - Click **File** → **New** → **Project** (hoặc `Ctrl+Shift+N`)
   - Tìm: `ASP.NET Core Web App (Model-View-Controller)`
   - **Quan trọng:** Chọn template có icon MVC (không phải Razor Pages)
   - Click **Next**

3. **Cấu hình Project:**
   ```
   Project name: MyFirstRazorApp
   Location: D:\Projects (hoặc nơi bạn muốn)
   Solution name: MyFirstRazorApp
   Framework: .NET 8.0 (Long Term Support) ← QUAN TRỌNG!
   Authentication: No Authentication
   Configure for HTTPS: ✅ (checked)
   Enable OpenAPI support: ❌ (unchecked - không cần cho MVC)
   Use controllers (uncheck to use minimal APIs): ✅ (checked)
   ```
   - Click **Create**

#### Cách 2: Command Line (Terminal)

```bash
# Tạo project MVC
dotnet new mvc -n MyFirstRazorApp

# Di chuyển vào thư mục
cd MyFirstRazorApp

# Chạy ứng dụng
dotnet run
```

### 1.2. Kiểm tra Program.cs (.NET 8)

Mở file `Program.cs` - đây là file quan trọng nhất:

```csharp
// Program.cs - .NET 8 (Minimal Hosting Model)
var builder = WebApplication.CreateBuilder(args);

// ========== CONFIGURATION STAGE ==========
// Đăng ký services vào DI Container
builder.Services.AddControllersWithViews(); // ← Quan trọng cho MVC!

var app = builder.Build();

// ========== MIDDLEWARE PIPELINE STAGE ==========
// Cấu hình HTTP request pipeline

// 1. Exception Handling (Development)
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts(); // HTTP Strict Transport Security
}

// 2. HTTPS Redirection
app.UseHttpsRedirection();

// 3. Static Files (CSS, JS, images)
app.UseStaticFiles();

// 4. Routing
app.UseRouting();

// 5. Authorization (nếu có)
// app.UseAuthorization();

// 6. Map Controllers (MVC)
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// ========== RUN APPLICATION ==========
app.Run();
```

**Giải thích từng phần:**

1. **`builder.Services.AddControllersWithViews()`**
   - Đăng ký MVC services vào DI Container
   - Cho phép sử dụng Controllers và Views

2. **Middleware Pipeline:**
   - `UseHttpsRedirection()`: Chuyển HTTP → HTTPS
   - `UseStaticFiles()`: Phục vụ file tĩnh (CSS, JS)
   - `UseRouting()`: Bật routing
   - `MapControllerRoute()`: Định nghĩa routing pattern

3. **Routing Pattern:**
   ```
   {controller=Home}/{action=Index}/{id?}
   ```
   - `{controller=Home}`: Controller mặc định là Home
   - `{action=Index}`: Action mặc định là Index
   - `{id?}`: Tham số tùy chọn

### 1.3. Kiểm tra cấu trúc Project

Sau khi tạo, bạn sẽ thấy cấu trúc như sau:

```
MyFirstRazorApp/
├── Controllers/              ← Controllers (điều khiển logic)
│   └── HomeController.cs
├── Views/                    ← Views (giao diện HTML)
│   ├── Home/
│   │   ├── Index.cshtml
│   │   ├── Privacy.cshtml
│   │   └── Error.cshtml
│   ├── Shared/
│   │   ├── _Layout.cshtml    ← Layout chung
│   │   └── _ValidationScriptsPartial.cshtml
│   └── _ViewStart.cshtml     ← Chạy trước mỗi View
├── Models/                    ← Models (dữ liệu)
├── wwwroot/                   ← Static files (CSS, JS, images)
│   ├── css/
│   ├── js/
│   └── lib/
├── appsettings.json           ← Cấu hình
├── Program.cs                 ← Entry point (QUAN TRỌNG!)
└── MyFirstRazorApp.csproj     ← Project file
```

### 1.4. Chạy ứng dụng lần đầu

1. **Nhấn F5** hoặc click **Run** (▶️) hoặc `Ctrl+F5` (không debug)
2. **Trình duyệt sẽ mở** với URL: `https://localhost:5001` (port có thể khác)
3. **Bạn sẽ thấy trang Home** với nội dung mặc định

🎉 **Xin chúc mừng!** Bạn đã có ứng dụng web đầu tiên chạy được!

**Quan sát:**
- URL trong browser: `https://localhost:5001/`
- Trang hiển thị: "Welcome" với link "Learn about building Web apps"
- Đây chính là `HomeController.Index()` → `Views/Home/Index.cshtml`

### 1.2. Kiểm tra cấu trúc Project

Sau khi tạo, bạn sẽ thấy cấu trúc như sau:

```
MyFirstRazorApp/
├── Controllers/          ← Controllers (điều khiển logic)
│   └── HomeController.cs
├── Views/                ← Views (giao diện HTML)
│   ├── Home/
│   │   ├── Index.cshtml
│   │   ├── Privacy.cshtml
│   ├── Shared/
│   │   └── _Layout.cshtml
│   └── _ViewStart.cshtml
├── Models/               ← Models (dữ liệu)
├── wwwroot/              ← Static files (CSS, JS, images)
│   ├── css/
│   ├── js/
│   └── lib/
├── appsettings.json      ← Cấu hình
├── Program.cs            ← Entry point
└── MyFirstRazorApp.csproj
```

### 1.3. Chạy ứng dụng lần đầu

1. **Nhấn F5** hoặc click **Run** (▶️)
2. **Trình duyệt sẽ mở** với URL: `https://localhost:5001` (port có thể khác)
3. **Bạn sẽ thấy trang Home** với nội dung mặc định

🎉 **Xin chúc mừng!** Bạn đã có ứng dụng web đầu tiên chạy được!

---

## 🎨 BƯỚC 2: HIỂU LUỒNG ROUTING → CONTROLLER → VIEW (DỄ - NẮM VỮNG CƠ BẢN)

### 2.1. Minh họa luồng với ví dụ cụ thể

**Kịch bản:** User truy cập `https://localhost:5001/Home/Index`

```
┌──────────────────────────────────────────────────────────────┐
│ BƯỚC 1: USER GỬI REQUEST                                     │
│ Browser gửi: GET /Home/Index                                 │
└──────────────────────────────────────────────────────────────┘
                        ↓
┌──────────────────────────────────────────────────────────────┐
│ BƯỚC 2: MIDDLEWARE PIPELINE (Program.cs)                   │
│ UseHttpsRedirection() → UseStaticFiles() → UseRouting()    │
└──────────────────────────────────────────────────────────────┘
                        ↓
┌──────────────────────────────────────────────────────────────┐
│ BƯỚC 3: ROUTING (Program.cs - MapControllerRoute)           │
│ Pattern: {controller=Home}/{action=Index}/{id?}             │
│ URL: /Home/Index                                             │
│ → Parse: controller="Home", action="Index"                  │
│ → Tìm: HomeController.Index()                               │
└──────────────────────────────────────────────────────────────┘
                        ↓
┌──────────────────────────────────────────────────────────────┐
│ BƯỚC 4: CONTROLLER (HomeController.cs)                      │
│ public IActionResult Index()                                 │
│ {                                                            │
│     return View(); // Gọi View Engine                       │
│ }                                                            │
└──────────────────────────────────────────────────────────────┘
                        ↓
┌──────────────────────────────────────────────────────────────┐
│ BƯỚC 5: VIEW ENGINE (Razor)                                │
│ Tìm file: Views/Home/Index.cshtml                           │
│ Render HTML với Layout                                       │
└──────────────────────────────────────────────────────────────┘
                        ↓
┌──────────────────────────────────────────────────────────────┐
│ BƯỚC 6: RESPONSE                                            │
│ HTML được gửi về Browser                                    │
│ Browser hiển thị trang web                                 │
└──────────────────────────────────────────────────────────────┘
```

### 2.2. Xem HomeController

Mở file `Controllers/HomeController.cs`:

```csharp
using Microsoft.AspNetCore.Mvc;

namespace MyFirstRazorApp.Controllers
{
    public class HomeController : Controller
    {
        private readonly ILogger<HomeController> _logger;

        public HomeController(ILogger<HomeController> logger)
        {
            _logger = logger;
        }

        // Action: Index
        // URL: / hoặc /Home hoặc /Home/Index
        public IActionResult Index()
        {
            _logger.LogInformation("HomeController.Index() được gọi");
            return View(); // Trả về Views/Home/Index.cshtml
        }

        // Action: Privacy
        // URL: /Home/Privacy
        public IActionResult Privacy()
        {
            _logger.LogInformation("HomeController.Privacy() được gọi");
            return View(); // Trả về Views/Home/Privacy.cshtml
        }
    }
}
```

**Giải thích chi tiết:**

1. **`HomeController : Controller`**
   - Kế thừa từ `Controller` base class
   - Có quyền truy cập `View()`, `Json()`, `Redirect()`, etc.

2. **`public IActionResult Index()`**
   - `public`: Phải public để routing có thể gọi
   - `IActionResult`: Kiểu trả về (View, Json, Redirect, etc.)
   - `Index()`: Tên action

3. **`return View()`**
   - Gọi View Engine
   - Tự động tìm: `Views/Home/Index.cshtml`
   - Render HTML và trả về

**Thử nghiệm:**
1. Thêm `_logger.LogInformation()` vào Index()
2. Chạy ứng dụng
3. Xem Output window → Bạn sẽ thấy log khi truy cập `/Home/Index`

### 2.3. Xem View Index

Mở file `Views/Home/Index.cshtml`:

```html
@{
    ViewData["Title"] = "Home Page";
}

<div class="text-center">
    <h1 class="display-4">Welcome</h1>
    <p>Learn about <a href="https://docs.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.</p>
</div>
```

**Giải thích chi tiết:**

1. **`@{}`** = Razor code block
   - Chạy C# code
   - `ViewData["Title"]` = Truyền dữ liệu sang Layout

2. **HTML thông thường**
   - Phần còn lại là HTML
   - Razor sẽ render thành HTML cuối cùng

**Luồng View Engine:**

```
Views/Home/Index.cshtml
    ↓
_ViewStart.cshtml (chạy trước)
    ↓
_Layout.cshtml (wrap nội dung)
    ↓
HTML cuối cùng
```

### 2.4. Xem _ViewStart.cshtml

Mở file `Views/_ViewStart.cshtml`:

```csharp
@{
    Layout = "_Layout";
}
```

**Giải thích:**
- Chạy **trước mỗi View**
- Set Layout mặc định = `_Layout.cshtml`
- Tất cả Views sẽ dùng Layout này (trừ khi override)

### 2.5. Xem Layout

Mở file `Views/Shared/_Layout.cshtml`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - MyFirstRazorApp</title>
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container">
                <a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">MyFirstRazorApp</a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target=".navbar-collapse">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex justify-content-between">
                    <ul class="navbar-nav flex-grow-1">
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Index">Home</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Privacy">Privacy</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    <div class="container">
        <main role="main" class="pb-3">
            @RenderBody() <!-- Nội dung của từng trang sẽ hiển thị ở đây -->
        </main>
    </div>

    <footer class="border-top footer text-muted">
        <div class="container">
            &copy; 2024 - MyFirstRazorApp
        </div>
    </footer>
    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/site.js"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

**Giải thích chi tiết:**

1. **`@RenderBody()`**
   - Nơi hiển thị nội dung của từng View
   - `Views/Home/Index.cshtml` sẽ được render vào đây

2. **Tag Helpers:**
   - `asp-controller="Home"` → Tạo URL đến HomeController
   - `asp-action="Index"` → Tạo URL đến action Index
   - Tự động tạo: `<a href="/Home/Index">Home</a>`

3. **Layout = Template chung**
   - Header, Footer, Navigation giống nhau
   - Chỉ phần `@RenderBody()` thay đổi

**Luồng Render View:**

```
1. Controller.Index() gọi View()
   ↓
2. View Engine tìm Views/Home/Index.cshtml
   ↓
3. Chạy _ViewStart.cshtml (set Layout)
   ↓
4. Render Index.cshtml
   ↓
5. Wrap vào _Layout.cshtml (@RenderBody())
   ↓
6. HTML cuối cùng gửi về Browser
```

### 2.6. Test luồng với Debug

**Thực hành:** Thêm breakpoint để xem luồng

1. **Thêm breakpoint** vào `HomeController.Index()`:
   ```csharp
   public IActionResult Index()
   {
       _logger.LogInformation("Bước 1: Controller được gọi"); // ← Breakpoint ở đây
       return View();
   }
   ```

2. **Chạy Debug** (F5)

3. **Truy cập:** `https://localhost:5001/`

4. **Quan sát:**
   - Breakpoint dừng ở Controller
   - Step Over (F10) → Xem `return View()` được gọi
   - View Engine sẽ render HTML

5. **Xem Output:**
   - Log sẽ hiển thị: "Bước 1: Controller được gọi"

---

## 🚀 BƯỚC 3: TẠO CONTROLLER VÀ ACTION MỚI

### 3.1. Tạo AboutController

1. **Right-click** vào thư mục `Controllers`
2. Chọn **Add** → **Controller**
3. Chọn **MVC Controller - Empty**
4. Đặt tên: `AboutController`
5. Click **Add**

### 3.2. Thêm Actions vào AboutController

Mở `Controllers/AboutController.cs` và sửa như sau:

```csharp
using Microsoft.AspNetCore.Mvc;

namespace MyFirstRazorApp.Controllers
{
    public class AboutController : Controller
    {
        // Action: Index
        // URL: /About hoặc /About/Index
        public IActionResult Index()
        {
            return View();
        }

        // Action: Team
        // URL: /About/Team
        public IActionResult Team()
        {
            return View();
        }

        // Action: History
        // URL: /About/History
        public IActionResult History()
        {
            return View();
        }
    }
}
```

**Giải thích:**
- `AboutController` = Controller mới
- `Index()`, `Team()`, `History()` = 3 Actions
- Mỗi Action sẽ tìm View tương ứng trong `Views/About/`

---

## 📄 BƯỚC 4: TẠO VIEWS CHO ABOUT

### 4.1. Tạo thư mục Views/About

1. **Right-click** vào thư mục `Views`
2. Chọn **Add** → **New Folder**
3. Đặt tên: `About`

### 4.2. Tạo View Index.cshtml

1. **Right-click** vào thư mục `Views/About`
2. Chọn **Add** → **Razor View**
3. Đặt tên: `Index`
4. Chọn template: **Empty**
5. Click **Add**

Sửa nội dung `Views/About/Index.cshtml`:

```html
@{
    ViewData["Title"] = "About Us";
}

<div class="container">
    <h1>Giới thiệu về chúng tôi</h1>
    <p>Đây là trang giới thiệu của ứng dụng MyFirstRazorApp.</p>
    <p>Chúng tôi đang học ASP.NET Core MVC!</p>
    
    <h2>Thông tin liên hệ</h2>
    <ul>
        <li>Email: contact@example.com</li>
        <li>Phone: 0123-456-789</li>
        <li>Address: Đà Nẵng, Việt Nam</li>
    </ul>
</div>
```

### 4.3. Tạo View Team.cshtml

Tương tự, tạo `Views/About/Team.cshtml`:

```html
@{
    ViewData["Title"] = "Our Team";
}

<div class="container">
    <h1>Đội ngũ của chúng tôi</h1>
    
    <div class="row">
        <div class="col-md-4">
            <h3>Nguyễn Văn A</h3>
            <p>Giám đốc</p>
        </div>
        <div class="col-md-4">
            <h3>Trần Thị B</h3>
            <p>Phó giám đốc</p>
        </div>
        <div class="col-md-4">
            <h3>Lê Văn C</h3>
            <p>Trưởng phòng kỹ thuật</p>
        </div>
    </div>
</div>
```

### 4.4. Tạo View History.cshtml

Tạo `Views/About/History.cshtml`:

```html
@{
    ViewData["Title"] = "Our History";
}

<div class="container">
    <h1>Lịch sử phát triển</h1>
    
    <div class="timeline">
        <div class="timeline-item">
            <h3>2020</h3>
            <p>Thành lập công ty</p>
        </div>
        <div class="timeline-item">
            <h3>2022</h3>
            <p>Ra mắt sản phẩm đầu tiên</p>
        </div>
        <div class="timeline-item">
            <h3>2024</h3>
            <p>Mở rộng quy mô</p>
        </div>
    </div>
</div>
```

### 4.5. Test các trang mới

1. **Chạy ứng dụng** (F5)
2. **Truy cập các URL:**
   - `https://localhost:5001/About` → Trang About
   - `https://localhost:5001/About/Team` → Trang Team
   - `https://localhost:5001/About/History` → Trang History

🎉 **Bạn đã tạo thành công 3 trang mới!**

---

## 🗺️ BƯỚC 5: HIỂU ROUTING (TRUNG BÌNH - QUAN TRỌNG)

### 5.1. Routing là gì? (Dễ - Khái niệm)

**Routing** = Quá trình ánh xạ URL → Controller + Action

```
URL: /Home/Index
     ↓
Routing Engine phân tích:
  - controller = "Home"
  - action = "Index"
     ↓
Tìm: HomeController.Index()
```

### 5.2. Conventional Routing (Dễ - Mặc định)

Routing mặc định được cấu hình trong `Program.cs` (.NET 8):

```csharp
// Program.cs
var app = builder.Build();

// ... middleware ...

// Map Controllers với Conventional Routing
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Giải thích chi tiết Pattern:**

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

**Luồng Routing hoạt động:**

```
1. User truy cập: /About/Team
   ↓
2. Routing Engine parse:
   - controller = "About"
   - action = "Team"
   - id = null (không có)
   ↓
3. Tìm Controller:
   - Tìm class: AboutController
   - Tìm method: Team()
   ↓
4. Gọi: AboutController.Team()
   ↓
5. Controller trả về View
```

**Ví dụ URL cụ thể:**

| URL | Controller | Action | Giải thích |
|-----|-----------|--------|------------|
| `/` | Home | Index | Mặc định (controller=Home, action=Index) |
| `/Home` | Home | Index | Controller=Home, action mặc định=Index |
| `/Home/Privacy` | Home | Privacy | Đầy đủ controller và action |
| `/About` | About | Index | Controller=About, action mặc định=Index |
| `/About/Team` | About | Team | Đầy đủ |
| `/Products/Details/1` | Products | Details | Có tham số id=1 |
| `/Products/Details` | Products | Details | id=null (optional) |

### 5.3. Test Routing với Debug (Trung bình)

**Thực hành:** Thêm logging để xem routing hoạt động

1. **Thêm vào `Program.cs`:**

```csharp
var app = builder.Build();

// Log routing information
app.Use(async (context, next) =>
{
    var routeData = context.Request.RouteValues;
    Console.WriteLine($"Route: {context.Request.Path}");
    Console.WriteLine($"Controller: {routeData["controller"]}");
    Console.WriteLine($"Action: {routeData["action"]}");
    Console.WriteLine($"Id: {routeData["id"]}");
    await next();
});

// ... rest of middleware ...
```

2. **Chạy ứng dụng và truy cập các URL**
3. **Xem Output window** → Bạn sẽ thấy routing information

### 5.4. Route Constraints (Khó - Nâng cao)

Thêm constraints để validate parameters:

```csharp
// Program.cs
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id:int?}"); // ← id phải là int
```

**Các constraints phổ biến:**

```csharp
{id:int}        // Phải là số nguyên
{id:guid}       // Phải là GUID
{id:min(1)}     // Tối thiểu là 1
{id:max(100)}   // Tối đa là 100
{id:range(1,100)} // Từ 1 đến 100
{name:alpha}    // Chỉ chữ cái
{name:required} // Bắt buộc (không optional)
```

**Ví dụ:**

```csharp
// Chỉ nhận id là số
app.MapControllerRoute(
    name: "products",
    pattern: "Products/Details/{id:int}",
    defaults: new { controller = "Products", action = "Details" });

// URL hợp lệ: /Products/Details/1
// URL không hợp lệ: /Products/Details/abc → 404
```

### 5.5. Attribute Routing (Trung bình - Khuyên dùng)

**Tại sao dùng Attribute Routing?**
- ✅ URL thân thiện hơn
- ✅ Linh hoạt hơn Conventional Routing
- ✅ Dễ đọc và maintain

**Ví dụ 1: Route đơn giản (Dễ)**

Thêm Attribute Routing vào `AboutController`:

```csharp
using Microsoft.AspNetCore.Mvc;

namespace MyFirstRazorApp.Controllers
{
    [Route("gioi-thieu")] // Route prefix cho toàn bộ controller
    public class AboutController : Controller
    {
        // URL: /gioi-thieu hoặc /gioi-thieu/index
        [Route("")]
        [Route("index")]
        public IActionResult Index()
        {
            return View();
        }

        // URL: /gioi-thieu/doi-ngu
        [Route("doi-ngu")]
        public IActionResult Team()
        {
            return View();
        }

        // URL: /gioi-thieu/lich-su
        [Route("lich-su")]
        public IActionResult History()
        {
            return View();
        }
    }
}
```

**Luồng Attribute Routing:**

```
1. User truy cập: /gioi-thieu/doi-ngu
   ↓
2. Routing Engine tìm controller có [Route("gioi-thieu")]
   → Tìm thấy: AboutController
   ↓
3. Tìm action có [Route("doi-ngu")]
   → Tìm thấy: Team()
   ↓
4. Gọi: AboutController.Team()
```

**Ví dụ 2: Route với parameters (Khó)**

```csharp
[Route("san-pham")]
public class ProductsController : Controller
{
    // URL: /san-pham
    [HttpGet("")]
    [HttpGet("danh-sach")]
    public IActionResult Index()
    {
        return View();
    }

    // URL: /san-pham/chi-tiet/1
    [HttpGet("chi-tiet/{id:int}")]
    public IActionResult Details(int id)
    {
        // id phải là số nguyên
        return View();
    }

    // URL: /san-pham/them-moi
    [HttpGet("them-moi")]
    public IActionResult Create()
    {
        return View();
    }

    // URL: /san-pham/them-moi (POST)
    [HttpPost("them-moi")]
    public IActionResult Create(Product product)
    {
        // Xử lý POST request
        return RedirectToAction(nameof(Index));
    }
}
```

**Giải thích:**
- `[HttpGet]` = Chỉ nhận GET request
- `[HttpPost]` = Chỉ nhận POST request
- `{id:int}` = Constraint: id phải là số nguyên
- URL thân thiện: `/san-pham/chi-tiet/1` thay vì `/Products/Details/1`

**Test:**
- `/gioi-thieu` → AboutController.Index()
- `/gioi-thieu/doi-ngu` → AboutController.Team()
- `/san-pham` → ProductsController.Index()
- `/san-pham/chi-tiet/1` → ProductsController.Details(1)
- `/san-pham/chi-tiet/abc` → 404 (vì id phải là int)

### 5.6. So sánh Conventional vs Attribute Routing

| Tiêu chí | Conventional | Attribute |
|----------|-------------|-----------|
| **Vị trí** | Program.cs | Controller/Action |
| **URL** | `/Controller/Action` | Custom (linh hoạt) |
| **Dễ đọc** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Linh hoạt** | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Khuyên dùng** | Dự án nhỏ | Dự án lớn |

**Kết luận:** Attribute Routing khuyên dùng cho dự án thực tế!

---

## 📊 BƯỚC 6: TRUYỀN DỮ LIỆU TỪ CONTROLLER SANG VIEW

### 6.1. Sử dụng ViewData

Sửa `AboutController.Index()`:

```csharp
public IActionResult Index()
{
    ViewData["Title"] = "Giới thiệu";
    ViewData["CompanyName"] = "MyFirstRazorApp";
    ViewData["FoundedYear"] = 2020;
    ViewData["EmployeeCount"] = 50;
    
    return View();
}
```

Sửa `Views/About/Index.cshtml`:

```html
@{
    ViewData["Title"] = "About Us";
}

<div class="container">
    <h1>Giới thiệu về @ViewData["CompanyName"]</h1>
    <p>Thành lập năm: @ViewData["FoundedYear"]</p>
    <p>Số nhân viên: @ViewData["EmployeeCount"]</p>
</div>
```

### 6.2. Sử dụng ViewBag

Sửa `AboutController.Team()`:

```csharp
public IActionResult Team()
{
    ViewBag.Title = "Đội ngũ";
    ViewBag.TeamMembers = new List<string>
    {
        "Nguyễn Văn A - Giám đốc",
        "Trần Thị B - Phó giám đốc",
        "Lê Văn C - Trưởng phòng kỹ thuật"
    };
    
    return View();
}
```

Sửa `Views/About/Team.cshtml`:

```html
@{
    ViewData["Title"] = ViewBag.Title;
}

<div class="container">
    <h1>@ViewBag.Title</h1>
    
    <ul>
        @foreach (var member in ViewBag.TeamMembers)
        {
            <li>@member</li>
        }
    </ul>
</div>
```

### 6.3. Sử dụng Strongly-Typed Model (Khuyên dùng)

**Tạo Model:**

1. **Right-click** vào thư mục `Models`
2. Chọn **Add** → **Class**
3. Đặt tên: `CompanyInfo.cs`

Sửa `Models/CompanyInfo.cs`:

```csharp
namespace MyFirstRazorApp.Models
{
    public class CompanyInfo
    {
        public string Name { get; set; } = string.Empty;
        public int FoundedYear { get; set; }
        public int EmployeeCount { get; set; }
        public string Address { get; set; } = string.Empty;
        public string Email { get; set; } = string.Empty;
    }
}
```

**Sửa Controller:**

```csharp
using MyFirstRazorApp.Models;

public IActionResult Index()
{
    var company = new CompanyInfo
    {
        Name = "MyFirstRazorApp",
        FoundedYear = 2020,
        EmployeeCount = 50,
        Address = "Đà Nẵng, Việt Nam",
        Email = "contact@example.com"
    };
    
    return View(company); // Truyền model sang View
}
```

**Sửa View:**

```html
@model MyFirstRazorApp.Models.CompanyInfo

@{
    ViewData["Title"] = "About Us";
}

<div class="container">
    <h1>Giới thiệu về @Model.Name</h1>
    <p>Thành lập năm: @Model.FoundedYear</p>
    <p>Số nhân viên: @Model.EmployeeCount</p>
    <p>Địa chỉ: @Model.Address</p>
    <p>Email: @Model.Email</p>
</div>
```

**Giải thích:**
- `@model` = Khai báo kiểu model
- `@Model` = Truy cập properties của model
- **Ưu điểm:** Type-safe, IntelliSense, Compile-time checking

---

## 🎨 BƯỚC 7: TẠO CONTROLLER VÀ VIEW PHỨC TẠP HƠN

### 7.1. Tạo ProductsController

Tạo `Controllers/ProductsController.cs`:

```csharp
using Microsoft.AspNetCore.Mvc;
using MyFirstRazorApp.Models;

namespace MyFirstRazorApp.Controllers
{
    public class ProductsController : Controller
    {
        // Danh sách sản phẩm (giả lập)
        private static List<Product> _products = new List<Product>
        {
            new Product { Id = 1, Name = "Laptop", Price = 15000000, Stock = 10 },
            new Product { Id = 2, Name = "Mouse", Price = 200000, Stock = 50 },
            new Product { Id = 3, Name = "Keyboard", Price = 500000, Stock = 30 }
        };

        // GET: /Products hoặc /Products/Index
        public IActionResult Index()
        {
            return View(_products);
        }

        // GET: /Products/Details/1
        public IActionResult Details(int id)
        {
            var product = _products.FirstOrDefault(p => p.Id == id);
            if (product == null)
            {
                return NotFound(); // Trả về 404
            }
            return View(product);
        }

        // GET: /Products/Create
        public IActionResult Create()
        {
            return View();
        }

        // POST: /Products/Create
        [HttpPost]
        public IActionResult Create(Product product)
        {
            if (ModelState.IsValid)
            {
                product.Id = _products.Count + 1;
                _products.Add(product);
                return RedirectToAction(nameof(Index));
            }
            return View(product);
        }
    }
}
```

### 7.2. Tạo Model Product

Tạo `Models/Product.cs`:

```csharp
using System.ComponentModel.DataAnnotations;

namespace MyFirstRazorApp.Models
{
    public class Product
    {
        public int Id { get; set; }
        
        [Required(ErrorMessage = "Tên sản phẩm là bắt buộc")]
        [StringLength(100)]
        public string Name { get; set; } = string.Empty;
        
        [Required]
        [Range(0, double.MaxValue, ErrorMessage = "Giá phải lớn hơn 0")]
        public decimal Price { get; set; }
        
        [Required]
        [Range(0, int.MaxValue, ErrorMessage = "Số lượng phải >= 0")]
        public int Stock { get; set; }
    }
}
```

### 7.3. Tạo Views cho Products

**Views/Products/Index.cshtml:**

```html
@model List<MyFirstRazorApp.Models.Product>

@{
    ViewData["Title"] = "Danh sách sản phẩm";
}

<div class="container">
    <h1>Danh sách sản phẩm</h1>
    
    <p>
        <a asp-action="Create" class="btn btn-primary">Thêm sản phẩm mới</a>
    </p>
    
    <table class="table table-striped">
        <thead>
            <tr>
                <th>ID</th>
                <th>Tên sản phẩm</th>
                <th>Giá</th>
                <th>Tồn kho</th>
                <th>Thao tác</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var product in Model)
            {
                <tr>
                    <td>@product.Id</td>
                    <td>@product.Name</td>
                    <td>@product.Price.ToString("N0") đ</td>
                    <td>@product.Stock</td>
                    <td>
                        <a asp-action="Details" asp-route-id="@product.Id" class="btn btn-info">Chi tiết</a>
                    </td>
                </tr>
            }
        </tbody>
    </table>
</div>
```

**Views/Products/Details.cshtml:**

```html
@model MyFirstRazorApp.Models.Product

@{
    ViewData["Title"] = "Chi tiết sản phẩm";
}

<div class="container">
    <h1>Chi tiết sản phẩm</h1>
    
    <dl class="row">
        <dt class="col-sm-2">ID</dt>
        <dd class="col-sm-10">@Model.Id</dd>
        
        <dt class="col-sm-2">Tên sản phẩm</dt>
        <dd class="col-sm-10">@Model.Name</dd>
        
        <dt class="col-sm-2">Giá</dt>
        <dd class="col-sm-10">@Model.Price.ToString("N0") đ</dd>
        
        <dt class="col-sm-2">Tồn kho</dt>
        <dd class="col-sm-10">@Model.Stock</dd>
    </dl>
    
    <div>
        <a asp-action="Index" class="btn btn-secondary">Quay lại danh sách</a>
    </div>
</div>
```

**Views/Products/Create.cshtml:**

```html
@model MyFirstRazorApp.Models.Product

@{
    ViewData["Title"] = "Thêm sản phẩm mới";
}

<div class="container">
    <h1>Thêm sản phẩm mới</h1>
    
    <form asp-action="Create" method="post">
        <div class="form-group">
            <label asp-for="Name" class="control-label"></label>
            <input asp-for="Name" class="form-control" />
            <span asp-validation-for="Name" class="text-danger"></span>
        </div>
        
        <div class="form-group">
            <label asp-for="Price" class="control-label"></label>
            <input asp-for="Price" class="form-control" type="number" step="0.01" />
            <span asp-validation-for="Price" class="text-danger"></span>
        </div>
        
        <div class="form-group">
            <label asp-for="Stock" class="control-label"></label>
            <input asp-for="Stock" class="form-control" type="number" />
            <span asp-validation-for="Stock" class="text-danger"></span>
        </div>
        
        <div class="form-group">
            <input type="submit" value="Tạo mới" class="btn btn-primary" />
            <a asp-action="Index" class="btn btn-secondary">Hủy</a>
        </div>
    </form>
</div>

@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}
```

### 7.4. Thêm Validation Scripts

Thêm vào `Views/Shared/_Layout.cshtml` (nếu chưa có):

```html
<script src="~/lib/jquery-validation/dist/jquery.validate.min.js"></script>
<script src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js"></script>
```

Hoặc tạo `Views/Shared/_ValidationScriptsPartial.cshtml`:

```html
<script src="~/lib/jquery-validation/dist/jquery.validate.min.js"></script>
<script src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js"></script>
```

---

## 🧪 BƯỚC 8: TEST ỨNG DỤNG

### 8.1. Test các trang

1. **Chạy ứng dụng** (F5)
2. **Truy cập các URL:**
   - `/` → Trang Home
   - `/About` → Trang About
   - `/About/Team` → Trang Team
   - `/Products` → Danh sách sản phẩm
   - `/Products/Details/1` → Chi tiết sản phẩm
   - `/Products/Create` → Form thêm sản phẩm

### 8.2. Test Routing

Thử các URL sau và quan sát kết quả:

```
/                           → HomeController.Index()
/Home                       → HomeController.Index()
/Home/Privacy               → HomeController.Privacy()
/About                      → AboutController.Index()
/About/Team                 → AboutController.Team()
/Products                   → ProductsController.Index()
/Products/Details/1         → ProductsController.Details(1)
/Products/Create            → ProductsController.Create() [GET]
```

### 8.3. Test Form

1. Truy cập `/Products/Create`
2. Thử submit form:
   - **Không nhập gì** → Xem validation errors
   - **Nhập đầy đủ** → Sản phẩm được thêm vào danh sách

---

## 📝 TÓM TẮT KIẾN THỨC

### ✅ MVC Pattern:
- **Model**: Dữ liệu (Product, CompanyInfo)
- **View**: Giao diện HTML (Index.cshtml, Details.cshtml)
- **Controller**: Logic xử lý (HomeController, ProductsController)

### ✅ Routing:
- **Conventional Routing**: `{controller}/{action}/{id?}`
- **Attribute Routing**: `[Route("custom-url")]`

### ✅ Truyền dữ liệu:
- **ViewData**: Dictionary
- **ViewBag**: Dynamic object
- **Model**: Strongly-typed (khuyên dùng)

### ✅ Tag Helpers:
- `asp-controller`, `asp-action`: Tạo URL
- `asp-for`: Model binding
- `asp-validation-for`: Validation messages

---

## 🎯 BÀI TẬP MỞ RỘNG

### Bài 1: Tạo BlogController
- Tạo `BlogController` với các actions:
  - Index (danh sách bài viết)
  - Details (chi tiết bài viết)
  - Create (tạo bài viết mới)
- Tạo Model `BlogPost` (Title, Content, Author, PublishedDate)
- Tạo Views tương ứng

### Bài 2: Custom Routing
- Thêm Attribute Routing cho BlogController
- URL thân thiện: `/tin-tuc`, `/tin-tuc/chi-tiet/1`

### Bài 3: Truyền dữ liệu phức tạp
- Tạo ViewModel chứa nhiều thông tin
- Hiển thị dữ liệu trong View với vòng lặp, điều kiện

---

## ❓ CÂU HỎI THƯỜNG GẶP

### Q1: Tại sao View không tìm thấy?
**A:** Đảm bảo:
- View nằm trong `Views/{Controller}/{Action}.cshtml`
- Tên file đúng (phân biệt hoa thường)
- Controller trả về `View()` không có tham số

### Q2: Routing không hoạt động?
**A:** Kiểm tra:
- `Program.cs` có `app.MapControllerRoute()`
- Controller/Action có `public`
- URL đúng format

### Q3: Model không hiển thị trong View?
**A:** Đảm bảo:
- View có `@model` directive
- Controller truyền model vào `View(model)`
- Properties của model có `get; set;`

---

## 🎉 CHÚC MỪNG!

Bạn đã hoàn thành bài thực hành đầu tiên! Bạn đã:

- ✅ Tạo ứng dụng web Razor từ đầu
- ✅ Hiểu MVC Pattern
- ✅ Nắm vững Routing
- ✅ Truyền dữ liệu Controller → View
- ✅ Tạo form với validation

**Tiếp theo:** Học Model Binding & Validation chi tiết hơn!

---

**File này là tài liệu tham khảo. Hãy thực hành từng bước và tự khám phá thêm!** 🚀

