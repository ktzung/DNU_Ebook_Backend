# 🟦 THỰC HÀNH 02 (Rider Version): SQL SERVER & EF CORE VỚI JETBRAINS RIDER

## 🎯 MỤC TIÊU
- **Làm chủ JetBrains Rider**: IDE mạnh mẽ nhất hiện nay cho .NET Developer.
- **Thao tác Visual (Giao diện)**: Thay vì gõ lệnh, ta sẽ tận dụng tối đa các công cụ GUI của Rider để quản lý NuGet, Database.
- **Hiểu sâu về Code First**: Cách biến Code C# thành Database SQL chuẩn chỉnh.
- **Kỹ năng Data Seeding**: Tạo dữ liệu giả thông minh để test ứng dụng.

---

## 1. CHUẨN BỊ MÔI TRƯỜNG

1.  **SQL Server**: Đã cài đặt (Developer hoặc Express). Đây là nơi chứa dữ liệu của chúng ta.
2.  **JetBrains Rider**: Đã cài đặt và kích hoạt. Rider sẽ là "vũ khí" chính.
3.  **Tư duy**: Chúng ta sẽ tiếp cận theo hướng **Code First** (Viết code trước -> Sinh database sau).

---

## 2. BÀI TOÁN: HỆ THỐNG EBOOK 📚
Chúng ta xây dựng Backend quản lý sách với 2 thực thể chính:
1.  **Categories (Thể loại)**: Ví dụ "Tiểu thuyết", "Công nghệ".
2.  **Books (Sách)**: Chi tiết từng cuốn sách.
3.  **Quan hệ**: Một Category có nhiều Book (**1-N**).

---

## 3. CÁC BƯỚC THỰC HIỆN TRÊN RIDER (CHI TIẾT)

### Bước 1: Tạo Project mới
1.  Mở Rider -> Chọn **New Solution**.
2.  Ở cột bên trái, chọn template **ASP.NET Core Web API**.
    *   *Tại sao chọn Web API?* Vì chúng ta đang làm Backend, cung cấp dữ liệu dạng JSON cho Frontend (Mobile/Web).
3.  Điền thông tin:
    *   **Solution name**: `DNU.Ebook` (Tên gói giải pháp tổng).
    *   **Project name**: `DNU.Ebook.Backend` (Tên dự án cụ thể).
4.  Nhấn **Create**. Rider sẽ mất vài giây để khởi tạo cấu trúc dự án chuẩn.

### Bước 2: Cài đặt thư viện (NuGet) qua GUI
**Tại sao dùng GUI?** Rider có giao diện quản lý gói (Package Manager) rất trực quan, giúp tìm kiếm phiên bản và xem sự phụ thuộc dễ dàng hơn gõ lệnh.

1.  Nhìn sang cửa sổ **Solution Explorer** (bên trái), chuột phải vào Project `DNU.Ebook.Backend`.
2.  Chọn **Manage NuGet Packages**.
3.  Một tab mới hiện ra ở giữa màn hình. Chọn thẻ **Packages** (hoặc Browse).
4.  Gõ vào ô tìm kiếm và cài đặt 3 gói "huyền thoại" sau (nhấn nút **+** màu xanh):
    *   `Microsoft.EntityFrameworkCore.SqlServer`: Driver để kết nối SQL Server.
    *   `Microsoft.EntityFrameworkCore.Tools`: Bộ công cụ để chạy lệnh Migration.
    *   `Bogus`: Thư viện sinh dữ liệu giả cực hay (dùng cho bước Seeding).

### Bước 3: Tạo Models (Entities) - "Xương sống" của ứng dụng
Chuột phải vào Project -> **Add** -> **Directory** -> đặt tên `Models`.
Sau đó tạo 2 class bên dưới. Chú ý đọc kỹ phần **Giải thích Code**.

**File: `Models/Category.cs`**
```csharp
using System.ComponentModel.DataAnnotations;

namespace DNU.Ebook.Backend.Models
{
    public class Category
    {
        [Key] // Đánh dấu khóa chính (Primary Key). SQL sẽ tự hiểu là Identity (Tự tăng).
        public int Id { get; set; }

        [Required] // Bắt buộc nhập (tương úng NOT NULL trong SQL).
        [MaxLength(100)] // Giới hạn độ dài chuỗi để tiết kiệm bộ nhớ DB.
        public required string Name { get; set; }
    }
}
```

**File: `Models/Book.cs`**
```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace DNU.Ebook.Backend.Models
{
    public class Book
    {
        [Key]
        public int Id { get; set; }

        [Required]
        [MaxLength(200)]
        public required string Title { get; set; }

        public string? Author { get; set; } // Dấu ? cho phép giá trị Null.
        
        [Range(0, double.MaxValue)] // Validate ngay tạo mức Code: Giá không được âm.
        public decimal Price { get; set; }

        // --- Thiết lập Khóa ngoại (Foreign Key) ---
        // Để EF Core hiểu Book nào thuộc Category nào.
        
        public int CategoryId { get; set; } // Lưu ID thực tế.
        
        [ForeignKey("CategoryId")] // Chỉ định rõ CategoryId là khóa ngoại cho biến Category bên dưới.
        public Category? Category { get; set; } // Navigation Property: giúp truy xuất thông tin Category từ Book.
    }
}
```

### Bước 4: Tạo DbContext - "Trái tim" của EF Core
Tạo thư mục `Data`, thêm class `ApplicationDbContext.cs`.
Class này đại diện cho cả Database của bạn trong code.

