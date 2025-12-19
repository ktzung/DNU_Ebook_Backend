# 🟦 CHƯƠNG 01
# **C# MODERN CHO BACKEND DEVELOPMENT**

Trước khi xây dựng ứng dụng web với ASP.NET Core, bạn cần nắm vững các tính năng hiện đại của C# — những công cụ mạnh mẽ giúp code ngắn gọn, hiệu quả và dễ bảo trì hơn.

Chương này tập trung vào **các tính năng C# 10+** quan trọng cho Backend Developer.

---

# 🎯 MỤC TIÊU HỌC TẬP

Sau chương này, bạn sẽ:

- Hiểu **Async/Await** và vì sao Backend bắt buộc phải dùng.
- Thành thạo **LINQ** để xử lý dữ liệu hiệu quả.
- Sử dụng **Nullable Reference Types** để tránh lỗi NullReferenceException.
- Áp dụng **Records** cho Data Transfer Objects (DTO).
- Hiểu **Pattern Matching** để viết code rõ ràng hơn.
- Biết cách sử dụng các tính năng C# 10+ trong dự án thực tế.

---

# 1. **TẠI SAO CẦN HỌC C# MODERN?**

## 🎒 Ví dụ đời sống

Hãy tưởng tượng C# là **ô tô**:
- C# 7.0 = Xe số sàn (phải thay số thủ công)
- C# 10+ = Xe số tự động với AI (tự động, thông minh hơn)

Backend hiện đại cần xử lý:
- **Hàng ngàn request đồng thời** → Cần Async/Await
- **Dữ liệu phức tạp từ Database** → Cần LINQ
- **Null safety** → Cần Nullable Reference Types
- **API trả về JSON** → Cần Records

---

# 2. **ASYNC/AWAIT — TẤM VÉ BẮT BUỘC CHO BACKEND**

## 2.1. Tại sao Backend phải dùng Async?

### ❌ Cách cũ (Synchronous):

```csharp
public List<Product> GetProducts()
{
    // Thread bị block ở đây, chờ Database trả về
    var products = _db.Products.ToList(); // 2 giây
    return products;
}
```

**Vấn đề:** 
- Nếu có 1000 requests → cần 1000 threads
- Server chỉ có ~100 threads → 900 requests phải đợi
- Hiệu suất thấp, dễ crash

### ✅ Cách mới (Asynchronous):

```csharp
public async Task<List<Product>> GetProductsAsync()
{
    // Thread được giải phóng trong lúc chờ Database
    var products = await _db.Products.ToListAsync(); // 2 giây
    return products;
}
```

**Lợi ích:**
- Thread được giải phóng khi chờ I/O
- Có thể xử lý nhiều requests hơn với ít threads hơn
- Hiệu suất cao, ít bị crash

---

## 2.2. Cú pháp Async/Await

### Quy tắc vàng:

```csharp
// 1. Hàm async phải trả về Task hoặc Task<T>
// 2. Tên hàm thường có hậu tố "Async"
// 3. Dùng "await" trước các hàm async

public async Task<string> GetUserNameAsync(int userId)
{
    // await: đợi kết quả nhưng không block thread
    var user = await _db.Users.FindAsync(userId);
    return user.Name;
}
```

### 🎯 Ví dụ thực tế: Gọi API bên ngoài

```csharp
public class WeatherService
{
    private readonly HttpClient _httpClient;
    
    public WeatherService(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }
    
    // ❌ Cách sai: Synchronous
    public string GetWeatherSync(string city)
    {
        var response = _httpClient.GetStringAsync($"api/weather/{city}").Result; // BLOCK!
        return response;
    }
    
    // ✅ Cách đúng: Asynchronous
    public async Task<string> GetWeatherAsync(string city)
    {
        var response = await _httpClient.GetStringAsync($"api/weather/{city}");
        return response;
    }
}
```

---

