# 🛒 E-SHOP ROADMAP - LỘ TRÌNH XÂY DỰNG DỰ ÁN

> **Mục tiêu:** Xây dựng hệ thống E-Shop hoàn chỉnh từ tuần 1 đến tuần 15

---

## 📊 TỔNG QUAN DỰ ÁN E-SHOP

### 🎯 Tính năng chính:

#### 👤 **User Features (Customer)**
- [x] Xem danh sách sản phẩm (phân trang, tìm kiếm)
- [x] Xem chi tiết sản phẩm
- [x] Đăng ký / Đăng nhập
- [x] Thêm vào giỏ hàng
- [x] Đặt hàng
- [x] Xem lịch sử đơn hàng
- [x] Cập nhật profile

#### 👨‍💼 **Admin Features**
- [x] Quản lý sản phẩm (CRUD)
- [x] Quản lý danh mục (CRUD)
- [x] Quản lý đơn hàng
- [x] Quản lý users
- [x] Xem thống kê

#### 🔧 **Technical Features**
- [x] RESTful API (cho Mobile/SPA)
- [x] MVC Web (cho Admin Panel)
- [x] JWT Authentication
- [x] Role-based Authorization
- [x] Entity Framework Core
- [x] SQL Server Database
- [x] Swagger Documentation
- [x] Global Error Handling
- [x] Logging với Serilog
- [x] Caching

---

## 🗓️ LỘ TRÌNH CHI TIẾT THEO TUẦN

### 📦 **TUẦN 1: KHỞI TẠO DỰ ÁN**

**Mục tiêu:** Có dự án E-Shop chạy được với Database và dữ liệu mẫu

#### ✅ Checklist:
- [ ] Tạo project ASP.NET Core Web API
- [ ] Cài đặt Entity Framework Core
- [ ] Thiết kế Database:
  - [ ] Bảng `Products` (Id, Name, Price, Description, Stock, CategoryId, ImageUrl)
  - [ ] Bảng `Categories` (Id, Name, Description)
  - [ ] Bảng `Users` (Id, Email, PasswordHash, FullName, Role)
- [ ] Code First với EF Core
- [ ] Tạo Migration và Update Database
- [ ] Seed data (10 sản phẩm, 3 categories)
- [ ] API đầu tiên: `GET /api/products` trả về JSON
- [ ] Test với Postman/Swagger

#### 📚 Bài học cần học:
- C# Modern cơ bản (Async/Await, LINQ) - **Học qua ví dụ trong dự án**
- Entity Framework Core - Code First
- Database Design

#### 🎯 Deliverable:
```
✅ Project EShopAPI chạy được
✅ Database có 3 bảng với dữ liệu mẫu
✅ API GET /api/products trả về JSON
✅ Swagger UI hiển thị API
```

#### 💡 Tips:
- Dùng `dotnet ef migrations add InitialCreate`
- Seed data trong `OnModelCreating` hoặc `Program.cs`
- Test API ngay với Swagger UI

---

### 🌐 **TUẦN 2: RESTFUL API PRODUCTS**

**Mục tiêu:** Hoàn thiện bộ API Products với CRUD đầy đủ

#### ✅ Checklist:
- [ ] API `GET /api/products` - Danh sách (có pagination)
- [ ] API `GET /api/products/{id}` - Chi tiết
- [ ] API `POST /api/products` - Tạo mới
- [ ] API `PUT /api/products/{id}` - Cập nhật
- [ ] API `DELETE /api/products/{id}` - Xóa
- [ ] DTOs: ProductDto, CreateProductRequest, UpdateProductRequest
- [ ] Validation với Data Annotations
- [ ] Error handling (404, 400, 500)
- [ ] Test tất cả endpoints với Postman

#### 📚 Bài học cần học:
- RESTful principles
- ApiController
- HTTP Methods & Status Codes
- DTOs pattern
- Swagger documentation

#### 🎯 Deliverable:
```
✅ Bộ API Products CRUD hoàn chỉnh
✅ Swagger documentation đầy đủ
✅ Postman collection test được
```

