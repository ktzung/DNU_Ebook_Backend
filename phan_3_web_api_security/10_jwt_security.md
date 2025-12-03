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

## 💡 Mẹo nhỏ
> [!WARNING]
> Không bao giờ lưu thông tin nhạy cảm (như password) vào Payload của JWT, vì ai cũng có thể giải mã Payload để xem nội dung (chỉ không sửa được thôi).

> [!TIP]
> Sử dụng trang [jwt.io](https://jwt.io) để dán token vào và xem nội dung bên trong (debug rất hữu ích).
