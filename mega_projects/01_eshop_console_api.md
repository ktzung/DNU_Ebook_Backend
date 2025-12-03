# 🧪 MEGA PROJECT 01
# **E-SHOP API — RESTful API HOÀN CHỈNH**

Dự án này xây dựng một **RESTful API hoàn chỉnh** cho ứng dụng thương mại điện tử E-Shop, bao gồm:
- Authentication & Authorization với JWT
- CRUD operations cho Products, Categories, Orders
- Repository Pattern & Service Layer
- Global Error Handling
- Swagger Documentation

---

# 🎯 MỤC TIÊU DỰ ÁN

Sau khi hoàn thành dự án, bạn sẽ:

- ✅ Xây dựng được API theo chuẩn RESTful
- ✅ Triển khai Authentication/Authorization với JWT
- ✅ Thiết kế kiến trúc 3-tier (Controller → Service → Repository)
- ✅ Áp dụng best practices: DI, Async/Await, Error Handling
- ✅ Tạo tài liệu API với Swagger
- ✅ Test API với Postman
- ✅ Deploy lên IIS hoặc Azure

---

# 1. **PHÂN TÍCH YÊU CẦU**

## 1.1. Functional Requirements

### 🟢 Public APIs (Không cần đăng nhập)

**Products:**
- GET /api/products — Danh sách sản phẩm (phân trang, tìm kiếm)
- GET /api/products/{id} — Chi tiết sản phẩm
- GET /api/products/category/{categoryId} — Sản phẩm theo danh mục

**Categories:**
- GET /api/categories — Danh sách danh mục
- GET /api/categories/{id} — Chi tiết danh mục

**Auth:**
- POST /api/auth/register — Đăng ký
- POST /api/auth/login — Đăng nhập (trả về JWT token)

---

### 🟡 Protected APIs (Cần đăng nhập — User role)

**Orders:**
- POST /api/orders — Tạo đơn hàng mới
- GET /api/orders/my — Xem đơn hàng của mình
- GET /api/orders/{id} — Chi tiết đơn hàng

**Profile:**
- GET /api/users/profile — Thông tin cá nhân
- PUT /api/users/profile — Cập nhật thông tin

---

### 🔴 Admin APIs (Chỉ Admin)

**Products Management:**
- POST /api/products — Tạo sản phẩm mới
- PUT /api/products/{id} — Cập nhật sản phẩm
- DELETE /api/products/{id} — Xóa sản phẩm

**Categories Management:**
- POST /api/categories — Tạo danh mục
- PUT /api/categories/{id} — Cập nhật danh mục
- DELETE /api/categories/{id} — Xóa danh mục

**Orders Management:**
- GET /api/orders — Tất cả đơn hàng
- PUT /api/orders/{id}/status — Cập nhật trạng thái

**Users Management:**
- GET /api/users — Danh sách user
- PUT /api/users/{id}/role — Thay đổi role

---

## 1.2. Non-Functional Requirements

- **Security:** JWT Authentication, Password hashing, HTTPS
- **Performance:** Caching, Async operations, Pagination
- **Scalability:** Stateless API, Database connection pooling
- **Maintainability:** Clean architecture, SOLID principles
- **Documentation:** Swagger/OpenAPI

---

# 2. **THIẾT KẾ DATABASE**

## 2.1. Database Schema

```sql
-- Users Table
CREATE TABLE Users (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Email NVARCHAR(200) NOT NULL UNIQUE,
    PasswordHash NVARCHAR(500) NOT NULL,
    FullName NVARCHAR(200) NOT NULL,
    Role NVARCHAR(50) NOT NULL DEFAULT 'User', -- 'User' or 'Admin'
    CreatedAt DATETIME2 NOT NULL DEFAULT GETDATE(),
    UpdatedAt DATETIME2 NULL
);

-- Categories Table
CREATE TABLE Categories (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Name NVARCHAR(100) NOT NULL,
    Description NVARCHAR(500) NULL,
    CreatedAt DATETIME2 NOT NULL DEFAULT GETDATE()
);

-- Products Table
CREATE TABLE Products (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Name NVARCHAR(200) NOT NULL,
    Description NVARCHAR(2000) NULL,
    Price DECIMAL(18,2) NOT NULL,
    Stock INT NOT NULL DEFAULT 0,
    ImageUrl NVARCHAR(500) NULL,
    CategoryId INT NOT NULL,
    IsActive BIT NOT NULL DEFAULT 1,
    CreatedAt DATETIME2 NOT NULL DEFAULT GETDATE(),
    UpdatedAt DATETIME2 NULL,
    FOREIGN KEY (CategoryId) REFERENCES Categories(Id)
);

-- Orders Table
CREATE TABLE Orders (
    Id INT PRIMARY KEY IDENTITY(1,1),
    UserId INT NOT NULL,
    TotalAmount DECIMAL(18,2) NOT NULL,
    Status NVARCHAR(50) NOT NULL DEFAULT 'Pending',
    CreatedAt DATETIME2 NOT NULL DEFAULT GETDATE(),
    UpdatedAt DATETIME2 NULL,
    FOREIGN KEY (UserId) REFERENCES Users(Id)
);

-- OrderItems Table
CREATE TABLE OrderItems (
    Id INT PRIMARY KEY IDENTITY(1,1),
    OrderId INT NOT NULL,
    ProductId INT NOT NULL,
    Quantity INT NOT NULL,
    UnitPrice DECIMAL(18,2) NOT NULL,
    FOREIGN KEY (OrderId) REFERENCES Orders(Id) ON DELETE CASCADE,
    FOREIGN KEY (ProductId) REFERENCES Products(Id)
);

-- Indexes
CREATE INDEX IX_Products_CategoryId ON Products(CategoryId);
CREATE INDEX IX_Orders_UserId ON Orders(UserId);
CREATE INDEX IX_OrderItems_OrderId ON OrderItems(OrderId);
```

