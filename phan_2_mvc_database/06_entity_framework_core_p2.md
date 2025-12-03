# 🟩 CHƯƠNG 06
# **ENTITY FRAMEWORK CORE — PHẦN 2: CRUD & RELATIONSHIPS**

Sau khi đã biết tạo Database bằng Code First, bây giờ chúng ta học cách thao tác dữ liệu: Create, Read, Update, Delete (CRUD) và làm việc với quan hệ giữa các bảng.

---

# 🎯 MỤC TIÊU HỌC TẬP

Sau chương này, bạn sẽ:

- Thành thạo CRUD operations với EF Core
- Hiểu Eager Loading, Lazy Loading, Explicit Loading
- Cấu hình quan hệ: 1-1, 1-N, N-N
- Xử lý cascade delete
- Tối ưu query performance
- Xây dựng chức năng CRUD hoàn chỉnh cho E-Shop

---

# 1. **CREATE — TẠO DỮ LIỆU MỚI**

## 1.1. Thêm 1 Entity

```csharp
public async Task<Product> CreateProductAsync(Product product)
{
    // Set timestamps
    product.CreatedAt = DateTime.UtcNow;
    
    // Add to DbSet
    _context.Products.Add(product);
    
    // Save to database
    await _context.SaveChangesAsync();
    
    // EF Core tự động set product.Id
    return product;
}
```

### 🎒 Ví dụ đời sống

Thêm sản phẩm giống như **thêm sách vào thư viện**:
1. Ghi thông tin sách vào phiếu (Add)
2. Đóng dấu xác nhận (SaveChanges)
3. Sách được gán mã số tự động (Auto-increment Id)

---

## 1.2. Thêm nhiều Entities

```csharp
public async Task CreateProductsAsync(List<Product> products)
{
    foreach (var product in products)
    {
        product.CreatedAt = DateTime.UtcNow;
    }
    
    // AddRange hiệu quả hơn Add nhiều lần
    _context.Products.AddRange(products);
    await _context.SaveChangesAsync();
}
```

---

## 1.3. Thêm Entity với Relationship

```csharp
public async Task<Product> CreateProductWithCategoryAsync(CreateProductRequest request)
{
    // Kiểm tra Category tồn tại
    var category = await _context.Categories.FindAsync(request.CategoryId);
    if (category == null)
        throw new Exception("Category not found");
    
    var product = new Product
    {
        Name = request.Name,
        Price = request.Price,
        CategoryId = request.CategoryId,
        CreatedAt = DateTime.UtcNow
    };
    
    _context.Products.Add(product);
    await _context.SaveChangesAsync();
    
    return product;
}
```

---

# 2. **READ — ĐỌC DỮ LIỆU**

## 2.1. Lấy tất cả

```csharp
// ❌ BAD: Load toàn bộ vào memory
public List<Product> GetAllProducts()
{
    return _context.Products.ToList(); // Sync - blocking!
}

// ✅ GOOD: Async
public async Task<List<Product>> GetAllProductsAsync()
{
    return await _context.Products.ToListAsync();
}
```

---

## 2.2. Lấy theo điều kiện (Where)

```csharp
// Lấy sản phẩm còn hàng
public async Task<List<Product>> GetInStockProductsAsync()
{
    return await _context.Products
        .Where(p => p.Stock > 0)
        .ToListAsync();
}

// Lấy sản phẩm theo category
public async Task<List<Product>> GetProductsByCategoryAsync(int categoryId)
{
    return await _context.Products
        .Where(p => p.CategoryId == categoryId)
        .ToListAsync();
}
```

---

## 2.3. Lấy 1 entity

### Find — Tìm theo Primary Key

```csharp
public async Task<Product?> GetProductByIdAsync(int id)
{
    // Find tìm trong memory trước, không tìm thấy mới query DB
    return await _context.Products.FindAsync(id);
}
```

### FirstOrDefault — Điều kiện phức tạp