## 2.3. Async trong Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly AppDbContext _db;
    
    public ProductsController(AppDbContext db)
    {
        _db = db;
    }
    
    // ✅ GET: api/products
    [HttpGet]
    public async Task<ActionResult<List<Product>>> GetProducts()
    {
        // Sử dụng async để không block thread
        var products = await _db.Products.ToListAsync();
        return Ok(products);
    }
    
    // ✅ GET: api/products/5
    [HttpGet("{id}")]
    public async Task<ActionResult<Product>> GetProduct(int id)
    {
        var product = await _db.Products.FindAsync(id);
        
        if (product == null)
            return NotFound();
            
        return Ok(product);
    }
    
    // ✅ POST: api/products
    [HttpPost]
    public async Task<ActionResult<Product>> CreateProduct(Product product)
    {
        _db.Products.Add(product);
        await _db.SaveChangesAsync(); // Async save!
        
        return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
    }
}
```

---

## 2.4. ❌ CÁC LỖI THƯỜNG GẶP VỚI ASYNC/AWAIT

### ❌ Lỗi 1: Dùng .Result hoặc .Wait()

```csharp
// ❌ Vấn đề: Block thread, có thể gây deadlock
var products = GetProductsAsync().Result; // Deadlock!
GetProductsAsync().Wait(); // Cũng block thread

// ✅ Giải pháp: Luôn dùng await
var products = await GetProductsAsync(); // ✅
```

**🔍 Giải thích:** `.Result` và `.Wait()` block thread hiện tại, có thể gây deadlock trong ASP.NET Core. Luôn dùng `await` trong async context.

---

### ❌ Lỗi 2: Quên await

```csharp
// ❌ Vấn đề: Trả về Task thay vì kết quả
public async Task<List<Product>> GetProductsAsync()
{
    var products = _db.Products.ToListAsync(); // ❌ Trả về Task<List<Product>>
    return products; // Lỗi compile: Cannot convert Task<List<Product>> to List<Product>
}

// ✅ Giải pháp: Thêm await
public async Task<List<Product>> GetProductsAsync()
{
    var products = await _db.Products.ToListAsync(); // ✅ Trả về List<Product>
    return products;
}
```

**🔍 Giải thích:** Quên `await` sẽ trả về `Task<T>` thay vì `T`. Luôn dùng `await` trước async method.

---

### ❌ Lỗi 3: Async void (chỉ dùng cho event handlers)

```csharp
// ❌ Vấn đề: Async void không thể catch exception
public async void ProcessData()
{
    await SomeAsyncMethod(); // Exception sẽ crash app
}

// ✅ Giải pháp: Dùng async Task
public async Task ProcessData()
{
    await SomeAsyncMethod(); // Exception có thể được catch
}
```

**🔍 Giải thích:** `async void` chỉ dùng cho event handlers. Với methods khác, luôn dùng `async Task`.

---

### ❌ Lỗi 4: Blocking async code trong sync method

```csharp
// ❌ Vấn đề: Block async code
public void ProcessData()
{
    var result = GetDataAsync().Result; // Block thread
}

// ✅ Giải pháp: Làm method async
public async Task ProcessData()
{
    var result = await GetDataAsync(); // Non-blocking
}
```

**🔍 Giải thích:** Không block async code. Nếu cần dùng async, làm method async luôn.

---

### ❌ Lỗi 5: ConfigureAwait(false) không cần thiết trong ASP.NET Core

```csharp
// ❌ Vấn đề: ConfigureAwait(false) không cần trong ASP.NET Core
var data = await GetDataAsync().ConfigureAwait(false);

// ✅ Giải pháp: Bỏ ConfigureAwait (ASP.NET Core không có SynchronizationContext)
var data = await GetDataAsync(); // ✅
```

**🔍 Giải thích:** ASP.NET Core không có SynchronizationContext, nên `ConfigureAwait(false)` không cần thiết (khác với .NET Framework).

---

# 3. **LINQ — CÔNG CỤ QUYỀN LỰC XỬ LÝ DỮ LIỆU**

## 3.1. LINQ là gì?

**Language Integrated Query** — Ngôn ngữ truy vấn tích hợp trong C#.

### 🏠 Ví dụ đời sống

LINQ giống như **Google Search cho Collection**:
- Thay vì duyệt từng phần tử → Gõ điều kiện là ra kết quả
- Thay vì viết vòng lặp phức tạp → 1 dòng LINQ xong

---

## 3.2. LINQ với List

### Ví dụ: Lọc sản phẩm giá > 1000

```csharp
// Cách cũ: Vòng lặp
List<Product> expensiveProducts = new List<Product>();
foreach (var product in products)
{
    if (product.Price > 1000)
    {
        expensiveProducts.Add(product);
    }
}

