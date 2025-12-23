# 🧪 LAB 03: BẢO MẬT API VỚI JWT AUTHENTICATION
**(Tương ứng: Module 03 - Week 4-5)**

## 🎯 Mục tiêu
- **Hiểu cốt lõi:** JWT (JSON Web Token) là gì? Khác gì với Session truyền thống?
- **Bảo mật:** Tại sao phải Hash mật khẩu (Password Hashing)?
- **Thực hành:** Tự xây dựng hệ thống Register/Login và bảo vệ API.

---

## 1. 🔍 GIẢI MÃ KHÁI NIỆM & TƯ DUY BẢO MẬT

### 1.1. JWT là gì? (Ví dụ: Hộ chiếu)
Hãy tưởng tượng **JWT giống như cuốn Hộ Chiếu (Passport)**.
*   Khi bạn nhập cảnh (Login thành công), hải quan (Server) cấp cho bạn cuốn Hộ chiếu có đóng dấu mộc đỏ.
*   **Điểm đặc biệt:** Server **KHÔNG lưu giữ** thông tin phiên làm việc của bạn trong bộ nhớ (Stateless).
*   Lần sau bạn đi qua cửa an ninh (Gọi API), bạn chỉ cần giơ Hộ chiếu ra. Server nhìn con dấu mộc (Signature) là biết hộ chiếu thật hay giả, không cần tra cứu lại danh sách công dân.

### 1.2. Mật khẩu & Hashing (Muối và Băm)
**Tuyệt đối KHÔNG bao giờ lưu mật khẩu gốc (Plain text) vào Database.**
*   Nếu lưu "123456", hacker hack được DB -> Mất hết tài khoản.
*   **Giải pháp:** Băm (Hash). "123456" -> `dk23@#$sdfsdf...`
*   Hacker có lấy được chuỗi băm cũng không thể dịch ngược lại thành "123456".

---

## 2. 💻 HƯỚNG DẪN THỰC HÀNH CHI TIẾT

### Bước 1: Cài đặt JWT Package
Thư viện giúp tạo và đọc Token.
```powershell
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

### Bước 2: Cấu hình Secret Key (Chữ ký điện tử)

Mở `appsettings.json`.
Secret Key là "con dấu mộc riêng" của Server. Nếu lộ key này, hacker có thể tự làm giả hộ chiếu Admin.

```json
{
  "Jwt": {
    "Key": "SuperSecretKey@123456_PhaiDuDai_Tren32KyTu!", 
    "Issuer": "https://localhost:7146",
    "Audience": "https://localhost:7146"
  }
}
```

### Bước 3: AuthController (Nơi cấp Hộ chiếu)

Tạo `Controllers/AuthController.cs`.

#### Phần 3.1: Đăng ký (Register) - Hash mật khẩu

```csharp
        // 1. REGISTER
        [HttpPost("register")]
        public async Task<IActionResult> Register(RegisterRequest request)
        {
            // Kiểm tra trùng Email
            if (await _context.Users.AnyAsync(u => u.Email == request.Email))
            {
                return BadRequest("Email này đã được sử dụng.");
            }

            // --- QUAN TRỌNG: HASHING PASSWORD ---
            // Cần cài gói: dotnet add package BCrypt.Net-Next
            // BCrypt sẽ tự động sinh Salt và Hash an toàn
            string passwordHash = BCrypt.Net.BCrypt.HashPassword(request.Password);

            var user = new User
            {
                Email = request.Email,
                PasswordHash = passwordHash, // Chỉ lưu chuỗi đã mã hóa
                FullName = request.FullName,
                Role = "Customer" // Mặc định ai đăng ký cũng là khách hàng
            };

            _context.Users.Add(user);
            await _context.SaveChangesAsync();

            return Ok("Đăng ký thành công!");
        }
```

#### Phần 3.2: Đăng nhập (Login) - Kiểm tra Hash và Cấp Token

```csharp
        // 2. LOGIN
        [HttpPost("login")]
        public async Task<IActionResult> Login(LoginRequest request)
        {
            // Tìm user theo email
            var user = await _context.Users.SingleOrDefaultAsync(u => u.Email == request.Email);
            
            // Nếu không tìm thấy user hoặc mật khẩu không khớp
            // Verify(): So sánh mật khẩu nhập vào (plain) với hash trong DB
            if (user == null || !BCrypt.Net.BCrypt.Verify(request.Password, user.PasswordHash))
            {
                return Unauthorized("Sai email hoặc mật khẩu.");
            }

            // Nếu đúng -> Cấp Token (Hộ chiếu)
            var token = CreateToken(user);

            return Ok(new { token });
        }
```

#### Phần 3.3: Hàm tạo Token (CreateToken) - Đóng dấu

```csharp
        private string CreateToken(User user)
        {
            // 1. Tạo Claims (Thông tin ghi trên hộ chiếu)
            // Đây là những gì Server cần biết về User mà không cần tra DB lại
            var claims = new List<Claim>
            {
                new Claim(ClaimTypes.Email, user.Email),
                new Claim(ClaimTypes.Role, user.Role), // Quan trọng: Ghi rõ Role để phân quyền
                new Claim("UserId", user.Id.ToString()) // Ghi Id để biết ai đang gọi API
            };

            // 2. Lấy Secret Key từ cấu hình để ký tên
            var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(
                _configuration.GetSection("Jwt:Key").Value!));

            // 3. Chọn thuật toán mã hóa (HmacSha512)
            var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha512Signature);

            // 4. Tạo Token object
            var token = new JwtSecurityToken(
                    claims: claims,
                    expires: DateTime.Now.AddDays(1), // Hộ chiếu có hạn 1 ngày
                    signingCredentials: creds
                );

            // 5. Viết ra chuỗi string
            var jwt = new JwtSecurityTokenHandler().WriteToken(token);
            return jwt;
        }
