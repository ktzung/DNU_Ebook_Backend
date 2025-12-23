# 🟩 CHƯƠNG 14
# **LAYOUT, PARTIAL VIEWS & TÁI SỬ DỤNG BỐ CỤC**

> **Mục tiêu:** Hiểu cách phân chia, kế thừa và tái sử dụng bố cục trang web trong ASP.NET Core MVC

---

## 🎯 MỤC TIÊU HỌC TẬP

Sau chương này, bạn sẽ:

- ✅ Hiểu **Layout** là gì và tại sao cần dùng
- ✅ Nắm vững **nguyên tắc mặc định** (_ViewStart, _ViewImports)
- ✅ Biết cách **tùy chỉnh Layout** cho từng trang
- ✅ Sử dụng **Partial Views** để tái sử dụng code
- ✅ Dùng **Sections** để inject content vào Layout
- ✅ Hiểu **View Components** (nâng cao)
- ✅ Áp dụng best practices cho dự án thực tế

---

## 🏠 1. LAYOUT LÀ GÌ? (DỄ - KHÁI NIỆM CƠ BẢN)

### 1.1. Vấn đề không dùng Layout

**Kịch bản:** Bạn có 10 trang web, mỗi trang cần:
- Header (Logo, Menu)
- Footer (Copyright, Links)
- CSS, JavaScript files

**Không dùng Layout:**

```html
<!-- Views/Home/Index.cshtml -->
<!DOCTYPE html>
<html>
<head>
    <title>Home</title>
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    <header>
        <nav>Menu...</nav>
    </header>
    <main>
        <h1>Home Page</h1>
    </main>
    <footer>
        <p>Copyright 2024</p>
    </footer>
    <script src="~/js/site.js"></script>
</body>
</html>

<!-- Views/Products/Index.cshtml -->
<!DOCTYPE html>
<html>
<head>
    <title>Products</title>
    <link rel="stylesheet" href="~/css/site.css" /> <!-- ❌ Lặp lại -->
</head>
<body>
    <header>
        <nav>Menu...</nav> <!-- ❌ Lặp lại -->
    </header>
    <main>
        <h1>Products</h1>
    </main>
    <footer>
        <p>Copyright 2024</p> <!-- ❌ Lặp lại -->
    </footer>
    <script src="~/js/site.js"></script> <!-- ❌ Lặp lại -->
</body>
</html>
```

**Vấn đề:**
- ❌ Code lặp lại nhiều lần
- ❌ Khó maintain (sửa menu phải sửa 10 file)
- ❌ Dễ lỗi (quên thêm CSS vào 1 trang)
- ❌ Không nhất quán

### 1.2. Giải pháp: Dùng Layout

**Layout = Template chung** cho tất cả các trang

```
┌─────────────────────────────────────┐
│         _Layout.cshtml              │
│  ┌───────────────────────────────┐  │
│  │ Header (Logo, Menu)           │  │
│  ├───────────────────────────────┤  │
│  │                               │  │
│  │  @RenderBody()                │  │ ← Nội dung từng trang
│  │  (Views/Home/Index.cshtml)    │  │
│  │                               │  │
│  ├───────────────────────────────┤  │
│  │ Footer (Copyright, Links)     │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

**Với Layout:**

```html
<!-- Views/Shared/_Layout.cshtml -->
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"] - MyApp</title>
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    <header>
        <nav>Menu...</nav>
    </header>
    <main>
        @RenderBody() <!-- Nội dung từng trang -->
    </main>
    <footer>
        <p>Copyright 2024</p>
    </footer>
    <script src="~/js/site.js"></script>
</body>
</html>

<!-- Views/Home/Index.cshtml -->
@{
    ViewData["Title"] = "Home";
}
<h1>Home Page</h1>

