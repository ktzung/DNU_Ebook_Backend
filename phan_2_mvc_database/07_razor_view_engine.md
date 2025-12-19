# 🟩 CHƯƠNG 07
# **RAZOR VIEW ENGINE & UI**

## 📖 1. Giới thiệu Razor View Engine

**Razor** là một markup syntax cho phép bạn nhúng mã C# vào trang HTML. Nó mạnh mẽ, linh hoạt và dễ học.

### Tại sao dùng Razor?
- **Dễ đọc**: Cú pháp `@` đơn giản, không cần thẻ đóng mở phức tạp.
- **IntelliSense**: Hỗ trợ nhắc lệnh C# tuyệt vời trong Visual Studio.
- **An toàn**: Tự động mã hóa HTML (HTML encoding) để chống XSS.

---

## 🛠️ 2. Cú pháp Razor cơ bản

Mọi mã Razor đều bắt đầu bằng ký tự `@`.

### 2.1. Biến và Biểu thức
```html
<!-- Hiển thị giá trị biến -->
<p>Xin chào, hôm nay là ngày: @DateTime.Now.ToString("dd/MM/yyyy")</p>

<!-- Khối code C# -->
@{
    var name = "Sinh viên DNU";
    var age = 20;
}
<p>Tên: @name, Tuổi: @age</p>
```

### 2.2. Cấu trúc điều khiển (If-Else)
```html
@{
    var isLoggedIn = true;
}

@if (isLoggedIn)
{
    <p>Chào mừng bạn quay lại!</p>
}
else
{
    <a href="/Login">Đăng nhập</a>
}
```

### 2.3. Vòng lặp (Loop)
```html
<ul>
    @for (int i = 0; i < 5; i++)
    {
        <li>Mục số @(i + 1)</li>
    }
</ul>

<!-- Duyệt danh sách -->
@foreach (var product in Model)
{
    <div class="card">
        <h3>@product.Name</h3>
        <p>@product.Price.ToString("C")</p>
    </div>
}
```

---

## 🎨 3. Layouts (Giao diện chung)

Thay vì copy menu, footer cho từng trang, ta dùng **Layout**.

### 3.1. Tạo `_Layout.cshtml`
File này thường nằm trong thư mục `Views/Shared`.

```html
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"] - EShop</title>
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    <header>
        <nav>
            <a href="/">Trang chủ</a>
            <a href="/Products">Sản phẩm</a>
        </nav>
    </header>

    <div class="container">
        <!-- Nội dung của từng trang con sẽ hiển thị ở đây -->
        @RenderBody()
    </div>

    <footer>
        <p>&copy; 2024 - DNU EShop</p>
    </footer>
</body>
</html>
```

### 3.2. Sử dụng Layout trong View
```html
@{
    Layout = "_Layout";
    ViewData["Title"] = "Danh sách sản phẩm";
}

<h1>Sản phẩm mới nhất</h1>
<!-- Nội dung này sẽ chui vào chỗ @RenderBody() -->
```

### 3.3. `_ViewStart.cshtml`
File này tự động chạy trước mỗi View. Dùng để set Layout mặc định.
```csharp
@{
    Layout = "_Layout";
}
```

---

## 🏷️ 4. Tag Helpers

Tag Helpers giúp viết code HTML giống như HTML chuẩn nhưng có sức mạnh của C#.

### 4.1. Anchor Tag Helper (Link)
Thay vì `href="/Products/Details/5"`, ta viết:

```html
<a asp-controller="Products" asp-action="Details" asp-route-id="5">Xem chi tiết</a>
```
- **Lợi ích**: Nếu bạn đổi cấu trúc Route, link này tự động cập nhật theo.

### 4.2. Form Tag Helpers
```html
<form asp-controller="Account" asp-action="Login" method="post">
    <div class="form-group">
        <label asp-for="Email"></label>
        <input asp-for="Email" class="form-control" />
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>
    <button type="submit">Đăng nhập</button>
</form>
```
- `asp-for`: Tự động sinh `id`, `name`, `value` và `type` dựa trên thuộc tính của Model.

