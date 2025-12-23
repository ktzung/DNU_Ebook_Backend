# 🟦 CHƯƠNG 05
# **ENTITY FRAMEWORK CORE — PHẦN 1: CODE FIRST & MIGRATIONS**

Entity Framework Core (EF Core) là ORM (Object-Relational Mapper) giúp bạn làm việc với Database bằng C# objects thay vì viết SQL thuần. Chương này tập trung vào **Code First** — cách tiếp cận hiện đại nhất.

---

# 🎯 MỤC TIÊU HỌC TẬP

Sau chương này, bạn sẽ:

- Hiểu ORM là gì và tại sao cần EF Core
- Thiết kế Database bằng C# classes (Code First)
- Cấu hình DbContext và Connection String
- Sử dụng Migrations để tạo/cập nhật Database
- Hiểu Data Annotations và Fluent API
- Tạo Database cho dự án E-Shop

---

# 1. **ORM VÀ ENTITY FRAMEWORK CORE**

## 1.1. ORM là gì?

**Object-Relational Mapping** — Ánh xạ giữa Object (C#) và Relational Database (SQL).

### 🏠 Ví dụ đời sống

ORM giống như **phiên dịch viên**:
- Bạn nói tiếng Việt (C# objects)
- Database hiểu tiếng Anh (SQL)
- ORM dịch giữa 2 ngôn ngữ

```csharp
// Bạn viết C#
var products = await _db.Products.Where(p => p.Price > 1000).ToListAsync();

// EF Core tự động tạo SQL
// SELECT * FROM Products WHERE Price > 1000
```

---

## 1.2. Tại sao dùng EF Core?

### ❌ Không dùng ORM (ADO.NET thuần)

```csharp
public List<Product> GetProducts()
{
    var products = new List<Product>();
    using var connection = new SqlConnection(connectionString);
    connection.Open();
    
    var command = new SqlCommand("SELECT Id, Name, Price FROM Products", connection);
    using var reader = command.ExecuteReader();
    
    while (reader.Read())
    {
        products.Add(new Product
        {
            Id = reader.GetInt32(0),
            Name = reader.GetString(1),
            Price = reader.GetDecimal(2)
        });
    }
    
    return products;
}
```

**Vấn đề:**
- Nhiều boilerplate code
- Dễ lỗi (typo, SQL injection)
- Khó maintain

### ✅ Dùng EF Core

```csharp
public async Task<List<Product>> GetProducts()
{
    return await _db.Products.ToListAsync();
}
```

**Lợi ích:**
- Ngắn gọn, dễ đọc
- Type-safe (compile-time checking)
- An toàn (tự động prevent SQL injection)
- Dễ maintain

---

# 2. **CODE FIRST WORKFLOW**

## 2.1. Code First là gì?

**Code First** = Viết C# code trước → Database được tạo tự động từ code.

### Workflow:

```
1. Viết Entity Classes (Product, Category)
   ↓
2. Tạo DbContext
   ↓
3. Add Migration
   ↓
4. Update Database
   ↓
5. Database được tạo tự động!
```

---

# 3. **ENTITY CLASSES — ĐỊNH NGHĨA MODEL**

## 3.1. Entity Class cơ bản

```csharp
// Models/Product.cs
public class Product
{
    public int Id { get; set; }                    // Primary Key (tự động)
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public int Stock { get; set; }
    public DateTime CreatedAt { get; set; }
    
    // Foreign Key
    public int CategoryId { get; set; }
    
    // Navigation Property
    public Category Category { get; set; } = null!;
}
```

---

## 3.2. Data Annotations

### 🟢 [Key] — Primary Key

```csharp
public class Product
{
    [Key]
    public int ProductId { get; set; } // Không phải "Id" → cần [Key]
}
```

**Lưu ý:** Nếu property tên là `Id` hoặc `{ClassName}Id` → tự động là Primary Key.

---

### 🟢 [Required] — Không cho phép NULL

```csharp
public class Product
{
    [Required]
    [MaxLength(200)]
    public string Name { get; set; } = string.Empty;
}
```

---

### 🟢 [MaxLength] / [StringLength] — Độ dài string

```csharp
public class Product
{
    [MaxLength(200)]
    public string Name { get; set; } = string.Empty;
    
    [StringLength(1000, MinimumLength = 10)]
    public string Description { get; set; } = string.Empty;
}
```

---

### 🟢 [Column] — Đặt tên cột khác

```csharp
public class Product
{
    [Column("product_name", TypeName = "nvarchar(200)")]
    public string Name { get; set; } = string.Empty;
}
```

---

### 🟢 [ForeignKey] — Foreign Key

```csharp
public class Product
{
    public int CategoryId { get; set; }
    
    [ForeignKey("CategoryId")]
    public Category Category { get; set; } = null!;
}
```

---

## 3.3. Ví dụ hoàn chỉnh: E-Shop Models

```csharp
// Models/Category.cs
public class Category
{
    public int Id { get; set; }
    
    [Required]
    [MaxLength(100)]
    public string Name { get; set; } = string.Empty;
    
    [MaxLength(500)]
    public string Description { get; set; } = string.Empty;
    
    public DateTime CreatedAt { get; set; }
    
    // Navigation Property (1 Category → Many Products)
    public ICollection<Product> Products { get; set; } = new List<Product>();
}

// Models/Product.cs
public class Product
{
    public int Id { get; set; }
    
    [Required]
    [MaxLength(200)]
    public string Name { get; set; } = string.Empty;
    
    [MaxLength(2000)]
    public string Description { get; set; } = string.Empty;
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal Price { get; set; }
    
    public int Stock { get; set; }
    
    [MaxLength(500)]
    public string? ImageUrl { get; set; }
    
    public DateTime CreatedAt { get; set; }
    public DateTime? UpdatedAt { get; set; }
    
    // Foreign Key
    public int CategoryId { get; set; }
    
    // Navigation Property
    public Category Category { get; set; } = null!;
}
```

---

# 4. **DBCONTEXT — CỔNG KẾT NỐI DATABASE**

## 4.1. DbContext là gì?

**DbContext** là class chính để tương tác với Database.

### 🏠 Ví dụ đời sống

DbContext giống như **quầy giao dịch ngân hàng**:
- Bạn muốn rút tiền (query data) → Qua quầy
- Bạn muốn gửi tiền (insert data) → Qua quầy
- Bạn muốn chuyển khoản (update data) → Qua quầy

---

## 4.2. Tạo DbContext

```csharp
// Data/AppDbContext.cs
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }
    
    // DbSet = Table trong Database
    public DbSet<Product> Products { get; set; } = null!;
    public DbSet<Category> Categories { get; set; } = null!;
    
    // Configuration (optional)
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // Fluent API configuration ở đây (Chương 6)
    }
}
```

---

## 4.3. Đăng ký DbContext trong Program.cs

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Đọc Connection String từ appsettings.json
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");

// Đăng ký DbContext với DI Container
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));

