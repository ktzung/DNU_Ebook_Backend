# 🧪 LAB 04: XÂY DỰNG ADMIN PANEL VỚI MVC
**(Tương ứng: Module 04 - Week 6-7)**

## 🎯 Mục tiêu
- **Hiểu concept MVC:** Mối quan hệ giữa Model (Dữ liệu), View (Giao diện) và Controller (Điều phối).
- **Thực hành:**
  - Xây dựng trang quản lý sản phẩm cho Admin.
  - Sử dụng Razor View Engine để trộn code C# vào HTML.
  - Hiểu cách dữ liệu di chuyển từ Form -> Controller -> Database.

---

## 1. 🔍 GIẢI MÃ MVC PATTERN

### 1.1. Ví dụ Nhà Hàng
*   **Controller (Bồi bàn):** Nhận yêu cầu của khách (URL), chạy xuống bếp báo đầu bếp làm (Gọi Service/DB), rồi mang món ăn ra cho khách (Trả về View).
*   **Model (Nguyên liệu/Món ăn):** Dữ liệu cần thiết (Product, User...).
*   **View (Bộ bát đĩa/Cách trình bày):** Giao diện HTML hiển thị cho người dùng.

### 1.2. Luồng đi của dữ liệu
1.  User truy cập `/Admin/Products`.
2.  **Controller** nhận request -> Gọi DB lấy `List<Product>`.
3.  **Controller** nhét list này vào một cái "hộp" (Model) và gửi sang **View**.
4.  **View** (`Index.cshtml`) mở hộp ra, dùng vòng lặp `@foreach` để in từng sản phẩm ra bảng HTML.

---

## 2. 💻 HƯỚNG DẪN THỰC HÀNH CHI TIẾT

### Bước 1: Kích hoạt MVC Service

Trong `Program.cs`, chúng ta cần bật tính năng MVC (vì mặc định template WebAPI chỉ có API).

```csharp
// Thay đổi từ AddControllers() -> AddControllersWithViews()
builder.Services.AddControllersWithViews(); 

// ...

// Cấu hình Route mặc định cho MVC
// Pattern: {controller}/{action}/{id}
// VD: /Home/Index -> Chạy HomeControler.Index()
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

### Bước 2: Tạo Layout (Khung sườn chung)

Giống như việc mọi trang trong quyển sách đều có tiêu đề và số trang. Web cũng cần Header/Footer chung.

1.  Tạo thư mục `Views/Shared`.
2.  Tạo file `_Layout.cshtml`.

**Views/Shared/_Layout.cshtml**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - EShop Admin</title>
    
    <!-- Bootstrap CDN: Thư viện CSS giúp web đẹp ngay lập tức -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <!-- Navbar (Thanh điều hướng trên cùng) -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="/">EShop Admin</a>
            <div class="collapse navbar-collapse">
                <ul class="navbar-nav me-auto">
                    <li class="nav-item">
                        <a class="nav-link" href="/Admin/Products">Quản lý Sản phẩm</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>

    <!-- Container chính: Nội dung các trang con sẽ được "rót" vào đây -->
    <div class="container mt-4">
        @RenderBody() 
    </div>

    <footer class="border-top footer text-muted mt-5 py-3 text-center">
        &copy; 2025 - EShop Backend Course
    </footer>
    
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

### Bước 3: ProductsController (Phiên bản MVC)

Tạo `Controllers/ProductsMvcController.cs`.
*Lưu ý: Ta đặt tên khác ProductsController (API) để tránh nhầm lẫn.*

```csharp
using EShop.API.Data;
using EShop.API.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

// Định tuyến URL bắt đầu bằng /Admin/Products
[Route("Admin/Products")]
public class ProductsMvcController : Controller
{
    private readonly AppDbContext _context;

    public ProductsMvcController(AppDbContext context)
    {
        _context = context;
    }

    // 1. TRANG DANH SÁCH (Index)
    // URL: /Admin/Products
    [HttpGet("")] 
    public async Task<IActionResult> Index()
    {
        // Lấy dữ liệu từ DB
        var products = await _context.Products.Include(p => p.Category).ToListAsync();
        
        // Trả về View cùng với dữ liệu (Model)
        return View(products); 
    }

    // 2. TRANG TẠO MỚI (Create - GET)
    // URL: /Admin/Products/Create
    // Nhiệm vụ: Chỉ đơn giản là hiển thị cái Form trống
    [HttpGet("Create")]
    public IActionResult Create()
    {
        // Để hiển thị Dropdown chọn danh mục, ta cần list danh mục
        // ViewBag là một "túi chứa đồ" tạm thời để chuyển data sang View
        ViewBag.Categories = _context.Categories.ToList();
        
        return View();
    }

    // 3. XỬ LÝ TẠO MỚI (Create - POST)
    // Nhiệm vụ: Nhận dữ liệu user điền, lưu vào DB
    [HttpPost("Create")]
    public async Task<IActionResult> Create(Product product)
    {
        // ModelState.IsValid: Kiểm tra xem dữ liệu có đúng luật không 
        // (VD: Tên có trống không? Giá có âm không? - Dựa vào Data Annotations ở Model)
        if (ModelState.IsValid)
        {
            _context.Products.Add(product);
            await _context.SaveChangesAsync();
            
            // Xong thì quay về trang danh sách
            return RedirectToAction(nameof(Index));
        }
        
        // Nếu lỗi (VD: Quên nhập tên), thì hiện lại Form cũ để nhập lại
        // Nhớ nạp lại danh mục cho Dropdown
        ViewBag.Categories = _context.Categories.ToList();
        return View(product);
    }
}
```

### Bước 4: Tạo Views (Giao diện HTML)

Tạo thư mục `Views/ProductsMvc`.

**1. Index.cshtml (Trang danh sách)**
```html
@model IEnumerable<Product> 
@* Dòng trên báo rằng: View này nhận vào một List các Product *@

