# 🟩 CHƯƠNG 13
# **MODEL BINDING & VALIDATION**

Model Binding là cơ chế tự động ánh xạ dữ liệu từ HTTP request vào C# objects. Validation đảm bảo dữ liệu nhập vào hợp lệ trước khi xử lý.

---

# 🎯 MỤC TIÊU HỌC TẬP

Sau chương này, bạn sẽ:

- Hiểu Model Binding từ Form, Query, Route
- Sử dụng Data Annotations để validate
- Tạo Custom Validators
- Xử lý validation errors trong Controller và View
- Hiểu ViewModel và DTO pattern
- Xây dựng forms an toàn và user-friendly

---

# 1. **MODEL BINDING**

## 1.1. Binding từ Form Data

```html
<!-- Views/Products/Create.cshtml -->
<form method="post" action="/products/create">
    <input type="text" name="Name" />
    <input type="number" name="Price" />
    <input type="number" name="Stock" />
    <button type="submit">Create</button>
</form>
```

```csharp
// Controller
[HttpPost("create")]
public IActionResult Create(Product product)
{
    // ASP.NET Core tự động bind form fields vào product
    // product.Name = form["Name"]
    // product.Price = form["Price"]
    // product.Stock = form["Stock"]
    
    _db.Products.Add(product);
    _db.SaveChanges();
    
    return RedirectToAction("Index");
}
```

---

## 1.2. Binding từ Query String

```csharp
// URL: /products?page=2&pageSize=10&sort=price
[HttpGet]
public IActionResult Index(int page = 1, int pageSize = 10, string? sort = null)
{
    // page = 2
    // pageSize = 10
    // sort = "price"
}
```

---

## 1.3. Binding từ Route Parameters

```csharp
// URL: /products/5
[HttpGet("{id}")]
public IActionResult Details(int id)
{
    // id = 5
}

// URL: /products/laptop/electronics
[HttpGet("{slug}/{category}")]
public IActionResult Details(string slug, string category)
{
    // slug = "laptop"
    // category = "electronics"
}
```

---

## 1.4. Binding từ Request Body (JSON)

```csharp
// POST /api/products
// Body: { "name": "Laptop", "price": 1500 }

[HttpPost]
public IActionResult Create([FromBody] Product product)
{
    // ASP.NET Core deserialize JSON → Product object
}
```

---

## 1.5. Binding Attributes

```csharp
public IActionResult Create(
    [FromForm] Product product,      // Form data
    [FromQuery] int page,             // Query string
    [FromRoute] int id,               // Route parameter
    [FromBody] CreateRequest request, // JSON body
    [FromHeader] string authorization // HTTP header
)
```

---

# 2. **DATA ANNOTATIONS — VALIDATION CƠ BẢN**

## 2.1. Required — Bắt buộc nhập

```csharp
public class Product
{
    public int Id { get; set; }
    
    [Required(ErrorMessage = "Product name is required")]
    public string Name { get; set; } = string.Empty;
    
    [Required]
    public decimal Price { get; set; }
}
```

---

## 2.2. StringLength — Độ dài chuỗi

```csharp
public class Product
{
    [Required]
    [StringLength(200, MinimumLength = 3, 
        ErrorMessage = "Name must be between 3 and 200 characters")]
    public string Name { get; set; } = string.Empty;
    
    [StringLength(2000)]
    public string Description { get; set; } = string.Empty;
}
```

---

## 2.3. Range — Giới hạn số

```csharp
public class Product
{
    [Range(0.01, 1000000, ErrorMessage = "Price must be between 0.01 and 1,000,000")]
    public decimal Price { get; set; }
    
    [Range(0, int.MaxValue, ErrorMessage = "Stock cannot be negative")]
    public int Stock { get; set; }
}
```

---

## 2.4. EmailAddress — Validate email

```csharp
public class User
{
    [Required]
    [EmailAddress(ErrorMessage = "Invalid email format")]
    public string Email { get; set; } = string.Empty;
}
```

---

## 2.5. RegularExpression — Pattern matching

```csharp
public class User
{
    [RegularExpression(@"^[0-9]{10}$", 
        ErrorMessage = "Phone must be 10 digits")]
    public string Phone { get; set; } = string.Empty;
    
    [RegularExpression(@"^[a-zA-Z0-9_-]+$", 
        ErrorMessage = "Username can only contain letters, numbers, _ and -")]
    public string Username { get; set; } = string.Empty;
}
```

---

## 2.6. Compare — So sánh 2 fields