#### 💡 Tips:
- Dùng `[ApiController]` để tự động validate
- Status codes: 200 (OK), 201 (Created), 204 (No Content), 404 (Not Found)
- DTOs để bảo mật (không expose Entity)

---

### 🔧 **TUẦN 3: API NÂNG CAO + DEPENDENCY INJECTION**

**Mục tiêu:** Refactor code với DI và Service Layer

#### ✅ Checklist:
- [ ] Tạo `IProductService` và `ProductService`
- [ ] Refactor `ProductsController` dùng Service
- [ ] Dependency Injection setup
- [ ] Global Exception Handler
- [ ] API Categories (tương tự Products):
  - [ ] GET /api/categories
  - [ ] GET /api/categories/{id}
  - [ ] POST /api/categories
  - [ ] PUT /api/categories/{id}
  - [ ] DELETE /api/categories/{id}
- [ ] Error responses nhất quán

#### 📚 Bài học cần học:
- Dependency Injection
- Service Layer pattern
- Repository pattern (optional)
- Error handling

#### 🎯 Deliverable:
```
✅ Code clean với Service layer
✅ DI container hoạt động
✅ Global exception handler
✅ API Categories hoàn chỉnh
```

---

### 🔐 **TUẦN 4: AUTHENTICATION**

**Mục tiêu:** User có thể đăng ký và đăng nhập

#### ✅ Checklist:
- [ ] Cài đặt ASP.NET Core Identity
- [ ] Cấu hình Identity trong Program.cs
- [ ] Migration cho Identity tables
- [ ] API `POST /api/auth/register`:
  - [ ] Validate input
  - [ ] Hash password
  - [ ] Tạo user
  - [ ] Trả về user info
- [ ] API `POST /api/auth/login`:
  - [ ] Kiểm tra email/password
  - [ ] Trả về user info (chưa có JWT)
- [ ] Test register/login với Postman

#### 📚 Bài học cần học:
- ASP.NET Core Identity
- Password hashing
- UserManager, SignInManager
- Authentication flow

#### 🎯 Deliverable:
```
✅ API Register hoạt động
✅ API Login hoạt động
✅ Users được lưu trong Database
✅ Password được hash an toàn
```

---

### 🎫 **TUẦN 5: JWT SECURITY**

**Mục tiêu:** Bảo vệ API với JWT token

#### ✅ Checklist:
- [ ] Cài đặt JWT packages
- [ ] Tạo `TokenService`:
  - [ ] Generate JWT token
  - [ ] Claims (UserId, Email, Role)
  - [ ] Expiration (1 hour)
- [ ] Cấu hình JWT trong Program.cs
- [ ] API Login trả về JWT token
- [ ] Protect API `POST /api/products` với `[Authorize]`
- [ ] Test với Postman:
  - [ ] Login lấy token
  - [ ] Gọi API với Bearer token
  - [ ] Gọi API không có token (401)

#### 📚 Bài học cần học:
- JWT concept
- Token generation
- JWT middleware
- Authorization header

#### 🎯 Deliverable:
```
✅ Login trả về JWT token
✅ API được bảo vệ với [Authorize]
✅ Test thành công với Postman
```

---

### 🎨 **TUẦN 6: MVC ADMIN PANEL**

**Mục tiêu:** Giao diện web để quản lý sản phẩm

#### ✅ Checklist:
- [ ] Thêm MVC vào project (hoặc tạo project riêng)
- [ ] Tạo `AdminController`:
  - [ ] GET /admin/products - Danh sách
  - [ ] GET /admin/products/create - Form tạo
  - [ ] POST /admin/products/create - Xử lý form
  - [ ] GET /admin/products/{id}/edit - Form sửa
  - [ ] POST /admin/products/{id}/edit - Xử lý form
  - [ ] POST /admin/products/{id}/delete - Xóa
- [ ] Views:
  - [ ] Layout Admin
  - [ ] Products/Index.cshtml
  - [ ] Products/Create.cshtml
  - [ ] Products/Edit.cshtml
- [ ] Bootstrap UI đẹp
- [ ] Gọi API từ Controller (hoặc dùng Service trực tiếp)

