# 🧪 LAB 02: XÂY DỰNG RESTFUL API (PRODUCTS & CATEGORIES)
**(Tương ứng: Module 02 - Week 2-3)**

## 🎯 Mục tiêu
- **Hiểu kiến trúc REST:** HTTP Methods (GET, POST, PUT, DELETE) hoạt động ra sao.
- **Tư duy DTO (Data Transfer Object):** Tại sao không nên lộ Database Entity ra ngoài?
- **Thực hành:** Viết API quản lý sản phẩm hoàn chỉnh, tuân thủ chuẩn "Clean & Safe".

---

## 1. 🔍 PHÂN TÍCH & TƯ DUY THIẾT KẾ

### 1.1. RESTful API là gì?
Hãy tưởng tượng API giống như người phục vụ nhà hàng (Waiter).
*   **Resource (Món ăn):** Ở đây là `Product`.
*   **HTTP Verbs (Hành động):**
    *   `GET` = Xem thực đơn (Lấy dữ liệu).
    *   `POST` = Gọi món mới (Tạo dữ liệu).
    *   `PUT` = Đổi món (Cập nhật dữ liệu).
    *   `DELETE` = Hủy món (Xóa dữ liệu).

### 1.2. Tại sao cần DTO (Data Transfer Object)?
Database Model (Entity) chứa mọi thứ lưu trong CSDL. Nhưng khi trả về cho Client hoặc nhận từ Client, chúng ta cần "bộ lọc":
1.  **Bảo mật:** Không lộ thông tin nhạy cảm (VD: Người tạo, Ngày sửa, Cấu trúc bảng user).
2.  **Gọn nhẹ:** Chỉ gửi những gì Client cần (VD: List sản phẩm chỉ cần Tên, Giá, Ảnh - không cần chi tiết mô tả dài dòng).
3.  **Tách biệt:** Database đổi tên cột, API không bị chết (do DTO giữ nguyên).

**Chiến lược DTO:**
- **ProductDto (Output):** Dùng để trả dữ liệu ra. (Có Id, Name, Price, CategoryName).
- **ProductCreateDto (Input):** Dùng để nhận dữ liệu tạo mới. (Không cần Id, chỉ cần Name, Price, CategoryId).

---

## 2. 💻 HƯỚNG DẪN THỰC HÀNH CHI TIẾT

### Bước 1: Tạo các DTO Classes (Cái vỏ chứa dữ liệu)

Tạo thư mục `DTOs` trong project.

**DTOs/ProductDto.cs**
```csharp
namespace EShop.API.DTOs
{
    // Class này dùng để TRẢ VỀ dữ liệu (Response)
    public class ProductDto
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public decimal Price { get; set; }
        public int Stock { get; set; }
        
        // Ta không trả về object Category, chỉ trả về tên cho nhẹ
        public string? CategoryName { get; set; } 
    }

    // Class này dùng để NHẬN dữ liệu tạo mới (Request)
    // Validate dữ liệu ngay tại đây
    public class ProductCreateDto
    {
        [Required(ErrorMessage = "Tên sản phẩm không được để trống")]
        public string Name { get; set; } = string.Empty;

        [Range(1000, double.MaxValue, ErrorMessage = "Giá phải ít nhất 1000đ")]
        public decimal Price { get; set; }

        public int Stock { get; set; }

        [Required]
        public int CategoryId { get; set; } // Buộc phải chọn danh mục
    }
}
```

### Bước 2: Tạo ProductsController (Bộ não xử lý)

Tạo file `Controllers/ProductsController.cs`. Chúng ta sẽ code từng phần và giải thích kỹ.

#### Phần 2.1: Khai báo và Dependency Injection

```csharp
using EShop.API.Data;
using EShop.API.DTOs;
using EShop.API.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace EShop.API.Controllers
{
    [Route("api/[controller]")] // URL sẽ là: domain.com/api/products
    [ApiController]             // Báo cho .NET biết đây là API Controller (tự động validate model)
    public class ProductsController : ControllerBase
    {
        private readonly AppDbContext _context;

        // Constructor Injection:
        // Yêu cầu hệ thống "bơm" DbContext vào để Controller dùng
        public ProductsController(AppDbContext context)
        {
            _context = context;
        }

        // ... Các Action Methods sẽ viết ở dưới ...
    }
}
```

#### Phần 2.2: Lấy danh sách (GET)

```csharp
        // 1. GET: api/products
        // Nhiệm vụ: Lấy tất cả sản phẩm, kèm tên danh mục, chuyển sang DTO
        [HttpGet]
        public async Task<IActionResult> GetAll()
        {
            var products = await _context.Products
                .Include(p => p.Category) // KỸ THUẬT EAGER LOADING: Load luôn bảng Category
                .Select(p => new ProductDto // PROJECTION: Chuyển đổi Entity -> DTO
                {
                    Id = p.Id,
                    Name = p.Name,
                    Price = p.Price,
                    Stock = p.Stock,
                    CategoryName = p.Category.Name // Lấy tên từ bảng Category đã include
                })
                .ToListAsync();

            return Ok(products); // Trả về HTTP 200 kèm data
        }
```