// ... rest of code
```

---

## 4.4. Connection String

```json
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=EShopDb;Trusted_Connection=True;TrustServerCertificate=True"
  }
}
```

**Giải thích:**
- `Server=.` — SQL Server trên máy local (. = localhost)
- `Database=EShopDb` — Tên database
- `Trusted_Connection=True` — Dùng Windows Authentication
- `TrustServerCertificate=True` — Bỏ qua SSL warning (chỉ dùng dev)

**Nếu dùng SQL Server Authentication:**

```json
"DefaultConnection": "Server=.;Database=EShopDb;User Id=sa;Password=YourPassword;TrustServerCertificate=True"
```

---

# 5. **MIGRATIONS — TẠO VÀ CẬP NHẬT DATABASE**

## 5.1. Migrations là gì?

**Migrations** = Lịch sử thay đổi Database dưới dạng code.

### 🏠 Ví dụ đời sống

Migrations giống như **Git cho Database**:
- Mỗi migration = 1 commit
- `Add-Migration` = git commit
- `Update-Database` = git push
- Có thể rollback về version trước

---

## 5.2. Cài đặt EF Core Tools

### Package Manager Console (Visual Studio)

```powershell
Install-Package Microsoft.EntityFrameworkCore.Tools
```

### .NET CLI (VS Code)

```powershell
dotnet tool install --global dotnet-ef
```

---

## 5.3. Tạo Migration đầu tiên

### Package Manager Console:

```powershell
Add-Migration InitialCreate
```

### .NET CLI:

```powershell
dotnet ef migrations add InitialCreate
```

**Kết quả:** Tạo folder `Migrations/` với file `XXXXXX_InitialCreate.cs`

---

## 5.4. Xem Migration Code

```csharp
// Migrations/20241202_InitialCreate.cs
public partial class InitialCreate : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // Tạo bảng Categories
        migrationBuilder.CreateTable(
            name: "Categories",
            columns: table => new
            {
                Id = table.Column<int>(nullable: false)
                    .Annotation("SqlServer:Identity", "1, 1"),
                Name = table.Column<string>(maxLength: 100, nullable: false),
                Description = table.Column<string>(maxLength: 500, nullable: false),
                CreatedAt = table.Column<DateTime>(nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Categories", x => x.Id);
            });
        
        // Tạo bảng Products
        migrationBuilder.CreateTable(
            name: "Products",
            columns: table => new
            {
                Id = table.Column<int>(nullable: false)
                    .Annotation("SqlServer:Identity", "1, 1"),
                Name = table.Column<string>(maxLength: 200, nullable: false),
                // ... other columns
                CategoryId = table.Column<int>(nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Products", x => x.Id);
                table.ForeignKey(
                    name: "FK_Products_Categories_CategoryId",
                    column: x => x.CategoryId,
                    principalTable: "Categories",
                    principalColumn: "Id",
                    onDelete: ReferentialAction.Cascade);
            });
    }
    
    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // Rollback: Xóa bảng
        migrationBuilder.DropTable(name: "Products");
        migrationBuilder.DropTable(name: "Categories");
    }
}
```

---

## 5.5. Apply Migration vào Database

### Package Manager Console:

```powershell
Update-Database
```

### .NET CLI:

```powershell
dotnet ef database update
```

**Kết quả:**
- Database `EShopDb` được tạo
- Bảng `Categories` và `Products` được tạo
- Foreign Key được tạo

---

## 5.6. Xem Database

Mở **SQL Server Management Studio (SSMS)**:

```sql
USE EShopDb;

