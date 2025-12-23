# 🟩 CHƯƠNG 17
# **TRANSACTIONS & CONCURRENCY**

Trong các ứng dụng thực tế (đặc biệt là E-Commerce), việc đảm bảo tính toàn vẹn dữ liệu khi có nhiều thao tác ghi cùng lúc là cực kỳ quan trọng. Chương này sẽ giúp bạn làm chủ **Database Transactions** và xử lý **Concurrency** (đồng thời).

---

# 🎯 MỤC TIÊU HỌC TẬP

Sau chương này, bạn sẽ:

- Hiểu nguyên lý ACID trong Database.
- Sử dụng `BeginTransaction` trong EF Core.
- Xử lý lỗi và Rollback dữ liệu.
- Hiểu về Concurrency (Optimistic Locking) và cách xử lý xung đột.
- Áp dụng vào quy trình Đặt hàng (Checkout).

---

# 1. **DATABASE TRANSACTION LÀ GÌ?**

## 1.1. Khái niệm

**Transaction** (Giao dịch) là một tập hợp các thao tác (Insert, Update, Delete) được thực hiện như một khối duy nhất.
Nguyên tắc: **"ALL OR NOTHING"** (Tất cả thành công, hoặc không có gì xảy ra).

### 🎒 Ví dụ đời sống: Chuyển khoản ngân hàng

Bạn chuyển 10 triệu cho bạn của bạn.
Quy trình gồm 2 bước:
1. Trừ 10 triệu tài khoản của bạn.
2. Cộng 10 triệu vào tài khoản bạn bè.

👉 **Vấn đề:** Nếu bước 1 xong, nhưng bước 2 bị lỗi (mất mạng, bank lỗi)?
-> Tiền bạn bị trừ nhưng bạn bè không nhận được. **MẤT TIỀN!** 😱

👉 **Giải pháp Transaction:** Khi bước 2 lỗi, hệ thống phải tự động **HOÀN TÁC (ROLLBACK)** bước 1. Tiền về lại ví bạn.

## 1.2. Mẫu hình ACID

Một Transaction chuẩn phải tuân thủ 4 tính chất (ACID):

1.  **A - Atomicity (Tính nguyên tử):** Chuỗi thao tác là không thể chia cắt. Thành công hết hoặc thất bại hết.
2.  **C - Consistency (Tính nhất quán):** Dữ liệu phải valid trước và sau transaction.
3.  **I - Isolation (Tính cô lập):** Transaction này không được ảnh hưởng tới transaction khác đang chạy song song.
4.  **D - Durability (Tính bền vững):** Khi đã Commit, dữ liệu được lưu vĩnh viễn dù có cúp điện.

---

# 2. **SỬ DỤNG TRANSACTION TRONG EF CORE**

EF Core hỗ trợ Transaction rất mạnh mẽ.

## 2.1. Tự động (Implicit Transaction)

Khi bạn gọi `SaveChanges()`, EF Core tự động bọc tất cả câu lệnh SQL vào 1 transaction.

```csharp
public void AddProductAndLog()
{
    _context.Products.Add(new Product { Name = "iPhone 15" });
    _context.AuditLogs.Add(new AuditLog { Action = "Add Product" });
    
    // Nếu Log lỗi -> Product cũng không được thêm
    _context.SaveChanges(); 
}
```

## 2.2. Thủ công (Explicit Transaction)

Dùng khi bạn cần kiểm soát chi tiết hơn, hoặc thực hiện nhiều lần `SaveChanges` trong 1 nghiệp vụ logic.

```csharp
public async Task CheckoutOrderAsync(Order order)
{
    // 1. Mở Transaction
    using var transaction = await _context.Database.BeginTransactionAsync();
    
    try
    {
        // Bước A: Tạo Order
        _context.Orders.Add(order);
        await _context.SaveChangesAsync(); // Lưu để lấy OrderId
        
        // Bước B: Trừ tồn kho (Khoảng thời gian này rất nhạy cảm)
        foreach(var item in order.OrderItems)
        {
            var product = await _context.Products.FindAsync(item.ProductId);
            
            // Check tồn kho
            if (product.Stock < item.Quantity)
                throw new Exception($"Hết hàng: {product.Name}");
                
            product.Stock -= item.Quantity;
        }
        await _context.SaveChangesAsync();
        
        // Bước C: Commit (Chốt giao dịch)
        await transaction.CommitAsync();
    }
    catch (Exception)
    {
        // Có lỗi -> Undo mọi thứ từ Bước A
        await transaction.RollbackAsync();
        throw; // Ném lỗi ra để Controller biết
    }
}
```

---

# 3. **CONCURRENCY (XỬ LÝ ĐỒNG THỜI)**

## 3.1. Vấn đề "Lost Update"