// Cách mới: LINQ
var expensiveProducts = products.Where(p => p.Price > 1000).ToList();
```

---

## 3.3. Các hàm LINQ quan trọng

### 🟢 Where: Lọc dữ liệu

```csharp
// Lấy sản phẩm còn hàng
var inStockProducts = products.Where(p => p.Stock > 0).ToList();

// Lấy sản phẩm theo category
var laptops = products.Where(p => p.CategoryId == 1).ToList();
```

### 🟢 Select: Chuyển đổi dữ liệu (Map)

```csharp
// Lấy danh sách tên sản phẩm
var productNames = products.Select(p => p.Name).ToList();

// Tạo DTO
var productDTOs = products.Select(p => new ProductDTO
{
    Id = p.Id,
    Name = p.Name,
    Price = p.Price
}).ToList();
```

### 🟢 OrderBy / OrderByDescending: Sắp xếp

```csharp
// Sắp xếp theo giá tăng dần
var sortedByPrice = products.OrderBy(p => p.Price).ToList();

// Sắp xếp theo giá giảm dần
var sortedByPriceDesc = products.OrderByDescending(p => p.Price).ToList();
```

### 🟢 First / FirstOrDefault: Lấy phần tử đầu

```csharp
// Lấy sản phẩm đầu tiên (throw exception nếu không có)
var firstProduct = products.First();

// Lấy sản phẩm đầu tiên hoặc null
var firstProduct = products.FirstOrDefault();

// Lấy sản phẩm đầu tiên với điều kiện
var cheapestProduct = products.OrderBy(p => p.Price).FirstOrDefault();
```

### 🟢 Skip / Take: Phân trang

```csharp
// Lấy 10 sản phẩm đầu tiên
var firstPage = products.Take(10).ToList();

// Bỏ qua 10 sản phẩm đầu, lấy 10 sản phẩm tiếp theo
var secondPage = products.Skip(10).Take(10).ToList();

// Công thức phân trang
int pageNumber = 2;
int pageSize = 10;
var page = products
    .Skip((pageNumber - 1) * pageSize)
    .Take(pageSize)
    .ToList();
```

### 🟢 Count / Any / All: Đếm và kiểm tra

```csharp
// Đếm số sản phẩm
int totalProducts = products.Count();

// Đếm số sản phẩm đắt tiền
int expensiveCount = products.Count(p => p.Price > 1000);

// Kiểm tra có sản phẩm nào không
bool hasProducts = products.Any();

// Kiểm tra có sản phẩm nào giá > 1000 không
bool hasExpensive = products.Any(p => p.Price > 1000);

// Kiểm tra TẤT CẢ sản phẩm có giá > 0
bool allPositivePrice = products.All(p => p.Price > 0);
```

### 🟢 GroupBy: Nhóm dữ liệu

```csharp
// Nhóm sản phẩm theo danh mục
var groupedByCategory = products.GroupBy(p => p.CategoryId);

foreach (var group in groupedByCategory)
{
    Console.WriteLine($"Category {group.Key}: {group.Count()} products");
}

// Thống kê số lượng theo category
var categoryStats = products
    .GroupBy(p => p.CategoryId)
    .Select(g => new
    {
        CategoryId = g.Key,
        Count = g.Count(),
        TotalValue = g.Sum(p => p.Price * p.Stock)
    })
    .ToList();
```

### 🟢 Join: Kết nối dữ liệu

```csharp
// Join Products với Categories
var productsWithCategory = products
    .Join(
        categories,
        p => p.CategoryId,      // Key từ products
        c => c.Id,              // Key từ categories
        (p, c) => new           // Kết quả
        {
            ProductName = p.Name,
            CategoryName = c.Name,
            Price = p.Price
        }
    )
    .ToList();
```

---

## 3.4. LINQ với Entity Framework Core

```csharp
public class ProductService
{
    private readonly AppDbContext _db;
    
    public ProductService(AppDbContext db)
    {
        _db = db;
    }
    