<!-- Views/Products/Index.cshtml -->
@{
    ViewData["Title"] = "Products";
}
<h1>Products</h1>
```

**Lợi ích:**
- ✅ Code không lặp lại
- ✅ Dễ maintain (sửa 1 lần, áp dụng tất cả)
- ✅ Nhất quán (tất cả trang giống nhau)
- ✅ Tách biệt logic (Layout vs Content)

---

## 🎨 2. LAYOUT MẶC ĐỊNH - _ViewStart.cshtml (DỄ)

### 2.1. _ViewStart.cshtml là gì?

**Convention:** File `_ViewStart.cshtml` chạy **trước mỗi View**

**Vị trí:**
```
Views/
├── _ViewStart.cshtml    ← Chạy trước mỗi View
├── Home/
│   └── Index.cshtml    ← Chạy sau _ViewStart
└── Products/
    └── Index.cshtml     ← Chạy sau _ViewStart
```

**Nội dung mặc định:**

```csharp
// Views/_ViewStart.cshtml
@{
    Layout = "_Layout"; // Set Layout mặc định
}
```

**Giải thích:**
- Chạy **tự động** trước mỗi View
- Set Layout mặc định = `_Layout.cshtml`
- Tất cả Views sẽ dùng Layout này (trừ khi override)

### 2.2. Luồng hoạt động

```
1. Controller.Index() gọi View()
   ↓
2. View Engine tìm Views/Home/Index.cshtml
   ↓
3. Chạy _ViewStart.cshtml (set Layout = "_Layout")
   ↓
4. Render Views/Home/Index.cshtml
   ↓
5. Wrap vào Views/Shared/_Layout.cshtml (@RenderBody())
   ↓
6. HTML cuối cùng gửi về Browser
```

### 2.3. Ví dụ minh họa

**Bước 1: Tạo _Layout.cshtml**

```html
<!-- Views/Shared/_Layout.cshtml -->
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - MyApp</title>
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    <header>
        <nav class="navbar">
            <div class="container">
                <a class="navbar-brand" asp-controller="Home" asp-action="Index">MyApp</a>
                <ul class="navbar-nav">
                    <li><a asp-controller="Home" asp-action="Index">Trang chủ</a></li>
                    <li><a asp-controller="Products" asp-action="Index">Sản phẩm</a></li>
                    <li><a asp-controller="About" asp-action="Index">Giới thiệu</a></li>
                </ul>
            </div>
        </nav>
    </header>

    <main class="container">
        @RenderBody() <!-- Nội dung từng trang -->
    </main>

    <footer class="footer">
        <div class="container">
            <p>&copy; 2024 - MyApp. All rights reserved.</p>
        </div>
    </footer>

    <script src="~/js/site.js"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

**Bước 2: Tạo _ViewStart.cshtml**

```csharp
// Views/_ViewStart.cshtml
@{
    Layout = "_Layout"; // Layout mặc định
}
```

**Bước 3: Tạo View**

```html
<!-- Views/Home/Index.cshtml -->
@{
    ViewData["Title"] = "Trang chủ";
}

<h1>Chào mừng đến với MyApp</h1>
<p>Đây là trang chủ của ứng dụng.</p>
```

**Kết quả:**
- View `Index.cshtml` tự động dùng `_Layout.cshtml`
- Header, Footer hiển thị tự động
- Chỉ cần viết nội dung trang

---

## 🔧 3. TÙY CHỈNH LAYOUT (TRUNG BÌNH)

### 3.1. Override Layout trong View

**Convention:** View có thể override Layout từ `_ViewStart.cshtml`

```html
<!-- Views/Home/Index.cshtml -->
@{
    Layout = "_CustomLayout"; // Override Layout mặc định
    ViewData["Title"] = "Home";
}

<h1>Home Page</h1>
```

**Ví dụ: Trang Admin dùng Layout khác**

```html
<!-- Views/Admin/Dashboard.cshtml -->
@{
    Layout = "_AdminLayout"; // Layout riêng cho Admin
    ViewData["Title"] = "Admin Dashboard";
}

<h1>Admin Dashboard</h1>
```

### 3.2. Không dùng Layout (Layout = null)

**Khi nào cần?**
- Trang Login (không có Header/Footer)
- Trang Error (minimal design)
- Email templates

```html
<!-- Views/Account/Login.cshtml -->
@{
    Layout = null; // Không dùng Layout
    ViewData["Title"] = "Đăng nhập";
}

<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"]</title>
    <link rel="stylesheet" href="~/css/login.css" />
</head>
<body>
    <div class="login-container">
        <h1>Đăng nhập</h1>
        <!-- Form login -->
    </div>
</body>
</html>
```

