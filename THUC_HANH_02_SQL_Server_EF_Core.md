# 🟦 THỰC HÀNH 02: CÀI ĐẶT SQL SERVER & TẠO DATABASE VỚI EF CORE

## 🎯 MỤC TIÊU
- Cài đặt thành công SQL Server và công cụ quản lý SSMS.
- Hiểu chuyên sâu về **Connection String** và cách cấu hình.
- Phân tích bài toán, thiết kế Entity và hiểu rõ các `Attribute` trong EF Core (Key, ForeignKey, Required...).
- Thực hiện quy trình **Code First** chuẩn chỉnh.
- **[MỚI]** Kỹ thuật sinh dữ liệu giả (**Fake Data**) bằng thư viện `Bogus` để phục vụ test.

---

## 1. CÀI ĐẶT SQL SERVER VÀ SSMS

Để làm việc với .NET Backend, chúng ta cần một hệ quản trị cơ sở dữ liệu (DBMS). SQL Server là lựa chọn "chính chủ" từ Microsoft, tương thích tốt nhất với .NET.

### Bước 1: Cài đặt SQL Server (Backend Engine)
Bạn có thể chọn bản **Developer** (đầy đủ tính năng như bản Enterprise nhưng miễn phí cho mục đích học tập/phát triển) hoặc **Express** (nhẹ, miễn phí).
1.  Tải về từ trang chủ Microsoft: [SQL Server Downloads](https://www.microsoft.com/en-us/sql-server/sql-server-downloads).
2.  Chọn **Developer** edition -> Download Now -> Chạy file cài đặt.
3.  Chọn chế độ **Basic** -> Accept -> Install.
4.  **QUAN TRỌNG:** Sau khi cài xong, màn hình sẽ hiển thị **Connection String**. Hãy copy và lưu lại chuỗi này, nó rất quan trọng để cấu hình sau này.

### Bước 2: Cài đặt SQL Server Management Studio (SSMS)
Đây là công cụ có giao diện (GUI) để quản lý Database, chạy câu lệnh SQL.
1.  Trong màn hình cài đặt SQL Server xong, có nút **Install SSMS**, hoặc tải tại: [Download SSMS](https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms).
2.  Tải về và cài đặt (Cứ Next -> Install là được).

---

## 2. LẤY VÀ HIỂU CHUỖI KẾT NỐI (CONNECTION STRING)

Connection String giống như "địa chỉ nhà" và "chìa khóa" giúp ứng dụng của bạn tìm và mở cửa vào Database.

### Server=localhost\SQLEXPRESS01;Database=master;Trusted_Connection=True;

### Cách lấy Server Name chính xác:
1.  Mở **SSMS**.
2.  Tại cửa sổ đăng nhập đầu tiên -> **Server name** chính là tên máy chủ (ví dụ: `DESKTOP-ABC\SQLEXPRESS` hoặc `localhost` hoặc dấu chấm `.`).
3.  Nhấn **Connect**.

### Phân tích cấu trúc Chuỗi kết nối:

**Kiểu 1: Windows Authentication (Khuyên dùng cho máy cá nhân)**
Sử dụng chính tài khoản Windows bạn đang login để xác thực.
```text
Server=YOUR_SERVER_NAME;Database=TEN_DATABASE_SE_TAO;Trusted_Connection=True;TrustServerCertificate=True;
```
*   `Server`: Địa chỉ máy chủ SQL. Dấu `.` đại diện cho localhost.
*   `Database`: Tên Database bạn muốn EF Core tạo ra.
*   `Trusted_Connection=True`: Dùng Windows Auth (không cần user/pass).
*   `TrustServerCertificate=True`: Bỏ qua lỗi chứng chỉ SSL (thường gặp ở môi trường dev).

**Kiểu 2: SQL Server Authentication (Dùng tài khoản sa)**
Dùng khi deploy lên server thật hoặc máy bạn đã set mật khẩu cho user `sa`.
```text
Server=YOUR_SERVER_NAME;Database=TEN_DATABASE;User Id=sa;Password=YOUR_PASSWORD;TrustServerCertificate=True;
```

---

## 3. BÀI TOÁN MẪU: HỆ THỐNG EBOOK 📚

**Mô tả:**
Xây dựng backend cho website đọc sách. Hệ thống cần quản lý sách và phân loại sách.

**Yêu cầu dữ liệu:**
1.  **Thể loại (Category)**: Tên thể loại (Ví dụ: Tiểu thuyết, Công nghệ).
2.  **Sách (Book)**: Tên sách, Tác giả, Giá, Mô tả.
3.  **Quan hệ**: Một Thể loại có nhiều Sách (1-N).

---

## 4. QUY TRÌNH CODE FIRST (CHI TIẾT)

Chúng ta sẽ viết code C# trước, sau đó EF Core sẽ "dịch" code này thành bảng trong Database.

### Bước 1: Tạo Project & Cài thư viện
1.  Tạo project **ASP.NET Core Web API** (tên `DNU.Ebook`).
2.  Cài đặt các gói NuGet cần thiết (Chạy lệnh trong Terminal tại thư mục project):
    ```bash
    dotnet add package Microsoft.EntityFrameworkCore.SqlServer
    dotnet add package Microsoft.EntityFrameworkCore.Tools
    ```

### Bước 2: Tạo Models (Entities)
Tạo thư mục `Models`. Đây là nơi định nghĩa cấu trúc dữ liệu.

**File: `Models/Category.cs`**
```csharp
using System.ComponentModel.DataAnnotations;

namespace DNU.Ebook.Models
{
    public class Category
    {
        [Key] // Đánh dấu đây là Khóa chính (Primary Key)
        public int Id { get; set; }

        [Required] // Bắt buộc phải có dữ liệu (NOT NULL trong SQL)
        [MaxLength(100)] // Giới hạn độ dài tối đa 100 ký tự
        public required string Name { get; set; }

        // Navigation Property: Để EF Core hiểu quan hệ 1-N.
        // Một Category chứa danh sách các Book.
        // public ICollection<Book> Books { get; set; } = new List<Book>(); (Tùy chọn)
    }
}
```

**File: `Models/Book.cs`**
```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace DNU.Ebook.Models
{
    public class Book
    {
        [Key]
        public int Id { get; set; }

        [Required]
        [MaxLength(200)]
        public required string Title { get; set; }

        [MaxLength(100)]
        public string? Author { get; set; } // Dấu ? nghĩa là được phép Null

        [Range(0, double.MaxValue)] // Giá tiền không được âm
        public decimal Price { get; set; }

        // --- Thiết lập Khóa ngoại (Foreign Key) ---
        
        // 1. Lưu ID của Category
        public int CategoryId { get; set; }

        // 2. Object Category tương ứng (Navigation Property)
        // [ForeignKey("CategoryId")] dùng để chỉ rõ property nào làm khóa ngoại
        [ForeignKey("CategoryId")] 
        public Category? Category { get; set; }
    }
}
```

> **💡 Giải thích:**
> *   `[Key]`: Báo cho EF Core biết đây là cột ID, mặc định sẽ tự động tăng (Identity).
> *   `[ForeignKey]`: Định nghĩa mối liên kết giữa 2 bảng.
> *   `required` (C# 11+): Bắt buộc property phải được khởi tạo, giúp code an toàn hơn.

### Bước 3: Tạo DbContext (Lớp trung gian)
`DbContext` là "trái tim" của EF Core, cầu nối giữa code C# và SQL Server.

**File: `Data/ApplicationDbContext.cs`**
```csharp
using Microsoft.EntityFrameworkCore;
using DNU.Ebook.Models; // Nhớ using namespace chứa Models

namespace DNU.Ebook.Data
{
    public class ApplicationDbContext : DbContext
    {
        // Constructor này giúp nhận Connection String từ bên ngoài (Program.cs) vào
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
        {
        }

        // Khai báo các bảng sẽ được tạo trong Database
        public DbSet<Category> Categories { get; set; }
        public DbSet<Book> Books { get; set; }
    }
}
```

### Bước 4: Cấu hình kết nối & Đăng ký Service
**1. Tại `appsettings.json`:**
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=DNU_Ebook_Demo;Trusted_Connection=True;TrustServerCertificate=True;"
  }
}
```

**2. Tại `Program.cs`:**
Thêm đoạn code này VÀO TRƯỚC dòng `var app = builder.Build();`:
```csharp
using Microsoft.EntityFrameworkCore;
using DNU.Ebook.Data;