```

### Bước 4: Đăng ký Auth Service (Cổng kiểm soát)

Trong `Program.cs`, chúng ta phải báo cho ứng dụng biết cách kiểm tra Token.

```csharp
// ...
// --- CẤU HÌNH JWT AUTHEN ---
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            // Kiểm tra chữ ký: Có phải do đúng Server mình cấp không?
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(
                builder.Configuration.GetSection("Jwt:Key").Value!)),
            
            // Đơn giản hóa môi trường dev: Không check Issuer/Audience
            ValidateIssuer = false,
            ValidateAudience = false
        };
    });

var app = builder.Build();

// ...

app.UseHttpsRedirection();

// --- THỨ TỰ CỰC KỲ QUAN TRỌNG ---
// 1. Authen: Kiểm tra "Bạn là ai?" (Check Token)
app.UseAuthentication(); 

// 2. Author: Kiểm tra "Bạn được làm gì?" (Check Role)
app.UseAuthorization(); 

app.MapControllers();
// ...
```

### Bước 5: Bảo vệ API (Hàng rào an ninh)

Quay lại `ProductsController.cs`. Muốn bảo vệ action nào, chỉ cần gắn biển cấm vào đó.

```csharp
using Microsoft.AspNetCore.Authorization;

// ...

    // [Authorize]: Phải đăng nhập mới được vào
    // Roles = "Admin": Chỉ Admin mới được vào
    [HttpPost]
    [Authorize(Roles = "Admin")] 
    public async Task<IActionResult> Create(...) 
    {
        // ...
    }
```

---

## 3. 🧪 KIỂM THỬ THỰC TẾ (SWAGGER AUTHORIZE)

Để Swagger có nút nhập Token, cấu hình thêm đoạn code "thần thánh" này vào `Program.cs` (Copy từ hướng dẫn gốc Lab 03).

**Kịch bản Test:**
1.  **Đăng ký:** Tạo user `khachhang@gmail.com`.
2.  **Đăng nhập:** Lấy `token` của khách hàng.
3.  **Thử tạo sản phẩm (POST /api/products):**
    *   Bấm Authorize -> Dán token.
    *   Gọi API -> Mong đợi: **403 Forbidden** (Vì API yêu cầu Role Admin, mà user này là Customer).
4.  **Hack Role:** Vào Database, sửa cột Role của user thành "Admin".
5.  **Đăng nhập lại:** Để lấy token mới (Token cũ vẫn chứa Role Customer).
6.  **Thử lại:** Lần này sẽ thành công (201 Created).

---

## ✅ CHECKLIST TỰ ĐÁNH GIÁ
- [ ] Tôi hiểu tại sao JWT được gọi là "Stateless" (Server không cần nhớ Session).
- [ ] Tôi hiểu tầm quan trọng của việc Hash mật khẩu.
- [ ] Tôi biết cách đọc thông tin User (Id, Email) từ Token trong Controller (`User.FindFirst...`).

---

## 3. 🧪 KIỂM THỬ VỚI SWAGGER (Authorize Button)

Để Swagger hỗ trợ nhập Token, cần cấu hình thêm ở `Program.cs`:

```csharp
builder.Services.AddSwaggerGen(c =>
{
    c.AddSecurityDefinition("Bearer", new Microsoft.OpenApi.Models.OpenApiSecurityScheme
    {
        Description = "Nhập token theo định dạng: Bearer {your token}",
        Name = "Authorization",
        In = Microsoft.ParameterLocation.Header,
        Type = Microsoft.SecuritySchemeType.ApiKey,
        Scheme = "Bearer"
    });

    c.AddSecurityRequirement(new Microsoft.OpenApi.Models.OpenApiSecurityRequirement
    {
        {
            new Microsoft.OpenApi.Models.OpenApiSecurityScheme
            {
                Reference = new Microsoft.OpenApi.Models.OpenApiReference
                {
                    Type = Microsoft.OpenApi.Models.ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            new string[] {}
        }
    });
});
```

**Thực hành Test:**
1.  Gọi `POST /api/auth/register` để tạo User (Password: `123456`).
2.  Sửa DB thủ công (trong SSMS), set cột `Role` của user vừa tạo thành `Admin` (để test quyền admin).
3.  Gọi `POST /api/auth/login`, copy chuỗi `token` trả về.
4.  Trên Swagger, bấm nút **Authorize (hình ổ khóa)**, nhập `Bearer <paste_token_here>`.
5.  Gọi `POST /api/products` -> Sẽ thành công (201).
6.  Logout (hoặc nhập token sai) -> Gọi lại -> Sẽ lỗi 401 Unauthorized.

---

## ✅ CHECKLIST HOÀN THÀNH
- [ ] Cài đặt được gói `Microsoft.AspNetCore.Authentication.JwtBearer`.
- [ ] API Register lưu được password đã hash vào DB.
- [ ] API Login trả về JWT Token hợp lệ.
- [ ] Bảo vệ được API Products (chặn request không có token).
- [ ] Swagger cấu hình được nút Authorize.