### 3.3. Nhiều Layout cho các mục đích khác nhau

**Cấu trúc:**

```
Views/
├── Shared/
│   ├── _Layout.cshtml          ← Layout chính (User)
│   ├── _AdminLayout.cshtml     ← Layout Admin
│   ├── _AuthLayout.cshtml      ← Layout Login/Register
│   └── _PrintLayout.cshtml     ← Layout in ấn
```

**Ví dụ: _AdminLayout.cshtml**

```html
<!-- Views/Shared/_AdminLayout.cshtml -->
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"] - Admin</title>
    <link rel="stylesheet" href="~/css/admin.css" />
</head>
<body>
    <div class="admin-container">
        <aside class="sidebar">
            <h2>Admin Menu</h2>
            <ul>
                <li><a asp-controller="Admin" asp-action="Dashboard">Dashboard</a></li>
                <li><a asp-controller="Admin" asp-action="Products">Products</a></li>
                <li><a asp-controller="Admin" asp-action="Orders">Orders</a></li>
            </ul>
        </aside>
        <main class="admin-content">
            @RenderBody()
        </main>
    </div>
</body>
</html>
```

**Sử dụng:**

```html
<!-- Views/Admin/Dashboard.cshtml -->
@{
    Layout = "_AdminLayout"; // Dùng Layout Admin
    ViewData["Title"] = "Dashboard";
}

<h1>Admin Dashboard</h1>
```

---

## 🧩 4. PARTIAL VIEWS - TÁI SỬ DỤNG CODE (TRUNG BÌNH)

### 4.1. Partial View là gì?

**Partial View** = Một phần của View có thể tái sử dụng

**Ví dụ đời sống:**
- Header, Footer = Partial Views
- Product Card = Partial View (dùng nhiều lần)
- Comment Section = Partial View

### 4.2. Quy tắc đặt tên

**Convention:** Partial View thường bắt đầu bằng `_` (underscore)

```
Views/
├── Shared/
│   ├── _Header.cshtml      ← Partial View
│   ├── _Footer.cshtml      ← Partial View
│   └── _ProductCard.cshtml ← Partial View
```

**Lý do dùng `_`:**
- Phân biệt với View thông thường
- Không thể truy cập trực tiếp qua URL
- Chỉ được include vào View khác

### 4.3. Tạo Partial View

**Ví dụ: _ProductCard.cshtml**

```html
<!-- Views/Shared/_ProductCard.cshtml -->
@model Product

<div class="product-card">
    <img src="@Model.ImageUrl" alt="@Model.Name" />
    <h3>@Model.Name</h3>
    <p class="price">@Model.Price.ToString("N0") đ</p>
    <a asp-controller="Products" asp-action="Details" asp-route-id="@Model.Id" 
       class="btn btn-primary">Xem chi tiết</a>
</div>
```

### 4.4. Sử dụng Partial View

**Cách 1: Dùng `<partial>` Tag Helper (Khuyên dùng - .NET Core 2.1+)**

```html
<!-- Views/Products/Index.cshtml -->
@model List<Product>

<h1>Danh sách sản phẩm</h1>

<div class="row">
    @foreach (var product in Model)
    {
        <div class="col-md-4">
            <partial name="_ProductCard" model="product" />
        </div>
    }
</div>
```

**Giải thích:**
- `name="_ProductCard"` = Tên Partial View (bỏ .cshtml)
- `model="product"` = Truyền model vào Partial View

**Cách 2: Dùng `@await Html.PartialAsync()`**

```html
@foreach (var product in Model)
{
    <div class="col-md-4">
        @await Html.PartialAsync("_ProductCard", product)
    </div>
}
```

**Cách 3: Dùng `Html.RenderPartial()` (Synchronous - không khuyên dùng)**

```html
@foreach (var product in Model)
{
    <div class="col-md-4">
        @{ Html.RenderPartial("_ProductCard", product); }
    </div>
}
```

### 4.5. Partial View với ViewData

**Truyền ViewData vào Partial View:**