#### 📚 Bài học cần học:
- MVC Pattern
- Razor syntax
- Layouts & Tag Helpers
- Model Binding

#### 🎯 Deliverable:
```
✅ Admin Panel: /admin/products
✅ CRUD sản phẩm qua web
✅ UI đẹp với Bootstrap
```

---

### ✅ **TUẦN 7: VALIDATION & FORMS**

**Mục tiêu:** Form validation hoàn chỉnh

#### ✅ Checklist:
- [ ] ViewModels cho forms:
  - [ ] CreateProductViewModel
  - [ ] EditProductViewModel
- [ ] Data Annotations validation
- [ ] Server-side validation
- [ ] Client-side validation (jQuery Unobtrusive)
- [ ] Error messages hiển thị đẹp
- [ ] Success messages (TempData)

#### 📚 Bài học cần học:
- Model Binding
- Data Annotations
- Validation attributes
- Client-side validation

#### 🎯 Deliverable:
```
✅ Form validation hoàn chỉnh
✅ Error messages user-friendly
✅ Client-side validation
```

---

### 🛒 **TUẦN 8: ORDERS & RELATIONSHIPS**

**Mục tiêu:** User có thể đặt hàng

#### ✅ Checklist:
- [ ] Thiết kế bảng Orders, OrderItems
- [ ] Migration
- [ ] Relationships:
  - [ ] Order - User (Many-to-One)
  - [ ] Order - OrderItems (One-to-Many)
  - [ ] OrderItem - Product (Many-to-One)
- [ ] API `POST /api/orders`:
  - [ ] Nhận OrderItems từ request
  - [ ] Tính tổng tiền
  - [ ] Lưu Order và OrderItems
  - [ ] Transaction để đảm bảo data integrity
- [ ] API `GET /api/orders/my` - Lịch sử đơn hàng
- [ ] API `GET /api/orders/{id}` - Chi tiết đơn hàng

#### 📚 Bài học cần học:
- EF Core Relationships
- Eager Loading (Include)
- Transactions
- Complex queries

#### 🎯 Deliverable:
```
✅ API đặt hàng hoạt động
✅ Lịch sử đơn hàng
✅ Relationships đúng
```

---

### 🛍️ **TUẦN 9: SHOPPING CART**

**Mục tiêu:** Giỏ hàng và checkout

#### ✅ Checklist:
- [ ] Thiết kế Cart (Session hoặc Database)
- [ ] API `POST /api/cart/add` - Thêm vào giỏ
- [ ] API `GET /api/cart` - Xem giỏ hàng
- [ ] API `PUT /api/cart/{productId}` - Cập nhật số lượng
- [ ] API `DELETE /api/cart/{productId}` - Xóa khỏi giỏ
- [ ] MVC: Trang giỏ hàng
- [ ] Checkout flow:
  - [ ] Xem giỏ hàng
  - [ ] Xác nhận đơn hàng
  - [ ] Tạo Order từ Cart

#### 📚 Bài học cần học:
- Session/Cookie
- State management
- Cart logic

#### 🎯 Deliverable:
```
✅ Shopping Cart hoạt động
✅ Checkout flow hoàn chỉnh
✅ UI giỏ hàng đẹp
```

---

### 👑 **TUẦN 10: AUTHORIZATION**

**Mục tiêu:** Phân quyền Admin/User

#### ✅ Checklist:
- [ ] Tạo Roles: Admin, Customer
- [ ] Assign role khi register (mặc định Customer)
- [ ] Tạo Admin user (seed data)
- [ ] API Products:
  - [ ] GET: Public (ai cũng xem được)
  - [ ] POST/PUT/DELETE: Chỉ Admin
- [ ] API Orders:
  - [ ] POST: Customer (đặt hàng)
  - [ ] GET /api/orders: Admin (xem tất cả)
  - [ ] GET /api/orders/my: Customer (xem của mình)
- [ ] Admin Panel:
  - [ ] Chỉ Admin truy cập được
  - [ ] [Authorize(Roles = "Admin")]