Giả sử cửa hàng còn đúng **1 cái iPhone**.
- User A bấm mua. (Đọc: Stock = 1)
- User B bấm mua cùng lúc. (Đọc: Stock = 1)
- User A trừ kho: 1 - 1 = 0 -> Save.
- User B trừ kho: 1 - 1 = 0 -> Save.

👉 **Kết quả:** Bán được 2 cái iPhone trong khi chỉ có 1. Lỗi logic nghiêm trọng!

## 3.2. Giải pháp: Concurrency Check

Trong EF Core, ta dùng `ConcurrencyToken` (thường là cột `Version` hoặc `RowVersion`).

### Cấu hình Entity

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Stock { get; set; }
    
    // Đánh dấu cột này dùng để check đồng thời
    [Timestamp]
    public byte[] RowVersion { get; set; }
}
```

### Cách hoạt động

Khi Update, EF Core sẽ tạo câu SQL kiểu:
```sql
UPDATE Products 
SET Stock = 0 
WHERE Id = 1 AND RowVersion = [OldVersion]
```

- Nếu User A update xong, RowVersion đổi.
- User B gửi câu lệnh với `RowVersion` cũ -> Không tìm thấy dòng nào (RowsAffected = 0).
- EF Core ném ra ngoại lệ `DbUpdateConcurrencyException`.

### Xử lý trong Code (Retry Pattern)

```csharp
try 
{
    _context.SaveChanges();
}
catch (DbUpdateConcurrencyException ex)
{
    // Data đã bị người khác sửa -> Reload và thử lại
    // Hoặc báo lỗi cho User: "Sản phẩm vừa hết hàng, vui lòng thử lại."
}
```

---

# 4. **CASE STUDY: ORDER SERVICE HOÀN CHỈNH**

Kết hợp Transaction và Validation.

```csharp
public async Task<int> PlaceOrderAsync(CreateOrderRequest request)
{
    // Dùng ExecutionStrategy nếu Database là Cloud (Azure SQL) để tự retry khi kết nối chập chờn
    var strategy = _context.Database.CreateExecutionStrategy();

    return await strategy.ExecuteAsync(async () =>
    {
        using var transaction = await _context.Database.BeginTransactionAsync();
        try
        {
            // 1. Tạo Order Header
            var order = new Order 
            {
                UserId = request.UserId,
                OrderDate = DateTime.Now,
                Status = OrderStatus.New,
                TotalAmount = 0
            };
            
            _context.Orders.Add(order);
            await _context.SaveChangesAsync(); // Có OrderId
            
            // 2. Xử lý Items & Kho
            foreach (var itemDto in request.Items)
            {
                var product = await _context.Products.FindAsync(itemDto.ProductId);
                if (product == null) throw new Exception("Product not found");
                
                // Check tồn kho
                if (product.Stock < itemDto.Quantity)
                    throw new Exception($"Sản phẩm {product.Name} không đủ số lượng.");
                
                // Trừ kho
                product.Stock -= itemDto.Quantity;
                
                // Add Detail
                var detail = new OrderDetail
                {
                    OrderId = order.Id,
                    ProductId = product.Id,
                    Quantity = itemDto.Quantity,
                    UnitPrice = product.Price
                };
                
                order.OrderDetails.Add(detail);
                order.TotalAmount += detail.Quantity * detail.UnitPrice;
            }
            
            await _context.SaveChangesAsync();
            await transaction.CommitAsync();
            
            return order.Id;
        }
        catch
        {
            await transaction.RollbackAsync();
            throw;
        }
    });
}
```

---

# ❌ 5. CÁC LỖI THƯỜNG GẶP

### ❌ Lỗi 1: Quên Commit
```csharp
using var transaction = _context.Database.BeginTransaction();
_context.SaveChanges();
// Quên transaction.Commit(); -> Dữ liệu không được lưu!
```

### ❌ Lỗi 2: Transaction quá dài
- Tránh thực hiện các việc tốn thời gian (Gửi Email, Gọi API bên thứ 3) TRONG transaction.
- Nó sẽ khóa bảng (Lock table) lâu, làm chậm cả hệ thống.
- **Giải pháp:** Commit DB xong xuôi, ra ngoài transaction mới gửi Email.

---

# 📌 TÓM TẮT

- **Transaction** đảm bảo tính toàn vẹn dữ liệu (ALL OR NOTHING).
- Sử dụng `BeginTransaction()`, `Commit()`, `Rollback()`.
- **Concurrency** xảy ra khi nhiều người cùng sửa 1 dữ liệu.
- Sử dụng `[Timestamp]` để ngăn chặn xung đột dữ liệu (Lost Update).

**Chương tiếp theo: [18. Advanced Topics: Caching & Logging →](../Module_06_Advanced_Deploy/01_Advanced_Caching_Logging.md)**
