# 🟨 CHƯƠNG 09
# **AUTHENTICATION & IDENTITY**

ASP.NET Core Identity là framework hoàn chỉnh để quản lý User, Password, Role. Chương này hướng dẫn triển khai hệ thống đăng nhập/đăng ký an toàn cho E-Shop.

---

# 🎯 MỤC TIÊU HỌC TẬP

Sau chương này, bạn sẽ:

- Hiểu Authentication vs Authorization
- Cấu hình ASP.NET Core Identity
- Implement Register/Login/Logout
- Hash password an toàn
- Quản lý User Roles
- Sử dụng [Authorize] attribute
- Tích hợp Identity vào E-Shop

---

# 1. **AUTHENTICATION VS AUTHORIZATION**

## 1.1. Authentication (Xác thực) — "Bạn là ai?"

**Authentication** = Xác minh danh tính người dùng.

### 🏠 Ví dụ đời sống: Căn cước công dân

Khi vào sân bay:
1. Bạn xuất trình CCCD → **Authentication**
2. Nhân viên xác nhận: "Đúng là bạn"

```csharp
// User đăng nhập → Hệ thống xác nhận identity
var result = await _signInManager.PasswordSignInAsync(email, password);
if (result.Succeeded)
{
    // Authenticated! Biết user là ai
}
```

---

## 1.2. Authorization (Phân quyền) — "Bạn được làm gì?"

**Authorization** = Kiểm tra quyền hạn.

### 🏠 Ví dụ: Vé máy bay

1. Bạn đã qua cửa check-in (Authenticated)
2. Nhưng vé Economy không vào được phòng chờ Business (Not Authorized)

```csharp
[Authorize(Roles = "Admin")] // Chỉ Admin mới vào được
public IActionResult AdminDashboard()
{
    return View();
}
```

---

# 2. **CÀI ĐẶT ASP.NET CORE IDENTITY**

## 2.1. Install Packages

```powershell
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.AspNetCore.Identity.UI
```

---

## 2.2. Tạo ApplicationUser

```csharp
// Models/ApplicationUser.cs
using Microsoft.AspNetCore.Identity;

public class ApplicationUser : IdentityUser<int> // int = UserId type
{
    public string FullName { get; set; } = string.Empty;
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    
    // Navigation Properties
    public ICollection<Order> Orders { get; set; } = new List<Order>();
}
```

---

## 2.3. Update DbContext

```csharp
// Data/AppDbContext.cs
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;

public class AppDbContext : IdentityDbContext<ApplicationUser, IdentityRole<int>, int>
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }
    
    public DbSet<Product> Products { get; set; } = null!;
    public DbSet<Category> Categories { get; set; } = null!;
    public DbSet<Order> Orders { get; set; } = null!;
    
    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder); // QUAN TRỌNG: Gọi base.OnModelCreating
        
        // Custom configurations...
    }
}
```

---

## 2.4. Configure Identity trong Program.cs

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// DbContext
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Identity
builder.Services.AddIdentity<ApplicationUser, IdentityRole<int>>(options =>
{
    // Password settings
    options.Password.RequireDigit = true;
    options.Password.RequireLowercase = true;
    options.Password.RequireUppercase = true;
    options.Password.RequireNonAlphanumeric = false;
    options.Password.RequiredLength = 8;
    
    // User settings
    options.User.RequireUniqueEmail = true;
    
    // Lockout settings (khóa tài khoản sau n lần sai password)
    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
    options.Lockout.MaxFailedAccessAttempts = 5;
    
    // SignIn settings
    options.SignIn.RequireConfirmedEmail = false; // Set true nếu có email confirmation
})
.AddEntityFrameworkStores<AppDbContext>()
.AddDefaultTokenProviders();

// Cookie settings
builder.Services.ConfigureApplicationCookie(options =>
{
    options.LoginPath = "/Account/Login";
    options.LogoutPath = "/Account/Logout";
    options.AccessDeniedPath = "/Account/AccessDenied";
    options.ExpireTimeSpan = TimeSpan.FromDays(7);
    options.SlidingExpiration = true;
});

builder.Services.AddControllersWithViews();

var app = builder.Build();

// Middleware order QUAN TRỌNG!
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.UseAuthentication(); // TRƯỚC UseAuthorization
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

---

## 2.5. Create Migration

```powershell
dotnet ef migrations add AddIdentity
dotnet ef database update
```

