# 🟨 CHƯƠNG 11
# **AUTHORIZATION — PHÂN QUYỀN NÂNG CAO**

Sau khi có Authentication (xác thực), Authorization (phân quyền) quyết định user được làm gì. Chương này hướng dẫn các kỹ thuật phân quyền nâng cao trong ASP.NET Core.

---

# 🎯 MỤC TIÊU HỌC TẬP

Sau chương này, bạn sẽ:

- Phân biệt Role-based và Claims-based Authorization
- Implement Role-based Authorization
- Implement Policy-based Authorization
- Sử dụng Resource-based Authorization
- Custom Authorization Requirements
- Phân quyền trong Views
- Best practices phân quyền

---

# 1. **ROLE-BASED AUTHORIZATION**

## 1.1. Simple Role Check

```csharp
// Chỉ Admin mới vào được
[Authorize(Roles = "Admin")]
public class AdminController : Controller
{
    public IActionResult Dashboard()
    {
        return View();
    }
}

// Admin HOẶC Manager
[Authorize(Roles = "Admin,Manager")]
public IActionResult Reports()
{
    return View();
}

// Specific action
public class ProductsController : Controller
{
    // Public
    public IActionResult Index() { }
    
    // User đã đăng nhập
    [Authorize]
    public IActionResult Details(int id) { }
    
    // Chỉ Admin
    [Authorize(Roles = "Admin")]
    public IActionResult Create() { }
}
```

---

## 1.2. Multiple Roles (AND logic)

```csharp
// Phải vừa là Admin VỪA là Manager
[Authorize(Roles = "Admin")]
[Authorize(Roles = "Manager")]
public IActionResult SpecialAction()
{
    // User phải có CẢ 2 roles
}
```

---

## 1.3. Check Role trong Code

```csharp
public class OrdersController : Controller
{
    public async Task<IActionResult> Index()
    {
        List<Order> orders;
        
        if (User.IsInRole("Admin"))
        {
            // Admin xem tất cả orders
            orders = await _db.Orders.ToListAsync();
        }
        else
        {
            // User chỉ xem orders của mình
            var userId = User.FindFirstValue(ClaimTypes.NameIdentifier);
            orders = await _db.Orders
                .Where(o => o.UserId == int.Parse(userId))
                .ToListAsync();
        }
        
        return View(orders);
    }
}
```

---

# 2. **CLAIMS-BASED AUTHORIZATION**

## 2.1. Claims là gì?

**Claim** = Một thông tin về user (key-value pair).

### 🏠 Ví dụ: Thẻ CCCD

```
CCCD:
- Name: Nguyễn Văn A        ← Claim: "Name" = "Nguyễn Văn A"
- BirthYear: 1990           ← Claim: "BirthYear" = "1990"
- Address: Hà Nội           ← Claim: "Address" = "Hà Nội"
```

ASP.NET Core Identity tự động tạo claims:
- `ClaimTypes.NameIdentifier` → UserId
- `ClaimTypes.Email` → Email
- `ClaimTypes.Role` → Role

---

## 2.2. Custom Claims

```csharp
// Thêm custom claim khi user đăng ký
public async Task<IActionResult> Register(RegisterViewModel model)
{
    var user = new ApplicationUser
    {
        UserName = model.Email,
        Email = model.Email,
        FullName = model.FullName
    };
    
    var result = await _userManager.CreateAsync(user, model.Password);
    
    if (result.Succeeded)
    {
        // Thêm claims
        await _userManager.AddClaimsAsync(user, new[]
        {
            new Claim("FullName", model.FullName),
            new Claim("MemberSince", DateTime.UtcNow.ToString("yyyy-MM-dd")),
            new Claim("Country", "Vietnam")
        });
        
        await _signInManager.SignInAsync(user, false);
        return RedirectToAction("Index", "Home");
    }
    
    return View(model);
}
```

---

## 2.3. Require Claim