---

## 🧩 5. Partial Views

Dùng để tách nhỏ giao diện thành các thành phần tái sử dụng (ví dụ: thẻ sản phẩm, menu).

### 5.1. Tạo `_ProductCard.cshtml`
```html
@model Product

<div class="card">
    <img src="@Model.ImageUrl" alt="@Model.Name" />
    <div class="card-body">
        <h5>@Model.Name</h5>
        <p>@Model.Price</p>
        <a asp-controller="Cart" asp-action="Add" asp-route-id="@Model.Id">Mua ngay</a>
    </div>
</div>
```

### 5.2. Sử dụng
```html
@foreach (var item in Model)
{
    <partial name="_ProductCard" model="item" />
}
```

---

## 🧪 6. Bài tập thực hành

### Bài 1: Tạo Layout cho E-Shop
1. Tạo file `_Layout.cshtml` với Header (Logo, Menu) và Footer.
2. Tích hợp Bootstrap 5 vào Layout (dùng CDN).
3. Tạo trang chủ (`Home/Index.cshtml`) sử dụng Layout này.

### Bài 2: Hiển thị danh sách sản phẩm
1. Tạo `ProductController` với action `Index`.
2. Tạo danh sách giả (Mock data) gồm 5 sản phẩm.
3. Truyền danh sách sang View.
4. Trong View, dùng `@foreach` để hiển thị sản phẩm dưới dạng lưới (Grid) dùng Bootstrap Card.

### Bài 3: Form liên hệ
1. Tạo Model `ContactViewModel` (Name, Email, Message).
2. Tạo View `Contact.cshtml` dùng Tag Helpers để sinh form.
3. Xử lý submit form và hiển thị thông báo "Cảm ơn".

---

## ❌ 7. CÁC LỖI THƯỜNG GẶP

### ❌ Lỗi 1: View không tìm thấy

```csharp
// ❌ Vấn đề: View không tồn tại hoặc sai tên
public IActionResult Index()
{
    return View("NonExistentView"); // View không tìm thấy
}

// ✅ Giải pháp: Đảm bảo View tồn tại đúng convention
// Views/Home/Index.cshtml phải tồn tại
public IActionResult Index()
{
    return View(); // Tự động tìm Views/Home/Index.cshtml
}
```

**🔍 Giải thích:** ASP.NET Core tìm View theo convention: `Views/{Controller}/{Action}.cshtml`. Đảm bảo file tồn tại.

---

### ❌ Lỗi 2: Model null trong View

```html
<!-- ❌ Vấn đề: Model null nhưng vẫn truy cập -->
@model Product
<h1>@Model.Name</h1> <!-- NullReferenceException nếu Model null -->

<!-- ✅ Giải pháp: Kiểm tra null -->
@model Product?
@if (Model != null)
{
    <h1>@Model.Name</h1>
}
```

**🔍 Giải thích:** Luôn kiểm tra Model null trước khi truy cập properties.

---

### ❌ Lỗi 3: Quên @ trong Razor syntax

```html
<!-- ❌ Vấn đề: Quên @ -->
model Product
var name = "Test";

<!-- ✅ Giải pháp: Luôn dùng @ -->
@model Product
@{
    var name = "Test";
}
```

**🔍 Giải thích:** Razor cần `@` để phân biệt C# code và HTML.

---

### ❌ Lỗi 4: XSS Attack (Cross-Site Scripting)

```html
<!-- ❌ Vấn đề: Hiển thị user input không encode -->
<p>@Html.Raw(userInput)</p> <!-- Nguy hiểm! -->

<!-- ✅ Giải pháp: Razor tự động encode -->
<p>@userInput</p> <!-- ✅ Tự động encode HTML -->
```