-- Xem danh sách bảng
SELECT * FROM INFORMATION_SCHEMA.TABLES;

-- Xem cấu trúc bảng
EXEC sp_help 'Products';

-- Xem dữ liệu (rỗng lúc này)
SELECT * FROM Products;
SELECT * FROM Categories;
```

---

## 5.7. Thêm/Sửa Entity và Tạo Migration mới

### Ví dụ: Thêm property `IsActive` vào Product

```csharp
// Models/Product.cs
public class Product
{
    // ... existing properties
    
    public bool IsActive { get; set; } = true; // ← Thêm mới
}
```

### Tạo Migration:

```powershell
Add-Migration AddIsActiveToProduct
Update-Database
```

**Kết quả:** Cột `IsActive` được thêm vào bảng `Products`.

---

## 5.8. Seed Data — Dữ liệu mẫu

```csharp
// Data/AppDbContext.cs
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);
    
    // Seed Categories
    modelBuilder.Entity<Category>().HasData(
        new Category 
        { 
            Id = 1, 
            Name = "Electronics", 
            Description = "Electronic devices",
            CreatedAt = DateTime.Now
        },
        new Category 
        { 
            Id = 2, 
            Name = "Books", 
            Description = "Books and magazines",
            CreatedAt = DateTime.Now
        }
    );
    
    // Seed Products
    modelBuilder.Entity<Product>().HasData(
        new Product 
        { 
            Id = 1,
            Name = "Laptop Dell XPS 13",
            Description = "High-end laptop",
            Price = 1500,
            Stock = 10,
            CategoryId = 1,
            CreatedAt = DateTime.Now
        },
        new Product 
        { 
            Id = 2,
            Name = "iPhone 15 Pro",
            Description = "Latest iPhone",
            Price = 1200,
            Stock = 20,
            CategoryId = 1,
            CreatedAt = DateTime.Now
        }
    );
}
```

### Tạo Migration cho Seed Data:

```powershell
Add-Migration SeedData
Update-Database
```

---

# 6. **FLUENT API — CẤU HÌNH NÂNG CAO**

## 6.1. Data Annotations vs Fluent API

| Feature | Data Annotations | Fluent API |
|---------|------------------|------------|
| **Vị trí** | Trên property | Trong OnModelCreating |
| **Độ phức tạp** | Đơn giản | Phức tạp hơn |
| **Khả năng** | Hạn chế | Mạnh mẽ |
| **Ưu tiên** | Thấp | Cao (override Annotations) |

---

## 6.2. Cấu hình với Fluent API

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);
    
    // Configure Product
    modelBuilder.Entity<Product>(entity =>
    {
        // Table name
        entity.ToTable("Products");
        
        // Primary Key
        entity.HasKey(p => p.Id);
        
        // Property configuration
        entity.Property(p => p.Name)
            .IsRequired()
            .HasMaxLength(200);
        
        entity.Property(p => p.Price)
            .HasColumnType("decimal(18,2)")
            .IsRequired();
        
        entity.Property(p => p.CreatedAt)
            .HasDefaultValueSql("GETDATE()");
        
        // Index
        entity.HasIndex(p => p.Name);
        
        // Relationship (1 Category → Many Products)
        entity.HasOne(p => p.Category)
            .WithMany(c => c.Products)
            .HasForeignKey(p => p.CategoryId)
            .OnDelete(DeleteBehavior.Cascade);
    });
}
```

