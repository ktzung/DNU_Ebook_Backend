# 🟨 CHƯƠNG 08
# **RESTFUL WEB API**

## 📖 1. Giới thiệu RESTful API

**REST** (Representational State Transfer) là một kiểu kiến trúc phần mềm để tạo ra các Web Service.
**API** (Application Programming Interface) là phương thức để các phần mềm giao tiếp với nhau.

### Tại sao dùng Web API?
- **Đa nền tảng**: Một Backend phục vụ cho cả Web (React/Angular), Mobile (Flutter/Android/iOS) và Desktop.
- **Tách biệt**: Frontend và Backend phát triển độc lập.
- **Scalable**: Dễ dàng mở rộng hệ thống.

---

## 🛠️ 2. Cấu trúc một Controller

Trong ASP.NET Core, API Controller thường kế thừa từ `ControllerBase` (không có View) thay vì `Controller` (có View).

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // Code xử lý ở đây
}
```

- `[ApiController]`: Kích hoạt các tính năng tự động như Model Validation, Binding source inference.
- `[Route]`: Định nghĩa URL base. Ví dụ `ProductsController` sẽ có URL là `/api/products`.

---

## 📡 3. HTTP Methods & Actions

Các hành động CRUD (Create, Read, Update, Delete) tương ứng với các HTTP Verbs:

| Action | HTTP Verb | Mô tả | Status Code thành công |
|--------|-----------|-------|------------------------|
| Lấy danh sách | `GET` | Đọc dữ liệu | 200 OK |
| Lấy chi tiết | `GET` | Đọc 1 bản ghi | 200 OK |
| Tạo mới | `POST` | Thêm dữ liệu | 201 Created |
| Cập nhật | `PUT` | Sửa toàn bộ | 204 No Content / 200 OK |
| Xóa | `DELETE` | Xóa dữ liệu | 204 No Content |

### 3.1. GET - Lấy dữ liệu
```csharp
// GET: api/products
[HttpGet]
public IActionResult GetAll()
{
    var products = _service.GetProducts();
    return Ok(products); // Trả về 200 cùng dữ liệu JSON
}

// GET: api/products/5
[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    var product = _service.GetProductById(id);
    if (product == null)
    {
        return NotFound(); // Trả về 404 nếu không tìm thấy
    }
    return Ok(product);
}
```

### 3.2. POST - Tạo mới
```csharp
// POST: api/products
[HttpPost]
public IActionResult Create(ProductDto request)
{
    var newProduct = _service.Create(request);
    // Trả về 201 Created cùng Header Location trỏ đến resource mới
    return CreatedAtAction(nameof(GetById), new { id = newProduct.Id }, newProduct);
}
```

### 3.3. PUT - Cập nhật
```csharp
// PUT: api/products/5
[HttpPut("{id}")]
public IActionResult Update(int id, ProductDto request)
{
    if (id != request.Id) return BadRequest(); // 400
    
    var result = _service.Update(id, request);
    if (!result) return NotFound();
    
    return NoContent(); // 204: Thành công nhưng không trả về nội dung gì
}
```

### 3.4. DELETE - Xóa
```csharp
// DELETE: api/products/5
[HttpDelete("{id}")]
public IActionResult Delete(int id)
{
    var result = _service.Delete(id);
    if (!result) return NotFound();
    
    return NoContent(); // 204
}
```

---

## 📦 4. Return Types

ASP.NET Core hỗ trợ nhiều kiểu trả về:

1. **Specific Type**: `public Product Get()` - Chỉ trả về dữ liệu, khó chỉnh Status Code.
2. **IActionResult**: `public IActionResult Get()` - Linh hoạt nhất (`Ok()`, `NotFound()`, `BadRequest()`).
3. **ActionResult<T>**: `public ActionResult<Product> Get()` - Kết hợp cả hai (khuyên dùng).

---

## 📜 5. Swagger / OpenAPI

**Swagger** là công cụ giúp tạo tài liệu API tự động và cho phép test API ngay trên trình duyệt.

### Cài đặt (thường có sẵn trong template Web API):
Trong `Program.cs`:
```csharp
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(); // Truy cập tại /swagger
}
```

---

## 🧪 6. Bài tập thực hành

### Bài 1: Tạo Category API
1. Tạo `CategoriesController`.
2. Viết API lấy danh sách Category.
3. Viết API tạo mới Category.
4. Test bằng Swagger UI.

### Bài 2: Hoàn thiện Product API
1. Bổ sung tính năng tìm kiếm sản phẩm theo tên: `GET /api/products?search=iphone`.
2. Bổ sung phân trang: `GET /api/products?page=1&pageSize=10`.
3. Test các trường hợp lỗi (404, 400) bằng Postman.

---

## ❌ 7. CÁC LỖI THƯỜNG GẶP

### ❌ Lỗi 1: Quên [ApiController] attribute

```csharp
// ❌ Vấn đề: Không có [ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(ProductDto request)
    {
        // ModelState.IsValid không tự động check
        if (!ModelState.IsValid) // Phải check thủ công
            return BadRequest();
    }
}