```csharp
// Chỉ user có claim "Department" = "IT" mới vào được
[Authorize]
public class ITController : Controller
{
    public IActionResult Index()
    {
        if (!User.HasClaim("Department", "IT"))
        {
            return Forbid(); // 403 Forbidden
        }
        
        return View();
    }
}
```

---

# 3. **POLICY-BASED AUTHORIZATION**

## 3.1. Định nghĩa Policies

```csharp
// Program.cs
builder.Services.AddAuthorization(options =>
{
    // Policy 1: Chỉ Admin
    options.AddPolicy("RequireAdminRole", policy =>
        policy.RequireRole("Admin"));
    
    // Policy 2: Admin HOẶC Manager
    options.AddPolicy("RequireManagerRole", policy =>
        policy.RequireRole("Admin", "Manager"));
    
    // Policy 3: Phải có email verified
    options.AddPolicy("RequireVerifiedEmail", policy =>
        policy.RequireClaim(ClaimTypes.Email)
              .Requirements.Add(new EmailVerifiedRequirement()));
    
    // Policy 4: Tuổi >= 18
    options.AddPolicy("RequireAdultAge", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(18)));
    
    // Policy 5: Kết hợp nhiều điều kiện
    options.AddPolicy("RequirePremiumAccess", policy =>
        policy.RequireRole("User")
              .RequireClaim("MembershipLevel", "Premium", "Gold"));
});
```

---

## 3.2. Sử dụng Policy

```csharp
[Authorize(Policy = "RequireAdminRole")]
public class AdminController : Controller
{
    public IActionResult Dashboard() { }
}

[Authorize(Policy = "RequireManagerRole")]
public IActionResult Reports() { }

[Authorize(Policy = "RequireAdultAge")]
public IActionResult AdultContent() { }
```

---

# 4. **CUSTOM AUTHORIZATION REQUIREMENTS**

## 4.1. Minimum Age Requirement

```csharp
// Requirements/MinimumAgeRequirement.cs
public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }
    
    public MinimumAgeRequirement(int minimumAge)
    {
        MinimumAge = minimumAge;
    }
}

// Handlers/MinimumAgeHandler.cs
public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        MinimumAgeRequirement requirement)
    {
        var birthDateClaim = context.User.FindFirst(c => c.Type == "BirthDate");
        
        if (birthDateClaim == null)
        {
            return Task.CompletedTask; // Fail (không có claim)
        }
        
        var birthDate = DateTime.Parse(birthDateClaim.Value);
        var age = DateTime.Today.Year - birthDate.Year;
        
        if (birthDate.Date > DateTime.Today.AddYears(-age))
            age--;
        
        if (age >= requirement.MinimumAge)
        {
            context.Succeed(requirement); // Pass
        }
        
        return Task.CompletedTask;
    }
}

// Program.cs
builder.Services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();

builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("RequireAdult", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(18)));
});
```

---

## 4.2. Business Hours Requirement

```csharp
// Requirements/BusinessHoursRequirement.cs
public class BusinessHoursRequirement : IAuthorizationRequirement
{
}

// Handlers/BusinessHoursHandler.cs
public class BusinessHoursHandler : AuthorizationHandler<BusinessHoursRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        BusinessHoursRequirement requirement)
    {
        var currentTime = DateTime.Now.TimeOfDay;
        var startTime = new TimeSpan(9, 0, 0);  // 9 AM
        var endTime = new TimeSpan(17, 0, 0);   // 5 PM
        
        if (currentTime >= startTime && currentTime <= endTime)
        {
            context.Succeed(requirement);
        }
        
        return Task.CompletedTask;
    }
}

// Usage
[Authorize(Policy = "BusinessHoursOnly")]
public IActionResult ProcessPayment() { }
```

---

# 5. **RESOURCE-BASED AUTHORIZATION**

## 5.1. Authorization Service