```csharp
public class RegisterRequest
{
    [Required]
    [DataType(DataType.Password)]
    public string Password { get; set; } = string.Empty;
    
    [Required]
    [Compare("Password", ErrorMessage = "Passwords do not match")]
    [DataType(DataType.Password)]
    public string ConfirmPassword { get; set; } = string.Empty;
}
```

---

## 2.7. Url — Validate URL

```csharp
public class Product
{
    [Url(ErrorMessage = "Invalid URL format")]
    public string? ImageUrl { get; set; }
}
```

---

## 2.8. CreditCard — Validate thẻ tín dụng

```csharp
public class PaymentInfo
{
    [CreditCard(ErrorMessage = "Invalid credit card number")]
    public string CardNumber { get; set; } = string.Empty;
}
```

---

# 3. **CUSTOM VALIDATION**

## 3.1. Custom Validation Attribute

```csharp
// Custom validator: Giá phải chia hết cho 1000
public class PriceRoundingAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (value is decimal price)
        {
            if (price % 1000 != 0)
            {
                return new ValidationResult("Price must be rounded to thousands (e.g., 1000, 2000)");
            }
        }
        
        return ValidationResult.Success;
    }
}

// Sử dụng
public class Product
{
    [Required]
    [PriceRounding]
    public decimal Price { get; set; }
}
```

---

## 3.2. IValidatableObject — Validation phức tạp

```csharp
public class Product : IValidatableObject
{
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public decimal DiscountPrice { get; set; }
    
    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        // Validate: DiscountPrice phải < Price
        if (DiscountPrice >= Price)
        {
            yield return new ValidationResult(
                "Discount price must be less than regular price",
                new[] { nameof(DiscountPrice) }
            );
        }
        
        // Validate: Nếu có discount, phải > 10%
        if (DiscountPrice > 0)
        {
            var discountPercent = (Price - DiscountPrice) / Price * 100;
            if (discountPercent < 10)
            {
                yield return new ValidationResult(
                    "Discount must be at least 10%",
                    new[] { nameof(DiscountPrice) }
                );
            }
        }
    }
}
```

---

# 4. **VALIDATION TRONG CONTROLLER**

## 4.1. ModelState.IsValid

```csharp
[HttpPost("create")]
public async Task<IActionResult> Create(Product product)
{
    // Kiểm tra validation
    if (!ModelState.IsValid)
    {
        // Trả về view với errors
        return View(product);
    }
    
    // Validation pass → Xử lý logic
    await _db.Products.AddAsync(product);
    await _db.SaveChangesAsync();
    
    TempData["SuccessMessage"] = "Product created successfully!";
    return RedirectToAction("Index");
}
```

---

## 4.2. Manual Validation

```csharp
[HttpPost("create")]
public async Task<IActionResult> Create(Product product)
{
    // Custom validation logic
    if (await _db.Products.AnyAsync(p => p.Name == product.Name))
    {
        ModelState.AddModelError("Name", "Product name already exists");
    }
    
    if (product.Price < 1000)
    {
        ModelState.AddModelError("Price", "Price must be at least 1000");
    }
    
    if (!ModelState.IsValid)
    {
        return View(product);
    }
    
    // Save to database
    await _db.Products.AddAsync(product);
    await _db.SaveChangesAsync();
    
    return RedirectToAction("Index");
}
```

---

## 4.3. Validation Error Response (API)

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    public async Task<ActionResult<Product>> Create(CreateProductRequest request)
    {
        if (!ModelState.IsValid)
        {
            // Trả về 400 Bad Request với errors
            return BadRequest(ModelState);
        }
        
        // Process...
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
    }
}
```

**Response:**
```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Name": ["Product name is required"],
    "Price": ["Price must be between 0.01 and 1,000,000"]
  }
}
```

---

# 5. **VIEWMODEL & DTO PATTERN**

## 5.1. Vấn đề khi dùng Entity trực tiếp

```csharp
// ❌ BAD: Expose Entity trực tiếp
[HttpPost]
public IActionResult Create(Product product)
{
    // Nguy hiểm: Client có thể set bất kỳ property nào
    // Ví dụ: product.Id = 999 (hack!)
}
```

---

## 5.2. ViewModel/DTO Solution

```csharp
// ✅ GOOD: Dùng DTO
public record CreateProductRequest(
    string Name,
    string Description,
    decimal Price,
    int Stock,
    int CategoryId
);