```html
<!-- Views/Shared/_Header.cshtml -->
<header>
    <nav class="navbar">
        <div class="container">
            <a class="navbar-brand">@ViewData["AppName"]</a>
            <ul class="navbar-nav">
                @if (ViewData["IsLoggedIn"] as bool? == true)
                {
                    <li><a asp-controller="Account" asp-action="Logout">Đăng xuất</a></li>
                }
                else
                {
                    <li><a asp-controller="Account" asp-action="Login">Đăng nhập</a></li>
                }
            </ul>
        </div>
    </nav>
</header>
```

**Sử dụng trong Layout:**

```html
<!-- Views/Shared/_Layout.cshtml -->
@{
    ViewData["AppName"] = "MyApp";
    ViewData["IsLoggedIn"] = User.Identity.IsAuthenticated;
}

<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"]</title>
</head>
<body>
    <partial name="_Header" view-data="ViewData" />
    
    <main>
        @RenderBody()
    </main>
    
    <partial name="_Footer" />
</body>
</html>
```

### 4.6. Ví dụ thực tế: Header và Footer

**Tạo _Header.cshtml:**

```html
<!-- Views/Shared/_Header.cshtml -->
<header class="site-header">
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
        <div class="container">
            <a class="navbar-brand" asp-controller="Home" asp-action="Index">
                <img src="~/images/logo.png" alt="Logo" />
                MyApp
            </a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" 
                    data-bs-target="#navbarNav">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav ms-auto">
                    <li class="nav-item">
                        <a class="nav-link" asp-controller="Home" asp-action="Index">Trang chủ</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" asp-controller="Products" asp-action="Index">Sản phẩm</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" asp-controller="About" asp-action="Index">Giới thiệu</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" asp-controller="Contact" asp-action="Index">Liên hệ</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
</header>
```

**Tạo _Footer.cshtml:**

```html
<!-- Views/Shared/_Footer.cshtml -->
<footer class="site-footer bg-dark text-white mt-5">
    <div class="container py-4">
        <div class="row">
            <div class="col-md-4">
                <h5>Về chúng tôi</h5>
                <p>MyApp là ứng dụng thương mại điện tử hiện đại.</p>
            </div>
            <div class="col-md-4">
                <h5>Liên kết nhanh</h5>
                <ul class="list-unstyled">
                    <li><a asp-controller="Home" asp-action="Index" class="text-white">Trang chủ</a></li>
                    <li><a asp-controller="Products" asp-action="Index" class="text-white">Sản phẩm</a></li>
                    <li><a asp-controller="About" asp-action="Index" class="text-white">Giới thiệu</a></li>
                </ul>
            </div>
            <div class="col-md-4">
                <h5>Liên hệ</h5>
                <p>Email: contact@myapp.com</p>
                <p>Phone: 0123-456-789</p>
            </div>
        </div>
        <hr class="bg-white" />
        <div class="text-center">
            <p>&copy; 2024 - MyApp. All rights reserved.</p>
        </div>
    </div>
</footer>
```

**Sử dụng trong _Layout.cshtml:**

```html
<!-- Views/Shared/_Layout.cshtml -->
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - MyApp</title>
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    <partial name="_Header" />
    
    <main class="container my-4">
        @RenderBody()
    </main>
    
    <partial name="_Footer" />
    
    <script src="~/js/site.js"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

**Lợi ích:**
- ✅ Header/Footer chỉ viết 1 lần
- ✅ Dễ maintain (sửa 1 file, áp dụng tất cả)
- ✅ Code gọn gàng, dễ đọc

---

## 📦 5. SECTIONS - INJECT CONTENT VÀO LAYOUT (TRUNG BÌNH)

### 5.1. Section là gì?

**Section** = Vùng trong Layout có thể được View điền nội dung

**Ví dụ đời sống:**
- Layout có vùng "Scripts" → View có thể thêm JavaScript
- Layout có vùng "Sidebar" → View có thể thêm menu phụ

### 5.2. Định nghĩa Section trong Layout

```html
<!-- Views/Shared/_Layout.cshtml -->
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"]</title>
    <link rel="stylesheet" href="~/css/site.css" />
    @await RenderSectionAsync("Styles", required: false) <!-- Section: Styles -->
