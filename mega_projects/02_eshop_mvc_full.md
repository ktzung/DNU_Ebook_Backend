# 🛒 MEGA PROJECT 02
# **E-SHOP MVC FULL STACK — WEBSITE THƯƠNG MẠI ĐIỆN TỬ HOÀN CHỈNH**

> **Mục tiêu:** Xây dựng website thương mại điện tử hoàn chỉnh với ASP.NET Core MVC, có giao diện đẹp, đầy đủ tính năng từ Front Store đến Admin Panel.

---

## 📋 MỤC LỤC

1. [Mô tả bài toán](#1-mô-tả-bài-toán)
2. [Phân tích nghiệp vụ](#2-phân-tích-nghiệp-vụ)
3. [Thiết kế hệ thống](#3-thiết-kế-hệ-thống)
4. [Thiết kế Database](#4-thiết-kế-database)
5. [Kiến trúc dự án](#5-kiến-trúc-dự-án)
6. [Triển khai từng bước](#6-triển-khai-từng-bước)
7. [Giao diện người dùng](#7-giao-diện-người-dùng)
8. [Giao diện quản trị](#8-giao-diện-quản-trị)
9. [Testing & Deployment](#9-testing--deployment)
10. [Tài liệu bàn giao](#10-tài-liệu-bàn-giao)

---

# 1. MÔ TẢ BÀI TOÁN

## 1.1. Bối cảnh

**E-Shop** là một hệ thống thương mại điện tử (E-Commerce) cho phép:
- **Khách hàng** mua sắm trực tuyến, xem sản phẩm, thêm vào giỏ hàng, đặt hàng
- **Quản trị viên** quản lý sản phẩm, danh mục, đơn hàng, thống kê

## 1.2. Vấn đề cần giải quyết

### Vấn đề của khách hàng:
- ❌ Không thể xem sản phẩm trực tuyến
- ❌ Không thể mua hàng 24/7
- ❌ Khó tìm kiếm sản phẩm theo nhu cầu
- ❌ Không biết giá, thông tin sản phẩm trước khi mua

### Vấn đề của quản trị viên:
- ❌ Khó quản lý số lượng lớn sản phẩm
- ❌ Không theo dõi được đơn hàng hiệu quả
- ❌ Không có thống kê doanh thu, bán hàng
- ❌ Tốn thời gian xử lý đơn hàng thủ công

## 1.3. Giải pháp

Xây dựng **website E-Shop** với:
- ✅ **Front Store** (Cửa hàng): Giao diện đẹp, dễ sử dụng cho khách hàng
- ✅ **Admin Panel** (Quản trị): Giao diện quản lý đầy đủ cho admin
- ✅ **Hệ thống đăng nhập**: Phân quyền User/Admin
- ✅ **Quản lý sản phẩm**: CRUD đầy đủ, upload ảnh
- ✅ **Giỏ hàng & Đặt hàng**: Luồng mua hàng hoàn chỉnh
- ✅ **Thống kê**: Dashboard với biểu đồ, số liệu

---

# 2. PHÂN TÍCH NGHIỆP VỤ

## 2.1. Phân hệ Khách hàng (Front Store)

### 2.1.1. Trang chủ (Home Page)

**Mô tả:** Trang đầu tiên khi vào website

**Chức năng:**
- Hiển thị banner quảng cáo (slider)
- Hiển thị sản phẩm nổi bật (Featured Products)
- Hiển thị sản phẩm mới nhất (New Products)
- Hiển thị danh mục sản phẩm (Categories)
- Menu điều hướng: Trang chủ, Sản phẩm, Giới thiệu, Liên hệ

**Luồng nghiệp vụ:**
```
1. User truy cập trang chủ
2. Hệ thống load banner, sản phẩm nổi bật, sản phẩm mới
3. User click vào sản phẩm → Chuyển đến trang chi tiết
4. User click vào danh mục → Chuyển đến trang danh sách sản phẩm
```

### 2.1.2. Danh sách sản phẩm (Product List)

**Mô tả:** Hiển thị danh sách sản phẩm với phân trang, lọc, tìm kiếm

**Chức năng:**
- Hiển thị danh sách sản phẩm dạng grid/list
- Phân trang (Pagination): 12 sản phẩm/trang
- Lọc theo danh mục (Category Filter)
- Tìm kiếm theo tên (Search)
- Sắp xếp: Mới nhất, Giá tăng dần, Giá giảm dần, Bán chạy
- Hiển thị: Tên, Giá, Hình ảnh, Nút "Xem chi tiết"

**Luồng nghiệp vụ:**
```
1. User vào trang "Sản phẩm" hoặc click danh mục
2. Hệ thống hiển thị danh sách sản phẩm (trang 1)
3. User chọn lọc theo danh mục → Reload danh sách
4. User nhập từ khóa tìm kiếm → Reload danh sách
5. User chọn sắp xếp → Reload danh sách
6. User click "Xem chi tiết" → Chuyển đến trang chi tiết
```

### 2.1.3. Chi tiết sản phẩm (Product Details)

**Mô tả:** Hiển thị thông tin chi tiết của một sản phẩm

**Chức năng:**
- Hiển thị hình ảnh sản phẩm (có thể nhiều ảnh)
- Hiển thị tên, giá, mô tả
- Hiển thị thông tin: Danh mục, Tồn kho, SKU
- Nút "Thêm vào giỏ hàng" với số lượng
- Hiển thị sản phẩm liên quan (Related Products)

**Luồng nghiệp vụ:**
```
1. User click vào sản phẩm từ danh sách
2. Hệ thống hiển thị chi tiết sản phẩm
3. User chọn số lượng
4. User click "Thêm vào giỏ hàng"
   - Nếu chưa đăng nhập → Yêu cầu đăng nhập
   - Nếu đã đăng nhập → Thêm vào giỏ hàng, hiển thị thông báo
5. User click "Mua ngay" → Chuyển đến trang thanh toán
```

### 2.1.4. Giỏ hàng (Shopping Cart)

**Mô tả:** Quản lý sản phẩm trong giỏ hàng

**Chức năng:**
- Hiển thị danh sách sản phẩm đã thêm
- Mỗi sản phẩm: Hình ảnh, Tên, Giá, Số lượng, Tổng tiền
- Cập nhật số lượng
- Xóa sản phẩm khỏi giỏ hàng
- Tính tổng tiền (Subtotal, Shipping, Total)
- Nút "Tiếp tục mua sắm"
- Nút "Thanh toán" (Checkout)

**Luồng nghiệp vụ:**
```
1. User click icon giỏ hàng
2. Hệ thống hiển thị giỏ hàng
3. User thay đổi số lượng → Cập nhật tổng tiền
4. User xóa sản phẩm → Cập nhật giỏ hàng
5. User click "Thanh toán" → Chuyển đến trang checkout
```

### 2.1.5. Thanh toán (Checkout)

**Mô tả:** Trang nhập thông tin và hoàn tất đơn hàng

**Chức năng:**
- **Thông tin giao hàng:**
  - Họ tên, Email, Số điện thoại
  - Địa chỉ (Tỉnh/Thành phố, Quận/Huyện, Phường/Xã, Địa chỉ chi tiết)
- **Phương thức thanh toán:**
  - Thanh toán khi nhận hàng (COD)
  - Chuyển khoản ngân hàng
  - Ví điện tử (giả lập)
- **Tóm tắt đơn hàng:**
  - Danh sách sản phẩm, Tổng tiền
- **Nút "Đặt hàng"**

**Luồng nghiệp vụ:**
```
1. User click "Thanh toán" từ giỏ hàng
2. Hệ thống hiển thị form checkout
3. User điền thông tin giao hàng
4. User chọn phương thức thanh toán
5. User xem lại tóm tắt đơn hàng
6. User click "Đặt hàng"
   - Validate form
   - Tạo đơn hàng trong database
   - Xóa giỏ hàng
   - Hiển thị thông báo "Đặt hàng thành công"
   - Gửi email xác nhận (nếu có)
7. Chuyển đến trang "Đơn hàng của tôi"
```

### 2.1.6. Đăng ký / Đăng nhập (Register / Login)

**Mô tả:** Quản lý tài khoản khách hàng

**Chức năng đăng ký:**
- Form: Họ tên, Email, Mật khẩu, Xác nhận mật khẩu, Số điện thoại
- Validation: Email hợp lệ, Mật khẩu tối thiểu 6 ký tự, Xác nhận mật khẩu khớp
- Sau khi đăng ký thành công → Tự động đăng nhập

**Chức năng đăng nhập:**
- Form: Email, Mật khẩu
- Checkbox "Ghi nhớ đăng nhập"
- Link "Quên mật khẩu" (tùy chọn)
- Sau khi đăng nhập → Chuyển về trang trước hoặc trang chủ

**Luồng nghiệp vụ:**
```
Đăng ký:
1. User click "Đăng ký"
2. User điền form
3. User click "Đăng ký"
   - Validate form
   - Kiểm tra email đã tồn tại chưa
   - Nếu chưa → Tạo tài khoản mới, đăng nhập tự động
   - Nếu có → Hiển thị lỗi "Email đã được sử dụng"

Đăng nhập:
1. User click "Đăng nhập"
2. User nhập email, mật khẩu
3. User click "Đăng nhập"
   - Validate form
   - Kiểm tra email, mật khẩu
   - Nếu đúng → Đăng nhập, chuyển về trang trước
   - Nếu sai → Hiển thị lỗi "Email hoặc mật khẩu không đúng"
```

### 2.1.7. Đơn hàng của tôi (My Orders)

**Mô tả:** Xem lịch sử đơn hàng của khách hàng

**Chức năng:**
- Hiển thị danh sách đơn hàng (Mã đơn, Ngày đặt, Tổng tiền, Trạng thái)
- Xem chi tiết đơn hàng (Danh sách sản phẩm, Thông tin giao hàng, Trạng thái)
- Hủy đơn hàng (nếu trạng thái = "Chờ xác nhận")

**Luồng nghiệp vụ:**
```
1. User click "Đơn hàng của tôi"
2. Hệ thống hiển thị danh sách đơn hàng
3. User click vào một đơn hàng → Xem chi tiết
4. Nếu đơn hàng = "Chờ xác nhận" → User có thể hủy
```

---

## 2.2. Phân hệ Quản trị (Admin Panel)

### 2.2.1. Dashboard (Bảng điều khiển)

**Mô tả:** Trang tổng quan với thống kê

**Chức năng:**
- **Thống kê tổng quan:**
  - Tổng doanh thu (hôm nay, tháng này, năm nay)
  - Số đơn hàng mới (hôm nay)
  - Số sản phẩm
  - Số khách hàng
- **Biểu đồ:**
  - Doanh thu theo tháng (Line Chart)
  - Top 5 sản phẩm bán chạy (Bar Chart)
- **Danh sách đơn hàng mới nhất** (10 đơn hàng gần nhất)

**Luồng nghiệp vụ:**
```
1. Admin đăng nhập vào Admin Panel
2. Hệ thống hiển thị Dashboard với thống kê
3. Admin có thể xem chi tiết từng phần
```

### 2.2.2. Quản lý Sản phẩm (Product Management)

**Mô tả:** CRUD sản phẩm

**Chức năng:**
- **Danh sách sản phẩm:**
  - Hiển thị: Hình ảnh, Tên, Giá, Danh mục, Tồn kho, Trạng thái
  - Tìm kiếm theo tên
  - Lọc theo danh mục
  - Phân trang
  - Nút "Thêm mới", "Sửa", "Xóa"
- **Thêm/Sửa sản phẩm:**
  - Form: Tên, Mô tả, Giá, Số lượng tồn kho, Danh mục, Hình ảnh
  - Upload nhiều ảnh
  - Preview ảnh trước khi lưu
  - Validation: Tên không rỗng, Giá > 0, Số lượng >= 0

**Luồng nghiệp vụ:**
```
Thêm sản phẩm:
1. Admin click "Thêm mới"
2. Admin điền form
3. Admin upload ảnh
4. Admin click "Lưu"
   - Validate form
   - Lưu ảnh vào wwwroot/images/products
   - Tạo sản phẩm trong database
   - Hiển thị thông báo "Thêm thành công"

Sửa sản phẩm:
1. Admin click "Sửa" trên một sản phẩm
2. Hệ thống load form với dữ liệu hiện tại
3. Admin chỉnh sửa
4. Admin click "Cập nhật"
   - Validate form
   - Cập nhật database
   - Hiển thị thông báo "Cập nhật thành công"

Xóa sản phẩm:
1. Admin click "Xóa"
2. Hệ thống hiển thị xác nhận
3. Admin xác nhận
   - Kiểm tra sản phẩm có trong đơn hàng chưa
   - Nếu có → Không cho xóa, hiển thị cảnh báo
   - Nếu không → Xóa sản phẩm, xóa ảnh
```

### 2.2.3. Quản lý Danh mục (Category Management)

**Mô tả:** CRUD danh mục sản phẩm

**Chức năng:**
- **Danh sách danh mục:**
  - Hiển thị: Tên, Mô tả, Số sản phẩm, Trạng thái
  - Nút "Thêm mới", "Sửa", "Xóa"
- **Thêm/Sửa danh mục:**
  - Form: Tên, Mô tả, Hình ảnh (tùy chọn)
  - Validation: Tên không rỗng

**Luồng nghiệp vụ:**
```
Tương tự như Quản lý Sản phẩm
Lưu ý: Không cho xóa danh mục nếu có sản phẩm thuộc danh mục đó
```

### 2.2.4. Quản lý Đơn hàng (Order Management)

**Mô tả:** Xem và cập nhật trạng thái đơn hàng

**Chức năng:**
- **Danh sách đơn hàng:**
  - Hiển thị: Mã đơn, Khách hàng, Ngày đặt, Tổng tiền, Trạng thái
  - Lọc theo trạng thái
  - Tìm kiếm theo mã đơn, tên khách hàng
  - Phân trang
- **Chi tiết đơn hàng:**
  - Thông tin khách hàng
  - Danh sách sản phẩm
  - Thông tin giao hàng
  - Trạng thái đơn hàng
  - Nút cập nhật trạng thái: "Xác nhận", "Đang giao", "Đã giao", "Hủy"

**Luồng nghiệp vụ:**
```
1. Admin vào "Quản lý đơn hàng"
2. Admin xem danh sách đơn hàng
3. Admin click vào một đơn hàng → Xem chi tiết
4. Admin cập nhật trạng thái:
   - "Chờ xác nhận" → "Đã xác nhận"
   - "Đã xác nhận" → "Đang giao"
   - "Đang giao" → "Đã giao"
   - Có thể hủy đơn hàng (bất kỳ lúc nào trước khi "Đã giao")
5. Hệ thống cập nhật trạng thái, gửi email thông báo (nếu có)
```

### 2.2.5. Quản lý Người dùng (User Management)

**Mô tả:** Xem danh sách người dùng, phân quyền

**Chức năng:**
- **Danh sách người dùng:**
  - Hiển thị: Email, Họ tên, Vai trò, Ngày đăng ký
  - Tìm kiếm theo email, tên
  - Phân trang
- **Phân quyền:**
  - Thay đổi role: User ↔ Admin
  - Khóa/Mở khóa tài khoản

**Luồng nghiệp vụ:**
```
1. Admin vào "Quản lý người dùng"
2. Admin xem danh sách
3. Admin thay đổi role hoặc khóa tài khoản
4. Hệ thống cập nhật
```

---

# 3. PHÂN TÍCH VÀ THIẾT KẾ HỆ THỐNG (OOAD)

> **Quy trình OOAD:** Requirements Analysis → Domain Analysis → System Analysis → System Design → Object Design → Database Design

---

## 3.1. PHASE 1: REQUIREMENTS ANALYSIS (Phân tích yêu cầu)

### 3.1.1. Functional Requirements (Yêu cầu chức năng)

**Đã được mô tả chi tiết ở Phần 2: Phân tích nghiệp vụ**

**Tóm tắt:**
- **Customer Features:** 7 chức năng chính
- **Admin Features:** 5 chức năng chính
- **System Features:** Authentication, Authorization, Session Management

### 3.1.2. Non-Functional Requirements (Yêu cầu phi chức năng)

### 3.1.1. Performance Requirements

**Yêu cầu hiệu năng:**
- **Response Time:**
  - Trang chủ: < 2 giây
  - Danh sách sản phẩm: < 3 giây
  - Chi tiết sản phẩm: < 1 giây
  - Tìm kiếm: < 2 giây
- **Throughput:**
  - Hỗ trợ đồng thời 100 users
  - 1000 requests/phút
- **Scalability:**
  - Có thể mở rộng theo chiều ngang (Horizontal Scaling)
  - Database có thể tách riêng

#### 3.1.2.2. Security Requirements

**Yêu cầu bảo mật:**
- **Authentication:** ASP.NET Core Identity với password hashing (BCrypt)
- **Authorization:** Role-based (User, Admin)
- **Data Protection:**
  - HTTPS bắt buộc
  - SQL Injection prevention (EF Core parameterized queries)
  - XSS prevention (Razor encoding)
  - CSRF protection (Anti-forgery tokens)
- **Session Management:**
  - Session timeout: 30 phút
  - Secure cookies (HttpOnly, Secure flags)

#### 3.1.2.3. Reliability Requirements

**Yêu cầu độ tin cậy:**
- **Uptime:** 99.5% (tương đương downtime < 3.6 giờ/tháng)
- **Error Handling:**
  - Global exception handler
  - Logging tất cả errors
  - User-friendly error messages
- **Data Integrity:**
  - Transaction support cho đơn hàng
  - Foreign key constraints
  - Soft delete cho sản phẩm

#### 3.1.2.4. Usability Requirements

**Yêu cầu khả năng sử dụng:**
- **Responsive Design:** Hỗ trợ Desktop, Tablet, Mobile
- **Browser Support:** Chrome, Firefox, Edge, Safari (phiên bản mới nhất)
- **Accessibility:** Tuân thủ WCAG 2.1 Level AA
- **Internationalization:** Hỗ trợ tiếng Việt (có thể mở rộng)

---

---

## 3.2. PHASE 2: DOMAIN ANALYSIS (Phân tích miền)

### 3.2.1. Domain Model Overview

**Domain Model** mô tả các khái niệm nghiệp vụ chính trong hệ thống E-Shop:

```
┌─────────────────────────────────────────────────────────────┐
│                    DOMAIN MODEL                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Core Entities (Thực thể cốt lõi):                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  Product     │  │   Order      │  │   Category   │     │
│  │  - Business  │  │  - Business │  │  - Business │     │
│  │    Rules     │  │    Rules     │  │    Rules     │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                              │
│  Supporting Entities (Thực thể hỗ trợ):                    │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │  OrderItem   │  │   AppUser    │                        │
│  │  - Value     │  │  - Identity  │                        │
│  │    Object    │  │    Entity    │                        │
│  └──────────────┘  └──────────────┘                        │
│                                                              │
│  Value Objects (Đối tượng giá trị):                        │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │  Money       │  │  Address     │                        │
│  │  (Price)     │  │  (Shipping)  │                        │
│  └──────────────┘  └──────────────┘                        │
│                                                              │
│  Enumerations (Liệt kê):                                    │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │ OrderStatus  │  │ PaymentMethod│                        │
│  └──────────────┘  └──────────────┘                        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 3.2.2. Domain Rules (Quy tắc nghiệp vụ)

#### Product Domain Rules:

1. **Product Creation Rules:**
   - Tên sản phẩm không được rỗng
   - Giá phải > 0
   - Số lượng tồn kho >= 0
   - SKU phải unique (nếu có)
   - Phải thuộc một Category hợp lệ

2. **Product Update Rules:**
   - Không thể cập nhật giá nếu sản phẩm đã có trong đơn hàng đã giao
   - Không thể giảm số lượng tồn kho xuống < 0
   - Khi cập nhật, phải set `UpdatedAt`

3. **Product Deletion Rules:**
   - Không thể xóa cứng (Hard Delete) nếu sản phẩm đã có trong đơn hàng
   - Chỉ được phép Soft Delete (set `IsActive = false`)

#### Order Domain Rules:

1. **Order Creation Rules:**
   - Đơn hàng phải có ít nhất 1 OrderItem
   - Tổng tiền = SubTotal + ShippingFee
   - OrderNumber phải unique và tự động sinh
   - User phải đã đăng nhập

2. **Order Status Transition Rules:**
   ```
   Pending → Confirmed → Shipping → Delivered
        ↓
    Cancelled (có thể hủy từ bất kỳ trạng thái nào trước Delivered)
   ```

3. **Order Cancellation Rules:**
   - Chỉ có thể hủy nếu Status != Delivered
   - Khi hủy, phải hoàn lại số lượng tồn kho cho các sản phẩm

#### OrderItem Domain Rules:

1. **OrderItem Creation Rules:**
   - Quantity phải > 0
   - Price phải lưu lại giá tại thời điểm mua (không thay đổi)
   - Product phải còn tồn kho đủ

2. **Stock Management Rules:**
   - Khi tạo OrderItem → Trừ Stock của Product
   - Khi hủy Order → Cộng lại Stock

### 3.2.3. Domain Services (Dịch vụ miền)

**Domain Services** là các service xử lý logic nghiệp vụ phức tạp không thuộc về một Entity cụ thể:

1. **OrderNumberGeneratorService:**
   - Tạo số đơn hàng unique: `ORD-YYYYMMDD-XXXX`
   - Đảm bảo không trùng lặp

2. **StockManagementService:**
   - Kiểm tra tồn kho trước khi đặt hàng
   - Trừ/cộng tồn kho khi tạo/hủy đơn hàng
   - Đảm bảo tính nhất quán (transaction)

3. **PriceCalculationService:**
   - Tính SubTotal từ OrderItems
   - Tính ShippingFee dựa trên địa chỉ
   - Tính Total = SubTotal + ShippingFee

---

## 3.3. PHASE 3: SYSTEM ANALYSIS (Phân tích hệ thống)

### 3.3.1. Use Case Analysis

**Đã được mô tả ở phần 3.7 (Use Case Diagrams)**

### 3.3.2. Interaction Analysis

**Đã được mô tả ở phần 3.8 (Sequence Diagrams)**

### 3.3.3. Process Analysis

**Đã được mô tả ở phần 3.9 (Activity Diagrams)**

### 3.3.4. State Analysis (Phân tích trạng thái)

#### State Diagram: Order Lifecycle

```
                    ┌─────────────┐
                    │   [Start]   │
                    └──────┬───────┘
                           │
                           ▼
                    ┌─────────────┐
                    │   Pending   │◄──────────┐
                    │ (Chờ xác nhận)          │
                    └──────┬──────┘          │
                           │                 │
                    ┌──────┴──────┐          │
                    │              │          │
                    ▼              ▼          │
            ┌─────────────┐  ┌─────────────┐  │
            │ Confirmed    │  │ Cancelled  │  │
            │ (Đã xác nhận)│  │ (Đã hủy)   │  │
            └──────┬───────┘  └────────────┘  │
                   │                          │
                   ▼                          │
            ┌─────────────┐                   │
            │  Shipping    │                   │
            │ (Đang giao) │                   │
            └──────┬──────┘                   │
                   │                          │
                   ▼                          │
            ┌─────────────┐                   │
            │  Delivered  │                   │
            │ (Đã giao)   │                   │
            └─────────────┘                   │
                   │                          │
                   ▼                          │
            ┌─────────────┐                   │
            │    [End]    │                   │
            └─────────────┘                   │
                                              │
                    ┌─────────────────────────┘
                    │
                    │ Có thể hủy từ:
                    │ - Pending
                    │ - Confirmed  
                    │ - Shipping
                    │ (KHÔNG thể hủy từ Delivered)
```

**State Transition Table:**

| From State | To State | Trigger | Guard Condition | Action |
|------------|----------|---------|-----------------|--------|
| Pending | Confirmed | Admin confirms | Admin has permission | Update status, Send email |
| Pending | Cancelled | User/Admin cancels | Status != Delivered | Restore stock, Update status |
| Confirmed | Shipping | Admin ships | Order confirmed | Update status, Send tracking |
| Confirmed | Cancelled | User/Admin cancels | Status != Delivered | Restore stock, Update status |
| Shipping | Delivered | Delivery completed | Order shipped | Update status, Send email |
| Shipping | Cancelled | Admin cancels | Status != Delivered | Restore stock, Update status |
| Delivered | - | - | - | Final state, no transitions |

---

## 3.4. PHASE 4: SYSTEM DESIGN (Thiết kế hệ thống)

### 3.4.1. Architecture Design

**Đã được mô tả ở phần 3.6 (Kiến trúc tổng quan)**

### 3.4.2. Component Design

**Đã được mô tả ở phần 3.11 (Component Diagram)**

### 3.4.3. Deployment Design

**Đã được mô tả ở phần 3.12 (Deployment Diagram)**

### 3.4.4. Package Diagram (Sơ đồ gói)

```
┌─────────────────────────────────────────────────────────────┐
│                    EShop Solution                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  EShop.Core (Domain Package)                          │  │
│  │  ┌──────────────┐  ┌──────────────┐                  │  │
│  │  │ Entities     │  │ Interfaces   │                  │  │
│  │  │ - Product    │  │ - IProductRepo│                 │  │
│  │  │ - Order      │  │ - IOrderRepo │                  │  │
│  │  │ - Category   │  │ - IUnitOfWork│                 │  │
│  │  └──────────────┘  └──────────────┘                  │  │
│  │  ┌──────────────┐                                     │  │
│  │  │ Enums        │                                     │  │
│  │  │ - OrderStatus│                                     │  │
│  │  │ - PaymentMethod│                                  │  │
│  │  └──────────────┘                                     │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  EShop.Infrastructure (Data Access Package)          │  │
│  │  ┌──────────────┐  ┌──────────────┐                  │  │
│  │  │ Data         │  │ Repositories │                  │  │
│  │  │ - DbContext  │  │ - ProductRepo│                  │  │
│  │  │              │  │ - OrderRepo  │                  │  │
│  │  │              │  │ - UnitOfWork │                  │  │
│  │  └──────────────┘  └──────────────┘                  │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  EShop.Application (Business Logic Package)            │  │
│  │  ┌──────────────┐  ┌──────────────┐                  │  │
│  │  │ Services     │  │ DTOs         │                  │  │
│  │  │ - ProductSvc │  │ - ProductDto │                  │  │
│  │  │ - OrderSvc   │  │ - OrderDto   │                  │  │
│  │  └──────────────┘  └──────────────┘                  │  │
│  │  ┌──────────────┐                                     │  │
│  │  │ Mappings     │                                     │  │
│  │  │ - AutoMapper │                                     │  │
│  │  └──────────────┘                                     │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  EShop.Web (Presentation Package)                     │  │
│  │  ┌──────────────┐  ┌──────────────┐                  │  │
│  │  │ Areas/Store  │  │ Areas/Admin  │                  │  │
│  │  │ - Controllers│  │ - Controllers│                  │  │
│  │  │ - Views      │  │ - Views      │                  │  │
│  │  └──────────────┘  └──────────────┘                  │  │
│  │  ┌──────────────┐                                     │  │
│  │  │ ViewModels   │                                     │  │
│  │  │ - ProductVM  │                                     │  │
│  │  │ - CartVM     │                                     │  │
│  │  └──────────────┘                                     │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Package Dependencies:**

```
EShop.Web
    ↓ depends on
EShop.Application
    ↓ depends on
EShop.Infrastructure
    ↓ depends on
EShop.Core
```

**Nguyên tắc:**
- ✅ Dependency chỉ đi một chiều (từ trên xuống)
- ✅ Core không phụ thuộc vào bất kỳ package nào
- ✅ Infrastructure phụ thuộc Core
- ✅ Application phụ thuộc Core và Infrastructure
- ✅ Web phụ thuộc Application

---

## 3.5. PHASE 5: OBJECT DESIGN (Thiết kế đối tượng)

### 3.5.1. Detailed Class Design

**Đã được mô tả ở phần 3.10 (Class Diagrams)**

### 3.5.2. Design Patterns

**Đã được mô tả ở phần 3.13 (Design Patterns)**

### 3.5.3. Interface Design

#### 3.5.3.1. Repository Interfaces

```csharp
// EShop.Core/Interfaces/IProductRepository.cs
namespace EShop.Core.Interfaces;

/// <summary>
/// Repository interface cho Product entity
/// Tuân thủ Repository Pattern
/// </summary>
public interface IProductRepository
{
    // Query Methods
    Task<IEnumerable<Product>> GetAllAsync();
    Task<Product?> GetByIdAsync(int id);
    Task<IEnumerable<Product>> GetByCategoryIdAsync(int categoryId);
    Task<IEnumerable<Product>> SearchAsync(string keyword);
    Task<bool> ExistsAsync(int id);
    
    // Command Methods
    Task<Product> AddAsync(Product product);
    Task UpdateAsync(Product product);
    Task DeleteAsync(int id);
}
```

**Design Principles:**
- ✅ Interface Segregation: Chia nhỏ interfaces theo responsibility
- ✅ Dependency Inversion: Depend on abstractions, not concretions
- ✅ Single Responsibility: Mỗi repository chỉ quản lý một entity

#### 3.5.3.2. Service Interfaces

```csharp
// EShop.Application/Services/IProductService.cs
namespace EShop.Application.Services;

/// <summary>
/// Service interface cho Product business logic
/// Tuân thủ Service Layer Pattern
/// </summary>
public interface IProductService
{
    // Query Operations
    Task<IEnumerable<ProductDto>> GetAllAsync();
    Task<ProductDto?> GetByIdAsync(int id);
    Task<IEnumerable<ProductDto>> GetByCategoryIdAsync(int categoryId);
    Task<IEnumerable<ProductDto>> SearchAsync(string keyword);
    
    // Command Operations
    Task<ProductDto> CreateAsync(CreateProductDto dto);
    Task<ProductDto> UpdateAsync(int id, UpdateProductDto dto);
    Task DeleteAsync(int id);
}
```

**Design Principles:**
- ✅ Separation of Concerns: Service xử lý business logic, Repository xử lý data access
- ✅ DTO Pattern: Không expose domain entities ra ngoài
- ✅ Async/Await: Tất cả operations đều async

---

## 3.6. Kiến trúc tổng quan (Architecture Overview)

### 3.2.1. Kiến trúc phân lớp (N-Layer Architecture)

```
┌─────────────────────────────────────────────────────────┐
│              PRESENTATION LAYER                          │
│                    EShop.Web (MVC)                       │
│  ┌──────────────────┐  ┌──────────────────┐             │
│  │  Front Store     │  │  Admin Panel     │             │
│  │  (Areas/Store)   │  │  (Areas/Admin)   │             │
│  │  - Controllers   │  │  - Controllers   │             │
│  │  - Views         │  │  - Views         │             │
│  │  - ViewModels    │  │  - ViewModels    │             │
│  └──────────────────┘  └──────────────────┘             │
└──────────────────────┬──────────────────────────────────┘
                       │ HTTP Requests/Responses
                       │ DTOs
┌──────────────────────▼──────────────────────────────────┐
│            APPLICATION LAYER                             │
│              EShop.Application                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │ Product      │  │ Order        │  │ User         │   │
│  │ Service      │  │ Service      │  │ Service      │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
│  ┌──────────────┐  ┌──────────────┐                     │
│  │ DTOs         │  │ AutoMapper  │                     │
│  │ Validators   │  │ Profiles    │                     │
│  └──────────────┘  └──────────────┘                     │
└──────────────────────┬──────────────────────────────────┘
                       │ Domain Models
                       │ Repository Interfaces
┌──────────────────────▼──────────────────────────────────┐
│          INFRASTRUCTURE LAYER                             │
│         EShop.Infrastructure                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │ Product      │  │ Order        │  │ Category     │   │
│  │ Repository   │  │ Repository   │  │ Repository   │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
│  ┌──────────────┐  ┌──────────────┐                     │
│  │ DbContext    │  │ UnitOfWork  │                     │
│  │ Migrations   │  │             │                     │
│  └──────────────┘  └──────────────┘                     │
└──────────────────────┬──────────────────────────────────┘
                       │ Entity Framework Core
                       │ SQL Queries
┌──────────────────────▼──────────────────────────────────┐
│              DOMAIN LAYER                                 │
│                EShop.Core                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │ Entities     │  │ Interfaces   │  │ Enums        │   │
│  │ - Product    │  │ - IRepo      │  │ - OrderStatus│   │
│  │ - Order      │  │ - IService   │  │ - Payment    │   │
│  │ - Category   │  │             │  │              │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│                  SQL Server Database                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │ Products     │  │ Orders       │  │ Categories   │   │
│  │ OrderItems   │  │ AspNetUsers  │  │              │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
└─────────────────────────────────────────────────────────┘
```

**Giải thích kiến trúc:**
- **Presentation Layer:** Xử lý giao diện, nhận input từ user, hiển thị output
- **Application Layer:** Chứa business logic, validation, mapping
- **Infrastructure Layer:** Xử lý data access, external services
- **Domain Layer:** Chứa domain models, không phụ thuộc vào bất kỳ layer nào

**Lợi ích:**
- ✅ Separation of Concerns: Mỗi layer có trách nhiệm riêng
- ✅ Testability: Dễ test từng layer độc lập
- ✅ Maintainability: Dễ bảo trì, mở rộng
- ✅ Reusability: Có thể tái sử dụng services, repositories

---

---

## 3.7. Use Case Diagram (System Analysis)

### 3.3.1. Use Case cho Khách hàng (Customer)

```
                    ┌─────────────────┐
                    │   Customer      │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Xem sản phẩm │    │ Tìm kiếm     │    │ Xem chi tiết │
│              │    │ sản phẩm     │    │ sản phẩm     │
└──────────────┘    └──────────────┘    └──────────────┘
        │                    │                    │
        └────────────────────┼────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Thêm vào     │    │ Xem giỏ hàng │    │ Cập nhật     │
│ giỏ hàng     │    │              │    │ giỏ hàng     │
└──────────────┘    └──────────────┘    └──────────────┘
        │                    │                    │
        └────────────────────┼────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Đăng ký      │    │ Đăng nhập    │    │ Đặt hàng     │
│              │    │              │    │              │
└──────────────┘    └──────────────┘    └──────────────┘
                             │
                             ▼
                    ┌──────────────┐
                    │ Xem đơn hàng │
                    │ của tôi      │
                    └──────────────┘
```

### 3.3.2. Use Case cho Quản trị viên (Admin)

```
                    ┌─────────────────┐
                    │   Admin         │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Xem Dashboard│    │ Quản lý      │    │ Quản lý      │
│              │    │ Sản phẩm     │    │ Danh mục     │
└──────────────┘    └──────────────┘    └──────────────┘
        │                    │                    │
        └────────────────────┼────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Quản lý      │    │ Quản lý      │    │ Xem thống kê │
│ Đơn hàng     │    │ Người dùng   │    │              │
└──────────────┘    └──────────────┘    └──────────────┘
```

---

## 3.8. Sequence Diagrams (Interaction Analysis)

### 3.4.1. Sequence Diagram: Đặt hàng (Place Order)

```
Customer    Browser    ProductController    CartController    OrderService    OrderRepository    Database
   │           │              │                   │                │                 │              │
   │           │              │                   │                │                 │              │
   │──Click "Thanh toán"─────────────────────────>│                │                 │              │
   │           │              │                   │                │                 │              │
   │           │              │<──Get Cart from Session────────────│                 │              │
   │           │              │                   │                │                 │              │
   │           │              │──Display Checkout Form─────────────>│                 │              │
   │           │<──Render Checkout Page───────────│                │                 │              │
   │<──Checkout Page──────────│                   │                │                 │              │
   │           │              │                   │                │                 │              │
   │──Fill Form & Submit─────────────────────────>│                │                 │              │
   │           │              │                   │                │                 │              │
   │           │              │──Validate Form───>│                │                 │              │
   │           │              │                   │                │                 │              │
   │           │              │──Create Order───>│                 │              │              │
   │           │              │                   │──Generate Order Number───────────>│              │
   │           │              │                   │                │<──Query Last Order──────────>│
   │           │              │                   │<──Order Number────────────────────│              │
   │           │              │                   │──Create Order Items───────────────>│              │
   │           │              │                   │                │──Insert Order───────────────>│
   │           │              │                   │                │                 │──Save──────>│
   │           │              │                   │                │<──Order Saved─────────────────│
   │           │              │                   │<──Order Created────────────────────│              │
   │           │              │──Clear Cart───────>│                │                 │              │
   │           │              │                   │                │                 │              │
   │           │<──Order Success Page─────────────│                │                 │              │
   │<──Success Message────────│                   │                │                 │              │
```

### 3.4.2. Sequence Diagram: Thêm sản phẩm vào giỏ hàng

```
Customer    Browser    ProductController    ProductService    ProductRepository    CartController    Session
   │           │              │                   │                  │                   │              │
   │──Click "Thêm vào giỏ"──>│                   │                  │                   │              │
   │           │              │                   │                  │                   │              │
   │           │              │──Get Product by ID──────────────────>│                   │              │
   │           │              │                   │──Query Database───────────────────>│              │
   │           │              │                   │<──Product Data─────────────────────│              │
   │           │              │<──Product DTO───────────────────────│                   │              │
   │           │              │                   │                  │                   │              │
   │           │              │──Add to Cart──────>│                  │                   │              │
   │           │              │                   │                  │──Get Cart from Session─────────>│
   │           │              │                   │                  │<──Cart Data─────────────────────│
   │           │              │                   │                  │──Update Cart────────────────────>│
   │           │              │                   │                  │──Save Cart to Session──────────>│
   │           │              │                   │                  │                   │              │
   │           │<──Success Message────────────────│                  │                   │              │
   │<──"Đã thêm vào giỏ hàng"│                   │                  │                   │              │
```

---

## 3.9. Activity Diagrams (Process Analysis)

### 3.5.1. Activity Diagram: Quy trình đặt hàng

```
                    [Bắt đầu]
                       │
                       ▼
              ┌─────────────────┐
              │ Xem giỏ hàng    │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ Giỏ hàng trống? │
              └────────┬────────┘
                  No   │   Yes
                  │    │    │
                  │    └────┼────> [Kết thúc]
                  │         │
                  ▼         │
         ┌─────────────────┐│
         │ Click "Thanh toán"│
         └────────┬────────┘│
                  │         │
                  ▼         │
         ┌─────────────────┐│
         │ Đã đăng nhập?    ││
         └────────┬────────┘│
             No   │   Yes   │
             │    │    │    │
             │    └────┼────┼──> [Yêu cầu đăng nhập]
             │         │    │
             ▼         │    │
    ┌─────────────────┐│    │
    │ Đăng nhập       ││    │
    └────────┬────────┘│    │
             │         │    │
             └─────────┼────┘
                       │
                       ▼
              ┌─────────────────┐
              │ Điền thông tin   │
              │ giao hàng        │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ Validate form   │
              └────────┬────────┘
                  Invalid│   Valid
                  │      │    │
                  └──────┼────┘
                         │
                         ▼
                ┌─────────────────┐
                │ Chọn phương thức │
                │ thanh toán       │
                └────────┬────────┘
                         │
                         ▼
                ┌─────────────────┐
                │ Tạo đơn hàng    │
                └────────┬────────┘
                         │
                         ▼
                ┌─────────────────┐
                │ Trừ tồn kho     │
                └────────┬────────┘
                         │
                         ▼
                ┌─────────────────┐
                │ Xóa giỏ hàng    │
                └────────┬────────┘
                         │
                         ▼
                ┌─────────────────┐
                │ Hiển thị thông  │
                │ báo thành công  │
                └────────┬────────┘
                         │
                         ▼
                    [Kết thúc]
```

### 3.5.2. Activity Diagram: Quy trình quản lý đơn hàng (Admin)

```
                    [Bắt đầu]
                       │
                       ▼
              ┌─────────────────┐
              │ Admin đăng nhập  │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ Xem danh sách   │
              │ đơn hàng        │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ Chọn đơn hàng   │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ Xem chi tiết    │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ Cập nhật trạng  │
              │ thái             │
              └────────┬────────┘
                       │
                  ┌────┴────┐
                  │         │
                  ▼         ▼
         ┌─────────────┐ ┌─────────────┐
         │ Xác nhận    │ │ Hủy đơn     │
         │ đơn hàng    │ │ hàng        │
         └──────┬──────┘ └──────┬──────┘
                │              │
                └──────┬───────┘
                       │
                       ▼
              ┌─────────────────┐
              │ Cập nhật database│
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ Gửi email thông │
              │ báo (nếu có)    │
              └────────┬────────┘
                       │
                       ▼
                    [Kết thúc]
```

---

## 3.10. Class Diagram (Object Design)

### 3.6.1. Domain Model Class Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        EShop.Core                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐      ┌──────────────┐                    │
│  │  Category    │      │   Product    │                    │
│  ├──────────────┤      ├──────────────┤                    │
│  │ +Id: int     │      │ +Id: int     │                    │
│  │ +Name: string│      │ +Name: string│                    │
│  │ +Description │      │ +Description │                    │
│  │ +ImageUrl    │      │ +Price: decimal                   │
│  │ +IsActive    │      │ +Stock: int  │                    │
│  │ +CreatedAt   │      │ +SKU: string │                    │
│  │ +UpdatedAt   │      │ +ImageUrl    │                    │
│  │              │      │ +CategoryId  │                    │
│  │ +Products    │      │ +IsActive    │                    │
│  └──────┬───────┘      │ +CreatedAt   │                    │
│         │ 1            │ +UpdatedAt   │                    │
│         │              │              │                    │
│         │              │ +Category    │                    │
│         │              │ +OrderItems  │                    │
│         │              └──────┬───────┘                    │
│         │                     │                            │
│         │                     │ *                          │
│         └─────────────────────┘                            │
│                                                              │
│  ┌──────────────┐      ┌──────────────┐                    │
│  │    Order     │      │  OrderItem   │                    │
│  ├──────────────┤      ├──────────────┤                    │
│  │ +Id: int     │      │ +Id: int     │                    │
│  │ +OrderNumber │      │ +OrderId: int│                    │
│  │ +UserId: string     │ +ProductId: int                   │
│  │ +OrderDate   │      │ +Quantity: int                    │
│  │ +Status: OrderStatus│ +Price: decimal                   │
│  │ +ShippingAddress     │              │                    │
│  │ +SubTotal: decimal   │              │                    │
│  │ +Total: decimal      │              │                    │
│  │              │      │ +Order        │                    │
│  │ +User        │      │ +Product     │                    │
│  │ +OrderItems  │      └──────┬───────┘                    │
│  └──────┬───────┘             │                            │
│         │ 1                   │ *                          │
│         │                     │                            │
│         │                     │                            │
│         │                     │                            │
│         └─────────────────────┘                            │
│                                                              │
│  ┌──────────────┐                                           │
│  │   AppUser    │                                           │
│  ├──────────────┤                                           │
│  │ +Id: string  │                                           │
│  │ +Email: string                                           │
│  │ +FullName: string                                        │
│  │ +PhoneNumber: string                                    │
│  │ +Address: string                                        │
│  │              │                                           │
│  │ +Orders: ICollection<Order>                              │
│  └──────┬───────┘                                           │
│         │ 1                                                 │
│         │                                                   │
│         │                                                   │
└─────────┼───────────────────────────────────────────────────┘
          │
          │
┌─────────▼───────────────────────────────────────────────────┐
│                    Enums                                    │
├─────────────────────────────────────────────────────────────┤
│  OrderStatus: Pending, Confirmed, Shipping, Delivered,      │
│               Cancelled                                     │
│  PaymentMethod: COD, BankTransfer, EWallet                 │
└─────────────────────────────────────────────────────────────┘
```

### 3.6.2. Application Layer Class Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    EShop.Application                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────┐      ┌──────────────────┐           │
│  │ IProductService  │      │  ProductService  │           │
│  ├──────────────────┤      ├──────────────────┤           │
│  │ +GetAllAsync()   │      │ -_unitOfWork     │           │
│  │ +GetByIdAsync()  │      │ -_mapper         │           │
│  │ +CreateAsync()   │      │                  │           │
│  │ +UpdateAsync()   │      │ +GetAllAsync()   │           │
│  │ +DeleteAsync()   │      │ +GetByIdAsync()  │           │
│  └────────┬─────────┘      │ +CreateAsync()   │           │
│           │ implements     │ +UpdateAsync()   │           │
│           └────────────────>│ +DeleteAsync()  │           │
│                             └──────────────────┘           │
│                                                              │
│  ┌──────────────────┐      ┌──────────────────┐           │
│  │   ProductDto     │      │ CreateProductDto │           │
│  ├──────────────────┤      ├──────────────────┤           │
│  │ +Id: int         │      │ +Name: string    │           │
│  │ +Name: string    │      │ +Description     │           │
│  │ +Price: decimal  │      │ +Price: decimal  │           │
│  │ +Stock: int      │      │ +Stock: int      │           │
│  │ +CategoryName    │      │ +CategoryId: int │           │
│  └──────────────────┘      └──────────────────┘           │
│                                                              │
│  ┌──────────────────┐                                       │
│  │  MappingProfile  │                                       │
│  ├──────────────────┤                                       │
│  │ +Product→ProductDto                                      │
│  │ +CreateProductDto→Product                                │
│  └──────────────────┘                                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 3.11. Component Diagram (System Design)

```
┌─────────────────────────────────────────────────────────────┐
│                    EShop.Web (MVC)                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Areas/Store                                         │   │
│  │  ┌──────────────┐  ┌──────────────┐                 │   │
│  │  │ HomeController│  │ProductController│            │   │
│  │  │ CartController │  │OrderController │            │   │
│  │  └──────────────┘  └──────────────┘                 │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Areas/Admin                                         │   │
│  │  ┌──────────────┐  ┌──────────────┐                 │   │
│  │  │DashboardCtrl │  │ProductCtrl  │                 │   │
│  │  │OrderCtrl     │  │CategoryCtrl │                 │   │
│  │  └──────────────┘  └──────────────┘                 │   │
│  └──────────────────────────────────────────────────────┘   │
└───────────────────────┬─────────────────────────────────────┘
                        │ uses
┌───────────────────────▼─────────────────────────────────────┐
│              EShop.Application                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ProductService│  │OrderService  │  │UserService   │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │ProductDto    │  │OrderDto     │                        │
│  │CartItemDto   │  │             │                        │
│  └──────────────┘  └──────────────┘                        │
└───────────────────────┬─────────────────────────────────────┘
                        │ uses
┌───────────────────────▼─────────────────────────────────────┐
│          EShop.Infrastructure                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ProductRepo   │  │OrderRepo    │  │CategoryRepo  │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │EShopDbContext│  │UnitOfWork  │                        │
│  └──────────────┘  └──────────────┘                        │
└───────────────────────┬─────────────────────────────────────┘
                        │ uses
┌───────────────────────▼─────────────────────────────────────┐
│              EShop.Core                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │Product       │  │Order        │  │Category      │     │
│  │Order         │  │OrderItem    │  │AppUser       │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │IProductRepo  │  │IOrderRepo   │                        │
│  │IUnitOfWork   │  │             │                        │
│  └──────────────┘  └──────────────┘                        │
└───────────────────────┬─────────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────────┐
│              SQL Server Database                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 3.12. Deployment Diagram (System Design)

```
┌─────────────────────────────────────────────────────────────┐
│                    Client Tier                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Browser    │  │   Browser    │  │   Browser    │     │
│  │  (Desktop)   │  │  (Tablet)    │  │  (Mobile)    │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└───────────────────────┬─────────────────────────────────────┘
                        │ HTTPS
                        │
┌───────────────────────▼─────────────────────────────────────┐
│                  Web Server Tier                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              IIS / Kestrel Server                     │   │
│  │  ┌────────────────────────────────────────────────┐  │   │
│  │  │         EShop.Web Application                  │  │   │
│  │  │  - ASP.NET Core MVC                            │  │   │
│  │  │  - Session Management                          │  │   │
│  │  │  - Static Files (wwwroot)                      │  │   │
│  │  └────────────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────────┘   │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        │
┌───────────────────────▼─────────────────────────────────────┐
│                  Application Tier                            │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  EShop.Application                                   │   │
│  │  - Business Logic                                   │   │
│  │  - Services                                         │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  EShop.Infrastructure                                │   │
│  │  - Data Access                                       │   │
│  │  - Repositories                                      │   │
│  └──────────────────────────────────────────────────────┘   │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        │ ADO.NET / EF Core
                        │
┌───────────────────────▼─────────────────────────────────────┐
│                  Database Tier                                │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           SQL Server Database                       │   │
│  │  ┌──────────────┐  ┌──────────────┐               │   │
│  │  │ Products     │  │ Orders       │               │   │
│  │  │ Categories   │  │ OrderItems   │               │   │
│  │  │ AspNetUsers  │  │              │               │   │
│  │  └──────────────┘  └──────────────┘               │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## 3.13. Design Patterns (Object Design)

### 3.9.1. Repository Pattern

**Mục đích:** Tách biệt data access logic khỏi business logic

**Cách triển khai:**
```csharp
// Interface trong Core
public interface IProductRepository
{
    Task<Product?> GetByIdAsync(int id);
    Task<IEnumerable<Product>> GetAllAsync();
}

// Implementation trong Infrastructure
public class ProductRepository : IProductRepository
{
    private readonly EShopDbContext _context;
    // Implementation...
}
```

**Lợi ích:**
- ✅ Dễ test (có thể mock repository)
- ✅ Dễ thay đổi data source (SQL → NoSQL)
- ✅ Tập trung data access logic

### 3.9.2. Unit of Work Pattern

**Mục đích:** Quản lý transactions, đảm bảo consistency

**Cách triển khai:**
```csharp
public interface IUnitOfWork
{
    IProductRepository Products { get; }
    IOrderRepository Orders { get; }
    Task<int> SaveChangesAsync();
}
```

**Lợi ích:**
- ✅ Đảm bảo tất cả changes được commit cùng lúc
- ✅ Tránh partial updates
- ✅ Dễ rollback khi có lỗi

### 3.9.3. Dependency Injection Pattern

**Mục đích:** Giảm coupling, tăng testability

**Cách triển khai:**
```csharp
// Register trong Program.cs
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
```

**Lợi ích:**
- ✅ Dễ test (inject mock objects)
- ✅ Loose coupling
- ✅ Centralized configuration

### 3.9.4. DTO Pattern (Data Transfer Object)

**Mục đích:** Tách biệt domain models khỏi presentation layer

**Cách triển khai:**
```csharp
// Domain Model
public class Product { ... }

// DTO
public class ProductDto { ... }

// Mapping với AutoMapper
CreateMap<Product, ProductDto>();
```

**Lợi ích:**
- ✅ Bảo vệ domain models
- ✅ Chỉ expose data cần thiết
- ✅ Dễ versioning API

---

## 3.14. Security Design (System Design)

### 3.10.1. Authentication Flow

```
┌──────────┐         ┌──────────┐         ┌──────────┐
│  User    │         │   Web    │         │ Identity │
│          │         │          │         │          │
│──Login──>│         │          │         │          │
│          │──POST──>│          │         │          │
│          │         │──Validate│         │          │
│          │         │  Credentials──────>│          │
│          │         │          │         │──Check──>│
│          │         │          │         │  User    │
│          │         │          │<──Token─│          │
│          │<──Cookie│          │         │          │
│<──Success│         │          │         │          │
└──────────┘         └──────────┘         └──────────┘
```

### 3.10.2. Authorization Strategy

**Role-based Authorization:**
- **User Role:** Xem sản phẩm, đặt hàng, xem đơn hàng của mình
- **Admin Role:** Tất cả quyền của User + Quản lý sản phẩm, đơn hàng, users

**Implementation:**
```csharp
[Authorize(Roles = "Admin")]
public class ProductController : Controller
{
    // Chỉ Admin mới truy cập được
}
```

### 3.10.3. Data Protection

**SQL Injection Prevention:**
- Sử dụng EF Core (parameterized queries tự động)
- Không dùng string concatenation cho SQL

**XSS Prevention:**
- Razor tự động encode output
- Validate input từ user

**CSRF Protection:**
- Anti-forgery tokens trong forms
- Validate tokens trên server

---

## 3.15. Performance Design (System Design)

### 3.11.1. Caching Strategy

**In-Memory Caching:**
- Cache danh sách danh mục (ít thay đổi)
- Cache sản phẩm nổi bật
- TTL: 30 phút

**Implementation:**
```csharp
var categories = await _memoryCache.GetOrCreateAsync("categories", 
    async entry =>
    {
        entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(30);
        return await _categoryService.GetAllAsync();
    });
```

### 3.11.2. Database Optimization

**Indexes:**
- `Products.CategoryId` (Foreign Key)
- `Products.Name` (Search)
- `Orders.UserId` (Foreign Key)
- `Orders.OrderDate` (Sorting)

**Query Optimization:**
- Eager Loading với `Include()`
- `AsNoTracking()` cho read-only queries
- Pagination cho danh sách lớn

### 3.11.3. Image Optimization

**Strategy:**
- Resize images khi upload (max 800x800)
- Compress images (JPEG quality 80%)
- Lazy loading trong Views
- CDN cho production (tùy chọn)

---

## 3.16. Risk Analysis (System Design)

### 3.12.1. Technical Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Database performance | High | Medium | Indexes, Caching, Query optimization |
| Security vulnerabilities | High | Low | Regular security audits, HTTPS, Input validation |
| Session timeout | Medium | Low | Extend session timeout, Auto-save cart |
| Image storage full | Medium | Low | Monitor disk space, Auto-cleanup old images |

### 3.12.2. Business Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Concurrent order conflicts | High | Medium | Optimistic locking, Transaction isolation |
| Stock inconsistency | High | Medium | Check stock before checkout, Reserve stock |
| Payment failure | Medium | Low | Retry mechanism, Clear error messages |

---

## 3.17. Các thành phần chính (Chi tiết)

### 3.13.1. Presentation Layer (EShop.Web)

**Trách nhiệm:**
- Xử lý HTTP requests/responses
- Render Views (Razor)
- Validate input từ user
- Session management (giỏ hàng)
- Authentication/Authorization

**Cấu trúc:**
```
EShop.Web/
├── Areas/
│   ├── Store/          # Front Store
│   │   ├── Controllers/
│   │   └── Views/
│   └── Admin/          # Admin Panel
│       ├── Controllers/
│       └── Views/
├── Controllers/        # Account, Home
├── ViewModels/         # DTOs cho Views
├── Views/
│   ├── Shared/
│   │   ├── _Layout.cshtml
│   │   └── _AdminLayout.cshtml
│   └── Account/
└── wwwroot/            # Static files
```

### 3.13.2. Application Layer (EShop.Application)

**Trách nhiệm:**
- Business logic
- Validation rules
- Data transformation (Entity ↔ DTO)
- Orchestration (gọi nhiều repositories)

**Cấu trúc:**
```
EShop.Application/
├── Services/
│   ├── IProductService.cs
│   ├── ProductService.cs
│   ├── IOrderService.cs
│   └── OrderService.cs
├── DTOs/
│   ├── ProductDto.cs
│   ├── OrderDto.cs
│   └── CartItemDto.cs
└── Mappings/
    └── MappingProfile.cs
```

### 3.13.3. Infrastructure Layer (EShop.Infrastructure)

**Trách nhiệm:**
- Data access (EF Core)
- External services integration
- File storage
- Email sending (tùy chọn)

**Cấu trúc:**
```
EShop.Infrastructure/
├── Data/
│   └── EShopDbContext.cs
├── Repositories/
│   ├── ProductRepository.cs
│   ├── OrderRepository.cs
│   └── UnitOfWork.cs
└── Migrations/
```

### 3.13.4. Domain Layer (EShop.Core)

**Trách nhiệm:**
- Domain models (Entities)
- Business rules (trong entities)
- Interfaces (contracts)
- Enums

**Cấu trúc:**
```
EShop.Core/
├── Entities/
│   ├── Product.cs
│   ├── Category.cs
│   ├── Order.cs
│   └── OrderItem.cs
├── Interfaces/
│   ├── IProductRepository.cs
│   ├── IOrderRepository.cs
│   └── IUnitOfWork.cs
└── Enums/
    ├── OrderStatus.cs
    └── PaymentMethod.cs
```

---

# 4. THIẾT KẾ DATABASE

## 4.1. Entity Relationship Diagram (ERD)

### 4.1.1. ERD Tổng quan

```
┌─────────────────────────────────────────────────────────────┐
│                    DATABASE SCHEMA                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐                    ┌──────────────┐       │
│  │  Categories  │                    │   Products   │       │
│  ├──────────────┤                    ├──────────────┤       │
│  │ PK Id        │◄──────────────────┤ PK Id        │       │
│  │    Name      │  1              N  │ FK CategoryId│       │
│  │    Description                    │    Name      │       │
│  │    ImageUrl  │                    │    Price     │       │
│  │    IsActive  │                    │    Stock     │       │
│  │    CreatedAt │                    │    SKU       │       │
│  │    UpdatedAt │                    │    ImageUrl  │       │
│  └──────────────┘                    │    IsActive  │       │
│                                      │    CreatedAt │       │
│                                      │    UpdatedAt │       │
│                                      └──────┬───────┘       │
│                                             │ 1             │
│                                             │               │
│                                             │ N             │
│                                      ┌──────▼───────┐       │
│                                      │  OrderItems  │       │
│                                      ├──────────────┤       │
│                                      │ PK Id        │       │
│                                      │ FK OrderId   │       │
│                                      │ FK ProductId │       │
│                                      │    Quantity  │       │
│                                      │    Price     │       │
│                                      └──────┬───────┘       │
│                                             │ N             │
│                                             │               │
│                                      ┌──────▼───────┐       │
│                                      │    Orders    │       │
│                                      ├──────────────┤       │
│                                      │ PK Id        │       │
│                                      │    OrderNumber│      │
│                                      │ FK UserId    │       │
│                                      │    OrderDate │       │
│                                      │    Status    │       │
│                                      │    ShippingAddress   │
│                                      │    SubTotal  │       │
│                                      │    Total     │       │
│                                      └──────┬───────┘       │
│                                             │ N             │
│                                             │               │
│                                      ┌──────▼───────┐       │
│                                      │  AspNetUsers │       │
│                                      │  (AppUser)   │       │
│                                      ├──────────────┤       │
│                                      │ PK Id        │       │
│                                      │    Email     │       │
│                                      │    FullName  │       │
│                                      │    PhoneNumber│      │
│                                      │    Address   │       │
│                                      └──────────────┘       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 4.1.2. Relationships Chi tiết

**1. Category ↔ Product (One-to-Many)**
- 1 Category có nhiều Products
- Product phải thuộc 1 Category
- Foreign Key: `Products.CategoryId → Categories.Id`
- Delete Behavior: `Restrict` (không cho xóa Category nếu có Products)

**2. Product ↔ OrderItem (One-to-Many)**
- 1 Product có thể có nhiều OrderItems (trong các đơn hàng khác nhau)
- OrderItem phải thuộc 1 Product
- Foreign Key: `OrderItems.ProductId → Products.Id`
- Delete Behavior: `Restrict` (không cho xóa Product nếu có trong đơn hàng)

**3. Order ↔ OrderItem (One-to-Many)**
- 1 Order có nhiều OrderItems
- OrderItem phải thuộc 1 Order
- Foreign Key: `OrderItems.OrderId → Orders.Id`
- Delete Behavior: `Cascade` (xóa Order → xóa OrderItems)

**4. AppUser ↔ Order (One-to-Many)**
- 1 User có nhiều Orders
- Order phải thuộc 1 User
- Foreign Key: `Orders.UserId → AspNetUsers.Id`
- Delete Behavior: `Restrict` (không cho xóa User nếu có Orders)

---

## 4.2. Phân tích dữ liệu

### 4.1.1. Entity: Category (Danh mục)

**Mô tả:** Danh mục sản phẩm (Ví dụ: Điện thoại, Laptop, Phụ kiện)

**Thuộc tính:**
- `Id` (int, PK): Mã danh mục
- `Name` (string, required): Tên danh mục
- `Description` (string, nullable): Mô tả
- `ImageUrl` (string, nullable): Hình ảnh danh mục
- `IsActive` (bool): Trạng thái (true = hiển thị, false = ẩn)
- `CreatedAt` (DateTime): Ngày tạo
- `UpdatedAt` (DateTime, nullable): Ngày cập nhật

**Quan hệ:**
- 1 Category → N Products (One-to-Many)

### 4.1.2. Entity: Product (Sản phẩm)

**Mô tả:** Sản phẩm bán trong cửa hàng

**Thuộc tính:**
- `Id` (int, PK): Mã sản phẩm
- `Name` (string, required): Tên sản phẩm
- `Description` (string, nullable): Mô tả chi tiết
- `Price` (decimal): Giá bán
- `Stock` (int): Số lượng tồn kho
- `SKU` (string, unique): Mã SKU (Stock Keeping Unit)
- `ImageUrl` (string, nullable): Hình ảnh chính
- `CategoryId` (int, FK): Mã danh mục
- `IsActive` (bool): Trạng thái
- `CreatedAt` (DateTime): Ngày tạo
- `UpdatedAt` (DateTime, nullable): Ngày cập nhật

**Quan hệ:**
- N Products → 1 Category (Many-to-One)
- 1 Product → N OrderItems (One-to-Many)

### 4.1.3. Entity: AppUser (Người dùng)

**Mô tả:** Tài khoản người dùng (kế thừa từ IdentityUser)

**Thuộc tính:**
- `Id` (string, PK): Mã người dùng (từ Identity)
- `Email` (string, required, unique): Email
- `FullName` (string, nullable): Họ tên
- `PhoneNumber` (string, nullable): Số điện thoại
- `Address` (string, nullable): Địa chỉ
- `City` (string, nullable): Thành phố
- `PostalCode` (string, nullable): Mã bưu điện

**Quan hệ:**
- 1 AppUser → N Orders (One-to-Many)

### 4.1.4. Entity: Order (Đơn hàng)

**Mô tả:** Đơn hàng của khách hàng

**Thuộc tính:**
- `Id` (int, PK): Mã đơn hàng
- `OrderNumber` (string, unique): Số đơn hàng (tự động sinh: ORD-YYYYMMDD-XXXX)
- `UserId` (string, FK): Mã khách hàng
- `OrderDate` (DateTime): Ngày đặt hàng
- `Status` (OrderStatus enum): Trạng thái
- `ShippingAddress` (string): Địa chỉ giao hàng
- `ShippingCity` (string): Thành phố
- `ShippingPostalCode` (string): Mã bưu điện
- `ShippingPhone` (string): Số điện thoại người nhận
- `ShippingName` (string): Tên người nhận
- `PaymentMethod` (PaymentMethod enum): Phương thức thanh toán
- `SubTotal` (decimal): Tổng tiền sản phẩm
- `ShippingFee` (decimal): Phí vận chuyển
- `Total` (decimal): Tổng tiền
- `Notes` (string, nullable): Ghi chú

**Quan hệ:**
- 1 Order → 1 AppUser (Many-to-One)
- 1 Order → N OrderItems (One-to-Many)

### 4.1.5. Entity: OrderItem (Chi tiết đơn hàng)

**Mô tả:** Sản phẩm trong đơn hàng

**Thuộc tính:**
- `Id` (int, PK): Mã chi tiết
- `OrderId` (int, FK): Mã đơn hàng
- `ProductId` (int, FK): Mã sản phẩm
- `Quantity` (int): Số lượng
- `Price` (decimal): Giá tại thời điểm mua (lưu lại để tránh thay đổi)

**Quan hệ:**
- N OrderItems → 1 Order (Many-to-One)
- N OrderItems → 1 Product (Many-to-One)

## 4.4. Entity Relationship Diagram (ERD) Chi tiết

### 4.2.1. ERD với Cardinality

```
                    ┌─────────────────┐
                    │   Categories    │
                    ├─────────────────┤
                    │ PK Id (int)     │
                    │    Name         │
                    │    Description  │
                    │    ImageUrl     │
                    │    IsActive     │
                    │    CreatedAt    │
                    │    UpdatedAt    │
                    └────────┬────────┘
                             │
                             │ 1
                             │
                             │ has many
                             │
                             │ N
                    ┌────────▼────────┐
                    │    Products     │
                    ├─────────────────┤
                    │ PK Id (int)     │
                    │ FK CategoryId   │──┐
                    │    Name         │  │
                    │    Description  │  │
                    │    Price        │  │
                    │    Stock        │  │
                    │    SKU          │  │
                    │    ImageUrl     │  │
                    │    IsActive     │  │
                    │    CreatedAt    │  │
                    │    UpdatedAt    │  │
                    └────────┬────────┘  │
                             │           │
                             │ 1         │
                             │           │
                             │ has many  │
                             │           │
                             │ N         │
                    ┌────────▼────────┐  │
                    │   OrderItems   │  │
                    ├─────────────────┤  │
                    │ PK Id (int)     │  │
                    │ FK OrderId      │──┼──┐
                    │ FK ProductId    │──┘  │
                    │    Quantity     │     │
                    │    Price        │     │
                    └────────┬────────┘     │
                             │ N            │
                             │              │
                             │ belongs to   │
                             │              │
                             │ 1            │
                    ┌────────▼────────┐    │
                    │     Orders      │    │
                    ├─────────────────┤    │
                    │ PK Id (int)     │    │
                    │    OrderNumber │    │
                    │ FK UserId      │────┼──┐
                    │    OrderDate   │    │  │
                    │    Status      │    │  │
                    │    ShippingAddress│  │  │
                    │    SubTotal    │    │  │
                    │    Total       │    │  │
                    └────────┬────────┘    │  │
                             │ N           │  │
                             │             │  │
                             │ belongs to  │  │
                             │             │  │
                             │ 1           │  │
                    ┌────────▼────────┐    │  │
                    │   AspNetUsers  │    │  │
                    │   (AppUser)    │    │  │
                    ├─────────────────┤    │  │
                    │ PK Id (string) │◄────┘  │
                    │    Email       │       │
                    │    FullName    │       │
                    │    PhoneNumber │       │
                    │    Address     │       │
                    └────────────────┘       │
                                             │
                                             │
                    ┌────────────────────────┘
                    │
                    │ Relationship: Category → Product
                    │ Type: One-to-Many (1:N)
                    │ Cardinality: 1 Category → 0..N Products
                    │
                    │ Relationship: Product → OrderItem
                    │ Type: One-to-Many (1:N)
                    │ Cardinality: 1 Product → 0..N OrderItems
                    │
                    │ Relationship: Order → OrderItem
                    │ Type: One-to-Many (1:N)
                    │ Cardinality: 1 Order → 1..N OrderItems
                    │
                    │ Relationship: AppUser → Order
                    │ Type: One-to-Many (1:N)
                    │ Cardinality: 1 User → 0..N Orders
```

### 4.2.2. Bảng mô tả Relationships

| Relationship | Type | Cardinality | Foreign Key | Delete Behavior |
|--------------|------|-------------|-------------|-----------------|
| Category → Product | One-to-Many | 1:N | `Products.CategoryId` | Restrict |
| Product → OrderItem | One-to-Many | 1:N | `OrderItems.ProductId` | Restrict |
| Order → OrderItem | One-to-Many | 1:N | `OrderItems.OrderId` | Cascade |
| AppUser → Order | One-to-Many | 1:N | `Orders.UserId` | Restrict |

**Giải thích Delete Behavior:**
- **Restrict:** Không cho phép xóa parent nếu có child records
- **Cascade:** Xóa parent → tự động xóa child records

---

## 4.5. Database Schema (SQL Script - Physical Design)

```sql
-- Categories Table
CREATE TABLE Categories (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Name NVARCHAR(100) NOT NULL,
    Description NVARCHAR(MAX),
    ImageUrl NVARCHAR(500),
    IsActive BIT DEFAULT 1,
    CreatedAt DATETIME2 DEFAULT GETDATE(),
    UpdatedAt DATETIME2
);

-- Products Table
CREATE TABLE Products (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Name NVARCHAR(200) NOT NULL,
    Description NVARCHAR(MAX),
    Price DECIMAL(18,2) NOT NULL,
    Stock INT DEFAULT 0,
    SKU NVARCHAR(50) UNIQUE,
    ImageUrl NVARCHAR(500),
    CategoryId INT NOT NULL,
    IsActive BIT DEFAULT 1,
    CreatedAt DATETIME2 DEFAULT GETDATE(),
    UpdatedAt DATETIME2,
    FOREIGN KEY (CategoryId) REFERENCES Categories(Id)
);

-- AspNetUsers (Identity - mở rộng)
-- Thêm các cột: FullName, PhoneNumber, Address, City, PostalCode

-- Orders Table
CREATE TABLE Orders (
    Id INT PRIMARY KEY IDENTITY(1,1),
    OrderNumber NVARCHAR(50) UNIQUE NOT NULL,
    UserId NVARCHAR(450) NOT NULL,
    OrderDate DATETIME2 DEFAULT GETDATE(),
    Status INT NOT NULL, -- 0: Pending, 1: Confirmed, 2: Shipping, 3: Delivered, 4: Cancelled
    ShippingAddress NVARCHAR(500) NOT NULL,
    ShippingCity NVARCHAR(100),
    ShippingPostalCode NVARCHAR(20),
    ShippingPhone NVARCHAR(20) NOT NULL,
    ShippingName NVARCHAR(100) NOT NULL,
    PaymentMethod INT NOT NULL, -- 0: COD, 1: BankTransfer, 2: EWallet
    SubTotal DECIMAL(18,2) NOT NULL,
    ShippingFee DECIMAL(18,2) DEFAULT 0,
    Total DECIMAL(18,2) NOT NULL,
    Notes NVARCHAR(MAX),
    FOREIGN KEY (UserId) REFERENCES AspNetUsers(Id)
);

-- OrderItems Table
CREATE TABLE OrderItems (
    Id INT PRIMARY KEY IDENTITY(1,1),
    OrderId INT NOT NULL,
    ProductId INT NOT NULL,
    Quantity INT NOT NULL,
    Price DECIMAL(18,2) NOT NULL,
    FOREIGN KEY (OrderId) REFERENCES Orders(Id) ON DELETE CASCADE,
    FOREIGN KEY (ProductId) REFERENCES Products(Id)
);

-- Indexes
CREATE INDEX IX_Products_CategoryId ON Products(CategoryId);
CREATE INDEX IX_Orders_UserId ON Orders(UserId);
CREATE INDEX IX_Orders_OrderDate ON Orders(OrderDate);
CREATE INDEX IX_OrderItems_OrderId ON OrderItems(OrderId);
CREATE INDEX IX_OrderItems_ProductId ON OrderItems(ProductId);
```

## 4.3. Enums

```csharp
// EShop.Core/Enums/OrderStatus.cs
public enum OrderStatus
{
    Pending = 0,      // Chờ xác nhận
    Confirmed = 1,    // Đã xác nhận
    Shipping = 2,     // Đang giao
    Delivered = 3,    // Đã giao
    Cancelled = 4     // Đã hủy
}

// EShop.Core/Enums/PaymentMethod.cs
public enum PaymentMethod
{
    COD = 0,              // Thanh toán khi nhận hàng
    BankTransfer = 1,     // Chuyển khoản
    EWallet = 2           // Ví điện tử
}
```

---

# 5. KIẾN TRÚC DỰ ÁN

## 5.1. Cấu trúc Solution

```
EShop.sln
│
├── EShop.Core (Class Library)
│   ├── Entities/
│   │   ├── Product.cs
│   │   ├── Category.cs
│   │   ├── Order.cs
│   │   └── OrderItem.cs
│   ├── Enums/
│   │   ├── OrderStatus.cs
│   │   └── PaymentMethod.cs
│   └── Interfaces/
│       ├── IProductRepository.cs
│       ├── IOrderRepository.cs
│       └── IUnitOfWork.cs
│
├── EShop.Infrastructure (Class Library)
│   ├── Data/
│   │   ├── EShopDbContext.cs
│   │   └── Configurations/
│   │       ├── ProductConfiguration.cs
│   │       └── OrderConfiguration.cs
│   ├── Repositories/
│   │   ├── ProductRepository.cs
│   │   ├── OrderRepository.cs
│   │   └── UnitOfWork.cs
│   └── Migrations/
│
├── EShop.Application (Class Library)
│   ├── Services/
│   │   ├── IProductService.cs
│   │   ├── ProductService.cs
│   │   ├── IOrderService.cs
│   │   └── OrderService.cs
│   ├── DTOs/
│   │   ├── ProductDto.cs
│   │   ├── OrderDto.cs
│   │   └── CartItemDto.cs
│   └── Mappings/
│       └── MappingProfile.cs (AutoMapper)
│
└── EShop.Web (ASP.NET Core MVC)
    ├── Areas/
    │   ├── Store/
    │   │   ├── Controllers/
    │   │   │   ├── HomeController.cs
    │   │   │   ├── ProductController.cs
    │   │   │   ├── CartController.cs
    │   │   │   └── OrderController.cs
    │   │   └── Views/
    │   └── Admin/
    │       ├── Controllers/
    │       │   ├── DashboardController.cs
    │       │   ├── ProductController.cs
    │       │   ├── CategoryController.cs
    │       │   └── OrderController.cs
    │       └── Views/
    ├── Controllers/
    │   └── AccountController.cs
    ├── ViewModels/
    │   ├── ProductViewModel.cs
    │   ├── CartViewModel.cs
    │   └── CheckoutViewModel.cs
    ├── Views/
    │   ├── Shared/
    │   │   ├── _Layout.cshtml
    │   │   └── _AdminLayout.cshtml
    │   └── Account/
    ├── wwwroot/
    │   ├── css/
    │   ├── js/
    │   └── images/
    │       └── products/
    └── Program.cs
```

## 5.2. Dependencies

### EShop.Core
- Không có dependencies (Pure Domain)

### EShop.Infrastructure
- **EShop.Core** (reference)
- **Microsoft.EntityFrameworkCore.SqlServer**
- **Microsoft.EntityFrameworkCore.Tools**

### EShop.Application
- **EShop.Core** (reference)
- **EShop.Infrastructure** (reference)
- **AutoMapper**
- **AutoMapper.Extensions.Microsoft.DependencyInjection**

### EShop.Web
- **EShop.Application** (reference)
- **Microsoft.AspNetCore.Identity.EntityFrameworkCore**
- **Microsoft.EntityFrameworkCore.Design**
- **Bootstrap 5** (CDN hoặc npm)

---

# 6. TRIỂN KHAI TỪNG BƯỚC

## BƯỚC 1: Khởi tạo Solution và Projects

### 1.1. Tạo Solution

```powershell
# Tạo thư mục dự án
mkdir EShop
cd EShop

# Tạo Solution
dotnet new sln -n EShop
```

### 1.2. Tạo các Projects

```powershell
# Tạo Class Libraries
dotnet new classlib -n EShop.Core
dotnet new classlib -n EShop.Infrastructure
dotnet new classlib -n EShop.Application

# Tạo MVC Project
dotnet new mvc -n EShop.Web

# Thêm vào Solution
dotnet sln add EShop.Core
dotnet sln add EShop.Infrastructure
dotnet sln add EShop.Application
dotnet sln add EShop.Web
```

### 1.3. Thiết lập Dependencies

```powershell
# Infrastructure phụ thuộc Core
dotnet add EShop.Infrastructure reference EShop.Core

# Application phụ thuộc Core và Infrastructure
dotnet add EShop.Application reference EShop.Core
dotnet add EShop.Application reference EShop.Infrastructure

# Web phụ thuộc Application
dotnet add EShop.Web reference EShop.Application
```

### 1.4. Cài đặt NuGet Packages

```powershell
# Infrastructure
cd EShop.Infrastructure
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
cd ..

# Application
cd EShop.Application
dotnet add package AutoMapper
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
cd ..

# Web
cd EShop.Web
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.Design
cd ..
```

---

## BƯỚC 2: Xây dựng Domain Layer (EShop.Core)

### 2.1. Tạo Entities

**EShop.Core/Entities/Category.cs:**
```csharp
namespace EShop.Core.Entities;

public class Category
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string? Description { get; set; }
    public string? ImageUrl { get; set; }
    public bool IsActive { get; set; } = true;
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? UpdatedAt { get; set; }

    // Navigation Property
    public ICollection<Product> Products { get; set; } = new List<Product>();
}
```

**EShop.Core/Entities/Product.cs:**
```csharp
namespace EShop.Core.Entities;

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string? Description { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
    public string? SKU { get; set; }
    public string? ImageUrl { get; set; }
    public int CategoryId { get; set; }
    public bool IsActive { get; set; } = true;
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? UpdatedAt { get; set; }

    // Navigation Properties
    public Category Category { get; set; } = null!;
    public ICollection<OrderItem> OrderItems { get; set; } = new List<OrderItem>();
}
```

**EShop.Core/Entities/Order.cs:**
```csharp
using EShop.Core.Enums;

namespace EShop.Core.Entities;

public class Order
{
    public int Id { get; set; }
    public string OrderNumber { get; set; } = string.Empty;
    public string UserId { get; set; } = string.Empty;
    public DateTime OrderDate { get; set; } = DateTime.UtcNow;
    public OrderStatus Status { get; set; } = OrderStatus.Pending;
    public string ShippingAddress { get; set; } = string.Empty;
    public string? ShippingCity { get; set; }
    public string? ShippingPostalCode { get; set; }
    public string ShippingPhone { get; set; } = string.Empty;
    public string ShippingName { get; set; } = string.Empty;
    public PaymentMethod PaymentMethod { get; set; }
    public decimal SubTotal { get; set; }
    public decimal ShippingFee { get; set; }
    public decimal Total { get; set; }
    public string? Notes { get; set; }

    // Navigation Properties
    public AppUser User { get; set; } = null!;
    public ICollection<OrderItem> OrderItems { get; set; } = new List<OrderItem>();
}
```

**EShop.Core/Entities/OrderItem.cs:**
```csharp
namespace EShop.Core.Entities;

public class OrderItem
{
    public int Id { get; set; }
    public int OrderId { get; set; }
    public int ProductId { get; set; }
    public int Quantity { get; set; }
    public decimal Price { get; set; }

    // Navigation Properties
    public Order Order { get; set; } = null!;
    public Product Product { get; set; } = null!;
}
```

**EShop.Core/Entities/AppUser.cs:**
```csharp
using Microsoft.AspNetCore.Identity;

namespace EShop.Core.Entities;

public class AppUser : IdentityUser
{
    public string? FullName { get; set; }
    public string? Address { get; set; }
    public string? City { get; set; }
    public string? PostalCode { get; set; }

    // Navigation Properties
    public ICollection<Order> Orders { get; set; } = new List<Order>();
}
```

### 2.2. Tạo Enums

**EShop.Core/Enums/OrderStatus.cs:**
```csharp
namespace EShop.Core.Enums;

public enum OrderStatus
{
    Pending = 0,      // Chờ xác nhận
    Confirmed = 1,    // Đã xác nhận
    Shipping = 2,     // Đang giao
    Delivered = 3,    // Đã giao
    Cancelled = 4     // Đã hủy
}
```

**EShop.Core/Enums/PaymentMethod.cs:**
```csharp
namespace EShop.Core.Enums;

public enum PaymentMethod
{
    COD = 0,              // Thanh toán khi nhận hàng
    BankTransfer = 1,     // Chuyển khoản
    EWallet = 2           // Ví điện tử
}
```

### 2.3. Tạo Interfaces

**EShop.Core/Interfaces/IProductRepository.cs:**
```csharp
using EShop.Core.Entities;

namespace EShop.Core.Interfaces;

public interface IProductRepository
{
    Task<IEnumerable<Product>> GetAllAsync();
    Task<Product?> GetByIdAsync(int id);
    Task<IEnumerable<Product>> GetByCategoryIdAsync(int categoryId);
    Task<IEnumerable<Product>> SearchAsync(string keyword);
    Task<Product> AddAsync(Product product);
    Task UpdateAsync(Product product);
    Task DeleteAsync(int id);
    Task<bool> ExistsAsync(int id);
}
```

**EShop.Core/Interfaces/IOrderRepository.cs:**
```csharp
using EShop.Core.Entities;
using EShop.Core.Enums;

namespace EShop.Core.Interfaces;

public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(int id);
    Task<Order?> GetByOrderNumberAsync(string orderNumber);
    Task<IEnumerable<Order>> GetByUserIdAsync(string userId);
    Task<IEnumerable<Order>> GetAllAsync();
    Task<Order> AddAsync(Order order);
    Task UpdateAsync(Order order);
    Task<string> GenerateOrderNumberAsync();
}
```

**EShop.Core/Interfaces/IUnitOfWork.cs:**
```csharp
namespace EShop.Core.Interfaces;

public interface IUnitOfWork : IDisposable
{
    IProductRepository Products { get; }
    IOrderRepository Orders { get; }
    Task<int> SaveChangesAsync();
}
```

---

## BƯỚC 3: Xây dựng Infrastructure Layer (EShop.Infrastructure)

### 3.1. Tạo DbContext

**EShop.Infrastructure/Data/EShopDbContext.cs:**
```csharp
using EShop.Core.Entities;
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

namespace EShop.Infrastructure.Data;

public class EShopDbContext : IdentityDbContext<AppUser>
{
    public EShopDbContext(DbContextOptions<EShopDbContext> options)
        : base(options)
    {
    }

    public DbSet<Product> Products { get; set; }
    public DbSet<Category> Categories { get; set; }
    public DbSet<Order> Orders { get; set; }
    public DbSet<OrderItem> OrderItems { get; set; }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);

        // Product Configuration
        builder.Entity<Product>(entity =>
        {
            entity.HasKey(p => p.Id);
            entity.Property(p => p.Name).IsRequired().HasMaxLength(200);
            entity.Property(p => p.Price).HasColumnType("decimal(18,2)");
            entity.HasIndex(p => p.SKU).IsUnique();
            entity.HasOne(p => p.Category)
                  .WithMany(c => c.Products)
                  .HasForeignKey(p => p.CategoryId)
                  .OnDelete(DeleteBehavior.Restrict);
        });

        // Category Configuration
        builder.Entity<Category>(entity =>
        {
            entity.HasKey(c => c.Id);
            entity.Property(c => c.Name).IsRequired().HasMaxLength(100);
        });

        // Order Configuration
        builder.Entity<Order>(entity =>
        {
            entity.HasKey(o => o.Id);
            entity.Property(o => o.OrderNumber).IsRequired().HasMaxLength(50);
            entity.HasIndex(o => o.OrderNumber).IsUnique();
            entity.Property(o => o.SubTotal).HasColumnType("decimal(18,2)");
            entity.Property(o => o.ShippingFee).HasColumnType("decimal(18,2)");
            entity.Property(o => o.Total).HasColumnType("decimal(18,2)");
            entity.HasOne(o => o.User)
                  .WithMany(u => u.Orders)
                  .HasForeignKey(o => o.UserId)
                  .OnDelete(DeleteBehavior.Restrict);
        });

        // OrderItem Configuration
        builder.Entity<OrderItem>(entity =>
        {
            entity.HasKey(oi => oi.Id);
            entity.Property(oi => oi.Price).HasColumnType("decimal(18,2)");
            entity.HasOne(oi => oi.Order)
                  .WithMany(o => o.OrderItems)
                  .HasForeignKey(oi => oi.OrderId)
                  .OnDelete(DeleteBehavior.Cascade);
            entity.HasOne(oi => oi.Product)
                  .WithMany(p => p.OrderItems)
                  .HasForeignKey(oi => oi.ProductId)
                  .OnDelete(DeleteBehavior.Restrict);
        });
    }
}
```

### 3.2. Tạo Repositories

**EShop.Infrastructure/Repositories/ProductRepository.cs:**
```csharp
using EShop.Core.Entities;
using EShop.Core.Interfaces;
using EShop.Infrastructure.Data;
using Microsoft.EntityFrameworkCore;

namespace EShop.Infrastructure.Repositories;

public class ProductRepository : IProductRepository
{
    private readonly EShopDbContext _context;

    public ProductRepository(EShopDbContext context)
    {
        _context = context;
    }

    public async Task<IEnumerable<Product>> GetAllAsync()
    {
        return await _context.Products
            .Include(p => p.Category)
            .Where(p => p.IsActive)
            .ToListAsync();
    }

    public async Task<Product?> GetByIdAsync(int id)
    {
        return await _context.Products
            .Include(p => p.Category)
            .FirstOrDefaultAsync(p => p.Id == id);
    }

    public async Task<IEnumerable<Product>> GetByCategoryIdAsync(int categoryId)
    {
        return await _context.Products
            .Include(p => p.Category)
            .Where(p => p.CategoryId == categoryId && p.IsActive)
            .ToListAsync();
    }

    public async Task<IEnumerable<Product>> SearchAsync(string keyword)
    {
        return await _context.Products
            .Include(p => p.Category)
            .Where(p => p.IsActive && 
                       (p.Name.Contains(keyword) || 
                        p.Description != null && p.Description.Contains(keyword)))
            .ToListAsync();
    }

    public async Task<Product> AddAsync(Product product)
    {
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
        return product;
    }

    public async Task UpdateAsync(Product product)
    {
        product.UpdatedAt = DateTime.UtcNow;
        _context.Products.Update(product);
        await _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(int id)
    {
        var product = await _context.Products.FindAsync(id);
        if (product != null)
        {
            product.IsActive = false; // Soft delete
            product.UpdatedAt = DateTime.UtcNow;
            await _context.SaveChangesAsync();
        }
    }

    public async Task<bool> ExistsAsync(int id)
    {
        return await _context.Products.AnyAsync(p => p.Id == id);
    }
}
```

**EShop.Infrastructure/Repositories/OrderRepository.cs:**
```csharp
using EShop.Core.Entities;
using EShop.Core.Interfaces;
using EShop.Infrastructure.Data;
using Microsoft.EntityFrameworkCore;

namespace EShop.Infrastructure.Repositories;

public class OrderRepository : IOrderRepository
{
    private readonly EShopDbContext _context;

    public OrderRepository(EShopDbContext context)
    {
        _context = context;
    }

    public async Task<Order?> GetByIdAsync(int id)
    {
        return await _context.Orders
            .Include(o => o.User)
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
            .FirstOrDefaultAsync(o => o.Id == id);
    }

    public async Task<Order?> GetByOrderNumberAsync(string orderNumber)
    {
        return await _context.Orders
            .Include(o => o.User)
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
            .FirstOrDefaultAsync(o => o.OrderNumber == orderNumber);
    }

    public async Task<IEnumerable<Order>> GetByUserIdAsync(string userId)
    {
        return await _context.Orders
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
            .Where(o => o.UserId == userId)
            .OrderByDescending(o => o.OrderDate)
            .ToListAsync();
    }

    public async Task<IEnumerable<Order>> GetAllAsync()
    {
        return await _context.Orders
            .Include(o => o.User)
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
            .OrderByDescending(o => o.OrderDate)
            .ToListAsync();
    }

    public async Task<Order> AddAsync(Order order)
    {
        _context.Orders.Add(order);
        await _context.SaveChangesAsync();
        return order;
    }

    public async Task UpdateAsync(Order order)
    {
        _context.Orders.Update(order);
        await _context.SaveChangesAsync();
    }

    public async Task<string> GenerateOrderNumberAsync()
    {
        var today = DateTime.UtcNow;
        var prefix = $"ORD-{today:yyyyMMdd}-";
        
        var lastOrder = await _context.Orders
            .Where(o => o.OrderNumber.StartsWith(prefix))
            .OrderByDescending(o => o.OrderNumber)
            .FirstOrDefaultAsync();

        int sequence = 1;
        if (lastOrder != null)
        {
            var lastSequence = lastOrder.OrderNumber.Substring(prefix.Length);
            if (int.TryParse(lastSequence, out int lastNum))
            {
                sequence = lastNum + 1;
            }
        }

        return $"{prefix}{sequence:D4}";
    }
}
```

**EShop.Infrastructure/Repositories/UnitOfWork.cs:**
```csharp
using EShop.Core.Interfaces;
using EShop.Infrastructure.Data;
using EShop.Infrastructure.Repositories;

namespace EShop.Infrastructure.Repositories;

public class UnitOfWork : IUnitOfWork
{
    private readonly EShopDbContext _context;
    private IProductRepository? _products;
    private IOrderRepository? _orders;

    public UnitOfWork(EShopDbContext context)
    {
        _context = context;
    }

    public IProductRepository Products
    {
        get
        {
            _products ??= new ProductRepository(_context);
            return _products;
        }
    }

    public IOrderRepository Orders
    {
        get
        {
            _orders ??= new OrderRepository(_context);
            return _orders;
        }
    }

    public async Task<int> SaveChangesAsync()
    {
        return await _context.SaveChangesAsync();
    }

    public void Dispose()
    {
        _context.Dispose();
    }
}
```

---

## BƯỚC 4: Xây dựng Application Layer (EShop.Application)

### 4.1. Tạo DTOs

**EShop.Application/DTOs/ProductDto.cs:**
```csharp
namespace EShop.Application.DTOs;

public class ProductDto
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string? Description { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
    public string? SKU { get; set; }
    public string? ImageUrl { get; set; }
    public int CategoryId { get; set; }
    public string CategoryName { get; set; } = string.Empty;
    public bool IsActive { get; set; }
}
```

**EShop.Application/DTOs/CartItemDto.cs:**
```csharp
namespace EShop.Application.DTOs;

public class CartItemDto
{
    public int ProductId { get; set; }
    public string ProductName { get; set; } = string.Empty;
    public string? ImageUrl { get; set; }
    public decimal Price { get; set; }
    public int Quantity { get; set; }
    public decimal Total => Price * Quantity;
}
```

**EShop.Application/DTOs/OrderDto.cs:**
```csharp
using EShop.Core.Enums;

namespace EShop.Application.DTOs;

public class OrderDto
{
    public int Id { get; set; }
    public string OrderNumber { get; set; } = string.Empty;
    public DateTime OrderDate { get; set; }
    public OrderStatus Status { get; set; }
    public decimal Total { get; set; }
    public string ShippingName { get; set; } = string.Empty;
    public List<OrderItemDto> OrderItems { get; set; } = new();
}

public class OrderItemDto
{
    public int ProductId { get; set; }
    public string ProductName { get; set; } = string.Empty;
    public int Quantity { get; set; }
    public decimal Price { get; set; }
    public decimal Total => Quantity * Price;
}
```

### 4.2. Tạo Services

**EShop.Application/Services/IProductService.cs:**
```csharp
using EShop.Application.DTOs;

namespace EShop.Application.Services;

public interface IProductService
{
    Task<IEnumerable<ProductDto>> GetAllAsync();
    Task<ProductDto?> GetByIdAsync(int id);
    Task<IEnumerable<ProductDto>> GetByCategoryIdAsync(int categoryId);
    Task<IEnumerable<ProductDto>> SearchAsync(string keyword);
    Task<ProductDto> CreateAsync(CreateProductDto dto);
    Task<ProductDto> UpdateAsync(int id, UpdateProductDto dto);
    Task DeleteAsync(int id);
}

public class CreateProductDto
{
    public string Name { get; set; } = string.Empty;
    public string? Description { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
    public string? SKU { get; set; }
    public int CategoryId { get; set; }
}

public class UpdateProductDto
{
    public string Name { get; set; } = string.Empty;
    public string? Description { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
    public int CategoryId { get; set; }
}
```

**EShop.Application/Services/ProductService.cs:**
```csharp
using AutoMapper;
using EShop.Application.DTOs;
using EShop.Core.Entities;
using EShop.Core.Interfaces;

namespace EShop.Application.Services;

public class ProductService : IProductService
{
    private readonly IUnitOfWork _unitOfWork;
    private readonly IMapper _mapper;

    public ProductService(IUnitOfWork unitOfWork, IMapper mapper)
    {
        _unitOfWork = unitOfWork;
        _mapper = mapper;
    }

    public async Task<IEnumerable<ProductDto>> GetAllAsync()
    {
        var products = await _unitOfWork.Products.GetAllAsync();
        return _mapper.Map<IEnumerable<ProductDto>>(products);
    }

    public async Task<ProductDto?> GetByIdAsync(int id)
    {
        var product = await _unitOfWork.Products.GetByIdAsync(id);
        return product == null ? null : _mapper.Map<ProductDto>(product);
    }

    public async Task<IEnumerable<ProductDto>> GetByCategoryIdAsync(int categoryId)
    {
        var products = await _unitOfWork.Products.GetByCategoryIdAsync(categoryId);
        return _mapper.Map<IEnumerable<ProductDto>>(products);
    }

    public async Task<IEnumerable<ProductDto>> SearchAsync(string keyword)
    {
        var products = await _unitOfWork.Products.SearchAsync(keyword);
        return _mapper.Map<IEnumerable<ProductDto>>(products);
    }

    public async Task<ProductDto> CreateAsync(CreateProductDto dto)
    {
        var product = _mapper.Map<Product>(dto);
        product.CreatedAt = DateTime.UtcNow;
        
        var created = await _unitOfWork.Products.AddAsync(product);
        return _mapper.Map<ProductDto>(created);
    }

    public async Task<ProductDto> UpdateAsync(int id, UpdateProductDto dto)
    {
        var product = await _unitOfWork.Products.GetByIdAsync(id);
        if (product == null)
            throw new Exception("Product not found");

        product.Name = dto.Name;
        product.Description = dto.Description;
        product.Price = dto.Price;
        product.Stock = dto.Stock;
        product.CategoryId = dto.CategoryId;
        product.UpdatedAt = DateTime.UtcNow;

        await _unitOfWork.Products.UpdateAsync(product);
        return _mapper.Map<ProductDto>(product);
    }

    public async Task DeleteAsync(int id)
    {
        await _unitOfWork.Products.DeleteAsync(id);
    }
}
```

### 4.3. Cấu hình AutoMapper

**EShop.Application/Mappings/MappingProfile.cs:**
```csharp
using AutoMapper;
using EShop.Application.DTOs;
using EShop.Core.Entities;
using EShop.Application.Services;

namespace EShop.Application.Mappings;

public class MappingProfile : Profile
{
    public MappingProfile()
    {
        // Product Mappings
        CreateMap<Product, ProductDto>()
            .ForMember(dest => dest.CategoryName, opt => opt.MapFrom(src => src.Category.Name));

        CreateMap<CreateProductDto, Product>();
        CreateMap<UpdateProductDto, Product>();

        // Order Mappings
        CreateMap<Order, OrderDto>();
        CreateMap<OrderItem, OrderItemDto>()
            .ForMember(dest => dest.ProductName, opt => opt.MapFrom(src => src.Product.Name));
    }
}
```

---

## BƯỚC 5: Cấu hình EShop.Web

### 5.1. Cấu hình Program.cs

**EShop.Web/Program.cs:**
```csharp
using EShop.Core.Entities;
using EShop.Core.Interfaces;
using EShop.Infrastructure.Data;
using EShop.Infrastructure.Repositories;
using EShop.Application.Services;
using EShop.Application.Mappings;
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllersWithViews();

// Database
builder.Services.AddDbContext<EShopDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Identity
builder.Services.AddIdentity<AppUser, IdentityRole>(options =>
{
    options.Password.RequireDigit = false;
    options.Password.RequireLowercase = false;
    options.Password.RequireUppercase = false;
    options.Password.RequireNonAlphanumeric = false;
    options.Password.RequiredLength = 6;
})
.AddEntityFrameworkStores<EShopDbContext>()
.AddDefaultTokenProviders();

// Repositories & Services
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
builder.Services.AddScoped<IProductService, ProductService>();

// AutoMapper
builder.Services.AddAutoMapper(typeof(MappingProfile));

// Session (for Shopping Cart)
builder.Services.AddDistributedMemoryCache();
builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(30);
    options.Cookie.HttpOnly = true;
    options.Cookie.IsEssential = true;
});

var app = builder.Build();

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();

app.UseAuthentication();
app.UseAuthorization();

app.UseSession();

app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

### 5.2. Cấu hình appsettings.json

**EShop.Web/appsettings.json:**
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=EShopDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

---

## BƯỚC 6: Tạo Database Migration

```powershell
cd EShop.Web
dotnet ef migrations add InitialCreate --project ../EShop.Infrastructure --startup-project .
dotnet ef database update --project ../EShop.Infrastructure --startup-project .
```

---

# 7. GIAO DIỆN NGƯỜI DÙNG (FRONT STORE)

## 7.1. Tạo Area Store

```powershell
cd EShop.Web
dotnet new area -n Store
```

## 7.2. Controller: HomeController (Trang chủ)

**Areas/Store/Controllers/HomeController.cs:**
```csharp
using EShop.Application.Services;
using Microsoft.AspNetCore.Mvc;

namespace EShop.Web.Areas.Store.Controllers;

[Area("Store")]
public class HomeController : Controller
{
    private readonly IProductService _productService;

    public HomeController(IProductService productService)
    {
        _productService = productService;
    }

    public async Task<IActionResult> Index()
    {
        // Lấy sản phẩm nổi bật (ví dụ: 8 sản phẩm đầu tiên)
        var featuredProducts = (await _productService.GetAllAsync())
            .Take(8)
            .ToList();

        ViewBag.FeaturedProducts = featuredProducts;
        return View();
    }
}
```

**Areas/Store/Views/Home/Index.cshtml:**
```html
@{
    ViewData["Title"] = "Trang chủ";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<!-- Hero Section -->
<section class="hero-section bg-primary text-white py-5">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-md-6">
                <h1 class="display-4 fw-bold">Chào mừng đến với E-Shop</h1>
                <p class="lead">Nơi mua sắm trực tuyến tốt nhất</p>
                <a asp-area="Store" asp-controller="Product" asp-action="Index" 
                   class="btn btn-light btn-lg">Mua ngay</a>
            </div>
            <div class="col-md-6">
                <img src="~/images/hero.jpg" class="img-fluid" alt="Hero" />
            </div>
        </div>
    </div>
</section>

<!-- Featured Products -->
<section class="featured-products py-5">
    <div class="container">
        <h2 class="text-center mb-5">Sản phẩm nổi bật</h2>
        <div class="row">
            @foreach (var product in ViewBag.FeaturedProducts as List<EShop.Application.DTOs.ProductDto>)
            {
                <div class="col-md-3 mb-4">
                    <div class="card h-100">
                        <img src="@(product.ImageUrl ?? "/images/placeholder.jpg")" 
                             class="card-img-top" alt="@product.Name" style="height: 200px; object-fit: cover;">
                        <div class="card-body d-flex flex-column">
                            <h5 class="card-title">@product.Name</h5>
                            <p class="card-text text-primary fw-bold">@product.Price.ToString("N0") đ</p>
                            <a asp-area="Store" asp-controller="Product" asp-action="Details" asp-route-id="@product.Id" 
                               class="btn btn-primary mt-auto">Xem chi tiết</a>
                        </div>
                    </div>
                </div>
            }
        </div>
    </div>
</section>
```

## 7.3. Controller: ProductController (Sản phẩm)

**Areas/Store/Controllers/ProductController.cs:**
```csharp
using EShop.Application.Services;
using Microsoft.AspNetCore.Mvc;

namespace EShop.Web.Areas.Store.Controllers;

[Area("Store")]
public class ProductController : Controller
{
    private readonly IProductService _productService;
    private const int PageSize = 12;

    public ProductController(IProductService productService)
    {
        _productService = productService;
    }

    public async Task<IActionResult> Index(int? categoryId, string? search, int page = 1)
    {
        IEnumerable<EShop.Application.DTOs.ProductDto> products;

        if (!string.IsNullOrEmpty(search))
        {
            products = await _productService.SearchAsync(search);
        }
        else if (categoryId.HasValue)
        {
            products = await _productService.GetByCategoryIdAsync(categoryId.Value);
        }
        else
        {
            products = await _productService.GetAllAsync();
        }

        var totalItems = products.Count();
        var totalPages = (int)Math.Ceiling(totalItems / (double)PageSize);
        var pagedProducts = products.Skip((page - 1) * PageSize).Take(PageSize).ToList();

        ViewBag.CurrentPage = page;
        ViewBag.TotalPages = totalPages;
        ViewBag.CategoryId = categoryId;
        ViewBag.Search = search;

        return View(pagedProducts);
    }

    public async Task<IActionResult> Details(int id)
    {
        var product = await _productService.GetByIdAsync(id);
        if (product == null)
        {
            return NotFound();
        }

        // Lấy sản phẩm liên quan (cùng danh mục)
        var relatedProducts = (await _productService.GetByCategoryIdAsync(product.CategoryId))
            .Where(p => p.Id != id)
            .Take(4)
            .ToList();

        ViewBag.RelatedProducts = relatedProducts;
        return View(product);
    }
}
```

**Areas/Store/Views/Product/Index.cshtml:**
```html
@model List<EShop.Application.DTOs.ProductDto>
@{
    ViewData["Title"] = "Sản phẩm";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<div class="container my-5">
    <h2 class="mb-4">Danh sách sản phẩm</h2>

    <!-- Search & Filter -->
    <div class="row mb-4">
        <div class="col-md-6">
            <form method="get" asp-action="Index">
                <div class="input-group">
                    <input type="text" name="search" class="form-control" 
                           placeholder="Tìm kiếm sản phẩm..." value="@ViewBag.Search">
                    <button class="btn btn-primary" type="submit">Tìm kiếm</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Product Grid -->
    <div class="row">
        @foreach (var product in Model)
        {
            <div class="col-md-3 mb-4">
                <div class="card h-100">
                    <img src="@(product.ImageUrl ?? "/images/placeholder.jpg")" 
                         class="card-img-top" alt="@product.Name" style="height: 200px; object-fit: cover;">
                    <div class="card-body d-flex flex-column">
                        <h5 class="card-title">@product.Name</h5>
                        <p class="card-text text-primary fw-bold">@product.Price.ToString("N0") đ</p>
                        <a asp-action="Details" asp-route-id="@product.Id" 
                           class="btn btn-primary mt-auto">Xem chi tiết</a>
                    </div>
                </div>
            </div>
        }
    </div>

    <!-- Pagination -->
    @if (ViewBag.TotalPages > 1)
    {
        <nav>
            <ul class="pagination justify-content-center">
                @for (int i = 1; i <= ViewBag.TotalPages; i++)
                {
                    <li class="page-item @(i == ViewBag.CurrentPage ? "active" : "")">
                        <a class="page-link" asp-action="Index" 
                           asp-route-page="@i" 
                           asp-route-categoryId="@ViewBag.CategoryId"
                           asp-route-search="@ViewBag.Search">@i</a>
                    </li>
                }
            </ul>
        </nav>
    }
</div>
```

**Areas/Store/Views/Product/Details.cshtml:**
```html
@model EShop.Application.DTOs.ProductDto
@{
    ViewData["Title"] = Model.Name;
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<div class="container my-5">
    <div class="row">
        <div class="col-md-6">
            <img src="@(Model.ImageUrl ?? "/images/placeholder.jpg")" 
                 class="img-fluid" alt="@Model.Name">
        </div>
        <div class="col-md-6">
            <h1>@Model.Name</h1>
            <p class="text-primary fs-3 fw-bold">@Model.Price.ToString("N0") đ</p>
            <p>@Model.Description</p>
            <p><strong>Danh mục:</strong> @Model.CategoryName</p>
            <p><strong>Tồn kho:</strong> @Model.Stock</p>

            <form asp-area="Store" asp-controller="Cart" asp-action="Add" method="post">
                <input type="hidden" name="productId" value="@Model.Id">
                <div class="mb-3">
                    <label>Số lượng:</label>
                    <input type="number" name="quantity" value="1" min="1" max="@Model.Stock" class="form-control" style="width: 100px;">
                </div>
                <button type="submit" class="btn btn-primary btn-lg">Thêm vào giỏ hàng</button>
            </form>
        </div>
    </div>

    <!-- Related Products -->
    @if (ViewBag.RelatedProducts != null)
    {
        <div class="mt-5">
            <h3>Sản phẩm liên quan</h3>
            <div class="row">
                @foreach (var product in ViewBag.RelatedProducts as List<EShop.Application.DTOs.ProductDto>)
                {
                    <div class="col-md-3">
                        <div class="card">
                            <img src="@(product.ImageUrl ?? "/images/placeholder.jpg")" 
                                 class="card-img-top" alt="@product.Name">
                            <div class="card-body">
                                <h5 class="card-title">@product.Name</h5>
                                <p class="text-primary">@product.Price.ToString("N0") đ</p>
                                <a asp-action="Details" asp-route-id="@product.Id" class="btn btn-sm btn-primary">Xem</a>
                            </div>
                        </div>
                    </div>
                }
            </div>
        </div>
    }
</div>
```

## 7.4. Controller: CartController (Giỏ hàng)

**Areas/Store/Controllers/CartController.cs:**
```csharp
using EShop.Application.DTOs;
using EShop.Application.Services;
using Microsoft.AspNetCore.Mvc;
using System.Text.Json;

namespace EShop.Web.Areas.Store.Controllers;

[Area("Store")]
public class CartController : Controller
{
    private readonly IProductService _productService;
    private const string CartSessionKey = "Cart";

    public CartController(IProductService productService)
    {
        _productService = productService;
    }

    private List<CartItemDto> GetCart()
    {
        var cartJson = HttpContext.Session.GetString(CartSessionKey);
        return cartJson == null 
            ? new List<CartItemDto>() 
            : JsonSerializer.Deserialize<List<CartItemDto>>(cartJson) ?? new List<CartItemDto>();
    }

    private void SaveCart(List<CartItemDto> cart)
    {
        var cartJson = JsonSerializer.Serialize(cart);
        HttpContext.Session.SetString(CartSessionKey, cartJson);
    }

    [HttpPost]
    public async Task<IActionResult> Add(int productId, int quantity = 1)
    {
        var product = await _productService.GetByIdAsync(productId);
        if (product == null)
        {
            return NotFound();
        }

        var cart = GetCart();
        var existingItem = cart.FirstOrDefault(c => c.ProductId == productId);

        if (existingItem != null)
        {
            existingItem.Quantity += quantity;
        }
        else
        {
            cart.Add(new CartItemDto
            {
                ProductId = product.Id,
                ProductName = product.Name,
                ImageUrl = product.ImageUrl,
                Price = product.Price,
                Quantity = quantity
            });
        }

        SaveCart(cart);
        TempData["Success"] = "Đã thêm sản phẩm vào giỏ hàng!";
        return RedirectToAction("Index");
    }

    public IActionResult Index()
    {
        var cart = GetCart();
        ViewBag.Total = cart.Sum(c => c.Total);
        return View(cart);
    }

    [HttpPost]
    public IActionResult Update(int productId, int quantity)
    {
        var cart = GetCart();
        var item = cart.FirstOrDefault(c => c.ProductId == productId);
        
        if (item != null)
        {
            if (quantity <= 0)
            {
                cart.Remove(item);
            }
            else
            {
                item.Quantity = quantity;
            }
            SaveCart(cart);
        }

        return RedirectToAction("Index");
    }

    [HttpPost]
    public IActionResult Remove(int productId)
    {
        var cart = GetCart();
        var item = cart.FirstOrDefault(c => c.ProductId == productId);
        if (item != null)
        {
            cart.Remove(item);
            SaveCart(cart);
        }

        return RedirectToAction("Index");
    }
}
```

**Areas/Store/Views/Cart/Index.cshtml:**
```html
@model List<EShop.Application.DTOs.CartItemDto>
@{
    ViewData["Title"] = "Giỏ hàng";
    Layout = "~/Views/Shared/_Layout.cshtml";
    var total = Model.Sum(c => c.Total);
}

<div class="container my-5">
    <h2>Giỏ hàng của bạn</h2>

    @if (Model.Count == 0)
    {
        <p class="text-muted">Giỏ hàng của bạn đang trống.</p>
        <a asp-area="Store" asp-controller="Product" asp-action="Index" class="btn btn-primary">Tiếp tục mua sắm</a>
    }
    else
    {
        <table class="table">
            <thead>
                <tr>
                    <th>Hình ảnh</th>
                    <th>Sản phẩm</th>
                    <th>Giá</th>
                    <th>Số lượng</th>
                    <th>Tổng</th>
                    <th></th>
                </tr>
            </thead>
            <tbody>
                @foreach (var item in Model)
                {
                    <tr>
                        <td>
                            <img src="@(item.ImageUrl ?? "/images/placeholder.jpg")" 
                                 style="width: 80px; height: 80px; object-fit: cover;">
                        </td>
                        <td>@item.ProductName</td>
                        <td>@item.Price.ToString("N0") đ</td>
                        <td>
                            <form method="post" asp-action="Update">
                                <input type="hidden" name="productId" value="@item.ProductId">
                                <input type="number" name="quantity" value="@item.Quantity" min="1" 
                                       class="form-control" style="width: 80px; display: inline-block;">
                                <button type="submit" class="btn btn-sm btn-primary">Cập nhật</button>
                            </form>
                        </td>
                        <td>@item.Total.ToString("N0") đ</td>
                        <td>
                            <form method="post" asp-action="Remove">
                                <input type="hidden" name="productId" value="@item.ProductId">
                                <button type="submit" class="btn btn-sm btn-danger">Xóa</button>
                            </form>
                        </td>
                    </tr>
                }
            </tbody>
            <tfoot>
                <tr>
                    <td colspan="4" class="text-end"><strong>Tổng cộng:</strong></td>
                    <td><strong>@total.ToString("N0") đ</strong></td>
                    <td></td>
                </tr>
            </tfoot>
        </table>

        <div class="text-end">
            <a asp-area="Store" asp-controller="Product" asp-action="Index" class="btn btn-secondary">Tiếp tục mua sắm</a>
            <a asp-area="Store" asp-controller="Order" asp-action="Checkout" class="btn btn-primary">Thanh toán</a>
        </div>
    }
</div>
```

---

# 8. GIAO DIỆN QUẢN TRỊ (ADMIN PANEL)

## 8.1. Tạo Area Admin

```powershell
dotnet new area -n Admin
```

## 8.2. Controller: Admin ProductController

**Areas/Admin/Controllers/ProductController.cs:**
```csharp
using EShop.Application.Services;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace EShop.Web.Areas.Admin.Controllers;

[Area("Admin")]
[Authorize(Roles = "Admin")]
public class ProductController : Controller
{
    private readonly IProductService _productService;

    public ProductController(IProductService productService)
    {
        _productService = productService;
    }

    public async Task<IActionResult> Index()
    {
        var products = await _productService.GetAllAsync();
        return View(products);
    }

    public IActionResult Create()
    {
        return View();
    }

    [HttpPost]
    public async Task<IActionResult> Create(CreateProductDto dto, IFormFile? image)
    {
        if (ModelState.IsValid)
        {
            // Upload image
            if (image != null)
            {
                var fileName = Guid.NewGuid().ToString() + Path.GetExtension(image.FileName);
                var filePath = Path.Combine("wwwroot", "images", "products", fileName);
                Directory.CreateDirectory(Path.GetDirectoryName(filePath)!);
                using (var stream = new FileStream(filePath, FileMode.Create))
                {
                    await image.CopyToAsync(stream);
                }
                dto.ImageUrl = $"/images/products/{fileName}";
            }

            await _productService.CreateAsync(dto);
            TempData["Success"] = "Thêm sản phẩm thành công!";
            return RedirectToAction("Index");
        }

        return View(dto);
    }

    // Edit, Delete tương tự...
}
```

---

# 9. TESTING & DEPLOYMENT

## 9.1. Testing Checklist

### Functional Testing:
- [ ] Đăng ký/Đăng nhập
- [ ] Xem danh sách sản phẩm
- [ ] Tìm kiếm sản phẩm
- [ ] Thêm vào giỏ hàng
- [ ] Thanh toán
- [ ] Quản lý sản phẩm (Admin)
- [ ] Quản lý đơn hàng (Admin)

### Performance Testing:
- [ ] Load time < 3s
- [ ] Database queries optimized
- [ ] Images optimized

## 9.2. Deployment

### Deploy lên IIS:
1. Publish project: `dotnet publish -c Release`
2. Copy files lên IIS server
3. Cấu hình Connection String
4. Chạy migrations: `dotnet ef database update`

---

# 10. TÀI LIỆU BÀN GIAO

## 10.1. Checklist bàn giao

- [ ] Source code đầy đủ
- [ ] Database script hoặc backup
- [ ] Hướng dẫn cài đặt
- [ ] Hướng dẫn sử dụng
- [ ] API Documentation (nếu có)
- [ ] Video demo

## 10.2. Template báo cáo

**Cấu trúc báo cáo:**
1. Giới thiệu dự án
2. Phân tích yêu cầu
3. Thiết kế hệ thống
4. Công nghệ sử dụng
5. Hướng dẫn cài đặt
6. Hướng dẫn sử dụng
7. Kết quả đạt được
8. Kết luận

---

**Tài liệu này cung cấp hướng dẫn đầy đủ từng bước để xây dựng E-Shop MVC Full Stack. Hãy làm theo từng bước một cách cẩn thận!** 🚀
