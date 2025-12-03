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

## 💡 Mẹo nhỏ
> [!TIP]
> Sử dụng `ViewData["Title"]` để thay đổi tiêu đề tab trình duyệt cho từng trang.

> [!NOTE]
> Luôn đặt file Partial View bắt đầu bằng dấu gạch dưới `_` (ví dụ: `_LoginPartial.cshtml`) để phân biệt với View thường.