</head>
<body>
    <header>
        <partial name="_Header" />
    </header>
    
    <main>
        @RenderBody()
    </main>
    
    <aside>
        @await RenderSectionAsync("Sidebar", required: false) <!-- Section: Sidebar -->
    </aside>
    
    <footer>
        <partial name="_Footer" />
    </footer>
    
    <script src="~/js/site.js"></script>
    @await RenderSectionAsync("Scripts", required: false) <!-- Section: Scripts -->
</body>
</html>
```

**Giải thích:**
- `@await RenderSectionAsync("Scripts", required: false)`
  - `"Scripts"` = Tên section
  - `required: false` = Section không bắt buộc (View có thể không có)
  - `required: true` = Section bắt buộc (View phải có)

### 5.3. Điền nội dung vào Section trong View

```html
<!-- Views/Products/Index.cshtml -->
@{
    ViewData["Title"] = "Sản phẩm";
}

<h1>Danh sách sản phẩm</h1>
<!-- Nội dung chính -->

@section Scripts {
    <script>
        // JavaScript chỉ cho trang Products
        console.log("Products page loaded");
    </script>
}

@section Styles {
    <style>
        /* CSS chỉ cho trang Products */
        .product-card {
            border: 1px solid #ddd;
        }
    </style>
}
```

**Kết quả:**
- JavaScript trong `@section Scripts` được inject vào cuối Layout
- CSS trong `@section Styles` được inject vào `<head>`

### 5.4. Section bắt buộc vs Tùy chọn

**Section bắt buộc:**

```html
<!-- Layout -->
@await RenderSectionAsync("RequiredSection", required: true)

<!-- View PHẢI có section này -->
@section RequiredSection {
    <p>Nội dung bắt buộc</p>
}
```

**Section tùy chọn:**

```html
<!-- Layout -->
@await RenderSectionAsync("OptionalSection", required: false)

<!-- View có thể có hoặc không -->
@section OptionalSection {
    <p>Nội dung tùy chọn</p>
}
```

### 5.5. Ví dụ thực tế: Section Scripts

**Layout:**

```html
<!-- Views/Shared/_Layout.cshtml -->
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"]</title>
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    @RenderBody()
    
    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/site.js"></script>
    
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

**View với Script riêng:**

```html
<!-- Views/Products/Create.cshtml -->
@model Product

<h1>Thêm sản phẩm mới</h1>

<form asp-action="Create" method="post">
    <!-- Form fields -->
</form>

@section Scripts {
    <script src="~/lib/jquery-validation/dist/jquery.validate.min.js"></script>
    <script src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js"></script>
    <script>
        // Custom validation cho trang này
        $(document).ready(function() {
            // Validation logic
        });
    </script>
}
```

**Kết quả HTML:**

```html
<body>
    <!-- Nội dung form -->
    
    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/site.js"></script>
    
    <!-- Scripts từ Section -->
    <script src="~/lib/jquery-validation/dist/jquery.validate.min.js"></script>
    <script src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js"></script>
    <script>
        $(document).ready(function() {
            // Validation logic
        });
    </script>
</body>
```

### 5.6. Ví dụ: Section Sidebar

**Layout với Sidebar:**

```html
<!-- Views/Shared/_Layout.cshtml -->
<div class="container">
    <div class="row">
        <main class="col-md-9">
            @RenderBody()
        </main>
        <aside class="col-md-3">
            @await RenderSectionAsync("Sidebar", required: false)
        </aside>
    </div>
</div>
```

**View với Sidebar:**

```html
<!-- Views/Products/Index.cshtml -->
<h1>Danh sách sản phẩm</h1>
<!-- Nội dung chính -->

@section Sidebar {
    <div class="sidebar">
        <h3>Bộ lọc</h3>
        <form>
            <div class="form-group">
                <label>Giá từ</label>
                <input type="number" class="form-control" />
            </div>
            <div class="form-group">
                <label>Giá đến</label>
                <input type="number" class="form-control" />
            </div>
            <button type="submit" class="btn btn-primary">Lọc</button>
        </form>
    </div>
}
```