---

# 7. **BÀI TẬP THỰC HÀNH**

## 📝 Bài 1: Tạo Entity Order

Tạo entity `Order` với:
- Id (int, PK)
- OrderDate (DateTime)
- TotalAmount (decimal)
- Status (string) — "Pending", "Completed", "Cancelled"
- UserId (int, FK)

<details>
<summary>💡 Đáp án</summary>

```csharp
public class Order
{
    public int Id { get; set; }
    
    public DateTime OrderDate { get; set; }
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal TotalAmount { get; set; }
    
    [Required]
    [MaxLength(50)]
    public string Status { get; set; } = "Pending";
    
    public int UserId { get; set; }
}
```
</details>

---

## 📝 Bài 2: Tạo Migration

1. Thêm `DbSet<Order> Orders` vào `AppDbContext`
2. Tạo migration `AddOrderTable`
3. Apply migration

<details>
<summary>💡 Đáp án</summary>

```csharp
// AppDbContext.cs
public DbSet<Order> Orders { get; set; } = null!;
```

```powershell
Add-Migration AddOrderTable
Update-Database
```
</details>

---

## ❌ 6. CÁC LỖI THƯỜNG GẶP

### ❌ Lỗi 1: Quên chạy Migration

```csharp
// ❌ Vấn đề: Tạo migration nhưng không apply
dotnet ef migrations add InitialCreate
// Quên chạy: dotnet ef database update
// Database không được tạo → Lỗi khi chạy app

// ✅ Giải pháp: Luôn update database sau khi tạo migration
dotnet ef migrations add InitialCreate
dotnet ef database update // ✅
```

**🔍 Giải thích:** Migration chỉ là file script, phải chạy `database update` để apply vào database.

---

### ❌ Lỗi 2: Connection String sai

```csharp
// ❌ Vấn đề: Connection string không đúng
"Server=localhost;Database=MyDb;User=sa;Password=wrong" // ❌

// ✅ Giải pháp: Kiểm tra connection string
"Server=localhost;Database=MyDb;Integrated Security=True" // ✅ Windows Auth
// Hoặc
"Server=localhost;Database=MyDb;User Id=sa;Password=correct;TrustServerCertificate=True" // ✅ SQL Auth
```

**🔍 Giải thích:** Connection string sai sẽ không kết nối được database. Kiểm tra server name, database name, credentials.

---

### ❌ Lỗi 3: Migration conflict

```csharp
// ❌ Vấn đề: 2 người cùng tạo migration
// Dev A: dotnet ef migrations add AddProductTable
// Dev B: dotnet ef migrations add AddCategoryTable
// → Conflict khi merge

// ✅ Giải pháp: Merge migrations hoặc tạo migration mới
dotnet ef migrations add MergeMigrations
// Hoặc xóa migration conflict, tạo lại
```

