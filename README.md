# 🎓 DNU E-BOOK BACKEND
# **Tài liệu học tập: Thiết kế & Lập trình Back-end với ASP.NET Core**

![.NET](https://img.shields.io/badge/.NET-6%2F8-512BD4?logo=dotnet)
![C#](https://img.shields.io/badge/C%23-10%2B-239120?logo=csharp)
![ASP.NET Core](https://img.shields.io/badge/ASP.NET%20Core-Web%20API%20%2B%20MVC-512BD4)
![SQL Server](https://img.shields.io/badge/SQL%20Server-2019%2B-CC2927?logo=microsoftsqlserver)

---

## 📖 Giới thiệu

Đây là **tài liệu học tập đầy đủ** cho môn **Thiết kế, Lập trình Back-end (FIT4016)** tại Đại học Đà Nẵng, được xây dựng theo phong cách "cầm tay chỉ việc" với:

- ✅ Lý thuyết giải thích dễ hiểu
- ✅ Ví dụ đời sống minh họa
- ✅ Code mẫu có chú thích chi tiết
- ✅ Bài tập thực hành sau mỗi chương
- ✅ Dự án thực tế E-Shop xuyên suốt
- ✅ Mini test kiểm tra hiểu bài

---

## 🎯 Mục tiêu học tập

Sau khi hoàn thành khóa học, bạn sẽ:

1. **Vận dụng** thành thạo C# hiện đại (LINQ, Async/Await, Records)
2. **Xây dựng** hệ thống Back-end với ASP.NET Core (Web API + MVC)
3. **Thiết kế** và thao tác Database với Entity Framework Core
4. **Triển khai** bảo mật với JWT, Identity, Authorization
5. **Phát triển** dự án E-Shop hoàn chỉnh sẵn sàng deploy

---

## 📚 Cấu trúc tài liệu

### ⬜ Phần 0: Nhập môn C# (Tuần 0)
```
phan_0_csharp_basic/
├── 01_syntax_variables.md                 ← Biến, Kiểu dữ liệu, Vòng lặp
├── 02_oop_fundamental.md                  ← Class, Object, 4 tính chất OOP
└── 03_collections_generics.md             ← List, Dictionary, LINQ
```

**Nội dung:**
- Cú pháp C# cơ bản
- Tư duy hướng đối tượng (OOP)
- Làm việc với Collections

### 🟦 Phần 1: Nền tảng .NET Core (Tuần 1-2)
```
phan_1_dotnet_core_foundation/
├── 01_csharp_modern.md                    ← Async, LINQ, Records
└── 02_kien_truc_aspnet_core.md            ← DI, Middleware, Config
```

**Nội dung:**
- C# 10+ với các tính năng hiện đại
- Lập trình bất đồng bộ (Async/Await)
- LINQ cho xử lý dữ liệu
- Dependency Injection Container
- Middleware Pipeline

### 🟩 Phần 2: MVC & Database (Tuần 3-7)
```
phan_2_mvc_database/
├── 03_mvc_pattern.md                      ← Controller, Action, Routing
├── 04_model_binding_validation.md         ← Form, Validation
├── 05_entity_framework_core_p1.md         ← Code First, Migrations
├── 06_entity_framework_core_p2.md         ← CRUD, Relationships
└── 07_razor_view_engine.md                ← Razor, Tag Helpers
```

**Dự án tích hợp:**
- Quản lý sản phẩm (Products)
- Quản lý danh mục (Categories)
- Giao diện web với Bootstrap

### 🟨 Phần 3: Web API & Security (Tuần 8-11)
```
phan_3_web_api_security/
├── 08_restful_web_api.md                  ← REST, Swagger, Postman
├── 09_authentication_identity.md          ← Identity, Login/Register
├── 10_jwt_security.md                     ← JSON Web Token
└── 11_authorization.md                    ← Role-based, Policy-based
```

**Dự án tích hợp:**
- RESTful API cho Mobile/SPA
- Hệ thống đăng nhập bảo mật
- Phân quyền Admin/User

### 🟧 Phần 4: Nâng cao & Best Practices (Tuần 12)
```
phan_4_advanced/
└── 12_advanced_topics.md                  ← Caching, Logging, Deploy
```

**Nội dung:**
- In-Memory Caching
- Logging với Serilog
- Global Exception Handling
- Deploy lên IIS/Docker

### 🧪 Mega Projects (Tuần 13-15)
```
mega_projects/
├── 01_eshop_console_api.md                ← RESTful API hoàn chỉnh
├── 02_eshop_mvc_full.md                   ← Website E-Commerce
└── 03_eshop_microservices.md              ← Microservices (Optional)
```

---

## 🛠️ Công cụ cần thiết

| Công cụ | Phiên bản | Mục đích |
|---------|-----------|----------|
| **Visual Studio 2022** | Community+ | IDE chính |
| **.NET SDK** | 6.0 hoặc 8.0 | Framework |
| **SQL Server** | 2019+ Express | Database |
| **SSMS** | Latest | Quản lý DB |
| **Postman** | Latest | Test API |
| **Git** | Latest | Version Control |

### Cài đặt nhanh:

```powershell
# Kiểm tra .NET SDK
dotnet --version

# Tạo project mới
dotnet new webapi -n MyFirstAPI
cd MyFirstAPI
dotnet run
```

---

## 📖 Cách sử dụng tài liệu

### Cho sinh viên:

1. **Bắt đầu từ đầu** — Đọc `00_tai_lieu_tong_quan.md`
2. **Học tuần tự** — Theo thứ tự từ Phần 1 → Phần 4
3. **Gõ code** — Không copy/paste
4. **Chạy ngay** — Test sau mỗi thay đổi
5. **Làm bài tập** — Sau mỗi chương
6. **Làm mini test** — Kiểm tra hiểu bài
7. **Tích hợp dự án** — Xây dựng E-Shop từng tuần

### Cho giảng viên:

- Mỗi chương = 1-2 buổi học (3 tiết)
- Code mẫu sẵn sàng chạy
- Slide trích xuất từ markdown
- Bài tập có đáp án
- Rubric đánh giá dự án

---

## 🗓️ Lộ trình học 15 tuần

| Tuần | Chương | Nội dung chính | Deliverable |
|------|--------|----------------|-------------|
| 1 | 01 | C# Modern | Console App Async/LINQ |
| 2 | 02 | Kiến trúc ASP.NET | API đầu tiên với DI |
| 3 | 03 | MVC Pattern | Controller cơ bản |
| 4 | 04 | Model Binding | Form có validation |
| 5 | 05 | EF Core P1 | Database Code First |
| 6 | 06 | EF Core P2 | CRUD Products |
| 7 | 07 | Razor Views | UI với Bootstrap |
| 8 | 08 | Web API | API Products JSON |
| 9 | 09 | Authentication | Login/Register |
| 10 | 10 | JWT Security | Protected API |
| 11 | 11 | Authorization | Admin Dashboard |
| 12 | 12 | Advanced | Cache, Log, Deploy |
| 13 | - | Workshop 1 | Code Review |
| 14 | - | Workshop 2 | Hoàn thiện E-Shop |
| 15 | - | Final | Bảo vệ dự án |

---

## 🎯 Dự án E-Shop (Xuyên suốt khóa học)

### Tính năng chính:

#### 🛒 User Features:
- [ ] Xem danh sách sản phẩm (phân trang)
- [ ] Tìm kiếm sản phẩm
- [ ] Xem chi tiết sản phẩm
- [ ] Đăng ký / Đăng nhập
- [ ] Thêm vào giỏ hàng
- [ ] Đặt hàng
- [ ] Xem lịch sử đơn hàng

#### 👨‍💼 Admin Features:
- [ ] Quản lý sản phẩm (CRUD)
- [ ] Quản lý danh mục (CRUD)
- [ ] Quản lý đơn hàng
- [ ] Xem thống kê

#### 🔧 Technical Features:
- [ ] RESTful API (cho Mobile)
- [ ] MVC Web (cho Desktop)
- [ ] JWT Authentication
- [ ] Role-based Authorization
- [ ] Entity Framework Core
- [ ] SQL Server Database
- [ ] Swagger Documentation
- [ ] Global Error Handling
- [ ] Logging với Serilog

### Database Schema:

```
Users (Id, Email, PasswordHash, Role, CreatedAt)
Products (Id, Name, Price, Description, CategoryId, Stock)
Categories (Id, Name, Description)
Orders (Id, UserId, TotalAmount, Status, CreatedAt)
OrderItems (Id, OrderId, ProductId, Quantity, Price)
```

---

## 📊 Đánh giá

| Thành phần | Trọng số | Hình thức |
|------------|----------|-----------|
| Chuyên cần | 10% | Điểm danh + Bài tập |
| Giữa kỳ | 30% | Thi thực hành (90 phút) |
| Cuối kỳ | 60% | Bảo vệ dự án E-Shop |

### Yêu cầu bảo vệ cuối kỳ:

1. **Source code** (50%)
   - Cấu trúc dự án rõ ràng
   - Code sạch, có comment
   - Chạy không lỗi
   - Đầy đủ tính năng

2. **Database** (15%)
   - Thiết kế chuẩn hóa
   - Có dữ liệu mẫu
   - Relationships đúng

3. **Demo** (25%)
   - Chức năng hoạt động tốt
   - UI/UX đẹp
   - Xử lý lỗi tốt

4. **Báo cáo** (10%)
   - Tài liệu kỹ thuật
   - Hướng dẫn cài đặt
   - API Documentation

---

## 💡 Mẹo học tập

### ✅ Nên:

- Gõ code thay vì copy/paste
- Chạy code sau mỗi thay đổi
- Đọc error message kỹ
- Làm bài tập ngay
- Hỏi khi chưa hiểu
- Commit code thường xuyên

### ❌ Không nên:

- Bỏ qua lý thuyết
- Copy code không hiểu
- Làm nhiều chương cùng lúc
- Bỏ qua bài tập
- Học đơn độc
- Để cuối kỳ mới code

---

## 🔗 Tài nguyên bổ sung

### Documentation:
- [Microsoft .NET Docs](https://docs.microsoft.com/dotnet)
- [ASP.NET Core Docs](https://docs.microsoft.com/aspnet/core)
- [Entity Framework Core Docs](https://docs.microsoft.com/ef/core)

### Video Tutorials:
- [.NET YouTube Channel](https://www.youtube.com/dotnet)
- [Microsoft Learn](https://learn.microsoft.com)

### Community:
- [Stack Overflow](https://stackoverflow.com/questions/tagged/asp.net-core)
- [Reddit r/dotnet](https://reddit.com/r/dotnet)
- [Discord .NET Community](https://discord.gg/dotnet)

---

## 📝 Changelog

### Version 1.0 (Dec 2024)
- ✅ Phát hành phiên bản đầu tiên
- ✅ 12 chương lý thuyết
- ✅ 3 mega projects
- ✅ Hướng dẫn cài đặt
- ✅ Code examples

---

## 🤝 Đóng góp

Nếu bạn tìm thấy lỗi hoặc muốn đóng góp:

1. Fork repository
2. Tạo branch mới
3. Commit changes
4. Push và tạo Pull Request

---

## 📧 Liên hệ

- **Email**: [Bộ môn CNPM]
- **Discord**: [Link server]
- **GitHub Issues**: [Link issues]

---

## 📜 License

Tài liệu này được phát hành theo giấy phép [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) — Sử dụng cho mục đích giáo dục phi lợi nhuận.

---

**Chúc bạn học tốt và trở thành Backend Developer giỏi! 🚀**

*Bắt đầu học ngay → [00_tai_lieu_tong_quan.md](./00_tai_lieu_tong_quan.md)*