[HttpPost]
public async Task<IActionResult> Create(CreateProductRequest request)
{
    if (!ModelState.IsValid)
        return View(request);
    
    // Map DTO → Entity
    var product = new Product
    {
        Name = request.Name,
        Description = request.Description,
        Price = request.Price,
        Stock = request.Stock,
        CategoryId = request.CategoryId,
        CreatedAt = DateTime.UtcNow
    };
    
    await _db.Products.AddAsync(product);
    await _db.SaveChangesAsync();
    
    return RedirectToAction("Index");
}
```

---

## 5.3. ViewModel cho View phức tạp

```csharp
// ViewModel kết hợp nhiều data
public class ProductCreateViewModel
{
    [Required]
    [StringLength(200)]
    public string Name { get; set; } = string.Empty;
    
    [Required]
    [Range(0.01, 1000000)]
    public decimal Price { get; set; }
    
    [Required]
    public int CategoryId { get; set; }
    
    // Danh sách categories cho dropdown
    public List<Category> Categories { get; set; } = new();
}

// Controller
[HttpGet("create")]
public async Task<IActionResult> Create()
{
    var viewModel = new ProductCreateViewModel
    {
        Categories = await _db.Categories.ToListAsync()
    };
    
    return View(viewModel);
}

[HttpPost("create")]
public async Task<IActionResult> Create(ProductCreateViewModel model)
{
    if (!ModelState.IsValid)
    {
        // Reload categories nếu validation fail
        model.Categories = await _db.Categories.ToListAsync();
        return View(model);
    }
    
    // Map ViewModel → Entity
    var product = new Product
    {
        Name = model.Name,
        Price = model.Price,
        CategoryId = model.CategoryId
    };
    
    await _db.Products.AddAsync(product);
    await _db.SaveChangesAsync();
    
    return RedirectToAction("Index");
}
```

---

# 6. **VALIDATION TRONG VIEW (RAZOR)**

## 6.1. Validation Summary

```html
@model ProductCreateViewModel

<form method="post">
    <!-- Hiển thị tất cả errors -->
    <div asp-validation-summary="All" class="text-danger"></div>
    
    <div>
        <label asp-for="Name"></label>
        <input asp-for="Name" />
        <span asp-validation-for="Name" class="text-danger"></span>
    </div>
    
    <div>
        <label asp-for="Price"></label>
        <input asp-for="Price" />
        <span asp-validation-for="Price" class="text-danger"></span>
    </div>
    
    <button type="submit">Create</button>
</form>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

---

## 6.2. Client-Side Validation (jQuery)

ASP.NET Core tự động generate data attributes cho client-side validation:

```html
<input type="text" 
       name="Name" 
       id="Name"
       data-val="true" 
       data-val-required="Product name is required"
       data-val-length="Name must be between 3 and 200 characters"
       data-val-length-max="200"
       data-val-length-min="3" />
```

jQuery Unobtrusive Validation tự động validate trên client trước khi submit.

---

# 7. **BÀI TẬP THỰC HÀNH**

## 📝 Bài 1: Create User Form

Tạo form đăng ký User với validation:
- Email: required, email format
- Password: required, min 8 characters, must contain number and uppercase
- ConfirmPassword: must match Password
- FullName: required, 3-100 characters

<details>
<summary>💡 Đáp án</summary>

```csharp
public class RegisterViewModel
{
    [Required]
    [EmailAddress]
    public string Email { get; set; } = string.Empty;
    
    [Required]
    [StringLength(100, MinimumLength = 8)]
    [RegularExpression(@"^(?=.*[A-Z])(?=.*\d).+$", 
        ErrorMessage = "Password must contain at least one uppercase letter and one number")]
    [DataType(DataType.Password)]
    public string Password { get; set; } = string.Empty;
    
    [Required]
    [Compare("Password")]
    [DataType(DataType.Password)]
    public string ConfirmPassword { get; set; } = string.Empty;
    
    [Required]
    [StringLength(100, MinimumLength = 3)]
    public string FullName { get; set; } = string.Empty;
}
```
</details>

---

## 📝 Bài 2: Custom Validator

Tạo custom validator kiểm tra giá giảm phải nhỏ hơn giá gốc.

<details>
<summary>💡 Đáp án</summary>

```csharp
public class DiscountPriceAttribute : ValidationAttribute
{
    private readonly string _comparisonProperty;
    
    public DiscountPriceAttribute(string comparisonProperty)
    {
        _comparisonProperty = comparisonProperty;
    }
    
    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        var currentValue = (decimal?)value;
        
        var property = validationContext.ObjectType.GetProperty(_comparisonProperty);
        if (property == null)
            throw new ArgumentException("Property not found");
        
        var comparisonValue = (decimal?)property.GetValue(validationContext.ObjectInstance);
        
        if (currentValue.HasValue && comparisonValue.HasValue && currentValue >= comparisonValue)
        {
            return new ValidationResult("Discount price must be less than regular price");
        }
        
        return ValidationResult.Success;
    }
}

// Usage
public class Product
{
    public decimal Price { get; set; }
    
    [DiscountPrice("Price")]
    public decimal? DiscountPrice { get; set; }
}
```
</details>