**🔍 Giải thích:** Razor tự động encode HTML để chống XSS. Chỉ dùng `Html.Raw()` khi thực sự cần.

---

### ❌ Lỗi 5: Layout không được apply

```html
<!-- ❌ Vấn đề: Quên set Layout -->
@{
    // Không có Layout = View không dùng Layout
}

<!-- ✅ Giải pháp: Set Layout -->
@{
    Layout = "_Layout";
}
```

**🔍 Giải thích:** Phải set Layout nếu muốn dùng. Hoặc dùng `_ViewStart.cshtml` để set mặc định.

---

## 🎯 8. CASE STUDY / VÍ DỤ THỰC TẾ

### Case Study 1: E-Shop Layout hoàn chỉnh

**Yêu cầu:** Tạo Layout với Header, Navigation, Footer, và responsive design.

```html
<!-- Views/Shared/_Layout.cshtml -->
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - EShop</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet" />
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
            <div class="container">
                <a class="navbar-brand" asp-controller="Home" asp-action="Index">EShop</a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="collapse navbar-collapse" id="navbarNav">
                    <ul class="navbar-nav me-auto">
                        <li class="nav-item">
                            <a class="nav-link" asp-controller="Home" asp-action="Index">Trang chủ</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" asp-controller="Products" asp-action="Index">Sản phẩm</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" asp-controller="Cart" asp-action="Index">Giỏ hàng</a>
                        </li>
                    </ul>
                    <partial name="_LoginPartial" />
                </div>
            </div>
        </nav>
    </header>

    <main class="container my-4">
        @if (TempData["SuccessMessage"] != null)
        {
            <div class="alert alert-success alert-dismissible fade show" role="alert">
                @TempData["SuccessMessage"]
                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
            </div>
        }
        
        @if (TempData["ErrorMessage"] != null)
        {
            <div class="alert alert-danger alert-dismissible fade show" role="alert">
                @TempData["ErrorMessage"]
                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
            </div>
        }

        @RenderBody()
    </main>

    <footer class="bg-light py-4 mt-5">
        <div class="container text-center">
            <p>&copy; 2024 - EShop. All rights reserved.</p>
        </div>
    </footer>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

---

### Case Study 2: Product List View với Pagination

**Yêu cầu:** Hiển thị danh sách sản phẩm với pagination và search.

```html
<!-- Views/Products/Index.cshtml -->
@model ProductListViewModel
@{
    ViewData["Title"] = "Sản phẩm";
}

<div class="row mb-4">
    <div class="col-md-6">
        <h2>Sản phẩm</h2>
    </div>
    <div class="col-md-6">
        <form method="get" class="d-flex">
            <input class="form-control me-2" type="search" name="search" 
                   placeholder="Tìm kiếm..." value="@Model.SearchTerm" />
            <button class="btn btn-outline-success" type="submit">Tìm</button>
        </form>
    </div>
</div>

@if (Model.Products.Any())
{
    <div class="row">
        @foreach (var product in Model.Products)
        {
            <div class="col-md-4 mb-4">
                <div class="card h-100">
                    <img src="@product.ImageUrl" class="card-img-top" alt="@product.Name" />
                    <div class="card-body">
                        <h5 class="card-title">@product.Name</h5>
                        <p class="card-text">@product.Description</p>
                        <p class="text-primary fw-bold">@product.Price.ToString("C")</p>
                        <a asp-controller="Products" asp-action="Details" asp-route-id="@product.Id" 
                           class="btn btn-primary">Xem chi tiết</a>
                    </div>
                </div>
            </div>
        }
    </div>

    <!-- Pagination -->
    <nav aria-label="Page navigation">
        <ul class="pagination justify-content-center">
            @for (int i = 1; i <= Model.TotalPages; i++)
            {
                <li class="page-item @(i == Model.CurrentPage ? "active" : "")">
                    <a class="page-link" 
                       asp-controller="Products" 
                       asp-action="Index" 
                       asp-route-page="@i"
                       asp-route-search="@Model.SearchTerm">@i</a>
                </li>
            }
        </ul>
    </nav>
}
else
{
    <div class="alert alert-info">
        Không tìm thấy sản phẩm nào.
    </div>
}
```

---

## ✅ 9. BEST PRACTICES

### 9.1. Razor Best Practices

```html
<!-- ✅ Đúng: Strongly-typed model -->
@model Product
<h1>@Model.Name</h1>