**🔍 Giải thích:** Khi nhiều người cùng làm việc, có thể conflict migrations. Cần merge hoặc tạo migration mới.

---

### ❌ Lỗi 4: Quên cấu hình DbContext trong Program.cs

```csharp
// ❌ Vấn đề: Tạo DbContext nhưng không đăng ký
public class AppDbContext : DbContext { }
// Quên: builder.Services.AddDbContext<AppDbContext>()

// ✅ Giải pháp: Đăng ký trong Program.cs
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
```

**🔍 Giải thích:** DbContext phải được đăng ký trong DI container để sử dụng.

---

### ❌ Lỗi 5: Migration không sync với code

```csharp
// ❌ Vấn đề: Sửa Entity nhưng không tạo migration mới
public class Product
{
    public string Description { get; set; } // Thêm property mới
}
// Quên: dotnet ef migrations add AddDescription
// Database không có column Description → Lỗi

// ✅ Giải pháp: Luôn tạo migration khi thay đổi Entity
dotnet ef migrations add AddDescription
dotnet ef database update
```

**🔍 Giải thích:** Mọi thay đổi Entity phải có migration tương ứng. Database và code phải sync.

---

## 🎯 7. CASE STUDY / VÍ DỤ THỰC TẾ

### Case Study 1: Database Design cho E-Shop

**Yêu cầu:** Thiết kế database cho hệ thống bán hàng online với Products, Categories, Orders, Users.

```csharp
// 1. Category Entity
public class Category
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string? Description { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    
    // Navigation property
    public ICollection<Product> Products { get; set; } = new List<Product>();
}

// 2. Product Entity
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string? Description { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
    public string? ImageUrl { get; set; }
    public bool IsActive { get; set; } = true;
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    
    // Foreign Key
    public int CategoryId { get; set; }
    public Category Category { get; set; } = null!;
    
    // Navigation properties
    public ICollection<OrderItem> OrderItems { get; set; } = new List<OrderItem>();
}

// 3. User Entity
public class User
{
    public int Id { get; set; }
    public string Email { get; set; } = string.Empty;
    public string FullName { get; set; } = string.Empty;
    public string PasswordHash { get; set; } = string.Empty;
    public string Role { get; set; } = "Customer";
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    
    public ICollection<Order> Orders { get; set; } = new List<Order>();
}

// 4. Order Entity
public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; } = DateTime.UtcNow;
    public decimal TotalAmount { get; set; }
    public string Status { get; set; } = "Pending";
    public string? ShippingAddress { get; set; }
    
    public int UserId { get; set; }
    public User User { get; set; } = null!;
    
    public ICollection<OrderItem> OrderItems { get; set; } = new List<OrderItem>();
}

// 5. OrderItem Entity (Join table)
public class OrderItem
{
    public int Id { get; set; }
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
    
    public int OrderId { get; set; }
    public Order Order { get; set; } = null!;
    
    public int ProductId { get; set; }
    public Product Product { get; set; } = null!;
}

// 6. DbContext
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    
    public DbSet<Category> Categories { get; set; }
    public DbSet<Product> Products { get; set; }
    public DbSet<User> Users { get; set; }
    public DbSet<Order> Orders { get; set; }
    public DbSet<OrderItem> OrderItems { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Product - Category (Many-to-One)
        modelBuilder.Entity<Product>()
            .HasOne(p => p.Category)
            .WithMany(c => c.Products)
            .HasForeignKey(p => p.CategoryId)
            .OnDelete(DeleteBehavior.Restrict); // Không xóa Category nếu có Product
        
        // Order - User (Many-to-One)
        modelBuilder.Entity<Order>()
            .HasOne(o => o.User)
            .WithMany(u => u.Orders)
            .HasForeignKey(o => o.UserId)
            .OnDelete(DeleteBehavior.Restrict);
        
        // OrderItem - Order (Many-to-One)
        modelBuilder.Entity<OrderItem>()
            .HasOne(oi => oi.Order)
            .WithMany(o => o.OrderItems)
            .HasForeignKey(oi => oi.OrderId)
            .OnDelete(DeleteBehavior.Cascade); // Xóa OrderItem khi xóa Order
        
        // OrderItem - Product (Many-to-One)
        modelBuilder.Entity<OrderItem>()
            .HasOne(oi => oi.Product)
            .WithMany(p => p.OrderItems)
            .HasForeignKey(oi => oi.ProductId)
            .OnDelete(DeleteBehavior.Restrict);
        
        // Indexes
        modelBuilder.Entity<Product>()
            .HasIndex(p => p.Name);
        
        modelBuilder.Entity<User>()
            .HasIndex(u => u.Email)
            .IsUnique();
    }
}
```