---

## ❌ 6. CÁC LỖI THƯỜNG GẶP

### ❌ Lỗi 1: Quên kiểm tra ModelState.IsValid

```csharp
// ❌ Vấn đề: Không kiểm tra validation
[HttpPost]
public IActionResult Create(Product product)
{
    _db.Products.Add(product); // ❌ Có thể có dữ liệu không hợp lệ
    _db.SaveChanges();
    return RedirectToAction("Index");
}

// ✅ Giải pháp: Luôn kiểm tra ModelState
[HttpPost]
public IActionResult Create(Product product)
{
    if (!ModelState.IsValid)
    {
        return View(product); // Hiển thị lại form với errors
    }
    
    _db.Products.Add(product);
    _db.SaveChanges();
    return RedirectToAction("Index");
}
```

**🔍 Giải thích:** ModelState.IsValid kiểm tra tất cả validation rules. Quên kiểm tra có thể lưu dữ liệu không hợp lệ.

---

### ❌ Lỗi 2: Over-posting Attack

```csharp
// ❌ Vấn đề: Bind trực tiếp Entity, client có thể set IsAdmin
public class User
{
    public string Name { get; set; }
    public bool IsAdmin { get; set; } // ❌ Client có thể set = true
}

[HttpPost]
public IActionResult Create(User user) // ❌ Over-posting!
{
    _db.Users.Add(user);
    _db.SaveChanges();
}

// ✅ Giải pháp: Dùng ViewModel/DTO
public class CreateUserRequest
{
    public string Name { get; set; } // Chỉ có fields cần thiết
    // Không có IsAdmin
}

[HttpPost]
public IActionResult Create(CreateUserRequest request)
{
    var user = new User { Name = request.Name, IsAdmin = false };
    _db.Users.Add(user);
    _db.SaveChanges();
}
```

**🔍 Giải thích:** Bind trực tiếp Entity cho phép client set các property không mong muốn. Dùng ViewModel để kiểm soát.

---

### ❌ Lỗi 3: Validation không chạy

```csharp
// ❌ Vấn đề: Quên [ApiController] hoặc [Bind] attribute
public class ProductsController : Controller
{
    [HttpPost]
    public IActionResult Create(Product product) // Validation không chạy
    {
        // ModelState.IsValid luôn true
    }
}

// ✅ Giải pháp: Đảm bảo validation được trigger
[ApiController] // Tự động validate
public class ProductsController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(Product product)
    {
        if (!ModelState.IsValid) // ✅ Validation chạy
            return BadRequest(ModelState);
    }
}
```

**🔍 Giải thích:** [ApiController] tự động validate. Với MVC, cần đảm bảo validation attributes được áp dụng.

---

## 🎯 7. CASE STUDY / VÍ DỤ THỰC TẾ

### Case Study 1: Form đăng ký với Validation đầy đủ

**Yêu cầu:** Tạo form đăng ký với validation email, password strength, confirm password.

```csharp
// ViewModel
public class RegisterViewModel
{
    [Required(ErrorMessage = "Email là bắt buộc")]
    [EmailAddress(ErrorMessage = "Email không hợp lệ")]
    [Display(Name = "Email")]
    public string Email { get; set; } = string.Empty;
    
    [Required(ErrorMessage = "Mật khẩu là bắt buộc")]
    [StringLength(100, MinimumLength = 8, ErrorMessage = "Mật khẩu phải từ 8-100 ký tự")]
    [DataType(DataType.Password)]
    [Display(Name = "Mật khẩu")]
    public string Password { get; set; } = string.Empty;
    
    [Required(ErrorMessage = "Xác nhận mật khẩu là bắt buộc")]
    [DataType(DataType.Password)]
    [Display(Name = "Xác nhận mật khẩu")]
    [Compare("Password", ErrorMessage = "Mật khẩu không khớp")]
    public string ConfirmPassword { get; set; } = string.Empty;
    
    [Required(ErrorMessage = "Họ tên là bắt buộc")]
    [StringLength(100, ErrorMessage = "Họ tên không quá 100 ký tự")]
    [Display(Name = "Họ tên")]
    public string FullName { get; set; } = string.Empty;
    
    [Range(18, 120, ErrorMessage = "Tuổi phải từ 18-120")]
    [Display(Name = "Tuổi")]
    public int? Age { get; set; }
    
    [Display(Name = "Tôi đồng ý với điều khoản")]
    [MustBeTrue(ErrorMessage = "Bạn phải đồng ý với điều khoản")]
    public bool AgreeToTerms { get; set; }
}

// Controller
[HttpGet("register")]
public IActionResult Register()
{
    return View(new RegisterViewModel());
}

[HttpPost("register")]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Register(RegisterViewModel model)
{
    if (!ModelState.IsValid)
    {
        return View(model);
    }
    
    // Kiểm tra email đã tồn tại
    if (await _userService.EmailExistsAsync(model.Email))
    {
        ModelState.AddModelError(nameof(model.Email), "Email đã được sử dụng");
        return View(model);
    }
    
    // Tạo user
    var user = await _userService.CreateUserAsync(model);
    
    TempData["SuccessMessage"] = "Đăng ký thành công!";
    return RedirectToAction("Login");
}
```