**View không có Sidebar:**

```html
<!-- Views/Home/Index.cshtml -->
<h1>Trang chủ</h1>
<!-- Không có @section Sidebar → Sidebar không hiển thị -->
```

---

## 🔄 6. KẾ THỪA LAYOUT (KHÓ - NÂNG CAO)

### 6.1. Layout lồng nhau (Nested Layouts)

**Kịch bản:** Admin Panel có Layout riêng, nhưng vẫn kế thừa Layout chính

```
_Layout.cshtml (Layout chính)
    ↓
_AdminLayout.cshtml (Kế thừa _Layout)
    ↓
Admin/Dashboard.cshtml (Dùng _AdminLayout)
```

**Ví dụ:**

```html
<!-- Views/Shared/_Layout.cshtml -->
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"]</title>
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    <partial name="_Header" />
    <main>
        @RenderBody()
    </main>
    <partial name="_Footer" />
</body>
</html>
```

```html
<!-- Views/Shared/_AdminLayout.cshtml -->
@{
    Layout = "_Layout"; // Kế thừa Layout chính
}

<div class="admin-wrapper">
    <aside class="admin-sidebar">
        <h2>Admin Menu</h2>
        <ul>
            <li><a asp-controller="Admin" asp-action="Dashboard">Dashboard</a></li>
            <li><a asp-controller="Admin" asp-action="Products">Products</a></li>
        </ul>
    </aside>
    <main class="admin-content">
        @RenderBody() <!-- Nội dung từ Admin Views -->
    </main>
</div>
```

```html
<!-- Views/Admin/Dashboard.cshtml -->
@{
    Layout = "_AdminLayout"; // Dùng Admin Layout
    ViewData["Title"] = "Admin Dashboard";
}

<h1>Dashboard</h1>
```

**Luồng render:**

```
1. Admin/Dashboard.cshtml
   ↓
2. _AdminLayout.cshtml (kế thừa _Layout)
   ↓
3. _Layout.cshtml
   ↓
4. HTML cuối cùng
```

### 6.2. Override Section trong Layout con

**Layout cha:**

```html
<!-- Views/Shared/_Layout.cshtml -->
<head>
    <link rel="stylesheet" href="~/css/site.css" />
    @await RenderSectionAsync("Styles", required: false)
</head>
```

**Layout con:**

```html
<!-- Views/Shared/_AdminLayout.cshtml -->
@{
    Layout = "_Layout";
}

@section Styles {
    <link rel="stylesheet" href="~/css/admin.css" />
    @await RenderSectionAsync("Styles", required: false) <!-- Cho phép View override -->
}

<div class="admin-wrapper">
    @RenderBody()
</div>
```

**View:**

```html
<!-- Views/Admin/Dashboard.cshtml -->
@{
    Layout = "_AdminLayout";
}

@section Styles {
    <link rel="stylesheet" href="~/css/dashboard.css" />
}

<h1>Dashboard</h1>
```

**Kết quả:** CSS được load theo thứ tự:
1. `site.css` (từ _Layout)
2. `admin.css` (từ _AdminLayout)
3. `dashboard.css` (từ Dashboard View)

---

## 🧪 7. VIEW COMPONENTS - TÁI SỬ DỤNG NÂNG CAO (KHÓ)

### 7.1. View Component là gì?

**View Component** = Partial View nâng cao, có logic phức tạp

**So sánh:**

| Feature | Partial View | View Component |
|---------|-------------|----------------|
| **Logic** | Không có | Có thể có logic phức tạp |
| **Dependency Injection** | Không | Có |
| **Async** | Không | Có |
| **Test** | Khó | Dễ test |

### 7.2. Khi nào dùng View Component?

**Dùng Partial View khi:**
- ✅ Hiển thị dữ liệu đơn giản
- ✅ Không cần logic phức tạp
- ✅ Chỉ cần render HTML

**Dùng View Component khi:**
- ✅ Cần logic phức tạp (query database, tính toán)
- ✅ Cần Dependency Injection
- ✅ Cần async operations
- ✅ Cần test riêng

### 7.3. Tạo View Component

**Bước 1: Tạo ViewComponent class**