    // Lấy sản phẩm có phân trang và tìm kiếm
    public async Task<List<Product>> GetProductsAsync(
        string searchTerm, 
        int pageNumber, 
        int pageSize)
    {
        var query = _db.Products.AsQueryable();
        
        // Lọc theo tên (nếu có)
        if (!string.IsNullOrEmpty(searchTerm))
        {
            query = query.Where(p => p.Name.Contains(searchTerm));
        }
        
        // Phân trang
        var products = await query
            .OrderBy(p => p.Name)
            .Skip((pageNumber - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync();
            
        return products;
    }
    
    // Thống kê sản phẩm theo category
    public async Task<List<CategoryStats>> GetCategoryStatsAsync()
    {
        var stats = await _db.Products
            .GroupBy(p => p.CategoryId)
            .Select(g => new CategoryStats
            {
                CategoryId = g.Key,
                ProductCount = g.Count(),
                TotalValue = g.Sum(p => p.Price * p.Stock),
                AveragePrice = g.Average(p => p.Price)
            })
            .ToListAsync();
            
        return stats;
    }
}
```

---

# 4. **NULLABLE REFERENCE TYPES**

## 4.1. Vấn đề với Null

```csharp
// Trước C# 8.0
public string GetUserName(int userId)
{
    var user = _db.Users.Find(userId);
    return user.Name; // NullReferenceException nếu user = null!
}
```

## 4.2. Giải pháp với Nullable

```csharp
// C# 8.0+: Bật Nullable Reference Types trong .csproj
// <Nullable>enable</Nullable>

public string? GetUserName(int userId) // ? = có thể null
{
    var user = _db.Users.Find(userId);
    return user?.Name; // Safe navigation operator
}

public string GetUserNameSafe(int userId)
{
    var user = _db.Users.Find(userId);
    
    if (user == null)
        return "Unknown";
        
    return user.Name; // Compiler biết user không null ở đây
}
```

---

# 5. **RECORDS — DTO HOÀN HẢO**

## 5.1. Record là gì?

**Record** là kiểu dữ liệu bất biến (immutable) dùng để chứa data.

### 🎒 Ví dụ đời sống

Record giống như **phong bì đựng giấy tờ**:
- Một khi gửi đi, không sửa được
- Chỉ đọc, không ghi
- Nhẹ, nhanh, an toàn

---

## 5.2. Cú pháp Record

```csharp
// Cách cũ: Class
public class ProductDTO
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

// Cách mới: Record (C# 9+)
public record ProductDTO(int Id, string Name, decimal Price);

// Hoặc dạng đầy đủ
public record ProductDTO
{
    public int Id { get; init; } // init = chỉ set khi tạo
    public string Name { get; init; }
    public decimal Price { get; init; }
}
```

---

## 5.3. Sử dụng Record

```csharp
// Tạo instance
var product = new ProductDTO(1, "Laptop", 1500);

// ❌ Không thể thay đổi
// product.Name = "PC"; // Lỗi!

// ✅ Tạo bản copy với giá trị mới (with expression)
var discountedProduct = product with { Price = 1200 };

Console.WriteLine(product.Price);           // 1500
Console.WriteLine(discountedProduct.Price); // 1200
```

---

## 5.4. Record trong API

```csharp
// Request DTO
public record CreateProductRequest(string Name, decimal Price, int CategoryId);

// Response DTO
public record ProductResponse(int Id, string Name, decimal Price, string CategoryName);

[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly AppDbContext _db;
    
    [HttpPost]
    public async Task<ActionResult<ProductResponse>> CreateProduct(CreateProductRequest request)
    {
        var category = await _db.Categories.FindAsync(request.CategoryId);
        if (category == null)
            return BadRequest("Category not found");
        
        var product = new Product
        {
            Name = request.Name,
            Price = request.Price,
            CategoryId = request.CategoryId
        };
        
        _db.Products.Add(product);
        await _db.SaveChangesAsync();
        
        var response = new ProductResponse(
            product.Id,
            product.Name,
            product.Price,
            category.Name
        );
        
        return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, response);
    }
}
```

---

# 6. **PATTERN MATCHING**

## 6.1. Switch Expression

```csharp
// Cách cũ
public string GetDiscountMessage(int quantity)
{
    string message;
    switch (quantity)
    {
        case 1:
            message = "No discount";
            break;
        case 2:
        case 3:
            message = "10% discount";
            break;
        default:
            message = "20% discount";
            break;
    }
    return message;
}

// Cách mới: Switch Expression (C# 8+)
public string GetDiscountMessage(int quantity) => quantity switch
{
    1 => "No discount",
    >= 2 and <= 3 => "10% discount",
    > 3 => "20% discount",
    _ => "Invalid quantity"
};
```

---

## 6.2. Type Pattern

```csharp
public decimal CalculateDiscount(object discountInfo) => discountInfo switch
{
    int percentage => percentage / 100m,
    string couponCode => GetCouponDiscount(couponCode),
    decimal fixedAmount => fixedAmount,
    _ => 0
};
```

---

# 7. **BÀI TẬP THỰC HÀNH**

## 📝 Bài 1: Async/Await

Viết hàm `GetTopProductsAsync` lấy top 5 sản phẩm có giá cao nhất:

```csharp
public async Task<List<Product>> GetTopProductsAsync()
{
    // TODO: Implement
}
```

<details>
<summary>💡 Đáp án</summary>

```csharp
public async Task<List<Product>> GetTopProductsAsync()
{
    return await _db.Products
        .OrderByDescending(p => p.Price)
        .Take(5)
        .ToListAsync();
}
```
</details>

---

## 📝 Bài 2: LINQ

Cho danh sách:
```csharp
var products = new List<Product>
{
    new Product { Id = 1, Name = "Laptop", Price = 1500, CategoryId = 1, Stock = 10 },
    new Product { Id = 2, Name = "Mouse", Price = 25, CategoryId = 2, Stock = 50 },
    new Product { Id = 3, Name = "Keyboard", Price = 75, CategoryId = 2, Stock = 0 },
    new Product { Id = 4, Name = "Monitor", Price = 300, CategoryId = 1, Stock = 5 }
};
```

Viết LINQ để:
1. Lấy sản phẩm còn hàng (Stock > 0)
2. Sắp xếp theo giá tăng dần
3. Lấy chỉ tên và giá

<details>
<summary>💡 Đáp án</summary>

```csharp
var result = products
    .Where(p => p.Stock > 0)
    .OrderBy(p => p.Price)
    .Select(p => new { p.Name, p.Price })
    .ToList();
```
</details>

---

## 📝 Bài 3: Record

Tạo 2 records:
- `CreateOrderRequest(int UserId, List<OrderItemDTO> Items)`
- `OrderItemDTO(int ProductId, int Quantity)`

<details>
<summary>💡 Đáp án</summary>

```csharp
public record OrderItemDTO(int ProductId, int Quantity);
public record CreateOrderRequest(int UserId, List<OrderItemDTO> Items);
```
</details>

---

## 🎯 8. CASE STUDY / VÍ DỤ THỰC TẾ

### Case Study 1: API Service với Async/Await

**Yêu cầu:** Tạo service gọi nhiều API bên ngoài đồng thời và tổng hợp kết quả.

```csharp
public class ExternalApiService
{
    private readonly HttpClient _httpClient;
    
