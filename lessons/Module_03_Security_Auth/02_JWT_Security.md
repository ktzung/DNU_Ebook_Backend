# 🟨 CHƯƠNG 10
# **JWT SECURITY (JSON WEB TOKEN)**

## 📖 1. Giới thiệu JWT

**JSON Web Token (JWT)** là một chuẩn mở (RFC 7519) định nghĩa cách truyền tin an toàn giữa các bên dưới dạng đối tượng JSON.

### Cấu trúc của JWT
Một token gồm 3 phần, ngăn cách bởi dấu chấm (`.`):
`Header.Payload.Signature`

1. **Header**: Loại token (JWT) và thuật toán mã hóa (HS256).
2. **Payload**: Chứa thông tin (Claims) như ID người dùng, tên, quyền hạn...
3. **Signature**: Chữ ký để xác thực token không bị giả mạo.

---

## 🛠️ 2. Cài đặt thư viện

Để làm việc với JWT trong .NET Core, ta cần package:
`Microsoft.AspNetCore.Authentication.JwtBearer`

```powershell
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

---

## 🔑 3. Tạo Token (Generate Token)

Ta thường tạo một `TokenService` để chuyên trách việc sinh token khi user đăng nhập thành công.

```csharp
public class TokenService
{
    private readonly IConfiguration _config;

    public TokenService(IConfiguration config)
    {
        _config = config;
    }

    public string CreateToken(User user)
    {
        // 1. Tạo Claims (Thông tin muốn lưu trong token)
        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Name, user.UserName),
            new Claim(ClaimTypes.Email, user.Email),
            new Claim(ClaimTypes.Role, user.Role) // Ví dụ: "Admin"
        };

        // 2. Tạo Key bảo mật (Lấy từ appsettings.json)
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_config["Jwt:Key"]));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256Signature);

        // 3. Cấu hình Token
        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(claims),
            Expires = DateTime.Now.AddDays(7), // Hết hạn sau 7 ngày
            SigningCredentials = creds,
            Issuer = _config["Jwt:Issuer"],
            Audience = _config["Jwt:Audience"]
        };

        // 4. Sinh Token
        var tokenHandler = new JwtSecurityTokenHandler();
        var token = tokenHandler.CreateToken(tokenDescriptor);

        return tokenHandler.WriteToken(token);
    }
}
```

---

## ⚙️ 4. Cấu hình Authentication

Trong `Program.cs`, ta cần đăng ký dịch vụ Authentication và cấu hình JWT Bearer.

```csharp
// 1. Đăng ký Authentication
builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"])),
        ValidateIssuer = true,
        ValidIssuer = builder.Configuration["Jwt:Issuer"],
        ValidateAudience = true,
        ValidAudience = builder.Configuration["Jwt:Audience"],
        ValidateLifetime = true // Kiểm tra thời gian hết hạn
    };
});

// ...

var app = builder.Build();

// 2. Kích hoạt Middleware (Thứ tự quan trọng!)
app.UseAuthentication(); // Xác thực: Bạn là ai?
app.UseAuthorization();  // Phân quyền: Bạn được làm gì?

app.MapControllers();
```

---

## 🛡️ 5. Bảo vệ API

Sử dụng attribute `[Authorize]` để yêu cầu client phải gửi token hợp lệ mới được truy cập.

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // Ai cũng xem được
    [HttpGet]
    public IActionResult GetAll() { ... }

    // Chỉ user đã đăng nhập mới được xem chi tiết
    [Authorize]
    [HttpGet("{id}")]
    public IActionResult GetById(int id) { ... }

    // Chỉ Admin mới được tạo sản phẩm
    [Authorize(Roles = "Admin")]
    [HttpPost]
    public IActionResult Create(ProductDto request) { ... }
}
```

### Cách gửi Token từ Client
Client (Postman, React, Mobile) phải gửi token trong **Header** của request:

```
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
```

---

## 🧪 6. Bài tập thực hành

### Bài 1: Chức năng Login
1. Tạo `AuthController` với action `Login`.
2. Kiểm tra username/password (tạm thời fix cứng hoặc lấy từ DB).
3. Nếu đúng, gọi `TokenService` để trả về JWT string.

### Bài 2: Bảo vệ API
1. Thêm `[Authorize]` vào các API sửa/xóa sản phẩm.
2. Dùng Postman:
   - Gọi API khi chưa có token -> Kỳ vọng: 401 Unauthorized.
   - Login để lấy token.
   - Gọi lại API với token -> Kỳ vọng: 200 OK.

---

## ❌ 4. CÁC LỖI THƯỜNG GẶP

### ❌ Lỗi 1: Secret Key quá ngắn hoặc không an toàn

```csharp
// ❌ Vấn đề: Secret key ngắn, dễ bị crack
var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("secret")); // ❌ Quá ngắn!

// ✅ Giải pháp: Secret key dài, random, lưu trong config
var secretKey = _configuration["JwtSettings:SecretKey"]; // Ít nhất 32 ký tự
var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(secretKey));
```