// ✅ Giải pháp: Thêm [ApiController]
[ApiController] // ✅ Tự động validate
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(ProductDto request)
    {
        // ModelState.IsValid tự động check
        // Nếu invalid → tự động trả về 400 BadRequest
    }
}
```

**🔍 Giải thích:** `[ApiController]` tự động validate model và trả về 400 nếu invalid. Không có attribute phải check thủ công.

---

### ❌ Lỗi 2: Trả về Entity thay vì DTO

```csharp
// ❌ Vấn đề: Trả về Entity (lộ thông tin nhạy cảm)
[HttpGet("{id}")]
public async Task<ActionResult<Product>> GetProduct(int id)
{
    var product = await _context.Products.FindAsync(id);
    return Ok(product); // ❌ Lộ PasswordHash, InternalId, etc.
}

// ✅ Giải pháp: Dùng DTO
[HttpGet("{id}")]
public async Task<ActionResult<ProductDto>> GetProduct(int id)
{
    var product = await _context.Products.FindAsync(id);
    if (product == null) return NotFound();
    
    var dto = _mapper.Map<ProductDto>(product);
    return Ok(dto); // ✅ Chỉ trả về fields cần thiết
}
```

**🔍 Giải thích:** Entity có thể chứa thông tin nhạy cảm. DTO chỉ expose fields cần thiết.

---

### ❌ Lỗi 3: Status Code không đúng

```csharp
// ❌ Vấn đề: Trả về 200 cho mọi trường hợp
[HttpPost]
public IActionResult Create(ProductDto request)
{
    var product = _service.Create(request);
    return Ok(product); // ❌ Nên dùng 201 Created
}

// ✅ Giải pháp: Dùng status code đúng
[HttpPost]
public IActionResult Create(ProductDto request)
{
    var product = _service.Create(request);
    return CreatedAtAction(nameof(GetById), new { id = product.Id }, product); // ✅ 201
}
```

**🔍 Giải thích:** RESTful API cần status code đúng: 201 Created, 204 No Content, 404 Not Found.

---

### ❌ Lỗi 4: Không xử lý exception

```csharp
// ❌ Vấn đề: Exception không được handle
[HttpGet("{id}")]
public async Task<ActionResult<ProductDto>> GetProduct(int id)
{
    var product = await _service.GetProductAsync(id); // Có thể throw exception
    return Ok(product);
}

// ✅ Giải pháp: Try-catch hoặc global exception handler
[HttpGet("{id}")]
public async Task<ActionResult<ProductDto>> GetProduct(int id)
{
    try
    {
        var product = await _service.GetProductAsync(id);
        if (product == null) return NotFound();
        return Ok(product);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error getting product {Id}", id);
        return StatusCode(500, "Internal server error");
    }
}
```

**🔍 Giải thích:** API phải xử lý exception và trả về status code phù hợp, không để exception leak ra client.

---

### ❌ Lỗi 5: CORS không được cấu hình

```csharp
// ❌ Vấn đề: Frontend không gọi được API (CORS error)
// Browser: "Access to fetch at 'http://localhost:5000/api/products' from origin 'http://localhost:3000' has been blocked by CORS policy"

// ✅ Giải pháp: Cấu hình CORS
// Program.cs
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

