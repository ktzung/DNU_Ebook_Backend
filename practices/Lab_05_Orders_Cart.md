# 🧪 LAB 05: XỬ LÝ ĐƠN HÀNG & TRANSACTION
**(Tương ứng: Module 05 - Week 8-9)**

## 📖 1. BÀI TOÁN & PHÂN TÍCH (ĐỌC KỸ TRƯỚC KHI CODE)

### 1.1. Câu chuyện nghiệp vụ (User Story)
Hãy đặt mình vào vị trí khách hàng:
1.  Bạn lướt web, thấy "iPhone 15" và "Ốp lưng".
2.  Bạn chọn mua (Thêm vào giỏ).
3.  Bạn bấm **"Đặt hàng" (Checkout)**.

👉 **Việc của Backend là gì lúc này?**
Backend phải thực hiện một chuỗi hành động **"MỘT MẤT MỘT CÒN" (Transaction):**
1.  Tạo "Hóa đơn" (Order) ghi nhận bạn đã mua.
2.  Tạo chi tiết hóa đơn (OrderDetail) ghi rõ mua cái gì, giá bao nhiêu.
3.  **QUAN TRỌNG:** Phải **TRỪ KHO** (Stock) của iPhone và Ốp lưng.
4.  Nếu trừ kho lỗi (hết hàng) -> **HỦY TOÀN BỘ ĐƠN**, không được tạo đơn rác.

### 1.2. Thiết kế Dữ liệu (Database Design)
Chúng ta cần 2 bảng mới nối với bảng `Users` và `Products`.

*   **Bảng `Orders` (Hóa đơn):**
    *   Ai mua? (`UserId`)
    *   Tổng tiền bao nhiêu? (`TotalPrice`)
    *   Lúc nào? (`CreatedAt`)
    *   Trạng thái? (`Status`: Mới, Đã thanh toán, Hủy...)
*   **Bảng `OrderDetails` (Chi tiết):**
    *   Thuộc hóa đơn nào? (`OrderId`)
    *   Sản phẩm nào? (`ProductId`)
    *   Số lượng? (`Quantity`)
    *   Giá lúc mua? (`UnitPrice`) -> *Tại sao lưu giá? Vì giá sản phẩm có thể đổi sau này, nhưng giá trong hóa đơn cũ không được đổi.*

---

## 📦 PHẦN 1: THIẾT KẾ DATABASE (MODELS)

Chúng ta sẽ hiện thực hóa bản thiết kế trên vào code C#.

### Bước 1: Tạo Model Order
Tạo file `Models/Order.cs`.

```csharp
namespace EShop.API.Models
{
    public class Order
    {
        public int Id { get; set; }
        public DateTime CreatedAt { get; set; } = DateTime.Now;
        public decimal TotalPrice { get; set; }
        
        // Trạng thái đơn hàng: New, Paid, Shipped, Cancelled
        public string Status { get; set; } = "New"; 

        // --- KHÓA NGOẠI (Relationship) ---
        // Một đơn hàng thuộc về Một khách hàng
        public int UserId { get; set; }
        public User User { get; set; } = null!; // Navigation Property

        // Một đơn hàng có Nhiều chi tiết (danh sách sản phẩm)
        public List<OrderDetail> OrderDetails { get; set; } = new List<OrderDetail>();
    }
}
```

### Bước 2: Tạo Model OrderDetail
Tạo file `Models/OrderDetail.cs`.

```csharp
namespace EShop.API.Models
{
    public class OrderDetail
    {
        public int Id { get; set; }
        
        // Mua cái gì?
        public int ProductId { get; set; }
        public Product Product { get; set; } = null!;

        // Thuộc đơn nào?
        public int OrderId { get; set; }
        public Order Order { get; set; } = null!;

        public int Quantity { get; set; }
        
        // Lưu giá tại thời điểm mua để làm bằng chứng
        public decimal UnitPrice { get; set; } 
    }
}
```

### Bước 3: Cập nhật DbContext & Migration
Mở `Data/AppDbContext.cs`, khai báo 2 bảng mới:

```csharp
public DbSet<Order> Orders { get; set; }
public DbSet<OrderDetail> OrderDetails { get; set; }
```

Chạy lệnh Terminal để cập nhật Database:
```powershell
dotnet ef migrations add AddOrderTables
dotnet ef database update
```
> **Kiểm tra:** Mở SQL Server, đảm bảo đã có 2 bảng mới và có khóa ngoại kết nối với nhau.

---

## ⚙️ PHẦN 2: LOGIC NGHIỆP VỤ (SERVICE LAYER)

Để đặt hàng, Client sẽ gửi lên một danh sách: *"{Mua iPhone SL 1, Mua Áo SL 2}"*.
Ta cần một DTO để hứng dữ liệu này.

### Bước 4: Tạo DTO nhận đơn hàng

**DTOs/OrderDto.cs**
```csharp
namespace EShop.API.DTOs
{
    // DTO cho từng món hàng trong giỏ
    public class CartItemDto
    {
        public int ProductId { get; set; }
        public int Quantity { get; set; }
    }

    // DTO cho cả yêu cầu đặt hàng
    public class CreateOrderRequest
    {
        public List<CartItemDto> Items { get; set; } = new List<CartItemDto>();
    }
}
```

