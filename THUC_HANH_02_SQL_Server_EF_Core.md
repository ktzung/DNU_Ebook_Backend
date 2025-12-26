# 🟦 THỰC HÀNH 02: CÀI ĐẶT SQL SERVER & TẠO DATABASE VỚI EF CORE

## 🎯 MỤC TIÊU
- Cài đặt thành công SQL Server và công cụ quản lý SSMS.
- Hiểu cách lấy chuỗi kết nối (Connection String).
- Phân tích bài toán mẫu và thiết kế lược đồ CSDL cơ bản.
- Thực hiện quy trình **Code First** với Entity Framework Core để tạo Database.

---

## 1. CÀI ĐẶT SQL SERVER VÀ SSMS

Để làm việc với .NET Backend, chúng ta cần một hệ quản trị cơ sở dữ liệu (DBMS). SQL Server là lựa chọn phổ biến nhất.

### Bước 1: Cài đặt SQL Server (Backend Engine)
Bạn có thể chọn bản **Developer** (đầy đủ tính năng, miễn phí cho dev) hoặc **Express** (nhẹ, miễn phí).
1.  Tải về từ trang chủ Microsoft: [SQL Server Downloads](https://www.microsoft.com/en-us/sql-server/sql-server-downloads).
2.  Chọn **Developer** edition -> Download Now.
3.  Chạy file cài đặt -> Chọn **Basic** -> Accept -> Install.
4.  Sau khi cài xong, lưu lại thông tin **Connection String** hiển thị trên màn hình (để tham khảo).

### Bước 2: Cài đặt SQL Server Management Studio (SSMS)
Đây là công cụ giao diện để quản lý Database.
1.  Trong màn hình cài đặt SQL Server xong, có nút **Install SSMS**, hoặc tải trực tiếp tại: [Download SSMS](https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms).
2.  Tải về và cài đặt bình thường.

---

## 2. LẤY THAM SỐ KẾT NỐI (CONNECTION STRING)

Connection String là chuỗi thông tin giúp ứng dụng của bạn tìm và đăng nhập vào Database.

### Cách lấy Server Name:
1.  Mở **SSMS**.
2.  Tại cửa sổ **Connect to Server**, mục **Server name** chính là tên máy chủ của bạn (ví dụ: `DESKTOP-XYZ\SQLEXPRESS` hoặc `localhost` hoặc `.`).
3.  Nhấn **Connect**.

### Cấu trúc chuỗi kết nối:
Có 2 kiểu xác thực chính:

**Kiểu 1: Windows Authentication (Khuyên dùng cho Local)**
Sử dụng tài khoản Windows hiện tại, không cần pass.
```text
Server=YOUR_SERVER_NAME;Database=TEN_DATABASE_MION_TAO;Trusted_Connection=True;TrustServerCertificate=True;
```
*Ví dụ:*
`Server=.\SQLEXPRESS;Database=DNU_Ebook_Db;Trusted_Connection=True;TrustServerCertificate=True;`

**Kiểu 2: SQL Server Authentication (Dùng tài khoản sa)**
Dùng khi deploy hoặc cấu hình riêng user/pass.
```text
Server=YOUR_SERVER_NAME;Database=TEN_DATABASE_MION_TAO;User Id=sa;Password=YOUR_PASSWORD;TrustServerCertificate=True;
```

---

## 3. BÀI TOÁN MẪU: HỆ THỐNG QUẢN LÝ SÁCH (EBOOK)

**Mô tả:**
Xây dựng backend cho một website đọc sách online đơn giản. Hệ thống cần quản lý thông tin về các cuốn sách và phân loại của chúng.

**Yêu cầu dữ liệu:**
1.  **Thể loại (Category)**:
    - Cần lưu tên thể loại (Ví dụ: Tiểu thuyết, CNTT, Kinh tế).
    - Mỗi thể loại có nhiều sách.
2.  **Sách (Book)**:
    - Cần lưu tên sách, tác giả, giá tiền, mô tả.
    - Mỗi cuốn sách thuộc về một thể loại duy nhất.

---

## 4. XÁC ĐỊNH LƯỢC ĐỒ CSDL (DATABASE SCHEMA)

Dựa trên bài toán, ta thiết kế 2 bảng (Entity) có quan hệ **1-N** (Một - Nhiều).

**Bảng 1: Categories**
| Tên cột | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| **Id** | int | Khóa chính (PK), tự tăng |
| **Name** | nvarchar(100) | Tên thể loại |

**Bảng 2: Books**
| Tên cột | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| **Id** | int | Khóa chính (PK), tự tăng |
| **Title** | nvarchar(200) | Tên sách |
| **Author** | nvarchar(100) | Tác giả |
| **Price** | decimal | Giá bán |
| **CategoryId**| int | Khóa ngoại (FK) nối tới bảng Categories |

---

## 5. CÁC BƯỚC TẠO CSDL VỚI EF CORE (CODE FIRST)

Thực hiện trong Visual Studio hoặc VS Code.

### Bước 1: Tạo Project
Tạo mới một **ASP.NET Core Web API** project (ví dụ tên: `DNU.Ebook.Backend`).

### Bước 2: Cài đặt thư viện (NuGet Packages)
Mở Terminal tại thư mục project và chạy lệnh:
```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

### Bước 3: Tạo Models (Entities)
Tạo thư mục `Models` và thêm 2 class:

**Categories.cs**
```csharp
using System.ComponentModel.DataAnnotations;

namespace DNU.Ebook.Backend.Models
{
    public class Category
    {
        [Key]
        public int Id { get; set; }
        public required string Name { get; set; }

        // Quan hệ: 1 Category có nhiều Books
        // public ICollection<Book> Books { get; set; } // (Tuỳ chọn)
    }
}
```

**Book.cs**
```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace DNU.Ebook.Backend.Models
{
    public class Book
    {
        [Key]
        public int Id { get; set; }
        public required string Title { get; set; }
        public string? Author { get; set; }
        public decimal Price { get; set; }

        // Khóa ngoại
        public int CategoryId { get; set; }
        
        [ForeignKey("CategoryId")]
        public Category? Category { get; set; }
    }
}
```

### Bước 4: Tạo DbContext
Tạo thư mục `Data`, thêm class `ApplicationDbContext.cs`:
```csharp
using Microsoft.EntityFrameworkCore;
using DNU.Ebook.Backend.Models;

namespace DNU.Ebook.Backend.Data
{
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
        {
        }

        public DbSet<Category> Categories { get; set; }
        public DbSet<Book> Books { get; set; }
    }
}
```

### Bước 5: Cấu hình Connection String
Mở file `appsettings.json`, thêm đoạn `ConnectionStrings`:
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=DNU_Ebook_Demo;Trusted_Connection=True;TrustServerCertificate=True;"
  }
}
```
*(Thay dấu `.` bằng tên Server của bạn nếu cần)*

### Bước 6: Đăng ký DbContext trong Program.cs
Mở `Program.cs`, thêm trước dòng `var app = builder.Build();`:

```csharp
using Microsoft.EntityFrameworkCore;
using DNU.Ebook.Backend.Data;

// ...

// Đăng ký DbContext
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();
```

### Bước 7: Chạy Migration (Tạo Database)
Mở Terminal, chạy lần lượt:

1.  **Tạo bản snapshot đầu tiên:**
    ```bash
    dotnet ef migrations add InitialCreate
    ```
    *Lệnh này sẽ tạo thư mục `Migrations` chứa code tạo bảng.*

2.  **Đẩy vào Database:**
    ```bash
    dotnet ef database update
    ```
    *Lệnh này sẽ thực thi code trên SQL Server để tạo Database và Bảng.*

### Bước 8: Kiểm tra
Mở **SSMS**, Refresh thư mục **Databases**, bạn sẽ thấy `DNU_Ebook_Demo` cùng các bảng `Categories` và `Books`.

---
**🎉 CHÚC MỪNG! BẠN ĐÃ TẠO THÀNH CÔNG DATABASE TỪ CODE.**