```csharp
using Microsoft.EntityFrameworkCore;
using DNU.Ebook.Backend.Models;

namespace DNU.Ebook.Backend.Data
{
    public class ApplicationDbContext : DbContext // Kế thừa từ DbContext chuẩn
    {
        // Constructor nhận cấu hình (Option) từ bên ngoài (thường là từ Program.cs)
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options) {}

        // Khai báo: "Tôi muốn có 2 bảng này trong Database"
        public DbSet<Category> Categories { get; set; }
        public DbSet<Book> Books { get; set; }
    }
}
```

### Bước 5: Cấu hình Kết nối & Seeding
**1. `appsettings.json` (Cấu hình chuỗi kết nối)**
```json
// Thêm đoạn này vào trong ngoặc nhọn lớn nhất
"ConnectionStrings": {
  // Dấu chấm (.) nghĩa là localhost.
  // TrustServerCertificate=True để bỏ qua lỗi SSL khi chạy local.
  "DefaultConnection": "Server=.;Database=DNU_Ebook_Rider;Trusted_Connection=True;TrustServerCertificate=True;"
}
```

**2. `Data/DbSeeder.cs` (Sinh dữ liệu giả)**
Thay vì nhập tay từng dòng mệt mỏi, ta dùng code để sinh 1000 dòng cũng được!

```csharp
using Bogus;
using DNU.Ebook.Backend.Models;

namespace DNU.Ebook.Backend.Data
{
    public static class DbSeeder
    {
        public static void Seed(ApplicationDbContext context)
        {
            // Nếu DB đã có dữ liệu rồi thì thôi, không sinh thêm nữa tránh trùng lặp.
            if (context.Categories.Any()) return;

            // 1. Tạo 5 Category ngẫu nhiên
            var categoryFaker = new Faker<Category>()
                .RuleFor(c => c.Name, f => f.Commerce.Categories(1)[0]);
            var categories = categoryFaker.Generate(5);
            context.Categories.AddRange(categories);
            context.SaveChanges(); // Lưu ngay để lấy ID cho bước sau

            // 2. Tạo 20 cuốn sách ngẫu nhiên
            var bookFaker = new Faker<Book>()
                .RuleFor(b => b.Title, f => f.Lorem.Sentence(3)) // Tên sách 3 từ
                .RuleFor(b => b.Author, f => f.Name.FullName())   // Tên người thật
                .RuleFor(b => b.Price, f => f.Random.Decimal(50000, 500000)) // Giá random
                .RuleFor(b => b.CategoryId, f => f.PickRandom(categories).Id); // Chọn bừa 1 category
            
            var books = bookFaker.Generate(20);
            context.Books.AddRange(books);
            context.SaveChanges();
        }
    }
}
```

**3. `Program.cs` (Đấu nối mọi thứ)**
```csharp
// 1. Đăng ký DbContext vào hệ thống (Dependency Injection)
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// 2. Kích hoạt Seeding data ngay khi app chạy
using (var scope = app.Services.CreateScope())
{
    var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
    // context.Database.Migrate(); // Mẹo: Bỏ comment dòng này nếu muốn tự động update DB luôn, không cần gõ lệnh.
    DbSeeder.Seed(context);
}
```

### Bước 6: Chạy Migration (Rider Terminal)
Rider có sẵn Terminal xịn xò (giống VS Code).
1.  Nhấn `Alt + F12` để mở Terminal.
2.  Gõ lệnh tạo file migration:
    ```bash
    dotnet ef migrations add InitialCreate
    ```
    *Giải thích*: Lệnh này so sánh code Models của bạn với Database hiện tại, để tìm ra sự thay đổi và tạo script update.
3.  Gõ lệnh chạy update:
    ```bash
    dotnet ef database update
    ```
    *Giải thích*: Lệnh này thực thi script SQL lên Server thật.

---

## 4. [CỰC HAY] QUẢN LÝ DATABASE TRỰC TIẾP TRONG RIDER 🚀

Đây là tính năng "ăn tiền" của Rider so với VS Code. Bạn không cần cài SSMS nặng nề, cũng không cần chuyển qua lại giữa các cửa sổ.

1.  **Mở Database Tool**: Nhìn sang thanh dọc bên phải (Right Sidebar), click biểu tượng **Database** (hình thùng phi).
2.  **Thêm kết nối**: Nhấn dấu `+` -> **Data Source** -> **Microsoft SQL Server**.
3.  **Cấu hình**:
    *   **Host**: `.` (hoặc localhost)
    *   **Authentication**: Để `User & Password` nếu dùng sa/pass. Hoặc để không điền gì (hoặc chọn Windows Credentials) nếu dùng Trusted Connection.
    *   *Lưu ý*: Nếu Rider hiện dòng chữ "Download driver files", hãy bấm vào đó để nó tải driver SQL về.
4.  **Kiểm tra**: Nhấn **Test Connection**. Nếu hiện ✅ xanh là OK. Nhấn **OK** để lưu.
5.  **Xem dữ liệu**:
    *   Trong cây thư mục Database mới hiện ra, tìm `DNU_Ebook_Rider` -> `schemas` -> `dbo` -> `tables`.
    *   Double click vào bảng `Books`.
    *   **Kết quả**: Bạn sẽ thấy danh sách 20 cuốn sách với tên, giá, tác giả... được sinh ra y như thật!

Bạn có thể sửa dữ liệu trực tiếp trên lưới (Grid) này và nhấn `Ctrl + Enter` để lưu xuống DB. Rất tiện!

---
**🎉 TỔNG KẾT**
Bạn đã hoàn thành:
1.  Biết cách setup dự án .NET API trên Rider.
2.  Hiểu rõ từng thành phần: Model, DbContext, Seeder.
3.  Biết cách dùng Bogus "phù phép" ra dữ liệu.
4.  Biết cách dùng công cụ Database tích hợp của Rider.