<!-- ✅ Đúng: Kiểm tra null -->
@model Product?
@if (Model != null)
{
    <h1>@Model.Name</h1>
}

<!-- ✅ Đúng: Dùng Tag Helpers -->
<a asp-controller="Products" asp-action="Details" asp-route-id="@product.Id">Chi tiết</a>

<!-- ❌ Sai: Hard-code URL -->
<a href="/Products/Details/@product.Id">Chi tiết</a> <!-- ❌ -->
```

### 9.2. Layout Best Practices

```html
<!-- ✅ Đúng: _ViewStart.cshtml cho Layout mặc định -->
@{
    Layout = "_Layout";
}

<!-- ✅ Đúng: Sections cho scripts -->
@section Scripts {
    <script src="~/js/custom.js"></script>
}
```

### 9.3. Partial Views Best Practices

```html
<!-- ✅ Đúng: Partial view tái sử dụng -->
<partial name="_ProductCard" model="product" />

<!-- ✅ Đúng: Partial với ViewData -->
<partial name="_ProductCard" model="product" view-data="ViewData" />
```

---

# 📝 10. QUICK NOTES

### Razor Syntax:
- `@model Type`: Strongly-typed model
- `@{ }`: Code block
- `@variable`: Hiển thị biến
- `@if`, `@foreach`, `@for`: Control structures
- `@Html.Raw()`: Raw HTML (cẩn thận XSS)

### Tag Helpers:
- `asp-controller`, `asp-action`: Routing
- `asp-route-*`: Route parameters
- `asp-for`: Model binding
- `asp-validation-for`: Validation messages

### Layout:
- `_Layout.cshtml`: Main layout
- `_ViewStart.cshtml`: Set default layout
- `@RenderBody()`: Render view content
- `@RenderSection()`: Render section

### Best Practices:
- ✅ Strongly-typed models
- ✅ Tag Helpers thay vì HTML thuần
- ✅ Partial views cho reusable components
- ✅ Layout cho consistency
- ✅ Sections cho scripts/styles

---

# 🧪 11. MINI TEST

1. **Razor syntax bắt đầu bằng ký tự gì?**
   - A. #
   - B. @ ✅
   - C. $
   - D. %

2. **Tag Helper nào dùng để tạo link?**
   - A. `<a href>`
   - B. `<a asp-controller asp-action>` ✅
   - C. `<link>`
   - D. `<url>`

3. **Partial View thường bắt đầu bằng ký tự gì?**
   - A. #
   - B. _ ✅
   - C. $
   - D. Không có quy tắc

<details>
<summary>💡 Đáp án</summary>

1. **B** - Razor syntax bắt đầu bằng `@`
2. **B** - Tag Helpers: `asp-controller`, `asp-action`
3. **B** - Partial View thường bắt đầu bằng `_` (ví dụ: `_ProductCard.cshtml`)
</details>

---

# 📌 12. TÓM TẮT CHƯƠNG

✅ **Razor** là markup syntax nhúng C# vào HTML  
✅ **Layout** tái sử dụng giao diện chung  
✅ **Partial Views** cho components tái sử dụng  
✅ **Tag Helpers** tạo HTML với IntelliSense  
✅ **Sections** cho scripts/styles  
✅ **ViewData/ViewBag** truyền dữ liệu phụ  

---

**Chương tiếp theo: [08. RESTful Web API →](../phan_3_web_api_security/08_restful_web_api.md)**