    public ExternalApiService(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }
    
    // Gọi nhiều API đồng thời
    public async Task<CombinedData> GetCombinedDataAsync(int userId)
    {
        // Gọi 3 API đồng thời (parallel)
        var userTask = GetUserAsync(userId);
        var ordersTask = GetOrdersAsync(userId);
        var preferencesTask = GetPreferencesAsync(userId);
        
        // Đợi tất cả hoàn thành
        await Task.WhenAll(userTask, ordersTask, preferencesTask);
        
        return new CombinedData
        {
            User = await userTask,
            Orders = await ordersTask,
            Preferences = await preferencesTask
        };
    }
    
    private async Task<User> GetUserAsync(int userId)
    {
        var response = await _httpClient.GetAsync($"api/users/{userId}");
        response.EnsureSuccessStatusCode();
        var json = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<User>(json);
    }
    
    private async Task<List<Order>> GetOrdersAsync(int userId)
    {
        var response = await _httpClient.GetAsync($"api/users/{userId}/orders");
        response.EnsureSuccessStatusCode();
        var json = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<List<Order>>(json);
    }
    
    private async Task<Preferences> GetPreferencesAsync(int userId)
    {
        var response = await _httpClient.GetAsync($"api/users/{userId}/preferences");
        response.EnsureSuccessStatusCode();
        var json = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<Preferences>(json);
    }
}
```

**Giải thích:**
- Sử dụng `Task.WhenAll` để gọi nhiều API đồng thời
- Tăng hiệu suất đáng kể so với gọi tuần tự
- Async/await không block thread

---

### Case Study 2: Data Processing với LINQ và Records

**Yêu cầu:** Xử lý dữ liệu đơn hàng, tính toán thống kê, tạo DTOs.

```csharp
// Record cho DTO
public record OrderSummaryDto(
    int OrderId,
    string CustomerName,
    decimal Total,
    int ItemCount,
    DateTime OrderDate
);

public record CustomerStatsDto(
    string CustomerName,
    int TotalOrders,
    decimal TotalSpent,
    decimal AverageOrderValue
);

public class OrderService
{
    private readonly AppDbContext _db;
    