#### Phần 2.3: Lấy chi tiết (GET by ID)

```csharp
        // 2. GET: api/products/{id} (VD: api/products/5)
        [HttpGet("{id}")]
        public async Task<IActionResult> GetById(int id)
        {
            // Tìm sản phẩm trong DB
            var product = await _context.Products
                .Include(p => p.Category)
                .FirstOrDefaultAsync(p => p.Id == id);

            // Kiểm tra: Nếu không thấy thì báo lỗi 404 Not Found
            if (product == null) 
                return NotFound(new { message = "Không tìm thấy sản phẩm này" });

            // Manual Mapping: Chuyển Entity sang DTO thủ công
            // (Thực tế thường dùng AutoMapper để đỡ viết đoạn này)
            var productDto = new ProductDto
            {
                Id = product.Id,
                Name = product.Name,
                Price = product.Price,
                Stock = product.Stock,
                CategoryName = product.Category.Name
            };

            return Ok(productDto);
        }
```

#### Phần 2.4: Tạo mới (POST)

```csharp
        // 3. POST: api/products
        // Body nhận vào: ProductCreateDto (JSON)
        [HttpPost]
        public async Task<IActionResult> Create(ProductCreateDto request)
        {
            // 1. Validate nghiệp vụ: Kiểm tra Category ID có tồn tại thật không?
            var category = await _context.Categories.FindAsync(request.CategoryId);
            if (category == null) 
                return BadRequest(new { message = "Danh mục không tồn tại" });

            // 2. Chuyển DTO -> Entity để lưu vào DB
            var newProduct = new Product
            {
                Name = request.Name,
                Price = request.Price,
                Stock = request.Stock,
                CategoryId = request.CategoryId
            };

            // 3. Thêm vào DbContext (Lúc này chưa lưu xuống SQL)
            _context.Products.Add(newProduct);
            
            // 4. Lưu thật sự (Bắn câu lệnh INSERT INTO xuống SQL)
            await _context.SaveChangesAsync();

            // 5. Chuẩn REST: Trả về 201 Created + Header Location trỏ đến resource mới
            return CreatedAtAction(nameof(GetById), new { id = newProduct.Id }, newProduct);
        }
```

#### Phần 2.5: Cập nhật (PUT)

```csharp
        // 4. PUT: api/products/{id}
        // Cập nhật toàn bộ thông tin sản phẩm
        [HttpPut("{id}")]
        public async Task<IActionResult> Update(int id, ProductCreateDto request)
        {
            // 1. Tìm sản phẩm cần sửa
            var product = await _context.Products.FindAsync(id);
            if (product == null) return NotFound();

            // 2. Cập nhật thông tin mới
            product.Name = request.Name;
            product.Price = request.Price;
            product.Stock = request.Stock;
            product.CategoryId = request.CategoryId;

            // 3. Lưu thay đổi
            await _context.SaveChangesAsync();

            return NoContent(); // 204 No Content (Thành công nhưng không trả về data gì)
        }
```

#### Phần 2.6: Xóa (DELETE)

```csharp
        // 5. DELETE: api/products/{id}
        [HttpDelete("{id}")]
        public async Task<IActionResult> Delete(int id)
        {
            var product = await _context.Products.FindAsync(id);
            if (product == null) return NotFound();

            // Xóa khỏi DbContext
            _context.Products.Remove(product);
            
            // Lưu thay đổi
            await _context.SaveChangesAsync();

            return NoContent();
        }
```

### Bước 3: Kiểm thử với Swagger (Test Lab)

1.  Chạy ứng dụng (`F5`).
2.  Mở Swagger UI.
3.  Thử API **POST**:
    *   Nhập CategoryId: 999 (Không tồn tại) -> Mong đợi: Lỗi 400 "Danh mục không tồn tại".
    *   Nhập CategoryId: 1 (Hợp lệ) -> Mong đợi: 201 Created.
4.  Thử API **GET**: Xem sản phẩm vừa tạo có xuất hiện không.

---

## 3. 🧪 BÀI TẬP VỀ NHÀ (CHALLENGES)

### Bài tập 1: Categories API
Hãy tự tay viết `CategoriesController` với đủ 5 chức năng CRUD. Yêu cầu:
*   API `GET /api/categories/{id}` phải trả về kèm danh sách sản phẩm thuộc danh mục đó.
*   Gợi ý: Dùng DTO có property `List<ProductDto> Products`.

### Bài tập 2: Phân trang (Pagination) - Nâng cao
Sửa API `GET /api/products` để nhận tham số: `?page=1&limit=5`.
*   Logic:
    *   Bỏ qua: `(page - 1) * limit` sản phẩm.
    *   Lấy: `limit` sản phẩm.
    *   Code: `.Skip(...).Take(...)`.

---

## ✅ CHECKLIST TỰ ĐÁNH GIÁ
- [ ] Tôi hiểu tại sao phải tách DTO mà không dùng trực tiếp Entity Product.
- [ ] Tôi hiểu ý nghĩa của `.Include()` (như lệnh JOIN SQL).
- [ ] Tôi biết sự khác nhau giữa `200 OK`, `201 Created` và `204 No Content`.