---

## 2.2. Entity Relationship Diagram

```
Users (1) ────── (N) Orders
                       │
                       │ (1)
                       │
                       │
                      (N)
                  OrderItems
                       │
                       │ (N)
                       │
                       │ (1)
                    Products
                       │
                       │ (N)
                       │
                       │ (1)
                   Categories
```

---

# 3. **KIẾN TRÚC DỰ ÁN**

## 3.1. Project Structure

```
EShopAPI/
│
├── Controllers/              ← API Endpoints
│   ├── AuthController.cs
│   ├── ProductsController.cs
│   ├── CategoriesController.cs
│   ├── OrdersController.cs
│   └── UsersController.cs
│
├── Models/                   ← Entity Classes
│   ├── User.cs
│   ├── Product.cs
│   ├── Category.cs
│   ├── Order.cs
│   └── OrderItem.cs
│
├── DTOs/                     ← Data Transfer Objects
│   ├── Auth/
│   │   ├── RegisterRequest.cs
│   │   ├── LoginRequest.cs
│   │   └── AuthResponse.cs
│   ├── Products/
│   │   ├── ProductResponse.cs
│   │   ├── CreateProductRequest.cs
│   │   └── UpdateProductRequest.cs
│   └── Orders/
│       ├── CreateOrderRequest.cs
│       └── OrderResponse.cs
│
├── Services/                 ← Business Logic
│   ├── Interfaces/
│   │   ├── IAuthService.cs
│   │   ├── IProductService.cs
│   │   ├── IOrderService.cs
│   │   └── ITokenService.cs
│   └── Implementations/
│       ├── AuthService.cs
│       ├── ProductService.cs
│       ├── OrderService.cs
│       └── TokenService.cs
│
├── Repositories/             ← Data Access Layer
│   ├── Interfaces/
│   │   ├── IGenericRepository.cs
│   │   ├── IProductRepository.cs
│   │   └── IOrderRepository.cs
│   └── Implementations/
│       ├── GenericRepository.cs
│       ├── ProductRepository.cs
│       └── OrderRepository.cs
│
├── Data/
│   └── AppDbContext.cs       ← EF Core DbContext
│
├── Middleware/
│   └── GlobalExceptionMiddleware.cs
│
├── Helpers/
│   ├── JwtSettings.cs
│   └── PasswordHasher.cs
│
├── Program.cs                ← Entry point
├── appsettings.json
└── appsettings.Development.json
```

---

## 3.2. Kiến trúc 3-Tier

```
Client (Postman/Mobile App)
         ↓
    Controller ────────────────┐
         ↓                     │
    Service Layer              │ Dependency Injection
         ↓                     │
    Repository Layer           │
         ↓                     │
    DbContext ─────────────────┘
         ↓
    SQL Server Database
```

---

# 4. **IMPLEMENTATION**

## 4.1. Models (Entities)