var app = builder.Build();
app.UseCors("AllowAll");
```

**🔍 Giải thích:** CORS (Cross-Origin Resource Sharing) cần được cấu hình để frontend từ domain khác có thể gọi API.

---

## 🎯 8. CASE STUDY / VÍ DỤ THỰC TẾ

### Case Study 1: Products API hoàn chỉnh với Pagination, Search, Filtering

**Yêu cầu:** Tạo RESTful API cho Products với đầy đủ tính năng.

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;
    private readonly ILogger<ProductsController> _logger;
    
    public ProductsController(IProductService productService, ILogger<ProductsController> logger)
    {
        _productService = productService;
        _logger = logger;
    }
    
    // GET: api/products?page=1&pageSize=10&search=laptop&minPrice=1000&maxPrice=5000&categoryId=1
    [HttpGet]
    public async Task<ActionResult<PagedResult<ProductDto>>> GetProducts(
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 10,
        [FromQuery] string? search = null,
        [FromQuery] decimal? minPrice = null,
        [FromQuery] decimal? maxPrice = null,
        [FromQuery] int? categoryId = null)
    {
        try
        {
            var result = await _productService.GetProductsAsync(
                page, pageSize, search, minPrice, maxPrice, categoryId);
            
            return Ok(result);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error getting products");
            return StatusCode(500, new { message = "Internal server error" });
        }
    }
    
    // GET: api/products/5
    [HttpGet("{id:int}")]
    public async Task<ActionResult<ProductDto>> GetProduct(int id)
    {
        var product = await _productService.GetProductAsync(id);
        if (product == null)
        {
            return NotFound(new { message = $"Product with id {id} not found" });
        }
        
        return Ok(product);
    }
    
    // POST: api/products
    [HttpPost]
    public async Task<ActionResult<ProductDto>> CreateProduct(CreateProductRequest request)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }
        
        try
        {
            var product = await _productService.CreateProductAsync(request);
            return CreatedAtAction(
                nameof(GetProduct), 
                new { id = product.Id }, 
                product);
        }
        catch (ValidationException ex)
        {
            return BadRequest(new { message = ex.Message });
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating product");
            return StatusCode(500, new { message = "Internal server error" });
        }
    }
    
    // PUT: api/products/5
    [HttpPut("{id:int}")]
    public async Task<IActionResult> UpdateProduct(int id, UpdateProductRequest request)
    {
        if (id != request.Id)
        {
            return BadRequest(new { message = "Id mismatch" });
        }
        
        try
        {
            var success = await _productService.UpdateProductAsync(id, request);
            if (!success)
            {
                return NotFound(new { message = $"Product with id {id} not found" });
            }
            
            return NoContent();
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error updating product {Id}", id);
            return StatusCode(500, new { message = "Internal server error" });
        }
    }
    
    // DELETE: api/products/5
    [HttpDelete("{id:int}")]
    public async Task<IActionResult> DeleteProduct(int id)
    {
        try
        {
            var success = await _productService.DeleteProductAsync(id);
            if (!success)
            {
                return NotFound(new { message = $"Product with id {id} not found" });
            }
            
            return NoContent();
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error deleting product {Id}", id);
            return StatusCode(500, new { message = "Internal server error" });
        }
    }
}
```

**Best practices:**
- DTOs cho request/response
- Pagination và filtering
- Exception handling
- Logging
- Status codes đúng chuẩn REST

---

### Case Study 2: API Response Wrapper

**Yêu cầu:** Tạo wrapper cho API responses để nhất quán.

```csharp
// Response wrapper
public class ApiResponse<T>
{
    public bool Success { get; set; }
    public string? Message { get; set; }
    public T? Data { get; set; }
    public List<string>? Errors { get; set; }
    
    public static ApiResponse<T> SuccessResponse(T data, string? message = null)
    {
        return new ApiResponse<T>
        {
            Success = true,
            Data = data,
            Message = message
        };
    }
    
    public static ApiResponse<T> ErrorResponse(string message, List<string>? errors = null)
    {
        return new ApiResponse<T>
        {
            Success = false,
            Message = message,
            Errors = errors
        };
    }
}

// Controller sử dụng
[HttpGet("{id:int}")]
public async Task<ActionResult<ApiResponse<ProductDto>>> GetProduct(int id)
{
    var product = await _productService.GetProductAsync(id);
    if (product == null)
    {
        return NotFound(ApiResponse<ProductDto>.ErrorResponse("Product not found"));
    }
    
    return Ok(ApiResponse<ProductDto>.SuccessResponse(product));
}
```