Identity tự động tạo các bảng:
- `AspNetUsers`
- `AspNetRoles`
- `AspNetUserRoles`
- `AspNetUserClaims`
- `AspNetUserLogins`
- `AspNetUserTokens`
- `AspNetRoleClaims`

---

# 3. **REGISTER — ĐĂNG KÝ TÀI KHOẢN**

## 3.1. RegisterViewModel

```csharp
// ViewModels/RegisterViewModel.cs
public class RegisterViewModel
{
    [Required]
    [EmailAddress]
    [Display(Name = "Email")]
    public string Email { get; set; } = string.Empty;
    
    [Required]
    [StringLength(100, MinimumLength = 8)]
    [DataType(DataType.Password)]
    [Display(Name = "Password")]
    public string Password { get; set; } = string.Empty;
    
    [Required]
    [DataType(DataType.Password)]
    [Compare("Password", ErrorMessage = "Passwords do not match")]
    [Display(Name = "Confirm Password")]
    public string ConfirmPassword { get; set; } = string.Empty;
    
    [Required]
    [StringLength(100)]
    [Display(Name = "Full Name")]
    public string FullName { get; set; } = string.Empty;
}
```

---

## 3.2. AccountController — Register

```csharp
// Controllers/AccountController.cs
public class AccountController : Controller
{
    private readonly UserManager<ApplicationUser> _userManager;
    private readonly SignInManager<ApplicationUser> _signInManager;
    
    public AccountController(
        UserManager<ApplicationUser> userManager,
        SignInManager<ApplicationUser> signInManager)
    {
        _userManager = userManager;
        _signInManager = signInManager;
    }
    
    // GET: /Account/Register
    [HttpGet]
    public IActionResult Register()
    {
        return View();
    }
    
    // POST: /Account/Register
    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Register(RegisterViewModel model)
    {
        if (!ModelState.IsValid)
            return View(model);
        
        // Tạo user mới
        var user = new ApplicationUser
        {
            UserName = model.Email,
            Email = model.Email,
            FullName = model.FullName,
            CreatedAt = DateTime.UtcNow
        };
        
        // Tạo user với password (Identity tự hash password)
        var result = await _userManager.CreateAsync(user, model.Password);
        
        if (result.Succeeded)
        {
            // Thêm role "User" (phải tạo role trước)
            await _userManager.AddToRoleAsync(user, "User");
            
            // Tự động đăng nhập
            await _signInManager.SignInAsync(user, isPersistent: false);
            
            TempData["SuccessMessage"] = "Registration successful!";
            return RedirectToAction("Index", "Home");
        }
        
        // Có lỗi → Hiển thị
        foreach (var error in result.Errors)
        {
            ModelState.AddModelError(string.Empty, error.Description);
        }
        
        return View(model);
    }
}
```

---

## 3.3. Register View

```html
@model RegisterViewModel

<h2>Register</h2>

<form method="post" asp-action="Register">
    <div asp-validation-summary="All" class="text-danger"></div>
    
    <div class="form-group">
        <label asp-for="Email"></label>
        <input asp-for="Email" class="form-control" />
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>
    
    <div class="form-group">
        <label asp-for="FullName"></label>
        <input asp-for="FullName" class="form-control" />
        <span asp-validation-for="FullName" class="text-danger"></span>
    </div>
    
    <div class="form-group">
        <label asp-for="Password"></label>
        <input asp-for="Password" class="form-control" />
        <span asp-validation-for="Password" class="text-danger"></span>
    </div>
    
    <div class="form-group">
        <label asp-for="ConfirmPassword"></label>
        <input asp-for="ConfirmPassword" class="form-control" />
        <span asp-validation-for="ConfirmPassword" class="text-danger"></span>
    </div>
    
    <button type="submit" class="btn btn-primary">Register</button>
</form>
```

---

# 4. **LOGIN — ĐĂNG NHẬP**

## 4.1. LoginViewModel

```csharp
public class LoginViewModel
{
    [Required]
    [EmailAddress]
    public string Email { get; set; } = string.Empty;
    
    [Required]
    [DataType(DataType.Password)]
    public string Password { get; set; } = string.Empty;
    
    [Display(Name = "Remember me")]
    public bool RememberMe { get; set; }
}
```

---

## 4.2. Login Action