#### 📚 Bài học cần học:
- Role-based Authorization
- Policy-based Authorization
- [Authorize] attributes

#### 🎯 Deliverable:
```
✅ Phân quyền Admin/User
✅ API được bảo vệ đúng
✅ Admin Panel chỉ Admin vào được
```

---

### ⚡ **TUẦN 11: ADVANCED TOPICS**

**Mục tiêu:** Tối ưu performance và stability

#### ✅ Checklist:
- [ ] Caching:
  - [ ] Cache danh sách sản phẩm (10 phút)
  - [ ] Cache categories
  - [ ] Invalidate cache khi update
- [ ] Logging:
  - [ ] Setup Serilog
  - [ ] Log orders, errors
  - [ ] Log file + Console
- [ ] Global Exception Handler:
  - [ ] Custom error responses
  - [ ] Log errors
- [ ] Health Checks:
  - [ ] /health endpoint
  - [ ] Check database connection

#### 📚 Bài học cần học:
- In-Memory Caching
- Serilog logging
- Global exception handling
- Health checks

#### 🎯 Deliverable:
```
✅ Caching hoạt động
✅ Logging với Serilog
✅ Error handling tốt
✅ Health checks
```

---

### 🔗 **TUẦN 12: TÍCH HỢP & TESTING**

**Mục tiêu:** Tích hợp tất cả tính năng

#### ✅ Checklist:
- [ ] Tích hợp tất cả modules
- [ ] Test end-to-end:
  - [ ] Register → Login → Browse Products → Add to Cart → Checkout
  - [ ] Admin: Login → Manage Products → View Orders
- [ ] Fix bugs
- [ ] Performance testing
- [ ] Security review

#### 📚 Bài học cần học:
- Integration testing
- Performance optimization
- Security best practices

#### 🎯 Deliverable:
```
✅ Tất cả tính năng hoạt động
✅ Không có bug nghiêm trọng
✅ Performance tốt
```

---

### 👨‍🏫 **TUẦN 13-15: WORKSHOP & BẢO VỆ**

#### Tuần 13: Workshop 1 - Code Review
- Review code structure
- Refactoring
- Best practices

#### Tuần 14: Workshop 2 - Bug Fix & Polish
- Fix bugs
- UI/UX improvements
- Documentation

#### Tuần 15: Bảo vệ dự án
- Demo E-Shop
- Presentation
- Q&A

---

## 📋 CHECKLIST TỔNG QUAN

### Database:
- [ ] Products table
- [ ] Categories table
- [ ] Users table (Identity)
- [ ] Orders table
- [ ] OrderItems table
- [ ] Relationships đúng
- [ ] Seed data

### API Endpoints:
- [ ] Products API (CRUD)
- [ ] Categories API (CRUD)
- [ ] Auth API (Register/Login)
- [ ] Orders API
- [ ] Cart API
- [ ] Users API (Profile)

### Security:
- [ ] JWT Authentication
- [ ] Role-based Authorization
- [ ] Password hashing
- [ ] API protection

### UI:
- [ ] Admin Panel
- [ ] Product management
- [ ] Shopping Cart
- [ ] Orders history

### Advanced:
- [ ] Caching
- [ ] Logging
- [ ] Error handling
- [ ] Health checks

---

## 🎯 MILESTONES

| Tuần | Milestone | Status |
|------|-----------|--------|
| 1 | ✅ Project setup + Database | |
| 2-3 | ✅ API Products + Categories | |
| 4-5 | ✅ Authentication + JWT | |
| 6-7 | ✅ Admin Panel | |
| 8-9 | ✅ Orders + Cart | |
| 10-11 | ✅ Authorization + Advanced | |
| 12-15 | ✅ Hoàn thiện + Bảo vệ | |

---

**Lộ trình này giúp sinh viên:**
- ✅ Thấy mục đích học rõ ràng (xây dựng E-Shop)
- ✅ Mỗi tuần có sản phẩm cụ thể
- ✅ Học lý thuyết qua thực hành
- ✅ Tích lũy dần thành dự án hoàn chỉnh