```csharp
// Authorizatio/ProductAuthorizationHandler.cs
public class ProductOperationRequirement : IAuthorizationRequirement
{
    public string Operation { get; }
    
    public ProductOperationRequirement(string operation)
    {
        Operation = operation;
    }
}

public static class ProductOperations
{
    public static ProductOperationRequirement Create = new("Create");
    public static ProductOperationRequirement Read = new("Read");
    public static ProductOperationRequirement Update = new("Update");
    public static ProductOperationRequirement Delete = new("Delete");
}

public class ProductAuthorizationHandler : 
    AuthorizationHandler<ProductOperationRequirement, Product>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        ProductOperationRequirement requirement,
        Product resource)
    {
        // Admin có tất cả quyền
        if (context.User.IsInRole("Admin"))
        {
            context.Succeed(requirement);
            return Task.CompletedTask;
        }
        
        // Manager chỉ có quyền Create, Update
        if (context.User.IsInRole("Manager"))
        {
            if (requirement.Operation == "Create" || requirement.Operation == "Update")
            {
                context.Succeed(requirement);
            }
        }
        
        // User chỉ có quyền Read
        if (requirement.Operation == "Read")
        {
            context.Succeed(requirement);
        }
        
        return Task.CompletedTask;
    }
}

// Program.cs
builder.Services.AddScoped<IAuthorizationHandler, ProductAuthorizationHandler>();
```

---

## 5.2. Sử dụng trong Controller

```csharp
public class ProductsController : Controller
{
    private readonly IAuthorizationService _authorizationService;
    private readonly AppDbContext _db;
    
    public ProductsController(
        IAuthorizationService authorizationService,
        AppDbContext db)
    {
        _authorizationService = authorizationService;
        _db = db;
    }
    
    public async Task<IActionResult> Edit(int id)
    {
        var product = await _db.Products.FindAsync(id);
        if (product == null)
            return NotFound();
        
        // Kiểm tra quyền
        var authResult = await _authorizationService.AuthorizeAsync(
            User, 
            product, 
            ProductOperations.Update);
        
        if (!authResult.Succeeded)
        {
            return Forbid(); // 403 Forbidden
        }
        
        return View(product);
    }
    
    [HttpPost]
    public async Task<IActionResult> Delete(int id)
    {
        var product = await _db.Products.FindAsync(id);
        if (product == null)
            return NotFound();
        
        var authResult = await _authorizationService.AuthorizeAsync(
            User, 
            product, 
            ProductOperations.Delete);
        
        if (!authResult.Succeeded)
        {
            return Forbid();
        }
        
        _db.Products.Remove(product);
        await _db.SaveChangesAsync();
        
        return RedirectToAction("Index");
    }
}
```

---

# 6. **AUTHORIZATION TRONG VIEWS**

## 6.1. User.IsInRole

```html
@if (User.Identity?.IsAuthenticated == true)
{
    <p>Welcome, @User.Identity.Name!</p>
    
    @if (User.IsInRole("Admin"))
    {
        <a href="/Admin/Dashboard">Admin Dashboard</a>
    }
    
    @if (User.IsInRole("Manager"))
    {
        <a href="/Reports">View Reports</a>
    }
}
else
{
    <a href="/Account/Login">Login</a>
}
```

---

## 6.2. IAuthorizationService trong View

```html
@inject IAuthorizationService AuthorizationService

<h1>Products</h1>

@foreach (var product in Model)
{
    <div>
        <h3>@product.Name</h3>
        <p>@product.Description</p>
        
        @if ((await AuthorizationService.AuthorizeAsync(
            User, product, ProductOperations.Update)).Succeeded)
        {
            <a asp-action="Edit" asp-route-id="@product.Id">Edit</a>
        }
        
        @if ((await AuthorizationService.AuthorizeAsync(
            User, product, ProductOperations.Delete)).Succeeded)
        {
            <form method="post" asp-action="Delete" asp-route-id="@product.Id">
                <button type="submit">Delete</button>
            </form>
        }
    </div>
}
```

---

# 7. **VÍ DỤ THỰC TẾ: E-SHOP AUTHORIZATION**