```csharp
// GET: /Account/Login
[HttpGet]
public IActionResult Login(string? returnUrl = null)
{
    ViewData["ReturnUrl"] = returnUrl;
    return View();
}

// POST: /Account/Login
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Login(LoginViewModel model, string? returnUrl = null)
{
    ViewData["ReturnUrl"] = returnUrl;
    
    if (!ModelState.IsValid)
        return View(model);
    
    // Đăng nhập
    var result = await _signInManager.PasswordSignInAsync(
        model.Email, 
        model.Password, 
        model.RememberMe, 
        lockoutOnFailure: true); // Lockout sau n lần sai
    
    if (result.Succeeded)
    {
        // Redirect về trang trước đó hoặc Home
        if (!string.IsNullOrEmpty(returnUrl) && Url.IsLocalUrl(returnUrl))
            return Redirect(returnUrl);
        
        return RedirectToAction("Index", "Home");
    }
    
    if (result.IsLockedOut)
    {
        ModelState.AddModelError(string.Empty, "Account locked due to multiple failed login attempts");
        return View(model);
    }
    
    ModelState.AddModelError(string.Empty, "Invalid email or password");
    return View(model);
}
```

---

# 5. **LOGOUT — ĐĂNG XUẤT**

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Logout()
{
    await _signInManager.SignOutAsync();
    return RedirectToAction("Index", "Home");
}
```

---

# 6. **PROTECT CONTROLLERS/ACTIONS**

## 6.1. [Authorize] — Yêu cầu đăng nhập

```csharp
[Authorize] // Phải đăng nhập mới vào được
public class OrdersController : Controller
{
    public IActionResult MyOrders()
    {
        // Lấy current user
        var userId = User.FindFirstValue(ClaimTypes.NameIdentifier);
        
        // Query orders...
        return View();
    }
}
```

---

## 6.2. [Authorize(Roles)] — Yêu cầu role cụ thể

```csharp
[Authorize(Roles = "Admin")] // Chỉ Admin
public class AdminController : Controller
{
    public IActionResult Dashboard()
    {
        return View();
    }
}

[Authorize(Roles = "Admin,Manager")] // Admin HOẶC Manager
public IActionResult Reports()
{
    return View();
}
```

---

## 6.3. [AllowAnonymous] — Cho phép anonymous

```csharp
[Authorize] // Controller yêu cầu đăng nhập
public class AccountController : Controller
{
    [AllowAnonymous] // Ngoại lệ: Action này không cần đăng nhập
    public IActionResult Login()
    {
        return View();
    }
    
    [AllowAnonymous]
    public IActionResult Register()
    {
        return View();
    }
    
    // Logout vẫn yêu cầu đăng nhập
    public IActionResult Logout()
    {
        return View();
    }
}
```

---

# 7. **ROLE MANAGEMENT**

## 7.1. Seed Roles

```csharp
// Data/DbSeeder.cs
public static class DbSeeder
{
    public static async Task SeedRolesAndAdminAsync(IServiceProvider serviceProvider)
    {
        var roleManager = serviceProvider.GetRequiredService<RoleManager<IdentityRole<int>>>();
        var userManager = serviceProvider.GetRequiredService<UserManager<ApplicationUser>>();
        
        // Tạo roles
        string[] roleNames = { "Admin", "User", "Manager" };
        foreach (var roleName in roleNames)
        {
            if (!await roleManager.RoleExistsAsync(roleName))
            {
                await roleManager.CreateAsync(new IdentityRole<int>(roleName));
            }
        }
        
        // Tạo Admin user
        var adminEmail = "admin@eshop.com";
        var adminUser = await userManager.FindByEmailAsync(adminEmail);
        
        if (adminUser == null)
        {
            adminUser = new ApplicationUser
            {
                UserName = adminEmail,
                Email = adminEmail,
                FullName = "System Administrator",
                EmailConfirmed = true
            };
            
            await userManager.CreateAsync(adminUser, "Admin@123");
            await userManager.AddToRoleAsync(adminUser, "Admin");
        }
    }
}

// Program.cs
var app = builder.Build();

// Seed data
using (var scope = app.Services.CreateScope())
{
    var services = scope.ServiceProvider;
    await DbSeeder.SeedRolesAndAdminAsync(services);
}