**🔍 Giải thích:** Secret key phải đủ dài (ít nhất 32 bytes) và random để tránh bị brute-force.

---

### ❌ Lỗi 2: Token không expire

```csharp
// ❌ Vấn đề: Token không bao giờ hết hạn
var tokenDescriptor = new SecurityTokenDescriptor
{
    Expires = null, // ❌ Token vĩnh viễn!
};

// ✅ Giải pháp: Set expiration time
var tokenDescriptor = new SecurityTokenDescriptor
{
    Expires = DateTime.UtcNow.AddHours(1), // ✅ Hết hạn sau 1 giờ
};
```

**🔍 Giải thích:** Token không expire là rủi ro bảo mật lớn. Nếu bị lộ, attacker có thể dùng mãi.

---

### ❌ Lỗi 3: Lưu thông tin nhạy cảm trong Payload

```csharp
// ❌ Vấn đề: Lưu password trong token
var claims = new List<Claim>
{
    new Claim("password", user.PasswordHash), // ❌ Nguy hiểm!
};

// ✅ Giải pháp: Chỉ lưu thông tin cần thiết
var claims = new List<Claim>
{
    new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
    new Claim(ClaimTypes.Email, user.Email),
    new Claim(ClaimTypes.Role, user.Role)
    // ✅ Không lưu password, credit card, etc.
};
```

**🔍 Giải thích:** Payload có thể decode (chỉ không sửa được). Không lưu thông tin nhạy cảm.

---

### ❌ Lỗi 4: Không validate token trong middleware

```csharp
// ❌ Vấn đề: Cấu hình JWT sai
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        // Thiếu ValidateIssuer, ValidateAudience
    });

// ✅ Giải pháp: Validate đầy đủ
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = _configuration["JwtSettings:Issuer"],
            ValidAudience = _configuration["JwtSettings:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(_configuration["JwtSettings:SecretKey"]))
        };
    });
```

**🔍 Giải thích:** Phải validate đầy đủ: Issuer, Audience, Lifetime, SigningKey để đảm bảo token hợp lệ.

---

## 🎯 5. CASE STUDY / VÍ DỤ THỰC TẾ

### Case Study 1: JWT Authentication System hoàn chỉnh

**Yêu cầu:** Tạo hệ thống JWT authentication với login, refresh token, logout.

```csharp
// TokenService.cs
public interface ITokenService
{
    string GenerateToken(IdentityUser user, IList<string> roles);
    ClaimsPrincipal? ValidateToken(string token);
    string GenerateRefreshToken();
}

public class TokenService : ITokenService
{
    private readonly IConfiguration _configuration;
    
    public TokenService(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    public string GenerateToken(IdentityUser user, IList<string> roles)
    {
        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id),
            new Claim(ClaimTypes.Name, user.UserName ?? ""),
            new Claim(ClaimTypes.Email, user.Email ?? "")
        };
        
        // Add roles
        foreach (var role in roles)
        {
            claims.Add(new Claim(ClaimTypes.Role, role));
        }
        
        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(_configuration["JwtSettings:SecretKey"]!));
        
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
        
        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(claims),
            Expires = DateTime.UtcNow.AddHours(1),
            Issuer = _configuration["JwtSettings:Issuer"],
            Audience = _configuration["JwtSettings:Audience"],
            SigningCredentials = credentials
        };
        
        var tokenHandler = new JwtSecurityTokenHandler();
        var token = tokenHandler.CreateToken(tokenDescriptor);
        
        return tokenHandler.WriteToken(token);
    }
    
    public ClaimsPrincipal? ValidateToken(string token)
    {
        var tokenHandler = new JwtSecurityTokenHandler();
        var key = Encoding.UTF8.GetBytes(_configuration["JwtSettings:SecretKey"]!);
        
        try
        {
            var principal = tokenHandler.ValidateToken(token, new TokenValidationParameters
            {
                ValidateIssuerSigningKey = true,
                IssuerSigningKey = new SymmetricSecurityKey(key),
                ValidateIssuer = true,
                ValidIssuer = _configuration["JwtSettings:Issuer"],
                ValidateAudience = true,
                ValidAudience = _configuration["JwtSettings:Audience"],
                ValidateLifetime = true,
                ClockSkew = TimeSpan.Zero
            }, out SecurityToken validatedToken);
            
            return principal;
        }
        catch
        {
            return null;
        }
    }
    
    public string GenerateRefreshToken()
    {
        var randomNumber = new byte[64];
        using var rng = RandomNumberGenerator.Create();
        rng.GetBytes(randomNumber);
        return Convert.ToBase64String(randomNumber);
    }
}

// AuthController.cs
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly UserManager<IdentityUser> _userManager;
    private readonly ITokenService _tokenService;
    private readonly AppDbContext _context;
    
    [HttpPost("login")]
    public async Task<ActionResult<AuthResponse>> Login(LoginRequest request)
    {
        var user = await _userManager.FindByEmailAsync(request.Email);
        if (user == null)
            return Unauthorized(new { message = "Invalid credentials" });
        
        var isValid = await _userManager.CheckPasswordAsync(user, request.Password);
        if (!isValid)
            return Unauthorized(new { message = "Invalid credentials" });
        
        var roles = await _userManager.GetRolesAsync(user);
        var token = _tokenService.GenerateToken(user, roles);
        var refreshToken = _tokenService.GenerateRefreshToken();
        
        // Lưu refresh token vào database
        await SaveRefreshTokenAsync(user.Id, refreshToken);
        
        return Ok(new AuthResponse
        {
            Token = token,
            RefreshToken = refreshToken,
            ExpiresAt = DateTime.UtcNow.AddHours(1)
        });
    }
    
    [HttpPost("refresh")]
    public async Task<ActionResult<AuthResponse>> RefreshToken(RefreshTokenRequest request)
    {
        var principal = _tokenService.ValidateToken(request.Token);
        if (principal == null)
            return Unauthorized(new { message = "Invalid token" });
        
        var userId = principal.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        if (userId == null)
            return Unauthorized();
        
        // Kiểm tra refresh token
        var isValid = await ValidateRefreshTokenAsync(userId, request.RefreshToken);
        if (!isValid)
            return Unauthorized(new { message = "Invalid refresh token" });
        
        var user = await _userManager.FindByIdAsync(userId);
        if (user == null)
            return Unauthorized();
        
        var roles = await _userManager.GetRolesAsync(user);
        var newToken = _tokenService.GenerateToken(user, roles);
        var newRefreshToken = _tokenService.GenerateRefreshToken();
        
        await SaveRefreshTokenAsync(userId, newRefreshToken);
        
        return Ok(new AuthResponse
        {
            Token = newToken,
            RefreshToken = newRefreshToken,
            ExpiresAt = DateTime.UtcNow.AddHours(1)
        });
    }
}
```