@{ ViewData["Title"] = "Danh sách sản phẩm"; }

<div class="d-flex justify-content-between mb-3">
    <h2>@ViewData["Title"]</h2>
    <a asp-action="Create" class="btn btn-primary">Thêm mới</a>
</div>

<table class="table table-bordered table-hover">
    <thead class="table-light">
        <tr>
            <th>ID</th>
            <th>Tên sản phẩm</th>
            <th>Giá</th>
            <th>Kho</th>
            <th>Danh mục</th>
            <th>Hành động</th>
        </tr>
    </thead>
    <tbody>
        @* Dùng vòng lặp C# ngay trong HTML *@
        @foreach (var item in Model)
        {
            <tr>
                <td>@item.Id</td>
                <td>@item.Name</td>
                <td>@item.Price.ToString("N0") đ</td>
                <td>@item.Stock</td>
                <td>@item.Category.Name</td>
                <td>
                    <a href="/Admin/Products/Edit/@item.Id" class="btn btn-sm btn-warning">Sửa</a>
                    <!-- Bài tập: Làm nút Xóa -->
                </td>
            </tr>
        }
    </tbody>
</table>
```

**2. Create.cshtml (Form tạo mới)**
```html
@model Product
@* View này làm việc với 1 đối tượng Product *@

@{ ViewData["Title"] = "Thêm sản phẩm"; }

<h2>Thêm sản phẩm mới</h2>
<hr />

<div class="row">
    <div class="col-md-6">
        <!-- asp-action="Create": Khi submit sẽ gửi đến Action Create (POST) -->
        <form asp-action="Create" method="post">
            
            <div class="mb-3">
                <label asp-for="Name" class="form-label">Tên sản phẩm</label>
                <!-- Input này tự động bind vào property Name của Model -->
                <input asp-for="Name" class="form-control" />
                <!-- Hiện lỗi nếu có -->
                <span asp-validation-for="Name" class="text-danger"></span>
            </div>

            <div class="mb-3">
                <label asp-for="Price" class="form-label">Giá bán</label>
                <input asp-for="Price" type="number" class="form-control" />
            </div>

            <div class="mb-3">
                <label asp-for="CategoryId" class="form-label">Danh mục</label>
                <!-- asp-items: Tự động tạo các thẻ <option> từ list Categories -->
                <select asp-for="CategoryId" class="form-control" 
                        asp-items="@(new SelectList(ViewBag.Categories, "Id", "Name"))">
                    <option>-- Chọn danh mục --</option>
                </select>
            </div>

            <div class="mb-3">
                 <label asp-for="Stock" class="form-label">Số lượng tồn</label>
                 <input asp-for="Stock" class="form-control" />
            </div>

            <button type="submit" class="btn btn-success">Lưu lại</button>
            <a asp-action="Index" class="btn btn-secondary">Hủy bỏ</a>
        </form>
    </div>
</div>
```

---

## 3. 🧪 BÀI TẬP VỀ NHÀ

### Bài tập: Hoàn thiện Edit & Delete
Hiện tại nút "Sửa" và "Xóa" chưa hoạt động. Hãy code thêm:
1.  **Edit (GET):** Lấy sản phẩm theo ID, hiển thị lên form (tương tự Create nhưng có dữ liệu sẵn).
2.  **Edit (POST):** Cập nhật dữ liệu vào database.
3.  **Delete (GET/POST):** Hiển thị trang xác nhận "Bạn có chắc muốn xóa?", nếu đồng ý thì xóa khỏi DB.

---

## ✅ CHECKLIST TỰ ĐÁNH GIÁ
- [ ] Tôi hiểu cách dùng `ViewBag` để truyền danh sách Categories drop-down.
- [ ] Tôi hiểu cú pháp Razor `@Model`, `@foreach`.
- [ ] Tôi hiểu Tag Helper `asp-for` giúp map input HTML với C# property như thế nào.

---

## 3. 🧪 BÀI TẬP VỀ NHÀ

### Bài tập: Hoàn thiện Edit & Delete
Hiện tại nút "Sửa" và "Xóa" chưa hoạt động. Hãy code thêm:
1.  **Edit (GET):** Lấy sản phẩm theo ID, hiển thị lên form (tương tự Create nhưng có dữ liệu sẵn).
2.  **Edit (POST):** Cập nhật dữ liệu vào database.
3.  **Delete (GET/POST):** Hiển thị trang xác nhận "Bạn có chắc muốn xóa?", nếu đồng ý thì xóa khỏi DB.

---

## ✅ CHECKLIST HOÀN THÀNH
- [ ] Truy cập `/Admin/Products` thấy danh sách đẹp mắt (Bootstrap).
- [ ] Bấm "Thêm mới" hiện ra Form.
- [ ] Dropdown danh mục hiển thị đúng dữ liệu từ DB.
- [ ] Thêm thành công -> Chuyển hướng về trang danh sách.