// ...

// Đăng ký dịch vụ DbContext vào DI Container
// Giúp ta có thể tiêm (inject) ApplicationDbContext vào Controller sau này
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();
```

### Bước 5: Chạy Migration (Tạo Database thực tế)
Đây là bước "biến code thành bảng". Mở Terminal:

1.  **Tạo file hướng dẫn migration:**
    ```bash
    dotnet ef migrations add InitialDb
    ```
    *Nếu thành công, thư mục `Migrations` sẽ xuất hiện.*

2.  **Thực thi lệnh tạo database:**
    ```bash
    dotnet ef database update
    ```
    *Lúc này Database mới chính thức được tạo trong SQL Server.*

---

## 5. SINH DỮ LIỆU MẪU (DATA SEEDING) VỚI BOGUS 🎲

Để test API, ta cần dữ liệu. Thay vì nhập tay hàng chục dòng vào SQL, ta dùng code để sinh dữ liệu giả (nhưng nhìn như thật). Chúng ta sẽ dùng thư viện **Bogus**.

### Bước 1: Cài đặt Bogus
```bash
dotnet add package Bogus
```

### Bước 2: Viết hàm sinh dữ liệu
Tạo file `Data/DbSeeder.cs`:

```csharp
using Bogus;
using DNU.Ebook.Models;