**Best practices:**
- Refresh token để renew access token
- Lưu refresh token vào database
- Validate token đầy đủ
- Expiration time hợp lý

---

## ✅ 6. BEST PRACTICES

### 6.1. JWT Security Best Practices

```csharp
// ✅ Đúng: Secret key dài, random
var secretKey = _configuration["JwtSettings:SecretKey"]; // Ít nhất 32 bytes

// ✅ Đúng: Set expiration
Expires = DateTime.UtcNow.AddHours(1) // Không quá dài

// ✅ Đúng: Validate đầy đủ
ValidateIssuer = true,
ValidateAudience = true,
ValidateLifetime = true,
ValidateIssuerSigningKey = true

// ✅ Đúng: HTTPS only (production)
options.RequireHttpsMetadata = true;
```

### 6.2. Token Storage Best Practices

```csharp
// ✅ Đúng: Lưu trong httpOnly cookie (web) hoặc secure storage (mobile)
// Web: httpOnly cookie
// Mobile: Secure storage (Keychain, Keystore)

// ❌ Sai: Lưu trong localStorage (dễ bị XSS)
localStorage.setItem('token', token); // ❌
```

---

# 📝 7. QUICK NOTES

### JWT Structure:
- **Header**: Algorithm, token type
- **Payload**: Claims (user info, roles)
- **Signature**: Verify token integrity

### Token Lifecycle:
1. User login → Generate token
2. Client store token
3. Send token in Authorization header
4. Server validate token
5. Token expire → Use refresh token

### Security:
- Secret key: Ít nhất 32 bytes, random
- Expiration: Không quá dài (1-24h)
- HTTPS: Bắt buộc trong production
- Refresh token: Renew access token

### Best Practices:
- ✅ Validate đầy đủ (Issuer, Audience, Lifetime)
- ✅ Không lưu thông tin nhạy cảm trong payload
- ✅ Refresh token mechanism
- ✅ Token revocation (blacklist)

---

# 🧪 8. MINI TEST

1. **JWT gồm mấy phần?**
   - A. 2
   - B. 3 ✅
   - C. 4
   - D. 5

2. **Phần nào của JWT có thể decode để xem?**
   - A. Header
   - B. Payload ✅
   - C. Signature
   - D. Cả 3 phần

3. **Tại sao cần refresh token?**
   - A. Token nhanh hơn
   - B. Renew access token mà không cần login lại ✅
   - C. Token rẻ hơn
   - D. Không cần thiết

<details>
<summary>💡 Đáp án</summary>

1. **B** - JWT gồm 3 phần: Header.Payload.Signature
2. **B** - Payload có thể decode (base64), nhưng không sửa được vì có Signature
3. **B** - Refresh token cho phép renew access token mà không cần user login lại
</details>

---

# 📌 9. TÓM TẮT CHƯƠNG

✅ **JWT** là chuẩn token cho stateless authentication  
✅ **Cấu trúc**: Header.Payload.Signature  
✅ **Generate token** với Claims và SigningCredentials  
✅ **Validate token** trong middleware  
✅ **Refresh token** để renew access token  
✅ **Security**: Secret key dài, expiration, HTTPS  

---

**Chương tiếp theo: [11. MVC Pattern & Routing →](../Module_04_MVC_Frontend/01_MVC_Pattern_Controller.md)**