### Bước 5: Viết API Đặt hàng (Trái tim của bài Lab)

Tạo `Controllers/OrdersController.cs`. Đây là phần khó nhất, hãy đọc kỹ comment giải thích từng dòng.

```csharp
using System.Security.Claims;
using EShop.API.Data;
using EShop.API.DTOs;
using EShop.API.Models;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

[Route("api/[controller]")]
[ApiController]
[Authorize] // 1. Bắt buộc phải đăng nhập mới được đặt hàng
public class OrdersController : ControllerBase
{
    private readonly AppDbContext _context;

    public OrdersController(AppDbContext context)
    {
        _context = context;
    }

    // POST: api/orders
    [HttpPost]
    public async Task<IActionResult> PlaceOrder(CreateOrderRequest request)
    {
        // 2. Lấy ID người dùng từ Token (Claim "UserId" ta đã bỏ vào ở Lab 03)
        // Nếu không có Token -> User.FindFirst sẽ null -> lỗi
        var userIdString = User.FindFirst("UserId")?.Value;
        if (userIdString == null) return Unauthorized("Không tìm thấy thông tin User.");
        
        int userId = int.Parse(userIdString);

        // 3. START TRANSACTION (Khóa an toàn)
        // Nếu có bất kỳ lỗi nào xảy ra trong khối này, mọi thay đổi sẽ bị HỦY (Rollback)
        using var transaction = await _context.Database.BeginTransactionAsync();
        
        try
        {
            // A. Tạo vỏ đơn hàng
            var order = new Order
            {
                UserId = userId,
                CreatedAt = DateTime.Now,
                Status = "New",
                TotalPrice = 0, // Sẽ cộng dồn sau
                OrderDetails = new List<OrderDetail>()
            };

            // B. Duyệt qua từng món hàng khách chọn
            foreach (var item in request.Items)
            {
                // Tìm sản phẩm trong kho
                var product = await _context.Products.FindAsync(item.ProductId);
                
                // Check 1: Có tồn tại không?
                if (product == null) throw new Exception($"Sản phẩm ID {item.ProductId} không tồn tại.");

                // Check 2: Còn đủ hàng không?
                if (product.Stock < item.Quantity) 
                    throw new Exception($"Sản phẩm {product.Name} không đủ hàng (còn {product.Stock}).");

                // C. TRỪ KHO (Thao tác quan trọng!)
                product.Stock = product.Stock - item.Quantity;

                // D. Tạo chi tiết đơn hàng
                var detail = new OrderDetail
                {
                    ProductId = item.ProductId,
                    Quantity = item.Quantity,
                    UnitPrice = product.Price // Lấy giá hiện tại
                };
                
                // Thêm vào list & cộng tiền
                order.OrderDetails.Add(detail);
                order.TotalPrice += (detail.Quantity * detail.UnitPrice);
            }

            // E. Lưu mọi thứ xuống Database
            _context.Orders.Add(order);
            await _context.SaveChangesAsync();

            // F. CHỐT GIAO DỊCH
            await transaction.CommitAsync();

            return Ok(new { OrderId = order.Id, Message = "Đặt hàng thành công!", Total = order.TotalPrice });
        }
        catch (Exception ex)
        {
            // G. CÓ LỖI -> HOÀN TÁC TẤT CẢ
            await transaction.RollbackAsync();
            return BadRequest(new { Message = ex.Message });
        }
    }
}
```

---

## 🚀 PHẦN 3: KIỂM THỬ (TESTING)

Giai đoạn này giúp bạn hiểu luồng dữ liệu chạy thực tế.

### Bước 6: Kịch bản Test bằng Postman / Swagger
1.  **Chuẩn bị:**
    *   Đảm bảo có User (đã đăng ký ở Lab 03).
    *   Đảm bảo có Product (Id 1, Id 2...) và có Stock > 0.
2.  **Đăng nhập:** Gọi API Login -> Copy Token.
3.  **Authorize:** Dán Token vào ổ khóa trong Swagger.
4.  **Đặt hàng:**
    *   Gọi `POST /api/orders`.
    *   Body:
        ```json
        {
          "items": [
            { "productId": 1, "quantity": 1 },
            { "productId": 2, "quantity": 5 }
          ]
        }
        ```
5.  **Kết quả:**
    *   Nếu thành công: Trả về `OrderId` và `Total`.
    *   **Check DB:** Bảng `Products` phải bị trừ Stock. Bảng `Orders` có dòng mới.

### Bước 7: Test Transaction (Thử làm sai)
*   Hãy thử đặt số lượng lớn hơn Stock hiện có.
*   Mong đợi: API báo lỗi "Không đủ hàng".
*   **Quan trọng:** Kiểm tra DB xem `Orders` có bị tạo rác không? Nếu không tạo -> Transaction hoạt động tốt.

---

## ✅ CHECKLIST TỰ ĐÁNH GIÁ (SELF-REVIEW)
- [ ] Tôi hiểu tại sao cần bảng trung gian `OrderDetails` (Quan hệ N-N giữa Order và Product).
- [ ] Tôi hiểu `Transaction` dùng để làm gì (Bảo vệ dữ liệu khi 1 thành phần bị lỗi).
- [ ] Tôi hiểu logic trừ kho phải nằm cùng lúc với tạo đơn.