app.Run();
```

---

## 7.2. Assign Role to User

```csharp
// Admin assigns role to user
[Authorize(Roles = "Admin")]
[HttpPost]
public async Task<IActionResult> AssignRole(int userId, string roleName)
{
    var user = await _userManager.FindByIdAsync(userId.ToString());
    if (user == null)
        return NotFound();
    
    if (!await _roleManager.RoleExistsAsync(roleName))
        return BadRequest("Role does not exist");
    
    var result = await _userManager.AddToRoleAsync(user, roleName);
    
    if (result.Succeeded)
        return Ok();
    
    return BadRequest(result.Errors);
}
```

---

# 8. **GET CURRENT USER INFO**

```csharp
public class OrdersController : Controller
{
    private readonly UserManager<ApplicationUser> _userManager;
    
    public async Task<IActionResult> MyOrders()
    {
        // Cách 1: Lấy UserId từ Claims
        var userId = User.FindFirstValue(ClaimTypes.NameIdentifier);
        
        // Cách 2: Lấy full user object
        var user = await _userManager.GetUserAsync(User);
        
        // Cách 3: Kiểm tra role
        var isAdmin = User.IsInRole("Admin");
        
        // Query orders
        var orders = await _db.Orders
            .Where(o => o.UserId == int.Parse(userId))
            .ToListAsync();
        
        return View(orders);
    }
}
```

---

# 9. **BÀI TẬP THỰC HÀNH**

## 📝 Bài 1: Profile Page

Tạo trang Profile cho user xem/edit thông tin cá nhân.

<details>
<summary>💡 Gợi ý</summary>

```csharp
[Authorize]
public class ProfileController : Controller
{
    [HttpGet]
    public async Task<IActionResult> Index()
    {
        var user = await _userManager.GetUserAsync(User);
        return View(user);
    }
    
    [HttpPost]
    public async Task<IActionResult> Update(string fullName)
    {
        var user = await _userManager.GetUserAsync(User);
        user.FullName = fullName;
        await _userManager.UpdateAsync(user);
        return RedirectToAction("Index");
    }
}
```
</details>

---

## ❌ 6. CÁC LỖI THƯỜNG GẶP

### ❌ Lỗi 1: Password không được hash

```csharp
// ❌ Vấn đề: Lưu password plain text
var user = new IdentityUser
{
    UserName = email,
    PasswordHash = password // ❌ Plain text!
};

// ✅ Giải pháp: Dùng PasswordHasher hoặc UserManager
var user = new IdentityUser { UserName = email, Email = email };
var result = await _userManager.CreateAsync(user, password); // ✅ Tự động hash
```

**🔍 Giải thích:** Password phải được hash (bcrypt, PBKDF2) trước khi lưu. UserManager tự động hash.

---

### ❌ Lỗi 2: Quên cấu hình Identity trong Program.cs

```csharp
// ❌ Vấn đề: Tạo UserManager nhưng chưa đăng ký Identity
var userManager = serviceProvider.GetService<UserManager<IdentityUser>>(); // null!

// ✅ Giải pháp: Đăng ký Identity services
builder.Services.AddIdentity<IdentityUser, IdentityRole>(options =>
{
    options.Password.RequireDigit = true;
    options.Password.RequiredLength = 8;
})
.AddEntityFrameworkStores<AppDbContext>();
```

**🔍 Giải thích:** Identity services phải được đăng ký trong DI container trước khi sử dụng.

---

### ❌ Lỗi 3: Không kiểm tra kết quả CreateAsync

```csharp
// ❌ Vấn đề: Không kiểm tra kết quả
await _userManager.CreateAsync(user, password);
// Có thể fail nhưng không biết lý do