namespace DNU.Ebook.Data
{
    public static class DbSeeder
    {
        // Hàm này sẽ được gọi khi ứng dụng khởi chạy
        public static void Seed(ApplicationDbContext context)
        {
            // 1. Kiểm tra xem Database có dữ liệu chưa. Nếu có rồi thì không thêm nữa.
            if (context.Categories.Any() || context.Books.Any())
            {
                return;
            }

            // 2. Tạo dữ liệu giả cho Categories
            // Dùng thư viện Bogus để sinh tên tiếng Việt
            var categoryFaker = new Faker<Category>()
                .RuleFor(c => c.Name, f => f.Commerce.Categories(1)[0]); // Sinh tên danh mục ngẫu nhiên

            var categories = categoryFaker.Generate(5); // Tạo 5 danh mục
            context.Categories.AddRange(categories);
            context.SaveChanges(); // Lưu Categories trước để có ID dùng cho Book

            // 3. Tạo dữ liệu giả cho Books
            var bookFaker = new Faker<Book>()
                .RuleFor(b => b.Title, f => f.Lorem.Sentence(3)) // Tên sách: câu 3 từ
                .RuleFor(b => b.Author, f => f.Name.FullName())   // Tên tác giả
                .RuleFor(b => b.Price, f => f.Random.Decimal(50000, 500000)) // Giá từ 50k - 500k
                .RuleFor(b => b.CategoryId, f => f.PickRandom(categories).Id); // Chọn ngẫu nhiên 1 Category ID đã có

            var books = bookFaker.Generate(20); // Tạo 20 cuốn sách
            context.Books.AddRange(books);
            context.SaveChanges();
        }
    }
}
```

### Bước 3: Gọi hàm Seed trong Program.cs
Sửa file `Program.cs` (sau dòng `var app = builder.Build();` và trước `app.Run()`):

```csharp
// ...
var app = builder.Build();

// --- BẮT ĐẦU VÙNG SEED DATA ---
// Tạo ra một scope xử lý tạm thời để lấy DbContext
using (var scope = app.Services.CreateScope())
{
    var services = scope.ServiceProvider;
    var context = services.GetRequiredService<ApplicationDbContext>();
    
    // Gọi hàm Seed
    DbSeeder.Seed(context);
}
// --- KẾT THÚC VÙNG SEED DATA ---

// Configure the HTTP request pipeline...
```

### Bước 4: Kiểm tra kết quả
1.  Chạy dự án (`dotnet run` hoặc F5).
2.  Mở SQL Server Management Studio (SSMS).
3.  Query bảng `Books` và `Categories` -> Bạn sẽ thấy dữ liệu đã được tự động sinh ra! 🎉

---

## TỔNG KẾT
Bạn đã học được:
1.  Setup môi trường SQL Server + EF Core.
2.  Cách viết Model với các Attribute chuẩn.
3.  Cách cấu hình DI và Connection String.
4.  Cách dùng **Bogus** để tạo dữ liệu test nhanh chóng.