**Best practices:**
- Relationships rõ ràng
- Cascade delete phù hợp
- Indexes cho performance
- Timestamps (CreatedAt)
- Soft delete (IsActive)

---

## ✅ 8. BEST PRACTICES

### 8.1. Entity Design Best Practices

```csharp
// ✅ Đúng: Entity có Id, timestamps, navigation properties
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? UpdatedAt { get; set; }
    
    public int CategoryId { get; set; }
    public Category Category { get; set; } = null!;
}

// ❌ Sai: Thiếu Id, không có navigation properties
public class Product
{
    public string Name { get; set; } // ❌ Thiếu Id
    public int CategoryId { get; set; } // ❌ Không có navigation property
}
```

### 8.2. Migration Best Practices

```csharp
// ✅ Đúng: Migration name rõ ràng
dotnet ef migrations add AddProductDescriptionColumn
dotnet ef migrations add CreateOrderTable

// ❌ Sai: Tên migration không rõ ràng
dotnet ef migrations add Migration1
dotnet ef migrations add Update
```

### 8.3. DbContext Best Practices

```csharp
// ✅ Đúng: DbContext với constructor injection
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
}

// ✅ Đúng: Cấu hình trong OnModelCreating
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Product>()
        .Property(p => p.Price)
        .HasPrecision(18, 2);
}
```

---

# 📝 9. QUICK NOTES

### Code First Workflow:
1. Tạo Entity classes
2. Tạo DbContext
3. Đăng ký DbContext trong Program.cs
4. Tạo Migration: `dotnet ef migrations add MigrationName`
5. Apply Migration: `dotnet ef database update`

### Data Annotations:
- `[Key]`: Primary key
- `[Required]`: Not null
- `[MaxLength]`: Độ dài tối đa
- `[Column]`: Tên column trong DB
- `[Table]`: Tên table

### Fluent API:
- `HasKey()`: Primary key
- `Property().IsRequired()`: Not null
- `HasOne().WithMany()`: Relationships
- `OnDelete()`: Cascade delete

### Best Practices:
- ✅ Code First cho dự án mới
- ✅ Migration name rõ ràng
- ✅ Timestamps cho mọi Entity
- ✅ Navigation properties
- ✅ Indexes cho performance

---

# 🧪 10. MINI TEST

1. **Code First có nghĩa là gì?**
   - A. Viết code SQL trước
   - B. Viết C# classes trước, Database tự tạo
   - C. Tạo Database trước, sau đó code

2. **DbContext có vai trò gì?**
   - A. Kết nối Database
   - B. Lưu trữ dữ liệu
   - C. Cả hai

3. **Migration dùng để làm gì?**
   - A. Tạo và cập nhật Database schema
   - B. Backup Database
   - C. Tối ưu performance

<details>
<summary>💡 Đáp án</summary>

1. **B** - Viết C# classes trước, Database tự tạo
2. **A** - Kết nối Database (DbContext không lưu trữ, chỉ là cổng giao tiếp)
3. **A** - Tạo và cập nhật Database schema
</details>

---

# 📌 TÓM TẮT CHƯƠNG

✅ **EF Core** là ORM giúp làm việc với Database bằng C# objects  
✅ **Code First** = Viết code → Database tự tạo  
✅ **Entity Classes** định nghĩa cấu trúc Database  
✅ **DbContext** là cổng kết nối Database  
✅ **Migrations** quản lý lịch sử thay đổi Database  
✅ **Data Annotations** cấu hình đơn giản, **Fluent API** mạnh mẽ hơn  

---

**Chương tiếp theo: [06. RESTful Web API →](../Module_02_WebAPI_Service/01_Restful_Web_API.md)**
