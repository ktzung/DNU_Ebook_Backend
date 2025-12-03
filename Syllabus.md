# ĐỀ CƯƠNG HỌC PHẦN: THIẾT KẾ, LẬP TRÌNH BACK-END (PHIÊN BẢN TỐI ƯU HÓA)

**Mã học phần:** FIT4016 (Điều chỉnh)  
**Số tín chỉ:** 03  
**Thời lượng:** 15 Tuần (45 tiết)  
**Công nghệ:** C# 10+, .NET 6/8, ASP.NET Core Web API & MVC, SQL Server.

---

## 1. MÔ TẢ HỌC PHẦN (ĐIỀU CHỈNH)
Học phần trang bị kiến thức chuyên sâu về xây dựng hệ thống phần mềm phía máy chủ (Back-end) hiện đại. Sinh viên sẽ học cách phát triển ứng dụng Web theo kiến trúc **RESTful API** và **MVC** sử dụng **ASP.NET Core**. Nội dung bao gồm các kỹ thuật nâng cao: Entity Framework Core (ORM), Dependency Injection, Asynchronous Programming, Bảo mật (Authentication/Authorization) và Triển khai ứng dụng.

## 2. MỤC TIÊU & CHUẨN ĐẦU RA (CẬP NHẬT)
Sau khi học xong, sinh viên có khả năng:
1.  **Vận dụng** thành thạo các tính năng hiện đại của C# (LINQ, Async/Await) để xử lý logic nghiệp vụ.
2.  **Xây dựng** được hệ thống Back-end hoàn chỉnh gồm cả Web API (cho Mobile/SPA) và Web MVC truyền thống.
3.  **Thiết kế** cơ sở dữ liệu và thao tác dữ liệu hiệu quả bằng **Entity Framework Core** (Code First).
4.  **Triển khai** cơ chế bảo mật, đăng nhập, phân quyền (Identity, JWT).
5.  **Phát triển** dự án thực tế "E-Shop" theo mô hình 3 lớp (3-tier) hoặc kiến trúc sạch.

## 3. PHƯƠNG PHÁP ĐÁNH GIÁ (ĐỀ XUẤT MỚI)
Thay thế hình thức "Thuyết trình" bằng "Thi thực hành" để đánh giá chính xác kỹ năng lập trình.