```csharp
// Program.cs
builder.Services.AddAuthorization(options =>
{
    // User thường
    options.AddPolicy("UserPolicy", policy =>
        policy.RequireRole("User"));
    
    // Admin
    options.AddPolicy("AdminPolicy", policy =>
        policy.RequireRole("Admin"));
    
    // Quản lý sản phẩm (Admin hoặc Manager)
    options.AddPolicy("ManageProducts", policy =>
        policy.RequireRole("Admin", "Manager"));
    
    // Xem báo cáo
    options.AddPolicy("ViewReports", policy =>
        policy.RequireRole("Admin", "Manager"));
    
    // Quản lý users (chỉ Admin)
    options.AddPolicy("ManageUsers", policy =>
        policy.RequireRole("Admin"));
});

// Controllers
[Authorize(Policy = "UserPolicy")]
public class OrdersController : Controller
{
    public async Task<IActionResult> MyOrders() { }
}

[Authorize(Policy = "ManageProducts")]
public class ProductsAdminController : Controller
{
    public IActionResult Create() { }
    public IActionResult Edit(int id) { }
}

[Authorize(Policy = "AdminPolicy")]
public class UsersController : Controller
{
    public IActionResult Index() { }
    public IActionResult AssignRole(int userId, string role) { }
}
```

---

# 8. **BÀI TẬP THỰC HÀNH**

## 📝 Bài 1: Own Resource Authorization

Implement authorization: User chỉ có thể edit/delete orders của chính mình.

<details>
<summary>💡 Đáp án</summary>

```csharp
public class OrderOwnerRequirement : IAuthorizationRequirement { }

public class OrderOwnerHandler : 
    AuthorizationHandler<OrderOwnerRequirement, Order>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        OrderOwnerRequirement requirement,
        Order resource)
    {
        var userId = context.User.FindFirstValue(ClaimTypes.NameIdentifier);
        
        if (resource.UserId == int.Parse(userId) || context.User.IsInRole("Admin"))
        {
            context.Succeed(requirement);
        }
        
        return Task.CompletedTask;
    }
}

// Program.cs
builder.Services.AddScoped<IAuthorizationHandler, OrderOwnerHandler>();
options.AddPolicy("OrderOwner", policy =>
    policy.Requirements.Add(new OrderOwnerRequirement()));

// Controller
public async Task<IActionResult> Cancel(int id)
{
    var order = await _db.Orders.FindAsync(id);
    
    var authResult = await _authorizationService.AuthorizeAsync(
        User, order, "OrderOwner");
    
    if (!authResult.Succeeded)
        return Forbid();
    
    order.Status = "Cancelled";
    await _db.SaveChangesAsync();
    
    return RedirectToAction("MyOrders");
}
```
</details>

---

# 🧪 MINI TEST

1. **Role-based vs Policy-based?**
   - A. Role đơn giản, Policy linh hoạt hơn
   - B. Không khác gì
   - C. Policy nhanh hơn

2. **IAuthorizationService dùng để làm gì?**
   - A. Kiểm tra quyền trong code
   - B. Tạo user mới
   - C. Hash password

3. **Resource-based authorization dùng khi nào?**
   - A. Khi quyền phụ thuộc vào resource cụ thể
   - B. Khi có nhiều roles
   - C. Không cần dùng

<details>
<summary>💡 Đáp án</summary>

1. **A** - Role đơn giản (check role), Policy linh hoạt (custom logic)
2. **A** - Kiểm tra quyền trong code (không dùng [Authorize])
3. **A** - Khi quyền phụ thuộc vào resource (vd: chỉ owner mới edit được)
</details>

---

# 📌 TÓM TẮT CHƯƠNG

✅ **Role-based**: Đơn giản, dựa vào roles  
✅ **Claims-based**: Linh hoạt, dựa vào claims  
✅ **Policy-based**: Mạnh mẽ, custom logic phức tạp  
✅ **Resource-based**: Quyền phụ thuộc vào resource cụ thể  
✅ **Custom Requirements**: Tự định nghĩa logic phân quyền  
✅ **IAuthorizationService**: Check quyền trong code  

---

**Chương tiếp theo: [12. Advanced Topics →](../phan_4_advanced/12_advanced_topics.md)**