// ✅ Giải pháp: Luôn kiểm tra result
var result = await _userManager.CreateAsync(user, password);
if (!result.Succeeded)
{
    foreach (var error in result.Errors)
    {
        ModelState.AddModelError("", error.Description);
    }
    return View(model);
}
```

**🔍 Giải thích:** `CreateAsync` trả về `IdentityResult` chứa thông tin lỗi. Phải kiểm tra để xử lý.

---

## 🎯 7. CASE STUDY / VÍ DỤ THỰC TẾ

### Case Study 1: Authentication System hoàn chỉnh

**Yêu cầu:** Tạo hệ thống đăng ký, đăng nhập, quản lý user với Identity.

```csharp
// AuthController.cs
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly UserManager<IdentityUser> _userManager;
    private readonly SignInManager<IdentityUser> _signInManager;
    private readonly ITokenService _tokenService;
    
    public AuthController(
        UserManager<IdentityUser> userManager,
        SignInManager<IdentityUser> signInManager,
        ITokenService tokenService)
    {
        _userManager = userManager;
        _signInManager = signInManager;
        _tokenService = tokenService;
    }
    
    // POST: api/auth/register
    [HttpPost("register")]
    public async Task<ActionResult<AuthResponse>> Register(RegisterRequest request)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);
        
        var user = new IdentityUser
        {
            UserName = request.Email,
            Email = request.Email
        };
        
        var result = await _userManager.CreateAsync(user, request.Password);
        
        if (!result.Succeeded)
        {
            return BadRequest(new { errors = result.Errors.Select(e => e.Description) });
        }
        
        // Assign default role
        await _userManager.AddToRoleAsync(user, "Customer");
        
        // Generate token
        var token = _tokenService.GenerateToken(user);
        
        return Ok(new AuthResponse
        {
            Token = token,
            Email = user.Email,
            UserId = user.Id
        });
    }
    
    // POST: api/auth/login
    [HttpPost("login")]
    public async Task<ActionResult<AuthResponse>> Login(LoginRequest request)
    {
        var user = await _userManager.FindByEmailAsync(request.Email);
        if (user == null)
        {
            return Unauthorized(new { message = "Invalid credentials" });
        }
        
        var result = await _signInManager.CheckPasswordSignInAsync(
            user, request.Password, lockoutOnFailure: true);
        
        if (!result.Succeeded)
        {
            return Unauthorized(new { message = "Invalid credentials" });
        }
        
        var token = _tokenService.GenerateToken(user);
        
        return Ok(new AuthResponse
        {
            Token = token,
            Email = user.Email,
            UserId = user.Id
        });
    }
}
```

---

## ✅ 8. BEST PRACTICES

### 8.1. Password Best Practices

```csharp
// ✅ Đúng: Cấu hình password policy
builder.Services.AddIdentity<IdentityUser, IdentityRole>(options =>
{
    options.Password.RequireDigit = true;
    options.Password.RequiredLength = 8;
    options.Password.RequireUppercase = true;
    options.Password.RequireLowercase = true;
    options.Password.RequireNonAlphanumeric = true;
    
    options.User.RequireUniqueEmail = true;
    options.SignIn.RequireConfirmedEmail = false; // Set true cho production
});
```

### 8.2. Security Best Practices

```csharp
// ✅ Đúng: Lockout sau nhiều lần sai password
options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
options.Lockout.MaxFailedAccessAttempts = 5;
options.Lockout.AllowedForNewUsers = true;
```

---

# 📝 9. QUICK NOTES

### Identity Components:
- `UserManager<TUser>`: Quản lý users
- `SignInManager<TUser>`: Xử lý sign in/out
- `RoleManager<TRole>`: Quản lý roles

### Authentication Methods:
- Cookie-based: Cho web apps
- JWT: Cho APIs, mobile apps
- OAuth/OpenID: Third-party login

### Best Practices:
- ✅ Hash passwords (UserManager tự động)
- ✅ Password policy mạnh
- ✅ Account lockout
- ✅ Email confirmation (production)
- ✅ Two-factor authentication (2FA)

---

# 🧪 10. MINI TEST

1. **Authentication là gì?**
   - A. Phân quyền
   - B. Xác thực danh tính
   - C. Mã hóa password

2. **[Authorize(Roles = "Admin")] làm gì?**
   - A. Tất cả user đăng nhập đều vào được
   - B. Chỉ Admin mới vào được
   - C. Không ai vào được

3. **UserManager dùng để làm gì?**
   - A. Quản lý database
   - B. Quản lý users
   - C. Quản lý sessions

<details>
<summary>💡 Đáp án</summary>

1. **B** - Xác thực danh tính (Authentication = Who you are)
2. **B** - Chỉ Admin mới vào được
3. **B** - Quản lý users (create, update, delete, roles)
</details>

---

# 📌 TÓM TẮT CHƯƠNG

✅ **Authentication** xác thực danh tính, **Authorization** phân quyền  
✅ **ASP.NET Core Identity** quản lý User, Password (hashed), Roles  
✅ **Register/Login/Logout** với UserManager và SignInManager  
✅ **[Authorize]** bảo vệ Controllers/Actions  
✅ **Roles** phân quyền: Admin, User, Manager...  
✅ **Claims** lưu thông tin user trong cookie  

---

**Chương tiếp theo: [10. JWT Security →](./10_jwt_security.md)**
