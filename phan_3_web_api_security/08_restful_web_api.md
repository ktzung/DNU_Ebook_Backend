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

## 💡 Mẹo nhỏ
> [!TIP]
> Luôn sử dụng **DTO (Data Transfer Object)** để truyền nhận dữ liệu thay vì dùng trực tiếp Entity của Database. Điều này giúp bảo mật và linh hoạt hơn.

> [!IMPORTANT]
> Đừng quên kiểm tra `ModelState.IsValid` nếu bạn không dùng `[ApiController]`. Nhưng với `[ApiController]`, việc này là tự động!