```csharp
public async Task<Product?> GetProductByNameAsync(string name)
{
    return await _context.Products
        .FirstOrDefaultAsync(p => p.Name == name);
}

// First: Throw exception nếu không tìm thấy
// FirstOrDefault: Trả về null nếu không tìm thấy (Prefer)
```

### Single — Đảm bảo chỉ 1 kết quả

```csharp
public async Task<User?> GetUserByEmailAsync(string email)
{
    // SingleOrDefault: Throw exception nếu có > 1 kết quả
    return await _context.Users
        .SingleOrDefaultAsync(u => u.Email == email);
}
```

---

## 2.4. Phân trang

```csharp
public async Task<PagedResult<Product>> GetProductsPagedAsync(int pageNumber, int pageSize)
{
    var totalCount = await _context.Products.CountAsync();
    
    var products = await _context.Products
        .OrderBy(p => p.Name)
        .Skip((pageNumber - 1) * pageSize)
        .Take(pageSize)
        .ToListAsync();
    
    return new PagedResult<Product>
    {
        Items = products,
        TotalCount = totalCount,
        PageNumber = pageNumber,
        PageSize = pageSize,
        TotalPages = (int)Math.Ceiling(totalCount / (double)pageSize)
    };
}
```

---

## 2.5. Sắp xếp

```csharp
// Sắp xếp tăng dần
var products = await _context.Products
    .OrderBy(p => p.Price)
    .ToListAsync();

// Sắp xếp giảm dần
var products = await _context.Products
    .OrderByDescending(p => p.CreatedAt)
    .ToListAsync();

// Sắp xếp nhiều trường
var products = await _context.Products
    .OrderBy(p => p.CategoryId)
    .ThenByDescending(p => p.Price)
    .ToListAsync();
```

---

# 3. **UPDATE — CẬP NHẬT DỮ LIỆU**

## 3.1. Update cơ bản

```csharp
public async Task<Product> UpdateProductAsync(int id, UpdateProductRequest request)
{
    // 1. Lấy entity từ DB
    var product = await _context.Products.FindAsync(id);
    if (product == null)
        throw new Exception("Product not found");
    
    // 2. Cập nhật properties
    product.Name = request.Name;
    product.Description = request.Description;
    product.Price = request.Price;
    product.Stock = request.Stock;
    product.UpdatedAt = DateTime.UtcNow;
    
    // 3. Save changes
    await _context.SaveChangesAsync();
    
    return product;
}
```

**Lưu ý:** EF Core tự động track changes, không cần gọi `Update()`.

---

## 3.2. Update một phần (Patch)

```csharp
public async Task UpdateProductStockAsync(int id, int newStock)
{
    var product = await _context.Products.FindAsync(id);
    if (product == null)
        throw new Exception("Product not found");
    
    // Chỉ update 1 field
    product.Stock = newStock;
    product.UpdatedAt = DateTime.UtcNow;
    
    await _context.SaveChangesAsync();
}
```

---

## 3.3. Update không cần load entity (Attach)

```csharp
public async Task UpdateProductAsync(Product product)
{
    // Attach entity vào context
    _context.Products.Attach(product);
    
    // Đánh dấu entity đã modified
    _context.Entry(product).State = EntityState.Modified;
    
    await _context.SaveChangesAsync();
}
```

**Khi nào dùng:** API nhận toàn bộ object từ client, không cần load từ DB.

---

# 4. **DELETE — XÓA DỮ LIỆU**

## 4.1. Delete cơ bản

```csharp
public async Task DeleteProductAsync(int id)
{
    var product = await _context.Products.FindAsync(id);
    if (product == null)
        throw new Exception("Product not found");
    
    _context.Products.Remove(product);
    await _context.SaveChangesAsync();
}
```

---

## 4.2. Delete không cần load (ExecuteDelete - EF Core 7+)

```csharp
public async Task DeleteProductAsync(int id)
{
    await _context.Products
        .Where(p => p.Id == id)
        .ExecuteDeleteAsync();
}
```