    public OrderService(AppDbContext db)
    {
        _db = db;
    }
    
    // Lấy đơn hàng với LINQ và tạo DTOs
    public async Task<List<OrderSummaryDto>> GetOrderSummariesAsync(
        DateTime? fromDate = null,
        DateTime? toDate = null,
        int? customerId = null)
    {
        var query = _db.Orders.AsQueryable();
        
        // Filter động
        if (fromDate.HasValue)
            query = query.Where(o => o.OrderDate >= fromDate.Value);
        
        if (toDate.HasValue)
            query = query.Where(o => o.OrderDate <= toDate.Value);
        
        if (customerId.HasValue)
            query = query.Where(o => o.CustomerId == customerId.Value);
        
        // Project sang DTO
        var summaries = await query
            .Select(o => new OrderSummaryDto(
                o.Id,
                o.Customer.Name,
                o.Total,
                o.Items.Count,
                o.OrderDate
            ))
            .OrderByDescending(o => o.OrderDate)
            .ToListAsync();
        
        return summaries;
    }
    
    // Thống kê theo khách hàng
    public async Task<List<CustomerStatsDto>> GetCustomerStatsAsync()
    {
        var stats = await _db.Orders
            .GroupBy(o => new { o.CustomerId, o.Customer.Name })
            .Select(g => new CustomerStatsDto(
                g.Key.Name,
                g.Count(),
                g.Sum(o => o.Total),
                g.Average(o => o.Total)
            ))
            .OrderByDescending(s => s.TotalSpent)
            .ToListAsync();
        
        return stats;
    }
}
```

**Best practices:**
- Dùng Records cho DTOs (immutable, value equality)
- LINQ để filter và project dữ liệu
- Async/await cho database operations
- IQueryable để filter ở database

---

### Case Study 3: Nullable Reference Types trong API

**Yêu cầu:** Tạo API an toàn với null checking.

```csharp
#nullable enable

public class ProductService
{
    private readonly AppDbContext _db;
    
    public ProductService(AppDbContext db)
    {
        _db = db;
    }
    
    // Method trả về nullable
    public async Task<Product?> GetProductAsync(int id)
    {
        return await _db.Products.FindAsync(id);
    }
    
    // Method với null checking
    public async Task<ProductDto> GetProductDtoAsync(int id)
    {
        var product = await GetProductAsync(id);
        
        if (product == null)
            throw new NotFoundException($"Product {id} not found");
        
        // Compiler biết product không null ở đây
        return new ProductDto
        {
            Id = product.Id,
            Name = product.Name,
            Price = product.Price
        };
    }
    
    // Method với null-conditional
    public async Task<string?> GetProductDescriptionAsync(int id)
    {
        var product = await GetProductAsync(id);
        return product?.Description; // Trả về null nếu product null
    }
}

// Controller
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly ProductService _service;
    
