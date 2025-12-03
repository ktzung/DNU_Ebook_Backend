# 🛒 MEGA PROJECT 02
# **E-SHOP MVC FULL STACK**

## 📖 1. Giới thiệu dự án

Đây là dự án lớn thứ 2 trong khóa học, tập trung vào việc xây dựng một website thương mại điện tử hoàn chỉnh sử dụng **ASP.NET Core MVC**.

### Mục tiêu:
- Xây dựng ứng dụng Monolith chuẩn chỉnh.
- Áp dụng kiến trúc phân lớp (N-Layer Architecture).
- Sử dụng Entity Framework Core để quản lý dữ liệu.
- Tích hợp Identity để quản lý người dùng.
- Thanh toán online (giả lập).

---

## 🏗️ 2. Kiến trúc dự án (N-Layer)

Chúng ta sẽ chia Solution thành 4 Project:

1. **EShop.Core** (Class Library): Chứa Entities, Interfaces, Domain Logic. Không phụ thuộc vào ai.
2. **EShop.Infrastructure** (Class Library): Chứa DbContext, Repositories, Migrations. Phụ thuộc `Core`.
3. **EShop.Application** (Class Library): Chứa Services, DTOs, Business Logic. Phụ thuộc `Core` và `Infrastructure`.
4. **EShop.Web** (ASP.NET Core MVC): Chứa Controllers, Views, ViewModels. Phụ thuộc `Application`.

---

## 🛠️ 3. Các tính năng chính

### 3.1. Phân hệ Khách hàng (Front Store)
- **Trang chủ**: Hiển thị sản phẩm nổi bật, banner.
- **Danh sách sản phẩm**: Phân trang, lọc theo danh mục, tìm kiếm.
- **Chi tiết sản phẩm**: Hình ảnh, mô tả, nút thêm vào giỏ.
- **Giỏ hàng**: Xem, sửa số lượng, xóa, tính tổng tiền.
- **Thanh toán (Checkout)**: Nhập thông tin giao hàng, chọn phương thức thanh toán.
- **Tài khoản**: Đăng ký, đăng nhập, xem lịch sử đơn hàng.

### 3.2. Phân hệ Quản trị (Admin Panel)
- **Dashboard**: Thống kê doanh thu, số đơn hàng mới.
- **Quản lý Sản phẩm**: Thêm, sửa, xóa, upload ảnh.
- **Quản lý Danh mục**: CRUD danh mục.
- **Quản lý Đơn hàng**: Xem chi tiết, cập nhật trạng thái (Đang giao, Đã giao, Hủy).

---

## 🚀 4. Hướng dẫn thực hiện từng bước

### Bước 1: Khởi tạo Solution và Projects
```powershell
dotnet new sln -n EShop
dotnet new classlib -n EShop.Core
dotnet new classlib -n EShop.Infrastructure
dotnet new classlib -n EShop.Application
dotnet new mvc -n EShop.Web

# Add references
dotnet add EShop.Infrastructure reference EShop.Core
dotnet add EShop.Application reference EShop.Core
dotnet add EShop.Application reference EShop.Infrastructure
dotnet add EShop.Web reference EShop.Application
```

### Bước 2: Thiết kế Database (EShop.Core)
Tạo các Entity: `Product`, `Category`, `Order`, `OrderItem`, `AppUser`.

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string ImageUrl { get; set; }
    public int CategoryId { get; set; }
    public Category Category { get; set; }
}
```

### Bước 3: Cấu hình EF Core (EShop.Infrastructure)
- Cài đặt `Microsoft.EntityFrameworkCore.SqlServer`.
- Tạo `EShopDbContext` kế thừa `IdentityDbContext`.
- Chạy Migration để sinh Database.

### Bước 4: Xây dựng Repository & Service (EShop.Application)
- Tạo `IProductRepository` và `ProductRepository`.
- Tạo `ProductService` để xử lý logic (ví dụ: validate giá không được âm).

### Bước 5: Xây dựng Web UI (EShop.Web)
- Cấu hình DI trong `Program.cs`.
- Tạo `HomeController`, `ProductController`.
- Thiết kế giao diện dùng Bootstrap 5.
- Tạo Layout chung (`_Layout.cshtml`) và Admin Layout (`_AdminLayout.cshtml`).

---

## 🛒 5. Chức năng Giỏ hàng (Shopping Cart)

Giỏ hàng thường được lưu trong **Session** hoặc **Cookies**.

### CartItem ViewModel
```csharp
public class CartItem
{
    public int ProductId { get; set; }
    public string ProductName { get; set; }
    public decimal Price { get; set; }
    public int Quantity { get; set; }
    public decimal Total => Price * Quantity;
}
```

### CartController
- `AddToCart(int id)`: Thêm sản phẩm vào List<CartItem> trong Session.
- `Index()`: Hiển thị giỏ hàng.
- `Remove(int id)`: Xóa khỏi Session.

---

## 🧪 6. Yêu cầu nộp bài

1. **Source Code**: Đẩy lên GitHub.
2. **Database**: Script tạo DB hoặc file backup `.bak`.
3. **Video Demo**: Quay màn hình demo luồng mua hàng và quản trị.
4. **Báo cáo**: File PDF mô tả các chức năng đã làm được.

---

## 💡 Mẹo nhỏ
> [!TIP]
> Sử dụng **Areas** trong MVC để tách biệt phần Admin và phần User (`/Areas/Admin/Controllers/...`).

> [!NOTE]
> Để upload ảnh, sử dụng `IFormFile` trong ViewModel và lưu file vào thư mục `wwwroot/images`.