**Ưu điểm:** Không cần load entity vào memory, execute SQL DELETE trực tiếp.

---

## 4.3. Soft Delete (Khuyến nghị cho Production)

```csharp
// Thêm property vào Entity
public class Product
{
    // ... other properties
    public bool IsDeleted { get; set; } = false;
    public DateTime? DeletedAt { get; set; }
}

// Soft delete
public async Task SoftDeleteProductAsync(int id)
{
    var product = await _context.Products.FindAsync(id);
    if (product == null)
        throw new Exception("Product not found");
    
    product.IsDeleted = true;
    product.DeletedAt = DateTime.UtcNow;
    
    await _context.SaveChangesAsync();
}

// Query chỉ lấy chưa xóa
public async Task<List<Product>> GetActiveProductsAsync()
{
    return await _context.Products
        .Where(p => !p.IsDeleted)
        .ToListAsync();
}
```

---

# 5. **RELATIONSHIPS — QUAN HỆ GIỮA CÁC BẢNG**

## 5.1. One-to-Many (1-N) — Category → Products

### Cấu hình

```csharp
public class Category
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    
    // Navigation Property
    public ICollection<Product> Products { get; set; } = new List<Product>();
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    
    // Foreign Key
    public int CategoryId { get; set; }
    
    // Navigation Property
    public Category Category { get; set; } = null!;
}

// Fluent API (trong OnModelCreating)
modelBuilder.Entity<Product>()
    .HasOne(p => p.Category)
    .WithMany(c => c.Products)
    .HasForeignKey(p => p.CategoryId)
    .OnDelete(DeleteBehavior.Restrict); // Không cho xóa Category nếu có Products
```

---

### Query với Include (Eager Loading)

```csharp
// ❌ BAD: N+1 Query Problem
public async Task<List<Product>> GetProductsAsync()
{
    var products = await _context.Products.ToListAsync();
    
    // Mỗi lần access product.Category → 1 query mới
    foreach (var product in products)
    {
        var categoryName = product.Category.Name; // N queries!
    }
    
    return products;
}

// ✅ GOOD: Eager Loading
public async Task<List<Product>> GetProductsAsync()
{
    return await _context.Products
        .Include(p => p.Category) // JOIN trong 1 query duy nhất
        .ToListAsync();
}
```

---

## 5.2. Many-to-Many (N-N) — Products ↔ Tags

### EF Core 5+ (Auto junction table)

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    
    // Navigation Property
    public ICollection<Tag> Tags { get; set; } = new List<Tag>();
}

public class Tag
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    
    // Navigation Property
    public ICollection<Product> Products { get; set; } = new List<Product>();
}

// Fluent API
modelBuilder.Entity<Product>()
    .HasMany(p => p.Tags)
    .WithMany(t => t.Products);
```

EF Core tự động tạo bảng junction `ProductTag`.

---

### Manual Junction Table

```csharp
public class ProductTag
{
    public int ProductId { get; set; }
    public Product Product { get; set; } = null!;
    
    public int TagId { get; set; }
    public Tag Tag { get; set; } = null!;
    
    public DateTime CreatedAt { get; set; }
}

// Fluent API
modelBuilder.Entity<ProductTag>()
    .HasKey(pt => new { pt.ProductId, pt.TagId }); // Composite key
```

---

## 5.3. One-to-One (1-1) — User → Profile

```csharp
public class User
{
    public int Id { get; set; }
    public string Email { get; set; } = string.Empty;
    
    // Navigation Property
    public UserProfile? Profile { get; set; }
}

public class UserProfile
{
    public int Id { get; set; }
    
    public int UserId { get; set; } // Foreign Key
    public User User { get; set; } = null!;
    
    public string Bio { get; set; } = string.Empty;
    public string Avatar { get; set; } = string.Empty;
}

// Fluent API
modelBuilder.Entity<User>()
    .HasOne(u => u.Profile)
    .WithOne(p => p.User)
    .HasForeignKey<UserProfile>(p => p.UserId);