    public ProductsController(ProductService service)
    {
        _service = service;
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDto>> GetProduct(int id)
    {
        try
        {
            var product = await _service.GetProductDtoAsync(id);
            return Ok(product);
        }
        catch (NotFoundException ex)
        {
            return NotFound(ex.Message);
        }
    }
}
```

**Giải thích:**
- `#nullable enable` bật null checking
- `?` đánh dấu nullable types
- Compiler cảnh báo nếu không kiểm tra null
- Code an toàn hơn, ít lỗi runtime

---

## ✅ 9. BEST PRACTICES

### 9.1. Async/Await Best Practices

```csharp
// ✅ Đúng: Async all the way
public async Task<List<Product>> GetProductsAsync()
{
    return await _db.Products.ToListAsync();
}

// ❌ Sai: Mix sync và async
public List<Product> GetProducts()
{
    return _db.Products.ToListAsync().Result; // Block thread
}

// ✅ Đúng: Dùng Task.WhenAll cho parallel operations
var task1 = GetData1Async();
var task2 = GetData2Async();
await Task.WhenAll(task1, task2);
```

### 9.2. LINQ Best Practices

```csharp
// ✅ Đúng: Filter ở database
var products = await _db.Products
    .Where(p => p.Price > 100)
    .ToListAsync();

// ❌ Sai: Load tất cả rồi filter
var all = await _db.Products.ToListAsync();
var filtered = all.Where(p => p.Price > 100).ToList();

// ✅ Đúng: Select chỉ fields cần thiết
var dtos = await _db.Products
    .Select(p => new ProductDto { Id = p.Id, Name = p.Name })
    .ToListAsync();
```

### 9.3. Records Best Practices

```csharp
// ✅ Đúng: Dùng record cho DTOs
public record CreateProductRequest(string Name, decimal Price);

// ✅ Đúng: Dùng record với init cho mutable DTOs
public record ProductDto
{
    public int Id { get; init; }
    public string Name { get; init; }
    public decimal Price { get; init; }
}

// ❌ Sai: Dùng class cho simple DTOs (verbose)
public class ProductDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    
    // Phải override Equals, GetHashCode, etc.
}
```

### 9.4. Nullable Reference Types Best Practices

```csharp
#nullable enable

// ✅ Đúng: Đánh dấu nullable rõ ràng
public string? GetOptionalName() { return null; }
public string GetRequiredName() { return "Required"; }

// ✅ Đúng: Kiểm tra null
if (name != null)
{
    Console.WriteLine(name.Length); // Compiler biết không null
}

// ✅ Đúng: Null-conditional operator
var length = name?.Length ?? 0;
```

### 9.5. Pattern Matching Best Practices

```csharp
// ✅ Đúng: Switch expression cho simple cases
string GetGrade(int score) => score switch
{
    >= 90 => "A",
    >= 80 => "B",
    >= 70 => "C",
    _ => "D"
};

// ✅ Đúng: Type pattern matching
string Describe(object obj) => obj switch
{
    Product p => $"Product: {p.Name}",
    Order o => $"Order: {o.Id}",
    null => "null",
    _ => "unknown"
};
```

---

# 🧪 10. MINI TEST

1. **Async/Await** có tác dụng gì?
   - A. Làm code chạy nhanh hơn
   - B. Giải phóng thread khi chờ I/O
   - C. Tự động xử lý lỗi
   - D. Không có tác dụng gì

2. **LINQ** nào đúng để lấy 10 sản phẩm đầu tiên?
   - A. `products.Take(10)`
   - B. `products.First(10)`
   - C. `products.Get(10)`
   - D. `products[0..10]`

3. **Record** khác gì **Class**?
   - A. Record nhanh hơn
   - B. Record bất biến (immutable)
   - C. Record nhỏ hơn
   - D. Không khác gì

<details>
<summary>💡 Đáp án</summary>

1. **B** - Giải phóng thread khi chờ I/O
2. **A** - `products.Take(10)`
3. **B** - Record bất biến (immutable)
</details>

---

# 📝 11. QUICK NOTES

### Async/Await:
- `async Task<T>` cho methods trả về giá trị
- `async Task` cho methods không trả về giá trị
- `await` trước async method calls
- Không dùng `.Result` hoặc `.Wait()`
- `Task.WhenAll` cho parallel operations

### LINQ:
- `Where`: Filter
- `Select`: Project/Transform
- `OrderBy`: Sort
- `GroupBy`: Group
- `FirstOrDefault`: Lấy phần tử đầu (an toàn)
- `ToListAsync`: Execute query (EF Core)

### Records:
- Immutable by default
- Value equality
- Perfect for DTOs
- `with` expression để tạo copy

### Nullable Reference Types:
- `#nullable enable` bật checking
- `string?` = nullable
- `string` = non-nullable
- Null-conditional: `obj?.Property`
- Null-coalescing: `value ?? defaultValue`

### Pattern Matching:
- Switch expressions: `value switch { ... }`
- Type patterns: `obj is Type t`
- Property patterns: `obj is { Property: value }`

---

# 📌 12. TÓM TẮT CHƯƠNG

✅ **Async/Await** giúp Backend xử lý nhiều requests đồng thời  
✅ **LINQ** giúp xử lý dữ liệu ngắn gọn và rõ ràng  
✅ **Nullable Reference Types** tránh lỗi NullReferenceException  
✅ **Records** hoàn hảo cho DTO  
✅ **Pattern Matching** giúp code dễ đọc hơn  

---

**Chương tiếp theo: [02. Kiến trúc ASP.NET Core →](./02_kien_truc_aspnet_core.md)**