```csharp
// Models/User.cs
public class User
{
    public int Id { get; set; }
    
    [Required]
    [EmailAddress]
    [MaxLength(200)]
    public string Email { get; set; } = string.Empty;
    
    [Required]
    public string PasswordHash { get; set; } = string.Empty;
    
    [Required]
    [MaxLength(200)]
    public string FullName { get; set; } = string.Empty;
    
    [Required]
    [MaxLength(50)]
    public string Role { get; set; } = "User"; // User, Admin
    
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? UpdatedAt { get; set; }
    
    // Navigation Properties
    public ICollection<Order> Orders { get; set; } = new List<Order>();
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
    
    public int CategoryId { get; set; }
    
    public bool IsActive { get; set; } = true;
    
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? UpdatedAt { get; set; }
    
    // Navigation Properties
    public Category Category { get; set; } = null!;
    public ICollection<OrderItem> OrderItems { get; set; } = new List<OrderItem>();
}

// Models/Category.cs
public class Category
{
    public int Id { get; set; }
    
    [Required]
    [MaxLength(100)]
    public string Name { get; set; } = string.Empty;
    
    [MaxLength(500)]
    public string Description { get; set; } = string.Empty;
    
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    
    // Navigation Properties
    public ICollection<Product> Products { get; set; } = new List<Product>();
}

// Models/Order.cs
public class Order
{
    public int Id { get; set; }
    
    public int UserId { get; set; }
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal TotalAmount { get; set; }
    
    [Required]
    [MaxLength(50)]
    public string Status { get; set; } = "Pending"; // Pending, Processing, Completed, Cancelled
    
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? UpdatedAt { get; set; }
    
    // Navigation Properties
    public User User { get; set; } = null!;
    public ICollection<OrderItem> OrderItems { get; set; } = new List<OrderItem>();
}

// Models/OrderItem.cs
public class OrderItem
{
    public int Id { get; set; }
    
    public int OrderId { get; set; }
    public int ProductId { get; set; }
    
    public int Quantity { get; set; }
    
    [Column(TypeName = "decimal(18,2)")]
    public decimal UnitPrice { get; set; }
    
    // Navigation Properties
    public Order Order { get; set; } = null!;
    public Product Product { get; set; } = null!;
}
```

---

## 4.2. DTOs (Data Transfer Objects)

```csharp
// DTOs/Auth/RegisterRequest.cs
public record RegisterRequest(
    string Email,
    string Password,
    string FullName
);

// DTOs/Auth/LoginRequest.cs
public record LoginRequest(
    string Email,
    string Password
);

// DTOs/Auth/AuthResponse.cs
public record AuthResponse(
    int UserId,
    string Email,
    string FullName,
    string Role,
    string Token,
    DateTime ExpiresAt
);

// DTOs/Products/ProductResponse.cs
public record ProductResponse(
    int Id,
    string Name,
    string Description,
    decimal Price,
    int Stock,
    string? ImageUrl,
    int CategoryId,
    string CategoryName,
    bool IsActive,
    DateTime CreatedAt
);

// DTOs/Products/CreateProductRequest.cs
public record CreateProductRequest(
    string Name,
    string Description,
    decimal Price,
    int Stock,
    string? ImageUrl,
    int CategoryId
);

// DTOs/Orders/CreateOrderRequest.cs
public record CreateOrderRequest(
    List<OrderItemRequest> Items
);

public record OrderItemRequest(
    int ProductId,
    int Quantity
);

// DTOs/Orders/OrderResponse.cs
public record OrderResponse(
    int Id,
    decimal TotalAmount,
    string Status,
    DateTime CreatedAt,
    List<OrderItemResponse> Items
);

public record OrderItemResponse(
    int ProductId,
    string ProductName,
    int Quantity,
    decimal UnitPrice,
    decimal Subtotal
);
```

---

## 4.3. DbContext

```csharp
// Data/AppDbContext.cs
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }
    
    public DbSet<User> Users { get; set; } = null!;
    public DbSet<Product> Products { get; set; } = null!;
    public DbSet<Category> Categories { get; set; } = null!;
    public DbSet<Order> Orders { get; set; } = null!;
    public DbSet<OrderItem> OrderItems { get; set; } = null!;
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // Seed Categories
        modelBuilder.Entity<Category>().HasData(
            new Category { Id = 1, Name = "Electronics", Description = "Electronic devices" },
            new Category { Id = 2, Name = "Clothing", Description = "Fashion items" },
            new Category { Id = 3, Name = "Books", Description = "Books and magazines" }
        );
        
        // Configure relationships
        modelBuilder.Entity<Product>()
            .HasOne(p => p.Category)
            .WithMany(c => c.Products)
            .HasForeignKey(p => p.CategoryId)
            .OnDelete(DeleteBehavior.Restrict);
        
        modelBuilder.Entity<Order>()
            .HasOne(o => o.User)
            .WithMany(u => u.Orders)
            .HasForeignKey(o => o.UserId)
            .OnDelete(DeleteBehavior.Restrict);
        
        // Indexes
        modelBuilder.Entity<User>()
            .HasIndex(u => u.Email)
            .IsUnique();
        
        modelBuilder.Entity<Product>()
            .HasIndex(p => p.CategoryId);
        
        modelBuilder.Entity<Order>()
            .HasIndex(o => o.UserId);
    }
}
```

---

## 4.4. Repository Pattern