```

---

# 6. **LOADING STRATEGIES**

## 6.1. Eager Loading (Include) — ✅ Recommended

```csharp
// Load Products với Categories trong 1 query
var products = await _context.Products
    .Include(p => p.Category)
    .ToListAsync();

// Multiple includes
var orders = await _context.Orders
    .Include(o => o.User)
    .Include(o => o.OrderItems)
        .ThenInclude(oi => oi.Product) // Nested include
    .ToListAsync();
```

---

## 6.2. Explicit Loading

```csharp
var product = await _context.Products.FindAsync(1);

// Load Category sau
await _context.Entry(product)
    .Reference(p => p.Category)
    .LoadAsync();

// Load Collection
var category = await _context.Categories.FindAsync(1);
await _context.Entry(category)
    .Collection(c => c.Products)
    .LoadAsync();
```

---

## 6.3. Lazy Loading (Không khuyến nghị)

```csharp
// Cần cài package: Microsoft.EntityFrameworkCore.Proxies

// Program.cs
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString)
           .UseLazyLoadingProxies()); // Enable lazy loading

// Entity với virtual keyword
public class Product
{
    public int Id { get; set; }
    public int CategoryId { get; set; }
    
    public virtual Category Category { get; set; } = null!; // virtual!
}

// Tự động load khi access
var product = await _context.Products.FindAsync(1);
var categoryName = product.Category.Name; // Query DB tự động
```

**Vấn đề:** N+1 queries, khó debug.

---

# 7. **CASCADE DELETE**

```csharp
// OnDelete(DeleteBehavior.Cascade) — Xóa cả Products khi xóa Category
modelBuilder.Entity<Product>()
    .HasOne(p => p.Category)
    .WithMany(c => c.Products)
    .OnDelete(DeleteBehavior.Cascade);

// OnDelete(DeleteBehavior.Restrict) — Không cho xóa Category nếu có Products
modelBuilder.Entity<Product>()
    .HasOne(p => p.Category)
    .WithMany(c => c.Products)
    .OnDelete(DeleteBehavior.Restrict);

// OnDelete(DeleteBehavior.SetNull) — Set CategoryId = NULL khi xóa Category
modelBuilder.Entity<Product>()
    .HasOne(p => p.Category)
    .WithMany(c => c.Products)
    .OnDelete(DeleteBehavior.SetNull);
```

---

# 8. **VÍ DỤ THỰC TẾ: PRODUCT SERVICE**

```csharp
public class ProductService
{
    private readonly AppDbContext _context;
    
    public ProductService(AppDbContext context)
    {
        _context = context;
    }
    
    // CREATE
    public async Task<Product> CreateAsync(CreateProductRequest request)
    {
        var product = new Product
        {
            Name = request.Name,
            Description = request.Description,
            Price = request.Price,
            Stock = request.Stock,
            CategoryId = request.CategoryId,
            CreatedAt = DateTime.UtcNow
        };
        
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
        
        return product;
    }
    
    // READ
    public async Task<List<Product>> GetAllAsync()
    {
        return await _context.Products
            .Include(p => p.Category)
            .Where(p => !p.IsDeleted)
            .OrderBy(p => p.Name)
            .ToListAsync();
    }
    
    public async Task<Product?> GetByIdAsync(int id)
    {
        return await _context.Products
            .Include(p => p.Category)
            .FirstOrDefaultAsync(p => p.Id == id && !p.IsDeleted);
    }
    
    // UPDATE
    public async Task<Product> UpdateAsync(int id, UpdateProductRequest request)
    {
        var product = await _context.Products.FindAsync(id);
        if (product == null || product.IsDeleted)
            throw new Exception("Product not found");
        
        product.Name = request.Name;
        product.Description = request.Description;
        product.Price = request.Price;
        product.Stock = request.Stock;
        product.UpdatedAt = DateTime.UtcNow;
        
        await _context.SaveChangesAsync();
        
        return product;
    }
    