```csharp
// ViewComponents/ShoppingCartViewComponent.cs
using Microsoft.AspNetCore.Mvc;

namespace MyApp.ViewComponents
{
    public class ShoppingCartViewComponent : ViewComponent
    {
        private readonly ICartService _cartService;

        public ShoppingCartViewComponent(ICartService cartService)
        {
            _cartService = cartService;
        }

        public async Task<IViewComponentResult> InvokeAsync()
        {
            var cart = await _cartService.GetCartAsync();
            return View(cart);
        }
    }
}
```

**Bước 2: Tạo View cho ViewComponent**

```
Views/
└── Shared/
    └── Components/
        └── ShoppingCart/
            └── Default.cshtml
```

```html
<!-- Views/Shared/Components/ShoppingCart/Default.cshtml -->
@model CartViewModel

<div class="shopping-cart">
    <h3>Giỏ hàng</h3>
    <p>Số lượng: @Model.ItemCount</p>
    <p>Tổng tiền: @Model.TotalAmount.ToString("N0") đ</p>
    <a asp-controller="Cart" asp-action="Index" class="btn btn-primary">Xem giỏ hàng</a>
</div>
```

**Bước 3: Sử dụng ViewComponent**

```html
<!-- Views/Shared/_Layout.cshtml -->
<header>
    <partial name="_Header" />
    @await Component.InvokeAsync("ShoppingCart")
</header>
```

**Hoặc dùng Tag Helper:**

```html
<vc:shopping-cart />
```

### 7.4. ViewComponent với Parameters

```csharp
// ViewComponents/ProductListViewComponent.cs
public class ProductListViewComponent : ViewComponent
{
    private readonly IProductService _productService;

    public ProductListViewComponent(IProductService productService)
    {
        _productService = productService;
    }

    public async Task<IViewComponentResult> InvokeAsync(int categoryId, int count = 5)
    {
        var products = await _productService.GetProductsByCategoryAsync(categoryId, count);
        return View(products);
    }
}
```

**Sử dụng:**

```html
@await Component.InvokeAsync("ProductList", new { categoryId = 1, count = 10 })
```

---

## 📊 8. SO SÁNH CÁC CÁCH TÁI SỬ DỤNG

### 8.1. Bảng so sánh

| Feature | Layout | Partial View | View Component | Section |
|---------|--------|--------------|----------------|---------|
| **Mục đích** | Template chung | Tái sử dụng HTML | Tái sử dụng + Logic | Inject content |
| **Logic** | Không | Không | Có | Không |
| **DI** | Không | Không | Có | Không |
| **Async** | Không | Không | Có | Không |
| **Khi nào dùng** | Header/Footer chung | Product Card, Comment | Shopping Cart, Menu động | Scripts, Styles |

### 8.2. Ví dụ tổng hợp

**Dự án E-Shop:**

```
Views/
├── Shared/
│   ├── _Layout.cshtml              ← Layout chính
│   ├── _AdminLayout.cshtml          ← Layout Admin (kế thừa _Layout)
│   ├── _Header.cshtml               ← Partial View
│   ├── _Footer.cshtml               ← Partial View
│   ├── _ProductCard.cshtml          ← Partial View
│   └── Components/
│       ├── ShoppingCart/
│       │   └── Default.cshtml       ← View Component
│       └── CategoryMenu/
│           └── Default.cshtml      ← View Component
├── Home/
│   └── Index.cshtml                 ← Dùng _Layout
└── Admin/
    └── Dashboard.cshtml             ← Dùng _AdminLayout
```

**Sử dụng:**

```html
<!-- _Layout.cshtml -->
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"]</title>
    @await RenderSectionAsync("Styles", required: false)
</head>
<body>
    <partial name="_Header" />
    @await Component.InvokeAsync("ShoppingCart")
    
    <main>
        @RenderBody()
    </main>
    
    <partial name="_Footer" />
    
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

---

## ✅ 9. BEST PRACTICES

### 9.1. Layout Best Practices

```csharp
// ✅ ĐÚNG - Dùng _ViewStart.cshtml
@{
    Layout = "_Layout";
}