| Bài đánh giá | Trọng số | Hình thức | Nội dung |
| :--- | :--- | :--- | :--- |
| **Chuyên cần** | 10% | Phiếu học tập | Tham gia đầy đủ, hoàn thành bài code tại lớp. |
| **Kiểm tra Giữa kỳ** | 30% | **Thi thực hành (90')** | Xây dựng API CRUD cơ bản + Kết nối CSDL. |
| **Thi Kết thúc** | 60% | **Bảo vệ Đồ án** | Dự án "E-Shop" (API + MVC + Database + Auth). |

---

## 4. KẾ HOẠCH GIẢNG DẠY CHI TIẾT (15 TUẦN)

### GIAI ĐOẠN 1: NỀN TẢNG C# MODERN & KIẾN TRÚC .NET (Tuần 1-2)

#### Tuần 1: C# Nâng cao cho Backend
* **1.1. Hệ sinh thái .NET hiện đại:** .NET 6/8, Kestrel, CLI Tools.
* **1.2. Cú pháp C# Modern:** Nullable Reference Types, Records, Pattern Matching.
* **1.3. Lập trình Bất đồng bộ (Critical):** Cơ chế `Task`, `async`, `await`. Tại sao Backend bắt buộc phải dùng Async?
* **1.4. LINQ Chuyên sâu:** `Select`, `Where`, `Join`, `GroupBy`, `Skip/Take` (Phân trang).
* *Thực hành:* Viết tool xử lý dữ liệu sinh viên dùng LINQ và Async.

#### Tuần 2: Kiến trúc ASP.NET Core
* **2.1. Cấu trúc dự án:** `Program.cs`, Minimal APIs vs Controller-based.
* **2.2. Dependency Injection (DI):** Container là gì? Vòng đời `Transient`, `Scoped`, `Singleton`.
* **2.3. Middleware Pipeline:** Luồng đi của Request -> Response.
* **2.4. Configuration:** Đọc cấu hình từ `appsettings.json`, Biến môi trường.
* *Thực hành:* Cấu hình DI Container và viết Custom Middleware ghi log.

---

### GIAI ĐOẠN 2: MVC CORE & DATABASE (THE CORE) (Tuần 3-7)

#### Tuần 3: MVC Pattern & Routing
* **3.1. Mô hình MVC:** Vai trò của Controller, Model, View.
* **3.2. Routing:** Conventional Routing (`{controller}/{action}`) vs Attribute Routing (`[Route]`).
* **3.3. Action Results:** `IActionResult`, `ViewResult`, `JsonResult`.
* *Thực hành:* Tạo Controller đầu tiên cho dự án E-Shop.

#### Tuần 4: Model Binding & Validation
* **4.1. Cơ chế Binding:** Nhận dữ liệu từ QueryString, RouteData, Form Body.
* **4.2. Data Validation:** Sử dụng Data Annotations (`[Required]`, `[EmailAddress]`). Kiểm tra `ModelState.IsValid`.
* **4.3. ViewModel:** Tại sao không nên dùng Entity trực tiếp? Design Pattern DTO/ViewModel.
* *Thực hành:* Xây dựng Form tạo mới sản phẩm có Validate chặt chẽ.

#### Tuần 5: Entity Framework Core (ORM) - Phần 1
*(Đẩy lên sớm hơn để sinh viên có dữ liệu làm việc)*
* **5.1. Giới thiệu ORM:** EF Core vs ADO.NET.
* **5.2. Code First Workflow:** Định nghĩa Entity Class.
* **5.3. DbContext:** Cấu hình kết nối SQL Server.
* **5.4. Migrations:** Lệnh `add-migration`, `update-database`.
* *Thực hành:* Thiết kế CSDL E-Shop (Bảng Products, Categories) bằng Code First.

#### Tuần 6: Entity Framework Core - Phần 2
* **6.1. CRUD Operations:** Thêm, Sửa, Xóa, Đọc dữ liệu dùng EF Core.
* **6.2. Relationships:** Cấu hình quan hệ 1-N (Category-Product), N-N.
* **6.3. Eager Loading vs Lazy Loading:** Sử dụng `Include()` để lấy dữ liệu liên quan.
* *Thực hành:* Hoàn thiện chức năng Quản lý Sản phẩm (CRUD) lưu xuống SQL Server.

#### Tuần 7: View Engine (Razor) & UI Integration
* **7.1. Razor Syntax:** Cú pháp `@`, Loops, Conditionals.
* **7.2. Tag Helpers:** `asp-controller`, `asp-action`, `asp-for`.
* **7.3. Layouts & Partials:** Tái sử dụng giao diện (Header, Footer).
* *Thực hành:* Ghép giao diện Bootstrap vào dự án E-Shop.

---

### GIAI ĐOẠN 3: WEB API & SECURITY (THE MODERN WEB) (Tuần 8-11)
*(Bổ sung quan trọng để phù hợp nhu cầu tuyển dụng)*

#### Tuần 8: Xây dựng RESTful Web API
* **8.1. Khái niệm REST:** Resource, HTTP Verbs (GET, POST, PUT, DELETE), Status Codes (200, 201, 400, 404, 500).
* **8.2. ApiController:** Khác gì với MVC Controller?
* **8.3. Swagger/OpenAPI:** Cấu hình tài liệu API tự động.
* **8.4. Postman:** Kỹ năng test API.
* *Thực hành:* Viết bộ API `Products` trả về JSON cho Mobile App/Frontend gọi.

#### Tuần 9: Authentication & Authorization (Identity)
* **9.1. Cơ chế xác thực:** Cookie-based vs Token-based.
* **9.2. ASP.NET Core Identity:** Thư viện quản lý User, Role, Password Hashing.
* **9.3. Login/Register Logic:** Tự xây dựng hoặc dùng Scaffolding.
* *Thực hành:* Chức năng Đăng ký, Đăng nhập, Đăng xuất cho E-Shop.

#### Tuần 10: Bảo mật API với JWT (JSON Web Token)
* **10.1. JWT Concept:** Header, Payload, Signature.
* **10.2. Tạo Token:** Code chức năng Login trả về Access Token.
* **10.3. Bảo vệ API:** Sử dụng Middleware xác thực JWT cho các API nhạy cảm.
* *Thực hành:* Bảo mật API `CreateOrder` (chỉ user đã đăng nhập mới gọi được).

#### Tuần 11: Phân quyền (Authorization)
* **11.1. Role-based Authorization:** `[Authorize(Roles = "Admin")]`.
* **11.2. Policy-based Authorization:** Phân quyền theo chính sách linh hoạt.
* *Thực hành:* Tạo trang Admin chỉ cho phép tài khoản Admin truy cập quản lý sản phẩm.

---

### GIAI ĐOẠN 4: NÂNG CAO & HOÀN THIỆN DỰ ÁN (Tuần 12-15)

#### Tuần 12: Các chủ đề Nâng cao & Performance
* **12.1. Caching:** In-Memory Cache để tăng tốc độ tải trang.
* **12.2. Logging:** Ghi log lỗi với Serilog.
* **12.3. Global Exception Handling:** Xử lý lỗi tập trung, trang báo lỗi thân thiện.
* **12.4. Deploy:** Hướng dẫn publish lên IIS hoặc Docker (cơ bản).

#### Tuần 13: Workshop Dự án 1 - Review Code
* **Nội dung:** Giảng viên review cấu trúc dự án của các nhóm.
* **Yêu cầu:** Các nhóm phải hoàn thành khung sườn, Database và các chức năng CRUD chính.
* **Hoạt động:** Code Review, Refactoring (Tối ưu code).

#### Tuần 14: Workshop Dự án 2 - Tích hợp & Fix bug
* **Nội dung:** Hỗ trợ xử lý các lỗi khó (API connection, Auth, Database performance).
* **Yêu cầu:** Hoàn thiện UI/UX, đảm bảo luồng nghiệp vụ "Happy Path" chạy mượt.

#### Tuần 15: Tổng kết & Bảo vệ Thử
* Tổng hệ thống kiến thức toàn khóa.
* Các nhóm demo chạy thử sản phẩm.
* Hướng dẫn viết báo cáo và slide bảo vệ.

---

## 5. TÀI LIỆU THAM KHẢO KHUYẾN NGHỊ
1.  **Sách:** *Pro ASP.NET Core 6/7* - Adam Freeman (Apress).
2.  **Sách:** *C# 10 in a Nutshell* - Joseph Albahari.
3.  **Tài liệu:** Microsoft Learn (.NET Documentation).