    // DELETE (Soft)
    public async Task DeleteAsync(int id)
    {
        var product = await _context.Products.FindAsync(id);
        if (product == null)
            throw new Exception("Product not found");
        
        product.IsDeleted = true;
        product.DeletedAt = DateTime.UtcNow;
        
        await _context.SaveChangesAsync();
    }
    
    // SEARCH & FILTER
    public async Task<PagedResult<Product>> SearchAsync(
        string? searchTerm,
        int? categoryId,
        decimal? minPrice,
        decimal? maxPrice,
        int pageNumber,
        int pageSize)
    {
        var query = _context.Products
            .Include(p => p.Category)
            .Where(p => !p.IsDeleted)
            .AsQueryable();
        
        if (!string.IsNullOrEmpty(searchTerm))
            query = query.Where(p => p.Name.Contains(searchTerm));
        
        if (categoryId.HasValue)
            query = query.Where(p => p.CategoryId == categoryId.Value);
        
        if (minPrice.HasValue)
            query = query.Where(p => p.Price >= minPrice.Value);
        
        if (maxPrice.HasValue)
            query = query.Where(p => p.Price <= maxPrice.Value);
        
        var totalCount = await query.CountAsync();
        
        var products = await query
            .OrderBy(p => p.Name)
            .Skip((pageNumber - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync();
        
        return new PagedResult<Product>
        {
            Items = products,
            TotalCount = totalCount,
            PageNumber = pageNumber,
            PageSize = pageSize
        };
    }
}
```

---

# 9. **BÀI TẬP THỰC HÀNH**

## 📝 Bài 1: Implement Order CRUD

Tạo `OrderService` với CRUD cho Order và OrderItems.

<details>
<summary>💡 Gợi ý</summary>

```csharp
public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
{
    var order = new Order
    {
        UserId = request.UserId,
        Status = "Pending",
        CreatedAt = DateTime.UtcNow
    };
    
    foreach (var item in request.Items)
    {
        var product = await _context.Products.FindAsync(item.ProductId);
        if (product == null) throw new Exception("Product not found");
        
        order.OrderItems.Add(new OrderItem
        {
            ProductId = item.ProductId,
            Quantity = item.Quantity,
            UnitPrice = product.Price
        });
    }
    
    order.TotalAmount = order.OrderItems.Sum(oi => oi.Quantity * oi.UnitPrice);
    
    _context.Orders.Add(order);
    await _context.SaveChangesAsync();
    
    return order;
}
```
</details>

---

# 🧪 MINI TEST

1. **Eager Loading dùng method nào?**
   - A. Load()
   - B. Include()
   - C. Join()

2. **Soft Delete khác Hard Delete như thế nào?**
   - A. Không khác
   - B. Soft Delete đánh dấu, Hard Delete xóa hẳn
   - C. Soft Delete nhanh hơn

3. **N+1 Query Problem xảy ra khi nào?**
   - A. Không dùng Include
   - B. Dùng quá nhiều Include
   - C. Query quá phức tạp

<details>
<summary>💡 Đáp án</summary>

1. **B** - Include()
2. **B** - Soft Delete đánh dấu IsDeleted=true, Hard Delete xóa khỏi DB
3. **A** - Không dùng Include → Mỗi lần access navigation property = 1 query mới
</details>

---

# 📌 TÓM TẮT CHƯƠNG

✅ **CRUD**: Create (Add), Read (Find/Where), Update (modify + SaveChanges), Delete (Remove)  
✅ **Relationships**: 1-1, 1-N, N-N với Navigation Properties  
✅ **Loading**: Eager (Include - best), Explicit, Lazy (avoid)  
✅ **Cascade Delete**: Cascade, Restrict, SetNull  
✅ **Best Practices**: Async, Soft Delete, Paging, Include  

---

**Chương tiếp theo: [07. Razor View Engine →](./07_razor_view_engine.md)**