---

## ✅ 9. BEST PRACTICES

### 9.1. RESTful API Best Practices

```csharp
// ✅ Đúng: Dùng HTTP verbs đúng
[HttpGet] // GET cho read
[HttpPost] // POST cho create
[HttpPut] // PUT cho update
[HttpDelete] // DELETE cho delete

// ✅ Đúng: Status codes đúng
return Ok(data); // 200
return CreatedAtAction(...); // 201
return NoContent(); // 204
return NotFound(); // 404
return BadRequest(); // 400

// ✅ Đúng: Dùng DTOs
public class ProductDto { }
public class CreateProductRequest { }
```

### 9.2. Error Handling Best Practices

```csharp
// ✅ Đúng: Consistent error response
catch (NotFoundException ex)
{
    return NotFound(new { message = ex.Message });
}
catch (ValidationException ex)
{
    return BadRequest(new { message = ex.Message, errors = ex.Errors });
}
catch (Exception ex)
{
    _logger.LogError(ex, "Unexpected error");
    return StatusCode(500, new { message = "Internal server error" });
}
```

### 9.3. API Documentation Best Practices

```csharp
// ✅ Đúng: XML comments cho Swagger
/// <summary>
/// Lấy danh sách sản phẩm với pagination và search
/// </summary>
/// <param name="page">Số trang (bắt đầu từ 1)</param>
/// <param name="pageSize">Số items mỗi trang</param>
/// <param name="search">Từ khóa tìm kiếm</param>
/// <returns>Danh sách sản phẩm</returns>
[HttpGet]
[ProducesResponseType(typeof(PagedResult<ProductDto>), StatusCodes.Status200OK)]
public async Task<ActionResult<PagedResult<ProductDto>>> GetProducts(...)
```

---

# 📝 10. QUICK NOTES

### HTTP Methods:
- **GET**: Read (200 OK)
- **POST**: Create (201 Created)
- **PUT**: Update full (204 No Content)
- **PATCH**: Update partial (204 No Content)
- **DELETE**: Delete (204 No Content)

### Status Codes:
- **200 OK**: Success
- **201 Created**: Resource created
- **204 No Content**: Success, no body
- **400 Bad Request**: Invalid request
- **401 Unauthorized**: Not authenticated
- **403 Forbidden**: Not authorized
- **404 Not Found**: Resource not found
- **500 Internal Server Error**: Server error

### Return Types:
- `IActionResult`: Linh hoạt nhất
- `ActionResult<T>`: Recommended
- `T`: Chỉ dữ liệu, khó control status code

### Best Practices:
- ✅ Dùng [ApiController]
- ✅ DTOs thay vì Entities
- ✅ Status codes đúng chuẩn
- ✅ Exception handling
- ✅ Swagger documentation
- ✅ CORS configuration

---

# 🧪 11. MINI TEST

1. **HTTP method nào dùng để tạo resource mới?**
   - A. GET
   - B. POST ✅
   - C. PUT
   - D. DELETE

2. **Status code nào trả về khi tạo resource thành công?**
   - A. 200 OK
   - B. 201 Created ✅
   - C. 204 No Content
   - D. 400 Bad Request

3. **Tại sao nên dùng DTO thay vì Entity trong API?**
   - A. DTO nhanh hơn
   - B. Bảo mật, tránh expose thông tin nhạy cảm ✅
   - C. Entity không serialize được
   - D. Không có lý do

<details>
<summary>💡 Đáp án</summary>

1. **B** - POST dùng để tạo resource mới
2. **B** - 201 Created là status code chuẩn khi tạo resource thành công
3. **B** - DTO chỉ expose fields cần thiết, bảo vệ thông tin nhạy cảm trong Entity
</details>

---

# 📌 12. TÓM TẮT CHƯƠNG

✅ **RESTful API** là kiến trúc cho Web Services  
✅ **HTTP Methods**: GET (read), POST (create), PUT (update), DELETE (delete)  
✅ **Status Codes**: 200, 201, 204, 400, 404, 500  
✅ **DTOs** bảo mật hơn Entities  
✅ **Swagger** cho API documentation  
✅ **CORS** cần cấu hình cho cross-origin requests  

---

**Chương tiếp theo: [09. Authentication & Identity →](./09_authentication_identity.md)**