```csharp
// Repositories/Interfaces/IGenericRepository.cs
public interface IGenericRepository<T> where T : class
{
    Task<List<T>> GetAllAsync();
    Task<T?> GetByIdAsync(int id);
    Task<T> CreateAsync(T entity);
    Task<T> UpdateAsync(T entity);
    Task DeleteAsync(int id);
    Task<bool> ExistsAsync(int id);
}

// Repositories/Implementations/GenericRepository.cs
public class GenericRepository<T> : IGenericRepository<T> where T : class
{
    protected readonly AppDbContext _context;
    protected readonly DbSet<T> _dbSet;
    
    public GenericRepository(AppDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }
    
    public virtual async Task<List<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }
    
    public virtual async Task<T?> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }
    
    public virtual async Task<T> CreateAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
        await _context.SaveChangesAsync();
        return entity;
    }
    
    public virtual async Task<T> UpdateAsync(T entity)
    {
        _dbSet.Update(entity);
        await _context.SaveChangesAsync();
        return entity;
    }
    
    public virtual async Task DeleteAsync(int id)
    {
        var entity = await GetByIdAsync(id);
        if (entity != null)
        {
            _dbSet.Remove(entity);
            await _context.SaveChangesAsync();
        }
    }
    
    public virtual async Task<bool> ExistsAsync(int id)
    {
        return await _dbSet.FindAsync(id) != null;
    }
}

// Repositories/Interfaces/IProductRepository.cs
public interface IProductRepository : IGenericRepository<Product>
{
    Task<List<Product>> GetByCategoryAsync(int categoryId);
    Task<List<Product>> SearchAsync(string searchTerm, int pageNumber, int pageSize);
    Task<List<Product>> GetActiveProductsAsync();
}

// Repositories/Implementations/ProductRepository.cs
public class ProductRepository : GenericRepository<Product>, IProductRepository
{
    public ProductRepository(AppDbContext context) : base(context)
    {
    }
    
    public override async Task<List<Product>> GetAllAsync()
    {
        return await _dbSet
            .Include(p => p.Category)
            .ToListAsync();
    }
    
    public override async Task<Product?> GetByIdAsync(int id)
    {
        return await _dbSet
            .Include(p => p.Category)
            .FirstOrDefaultAsync(p => p.Id == id);
    }
    
    public async Task<List<Product>> GetByCategoryAsync(int categoryId)
    {
        return await _dbSet
            .Include(p => p.Category)
            .Where(p => p.CategoryId == categoryId && p.IsActive)
            .ToListAsync();
    }
    
    public async Task<List<Product>> SearchAsync(string searchTerm, int pageNumber, int pageSize)
    {
        return await _dbSet
            .Include(p => p.Category)
            .Where(p => p.IsActive && p.Name.Contains(searchTerm))
            .OrderBy(p => p.Name)
            .Skip((pageNumber - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync();
    }
    
    public async Task<List<Product>> GetActiveProductsAsync()
    {
        return await _dbSet
            .Include(p => p.Category)
            .Where(p => p.IsActive)
            .ToListAsync();
    }
}
```

---

# 5. **HƯỚNG DẪN HOÀN THÀNH**

## Giai đoạn 1: Setup Project (Tuần 13)
1. Tạo project ASP.NET Core Web API
2. Cài đặt packages cần thiết
3. Tạo Models và DbContext
4. Tạo Migration và Update Database
5. Seed data mẫu

## Giai đoạn 2: Repository & Service (Tuần 13)
1. Implement Repository Pattern
2. Implement Service Layer
3. Đăng ký DI trong Program.cs

## Giai đoạn 3: Authentication (Tuần 14)
1. Implement TokenService (JWT)
2. Implement AuthService
3. Tạo AuthController (Register, Login)
4. Test với Postman

## Giai đoạn 4: CRUD APIs (Tuần 14)
1. Products CRUD
2. Categories CRUD
3. Orders CRUD
4. Test với Postman

## Giai đoạn 5: Authorization (Tuần 14)
1. Implement [Authorize] attribute
2. Role-based authorization
3. Test với JWT token

## Giai đoạn 6: Polish & Deploy (Tuần 15)
1. Global Error Handling
2. Logging
3. Swagger Documentation
4. Deploy lên IIS/Azure

---

# 📌 TÓM TẮT

Dự án này tích hợp tất cả kiến thức đã học:
- ✅ C# Modern (Async, LINQ, Records)
- ✅ ASP.NET Core (DI, Middleware, Configuration)
- ✅ Entity Framework Core (Code First, Migrations, CRUD)
- ✅ RESTful API Design
- ✅ JWT Authentication & Authorization
- ✅ Repository Pattern & Clean Architecture

**File code đầy đủ sẽ được cung cấp trong repository GitHub của khóa học.**

---

**Bài tập mở rộng:**
1. Thêm chức năng giỏ hàng (Cart)
2. Thêm chức năng review sản phẩm
3. Thêm payment integration
4. Thêm email notification
5. Implement caching với Redis