**Best practices:**
- ViewModel riêng cho form
- Validation attributes đầy đủ
- Custom validation cho logic phức tạp
- Server-side validation bổ sung

---

## ✅ 8. BEST PRACTICES

### 8.1. Validation Best Practices

```csharp
// ✅ Đúng: Validation ở nhiều lớp
public class Product
{
    [Required]
    [StringLength(100)]
    public string Name { get; set; }
    
    [Range(0.01, 1000000)]
    public decimal Price { get; set; }
}

// Controller
if (!ModelState.IsValid)
    return View(model);

// Service layer validation
if (await _repository.NameExistsAsync(product.Name))
    throw new ValidationException("Tên sản phẩm đã tồn tại");
```

### 8.2. ViewModel Best Practices

```csharp
// ✅ Đúng: ViewModel cho mỗi use case
public class CreateProductViewModel { }
public class EditProductViewModel { }
public class ProductListViewModel { }

// ❌ Sai: Dùng Entity trực tiếp
public IActionResult Create(Product product) { } // ❌
```

### 8.3. Model Binding Best Practices

```csharp
// ✅ Đúng: [FromBody] cho API
[HttpPost]
public IActionResult Create([FromBody] CreateProductRequest request) { }

// ✅ Đúng: [FromForm] cho MVC form
[HttpPost]
public IActionResult Create([FromForm] CreateProductViewModel model) { }

// ✅ Đúng: [FromQuery] cho query parameters
[HttpGet]
public IActionResult Search([FromQuery] string keyword) { }
```

---

# 📝 9. QUICK NOTES

### Model Binding Sources:
- **Form Data**: `[FromForm]` (default cho POST)
- **Query String**: `[FromQuery]`
- **Route**: `[FromRoute]`
- **Body**: `[FromBody]` (JSON cho API)

### Data Annotations:
- `[Required]`: Bắt buộc
- `[StringLength]`: Độ dài chuỗi
- `[Range]`: Khoảng giá trị
- `[EmailAddress]`: Email
- `[Compare]`: So sánh với property khác
- `[RegularExpression]`: Regex pattern

### Validation:
- `ModelState.IsValid`: Kiểm tra validation
- `ModelState.AddModelError()`: Thêm lỗi thủ công
- Client-side: jQuery Unobtrusive Validation

### Best Practices:
- ✅ Dùng ViewModel thay vì Entity
- ✅ Validate ở nhiều lớp
- ✅ Kiểm tra ModelState.IsValid
- ✅ Custom validators cho logic phức tạp

---

# 🧪 10. MINI TEST

1. **ModelState.IsValid kiểm tra gì?**
   - A. Database connection
   - B. Validation errors
   - C. User authentication

2. **Data Annotation nào validate email?**
   - A. [Email]
   - B. [EmailAddress]
   - C. [ValidEmail]

3. **Tại sao nên dùng ViewModel thay vì Entity?**
   - A. ViewModel nhanh hơn
   - B. Bảo mật, tránh over-posting
   - C. Entity không validate được

<details>
<summary>💡 Đáp án</summary>

1. **B** - Validation errors
2. **B** - [EmailAddress]
3. **B** - Bảo mật, tránh client set các property không mong muốn (over-posting attack)
</details>

---

# 📌 TÓM TẮT CHƯƠNG

✅ **Model Binding** tự động map dữ liệu từ Form, Query, Route, Body vào objects  
✅ **Data Annotations** validate cơ bản: Required, StringLength, Range, Email...  
✅ **Custom Validators** cho logic phức tạp  
✅ **ModelState.IsValid** kiểm tra validation  
✅ **ViewModel/DTO** bảo mật hơn Entity  
✅ **Client-side validation** với jQuery Unobtrusive  

---

**Chương tiếp theo: [14. Layout, Partial Views & Bootstrap →](./04_Layout_Bootstrap.md)**