// ✅ ĐÚNG - Override khi cần
@{
    Layout = "_AdminLayout";
}

// ✅ ĐÚNG - Không dùng Layout khi cần
@{
    Layout = null;
}
```

### 9.2. Partial View Best Practices

```csharp
// ✅ ĐÚNG - Đặt tên bắt đầu bằng _
_ProductCard.cshtml
_Header.cshtml

// ✅ ĐÚNG - Đặt trong Shared hoặc Controller folder
Views/Shared/_ProductCard.cshtml
Views/Products/_ProductCard.cshtml

// ✅ ĐÚNG - Dùng <partial> Tag Helper
<partial name="_ProductCard" model="product" />
```

### 9.3. Section Best Practices

```csharp
// ✅ ĐÚNG - Section tùy chọn (thường dùng)
@await RenderSectionAsync("Scripts", required: false)

// ✅ ĐÚNG - Section bắt buộc (ít dùng)
@await RenderSectionAsync("RequiredContent", required: true)

// ✅ ĐÚNG - Đặt Scripts ở cuối body
<body>
    @RenderBody()
    @await RenderSectionAsync("Scripts", required: false)
</body>
```

---

## ❌ 10. CÁC LỖI THƯỜNG GẶP

### 10.1. Layout không được apply

```csharp
// ❌ Vấn đề: Quên _ViewStart.cshtml hoặc Layout = null
@{
    // Không có Layout
}

// ✅ Giải pháp: Tạo _ViewStart.cshtml hoặc set Layout
@{
    Layout = "_Layout";
}
```

### 10.2. Partial View không tìm thấy

```csharp
// ❌ Vấn đề: Tên sai hoặc đường dẫn sai
<partial name="ProductCard" /> // Thiếu _

// ✅ Giải pháp: Đúng tên và đường dẫn
<partial name="_ProductCard" /> // Có _
// Hoặc
<partial name="~/Views/Shared/_ProductCard.cshtml" />
```

### 10.3. Section không render

```csharp
// ❌ Vấn đề: Section required nhưng View không có
@await RenderSectionAsync("RequiredSection", required: true)
// View không có @section RequiredSection → Lỗi!

// ✅ Giải pháp: Dùng required: false hoặc thêm section vào View
@await RenderSectionAsync("OptionalSection", required: false)
```

---

## 📝 11. TÓM TẮT

### ✅ Layout:
- Template chung cho tất cả trang
- `_ViewStart.cshtml` set Layout mặc định
- Có thể override hoặc không dùng Layout

### ✅ Partial Views:
- Tái sử dụng HTML
- Đặt tên bắt đầu bằng `_`
- Dùng `<partial>` Tag Helper

### ✅ Sections:
- Inject content vào Layout
- `@section Scripts` cho JavaScript
- `@section Styles` cho CSS

### ✅ View Components:
- Partial View nâng cao
- Có logic và DI
- Dùng cho component phức tạp

### ✅ Best Practices:
- ✅ Dùng _ViewStart.cshtml
- ✅ Partial View đặt tên với `_`
- ✅ Section tùy chọn (required: false)
- ✅ View Component cho logic phức tạp

---

## 🎯 BÀI TẬP THỰC HÀNH

### Bài 1: Tạo Layout hoàn chỉnh
- Tạo `_Layout.cshtml` với Header, Footer
- Tạo `_Header.cshtml` và `_Footer.cshtml` (Partial Views)
- Tạo `_ViewStart.cshtml` set Layout mặc định
- Test với nhiều trang

### Bài 2: Tạo Partial View ProductCard
- Tạo `_ProductCard.cshtml`
- Hiển thị danh sách sản phẩm dùng Partial View
- Truyền model vào Partial View

### Bài 3: Sử dụng Sections
- Thêm `@section Scripts` vào một số trang
- Thêm `@section Styles` cho CSS riêng
- Test xem Scripts/Styles được inject đúng chưa

---

**Hiểu rõ Layout, Partial Views và Sections giúp bạn code gọn gàng, dễ maintain!** 🚀

---

**Chương tiếp theo: [15. EF Core Advanced Relationships →](../Module_05_Business_Logic/01_EF_Core_Advanced_Relations.md)**

