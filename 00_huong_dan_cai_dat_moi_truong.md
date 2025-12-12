# 🛠️ HƯỚNG DẪN CÀI ĐẶT MÔI TRƯỜNG PHÁT TRIỂN

**Môn học:** Thiết kế & Lập trình Back-end với ASP.NET Core  
**IDE:** JetBrains Rider (thay vì Visual Studio)  
**Cập nhật:** Tháng 12/2025

---

## 📑 MỤC LỤC

1. [Tổng quan môi trường](#tổng-quan-môi-trường)
2. [Yêu cầu hệ thống](#yêu-cầu-hệ-thống)
3. [Bước 1: Cài đặt .NET SDK](#bước-1-cài-đặt-net-sdk)
4. [Bước 2: Cài đặt JetBrains Rider](#bước-2-cài-đặt-jetbrains-rider)
5. [Bước 3: Cài đặt Database](#bước-3-cài-đặt-database)
6. [Bước 4: Cài đặt công cụ hỗ trợ](#bước-4-cài-đặt-công-cụ-hỗ-trợ)
7. [Bước 5: Cấu hình Rider](#bước-5-cấu-hình-rider)
8. [Bước 6: Kiểm tra cài đặt](#bước-6-kiểm-tra-cài-đặt)
9. [Troubleshooting](#troubleshooting)
10. [So sánh Rider vs Visual Studio](#so-sánh-rider-vs-visual-studio)

---

## 🎯 TỔNG QUAN MÔI TRƯỜNG

### Stack công nghệ cần cài đặt

```
┌─────────────────────────────────────────────┐
│  DEVELOPMENT ENVIRONMENT                    │
├─────────────────────────────────────────────┤
│  1. .NET SDK 8.0 (LTS)                     │
│  2. JetBrains Rider 2024.3                 │
│  3. SQL Server 2022 / PostgreSQL           │
│  4. Git & GitHub Desktop                   │
│  5. Postman / Insomnia                     │
│  6. Docker Desktop (Optional)              │
└─────────────────────────────────────────────┘
```

### Tại sao chọn Rider thay vì Visual Studio?

| Tiêu chí | JetBrains Rider | Visual Studio |
|----------|-----------------|---------------|
| **Cross-platform** | ✅ Win, Mac, Linux | ⚠️ Chỉ Windows (Mac khác) |
| **Performance** | ⚡ Nhẹ hơn, nhanh hơn | 🐌 Nặng, khởi động chậm |
| **Price** | 💰 Trả phí (Free cho SV) | 🆓 Community miễn phí |
| **Database tools** | ✅ Built-in DataGrip | ⚠️ Cần cài thêm |
| **Code analysis** | ✅ ReSharper integrated | ⚠️ Cần mua ReSharper |
| **Git integration** | ✅ Xuất sắc | ✅ Tốt |
| **Keyboard shortcuts** | ✅ Nhất quán JetBrains | ⚠️ Khác biệt |

**Kết luận:** Rider phù hợp hơn cho:
- Sinh viên có laptop yếu/trung bình
- Người dùng Mac/Linux
- Muốn học công cụ chuyên nghiệp
- Có license miễn phí (email .edu)

---

## 💻 YÊU CẦU HỆ THỐNG

### Cấu hình tối thiểu

```
CPU:    Intel Core i3 hoặc AMD Ryzen 3 (đời 8 trở lên)
RAM:    8 GB (khuyến nghị 16 GB)
HDD:    20 GB trống (khuyến nghị SSD)
OS:     Windows 10/11, macOS 11+, Linux (Ubuntu 20.04+)
```

### Cấu hình khuyến nghị

```
CPU:    Intel Core i5 / AMD Ryzen 5 trở lên
RAM:    16 GB hoặc 32 GB
SSD:    256 GB trở lên (NVMe càng tốt)
OS:     Windows 11, macOS Sonoma, Ubuntu 22.04 LTS
```

---

## 📥 BƯỚC 1: CÀI ĐẶT .NET SDK

### 1.1. Download .NET SDK

**Trang chính thức:** https://dotnet.microsoft.com/download

Chọn phiên bản **`.NET 8.0 (LTS)` - Long Term Support**

```
┌─────────────────────────────────────────┐
│  Khuyến nghị: .NET 8.0 (LTS)           │
│  - Hỗ trợ đến 11/2026                  │
│  - Ổn định, nhiều tài liệu             │
│  - Tương thích tốt với Rider           │
└─────────────────────────────────────────┘
```

### 1.2. Cài đặt theo hệ điều hành

#### 🪟 Windows

1. Download file: `dotnet-sdk-8.0.xxx-win-x64.exe`
2. Chạy file cài đặt
3. Chọn **Install**
4. Đợi quá trình cài đặt hoàn tất (3-5 phút)

#### 🍎 macOS

**Cách 1: Dùng Installer**
```bash
# Download file .pkg từ website
# Double-click và làm theo hướng dẫn
```

**Cách 2: Dùng Homebrew (khuyến nghị)**
```bash
# Cài Homebrew (nếu chưa có)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Cài .NET SDK
brew install --cask dotnet-sdk
```

#### 🐧 Linux (Ubuntu/Debian)

```bash
# Thêm Microsoft package repository
wget https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb

# Cài đặt .NET SDK
sudo apt-get update
sudo apt-get install -y dotnet-sdk-8.0
```

### 1.3. Kiểm tra cài đặt

Mở **Terminal** (hoặc Command Prompt trên Windows):

```bash
# Kiểm tra phiên bản
dotnet --version
```

**Kết quả mong đợi:**
```
8.0.xxx
```

```bash
# Xem thông tin chi tiết
dotnet --info
```

**Kết quả mong đợi:**
```
.NET SDK:
 Version:   8.0.100
 Commit:    xxxxx

Runtime Environment:
 OS Name:     Windows
 OS Version:  10.0.22631
 OS Platform: Windows
 RID:         win-x64
```

### 1.4. Test tạo project đầu tiên

```bash
# Tạo thư mục test
mkdir test-dotnet
cd test-dotnet

# Tạo console app
dotnet new console -n HelloDotnet
cd HelloDotnet

# Chạy thử
dotnet run
```

**Kết quả mong đợi:**
```
Hello, World!
```

✅ **Xong bước 1!** .NET SDK đã sẵn sàng.

---

## 🚀 BƯỚC 2: CÀI ĐẶT JETBRAINS RIDER

### 2.1. Đăng ký tài khoản JetBrains (Sinh viên miễn phí)

1. Truy cập: https://www.jetbrains.com/shop/eform/students
2. Chọn **"Apply now"**
3. Điền thông tin:
   - **Email sinh viên** (xxx@duytan.edu.vn hoặc xxx@student.duytan.edu.vn)
   - Họ tên, trường, ngành học
4. Xác nhận email
5. Nhận **JetBrains Educational License** (miễn phí 1 năm, gia hạn được)

**Lưu ý:** 
- ✅ Phải dùng email trường (.edu.vn)
- ✅ License bao gồm TẤT CẢ sản phẩm JetBrains (Rider, IntelliJ IDEA, WebStorm, PyCharm, ...)
- ⚠️ Nếu không có email trường, có thể dùng **Trial 30 ngày** hoặc **Free Community Edition** (giới hạn tính năng)

### 2.2. Download Rider

**Trang chính thức:** https://www.jetbrains.com/rider/download/

Chọn hệ điều hành của bạn:
- Windows: `JetBrains.Rider-2024.3.exe`
- macOS: `JetBrains.Rider-2024.3.dmg` (Apple Silicon hoặc Intel)
- Linux: `JetBrains.Rider-2024.3.tar.gz`

### 2.3. Cài đặt Rider

#### 🪟 Windows

1. Chạy file `.exe`
2. Chọn **Next**
3. Chọn thư mục cài đặt (mặc định: `C:\Program Files\JetBrains\Rider`)
4. **Tích chọn:**
   - ✅ Create Desktop Shortcut
   - ✅ Add "Open Folder as Project"
   - ✅ Add launchers dir to the PATH
   - ✅ `.cs` - Open files with .cs extension
   - ✅ `.csproj` - Open project files
5. Click **Install**
6. Đợi cài đặt (2-3 phút)
7. Click **Finish**

#### 🍎 macOS

1. Mở file `.dmg`
2. Kéo **Rider.app** vào thư mục **Applications**
3. Mở **Launchpad** → tìm **Rider**
4. Click chuột phải → **Open** (lần đầu cần xác nhận)

#### 🐧 Linux

```bash
# Giải nén
sudo tar -xzf JetBrains.Rider-2024.3.tar.gz -C /opt/

# Chạy Rider
cd /opt/JetBrains-Rider-2024.3/bin
./rider.sh
```

### 2.4. Khởi động và kích hoạt Rider lần đầu

1. **Mở Rider** lần đầu
2. **Import Settings**: Chọn **Do not import settings** (nếu mới dùng)
3. **Privacy Policy**: Đọc và **Accept**
4. **Activate Rider**:
   - Chọn **Log in to JetBrains Account**
   - Đăng nhập bằng tài khoản JetBrains đã đăng ký
   - Educational License sẽ tự động kích hoạt

### 2.5. Cấu hình ban đầu

#### Chọn Theme
```
┌─────────────────────────────────┐
│  Light Theme:  IntelliJ Light   │
│  Dark Theme:   Darcula          │
└─────────────────────────────────┘
```
**Khuyến nghị:** Chọn theo sở thích, theme không ảnh hưởng hiệu suất.

#### Cài đặt Plugins cần thiết

Rider sẽ hỏi cài plugins, chọn:
- ✅ **.NET Core** (đã có sẵn)
- ✅ **ASP.NET Core** (đã có sẵn)
- ✅ **Entity Framework Core** (đã có sẵn)
- ✅ **Markdown** (để xem .md files)
- ✅ **Git** (đã có sẵn)
- ⚠️ **GitHub Copilot** (optional, trả phí hoặc free cho SV)

### 2.6. Kết nối Rider với .NET SDK

Rider sẽ **tự động phát hiện** .NET SDK đã cài.

Kiểm tra:
1. Mở Rider
2. **File** → **Settings** (Windows/Linux) hoặc **Preferences** (macOS)
3. **Build, Execution, Deployment** → **.NET**
4. **SDK versions**: Phải thấy `.NET 8.0.xxx`

```
✅ .NET 8.0.100 (C:\Program Files\dotnet\sdk\8.0.100)
```

Nếu không thấy:
- Click **Add** → Chọn thư mục cài .NET SDK
- Windows: `C:\Program Files\dotnet\sdk\8.0.xxx`
- macOS/Linux: `/usr/local/share/dotnet/sdk/8.0.xxx`

---

## 🗄️ BƯỚC 3: CÀI ĐẶT DATABASE

Có **3 lựa chọn** phổ biến:

### Lựa chọn 1: SQL Server (Khuyến nghị cho Windows)

#### 3.1. Download SQL Server

**Trang chính thức:** https://www.microsoft.com/sql-server/sql-server-downloads

Chọn **SQL Server 2022 Developer Edition** (Miễn phí cho học tập)

#### 3.2. Cài đặt SQL Server

1. Chạy file cài đặt
2. Chọn **Basic Installation**
3. Chọn thư mục cài đặt (mặc định: `C:\Program Files\Microsoft SQL Server`)
4. Click **Install**
5. Đợi cài đặt (10-15 phút)
6. Lưu lại **Connection String** hiển thị sau khi cài:

```
Server=localhost;Database=master;Trusted_Connection=True;
```

#### 3.3. Cài đặt SQL Server Management Studio (SSMS)

**Download:** https://aka.ms/ssmsfullsetup

1. Chạy file `SSMS-Setup-ENU.exe`
2. Click **Install**
3. Đợi cài đặt (5-10 phút)
4. Mở SSMS và kết nối:
   - **Server name:** `localhost` hoặc `(localdb)\MSSQLLocalDB`
   - **Authentication:** Windows Authentication
   - Click **Connect**

✅ **Done!** SQL Server đã sẵn sàng.

---

### Lựa chọn 2: PostgreSQL (Cross-platform, khuyến nghị cho Mac/Linux)

#### 3.1. Download PostgreSQL

**Trang chính thức:** https://www.postgresql.org/download/

Chọn phiên bản **PostgreSQL 16**

#### 3.2. Cài đặt PostgreSQL

##### 🪟 Windows
1. Download file `.exe`
2. Chạy installer
3. Chọn components:
   - ✅ PostgreSQL Server
   - ✅ pgAdmin 4 (GUI tool)
   - ✅ Command Line Tools
4. Chọn thư mục cài đặt
5. Đặt **password** cho user `postgres` (ghi nhớ password này!)
6. Port mặc định: `5432`
7. Click **Next** → **Install**

##### 🍎 macOS
```bash
# Cài qua Homebrew
brew install postgresql@16

# Khởi động service
brew services start postgresql@16

# Tạo database đầu tiên
createdb mydb
```

##### 🐧 Linux (Ubuntu)
```bash
# Cài đặt
sudo apt update
sudo apt install postgresql postgresql-contrib

# Khởi động service
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Chuyển sang user postgres
sudo -u postgres psql
```

#### 3.3. Cài đặt pgAdmin (GUI)

**Download:** https://www.pgadmin.org/download/

1. Cài đặt pgAdmin
2. Mở pgAdmin
3. Kết nối đến server:
   - Host: `localhost`
   - Port: `5432`
   - Username: `postgres`
   - Password: (password bạn đã đặt)

---

### Lựa chọn 3: LocalDB (Nhẹ nhất, chỉ Windows)

**LocalDB** là phiên bản rút gọn của SQL Server, tự động cài cùng Visual Studio hoặc .NET SDK.

#### Kiểm tra LocalDB đã có chưa:

```bash
sqllocaldb info
```

Nếu chưa có, download: https://aka.ms/ssdt

**Connection String:**
```
Server=(localdb)\\mssqllocaldb;Database=MyDb;Trusted_Connection=True;
```

✅ **Đơn giản nhất** cho người mới bắt đầu.

---

### 🎯 Khuyến nghị cho sinh viên

| Hệ điều hành | Database khuyến nghị | Lý do |
|--------------|----------------------|-------|
| **Windows** | SQL Server Developer / LocalDB | Native, tài liệu nhiều, SSMS trực quan |
| **macOS** | PostgreSQL | Cross-platform, performance tốt |
| **Linux** | PostgreSQL | Tích hợp tốt, mã nguồn mở |

---

## 🔧 BƯỚC 4: CÀI ĐẶT CÔNG CỤ HỖ TRỢ

### 4.1. Git (Version Control)

#### 🪟 Windows

**Download:** https://git-scm.com/download/win

1. Chạy file `Git-2.xx.x-64-bit.exe`
2. Chọn **Next** cho tất cả (dùng cấu hình mặc định)
3. **Important:** Chọn **Git from the command line and also from 3rd-party software**
4. Click **Install**

Kiểm tra:
```bash
git --version
```

#### 🍎 macOS

```bash
# Cài qua Homebrew
brew install git

# Kiểm tra
git --version
```

#### 🐧 Linux

```bash
sudo apt install git
git --version
```

#### Cấu hình Git lần đầu

```bash
# Đặt tên và email (dùng cho commit)
git config --global user.name "Nguyen Van A"
git config --global user.email "nguyenvana@student.duytan.edu.vn"

# Kiểm tra
git config --list
```

### 4.2. GitHub Desktop (Optional, dễ dùng cho người mới)

**Download:** https://desktop.github.com/

Cài đặt và đăng nhập bằng tài khoản GitHub.

### 4.3. Postman (Test API)

**Download:** https://www.postman.com/downloads/

1. Download phiên bản phù hợp
2. Cài đặt
3. Đăng ký tài khoản miễn phí (optional)
4. Mở Postman và làm quen giao diện

**Alternative:** **Insomnia** (https://insomnia.rest/) - Nhẹ hơn Postman

### 4.4. Docker Desktop (Optional, cho học Microservices)

**Download:** https://www.docker.com/products/docker-desktop

**Yêu cầu:**
- Windows: Cần bật **WSL 2** (Windows Subsystem for Linux)
- Mac: Chọn đúng chip (Intel hoặc Apple Silicon)
- RAM: Tối thiểu 8GB

**Lưu ý:** Docker khá nặng, có thể bỏ qua nếu chỉ học Web API cơ bản.

---

## ⚙️ BƯỚC 5: CẤU HÌNH RIDER

### 5.1. Cài đặt Plugins bổ sung

**File** → **Settings** → **Plugins** → **Marketplace**

Tìm và cài:

1. **GitHub Copilot** (Optional)
   - AI code assistant
   - Free cho sinh viên: https://education.github.com/

2. **Markdown** (Recommended)
   - Xem và chỉnh sửa file .md
   - Preview real-time

3. **.ignore** (Recommended)
   - Quản lý .gitignore files
   - Templates sẵn có

4. **Rainbow Brackets** (Optional)
   - Tô màu dấu ngoặc `{}` `[]` `()`
   - Dễ nhìn code hơn

### 5.2. Cấu hình Editor

**File** → **Settings** → **Editor**

#### Font & Size
```
Font: JetBrains Mono (built-in, đẹp)
Size: 14 (hoặc 12 nếu màn hình nhỏ)
Line spacing: 1.2
```

#### Code Style (C#)
```
Settings → Editor → Code Style → C#

✅ Use auto-property, when possible
✅ Use 'var' when type is obvious
✅ Add 'readonly' modifier
✅ Make field readonly, when possible
```

**Shortcut format code:**
- Windows/Linux: `Ctrl + Alt + L`
- macOS: `Cmd + Option + L`

### 5.3. Cấu hình Database Tools

Rider có **DataGrip** tích hợp sẵn!

**View** → **Tool Windows** → **Database**

#### Kết nối SQL Server

1. Click **+** → **Data Source** → **Microsoft SQL Server**
2. Điền thông tin:
   ```
   Host: localhost
   Port: 1433 (SQL Server) hoặc để trống (LocalDB)
   Database: master
   Authentication: Windows (Integrated)
   ```
3. Click **Test Connection**
4. **Apply** → **OK**

#### Kết nối PostgreSQL

1. Click **+** → **Data Source** → **PostgreSQL**
2. Điền thông tin:
   ```
   Host: localhost
   Port: 5432
   Database: postgres
   User: postgres
   Password: (your password)
   ```
3. Click **Download missing driver files** (nếu cần)
4. **Test Connection** → **OK**

### 5.4. Cấu hình Build & Run

**File** → **Settings** → **Build, Execution, Deployment**

#### .NET CLI
```
✅ Use custom .NET CLI executable
Path: (auto-detected)

✅ Generate XML documentation
✅ Treat warnings as errors: No
```

#### NuGet
```
NuGet sources:
✅ nuget.org (default)

Package restore:
✅ Automatic restore on project load
```

### 5.5. Keyboard Shortcuts quan trọng

| Chức năng | Windows/Linux | macOS |
|-----------|---------------|-------|
| **Run project** | `Shift + F10` | `Ctrl + R` |
| **Debug project** | `Shift + F9` | `Ctrl + D` |
| **Build solution** | `Ctrl + Shift + B` | `Cmd + F9` |
| **Search everywhere** | `Double Shift` | `Double Shift` |
| **Go to definition** | `Ctrl + B` | `Cmd + B` |
| **Find usages** | `Alt + F7` | `Option + F7` |
| **Rename** | `Ctrl + R, R` | `Cmd + R, R` |
| **Format code** | `Ctrl + Alt + L` | `Cmd + Option + L` |
| **Show context menu** | `Alt + Enter` | `Option + Enter` |
| **Terminal** | `Alt + F12` | `Option + F12` |

**In PDF:** File → Settings → Keymap → Export (để in ra)

---

## ✅ BƯỚC 6: KIỂM TRA CÀI ĐẶT

### 6.1. Tạo project ASP.NET Core Web API đầu tiên

#### Dùng Terminal
```bash
# Tạo thư mục project
mkdir MyFirstAPI
cd MyFirstAPI

# Tạo Web API project
dotnet new webapi -n MyFirstAPI

# Mở trong Rider
cd MyFirstAPI
rider .
```

#### Dùng Rider GUI

1. **File** → **New** → **Solution...**
2. Chọn **ASP.NET Core Web API**
3. Điền thông tin:
   ```
   Solution name: MyFirstAPI
   Project name: MyFirstAPI
   Location: (chọn thư mục bạn muốn)
   
   Framework: .NET 8.0
   Authentication: None
   ✅ Enable OpenAPI support
   ✅ Use controllers
   ❌ Use minimal APIs
   ```
4. Click **Create**

### 6.2. Chạy project

**Cách 1: Dùng nút Run**
- Click nút ▶️ **Run** (màu xanh) trên toolbar
- Hoặc: `Shift + F10` (Win/Linux) / `Ctrl + R` (Mac)

**Cách 2: Dùng Terminal**
```bash
dotnet run
```

### 6.3. Kiểm tra kết quả

Rider sẽ tự động mở browser với URL:
```
https://localhost:7xxx/swagger
```

Bạn sẽ thấy **Swagger UI** hiển thị API documentation:

```
┌─────────────────────────────────────┐
│  Swagger UI                         │
├─────────────────────────────────────┤
│  GET /weatherforecast               │
│  Returns weather forecast data      │
│                                     │
│  [Try it out]                       │
└─────────────────────────────────────┘
```

Click **Try it out** → **Execute**

**Kết quả mong đợi:**
```json
[
  {
    "date": "2025-12-13",
    "temperatureC": 15,
    "temperatureF": 58,
    "summary": "Cool"
  },
  {
    "date": "2025-12-14",
    "temperatureC": 22,
    "temperatureF": 71,
    "summary": "Warm"
  }
]
```

✅ **Chúc mừng!** Môi trường đã sẵn sàng!

---

## 🐛 TROUBLESHOOTING

### Lỗi 1: "dotnet command not found"

**Nguyên nhân:** .NET SDK chưa được thêm vào PATH

**Giải pháp:**

#### Windows:
```powershell
# Kiểm tra PATH
$env:PATH

# Thêm .NET vào PATH thủ công
setx PATH "$env:PATH;C:\Program Files\dotnet"

# Khởi động lại Terminal
```

#### macOS/Linux:
```bash
# Thêm vào ~/.zshrc hoặc ~/.bashrc
export PATH="$PATH:/usr/local/share/dotnet"

# Reload
source ~/.zshrc
```

---

### Lỗi 2: Rider không phát hiện .NET SDK

**Giải pháp:**

1. **File** → **Settings** → **.NET**
2. Click **Add SDK**
3. Chọn thư mục:
   - Windows: `C:\Program Files\dotnet\sdk\8.0.xxx`
   - macOS: `/usr/local/share/dotnet/sdk/8.0.xxx`
   - Linux: `/usr/share/dotnet/sdk/8.0.xxx`
4. **Apply** → **OK**
5. **Restart Rider**

---

### Lỗi 3: "Unable to connect to database"

#### SQL Server (Windows)

```bash
# Kiểm tra SQL Server service có chạy không
services.msc

# Tìm "SQL Server (MSSQLSERVER)"
# Right-click → Start (nếu chưa chạy)
```

#### PostgreSQL

```bash
# Windows
net start postgresql-x64-16

# macOS
brew services start postgresql@16

# Linux
sudo systemctl start postgresql
```

---

### Lỗi 4: Port đã được sử dụng

**Lỗi:**
```
Failed to bind to address http://localhost:5000: address already in use
```

**Giải pháp:**

#### Windows:
```powershell
# Tìm process đang dùng port 5000
netstat -ano | findstr :5000

# Kill process (thay PID)
taskkill /PID <PID> /F
```

#### macOS/Linux:
```bash
# Tìm process
lsof -i :5000

# Kill process
kill -9 <PID>
```

Hoặc đổi port trong `launchSettings.json`:
```json
{
  "applicationUrl": "http://localhost:5001;https://localhost:7001"
}
```

---

### Lỗi 5: Certificate SSL không tin cậy

**Lỗi:** Browser hiển thị "Your connection is not private"

**Giải pháp:**

```bash
# Trust HTTPS certificate
dotnet dev-certs https --trust
```

Chọn **Yes** khi hệ thống hỏi.

---

### Lỗi 6: Rider chậm/lag

**Nguyên nhân:** RAM không đủ hoặc indexing

**Giải pháp:**

1. **Tăng RAM cho Rider:**
   - **Help** → **Change Memory Settings**
   - Tăng lên 4GB hoặc 8GB (nếu máy có)

2. **Disable unused plugins:**
   - **File** → **Settings** → **Plugins**
   - Tắt các plugin không dùng

3. **Đợi indexing xong:**
   - Lần đầu mở project, Rider cần index code (5-10 phút)
   - Thanh progress ở góc dưới cùng

4. **Exclude folders:**
   - Right-click folder `bin`, `obj` → **Mark Directory as** → **Excluded**

---

### Lỗi 7: NuGet restore failed

**Giải pháp:**

```bash
# Clear NuGet cache
dotnet nuget locals all --clear

# Restore lại packages
dotnet restore
```

Hoặc trong Rider:
- Right-click Solution → **Restore NuGet Packages**

---

## 🆚 SO SÁNH RIDER VS VISUAL STUDIO

### Chi tiết so sánh

| Feature | JetBrains Rider | Visual Studio 2022 |
|---------|-----------------|-------------------|
| **Khởi động** | ⚡ 5-10 giây | 🐌 20-30 giây |
| **RAM usage** | 💚 1-2 GB | 🟡 3-5 GB |
| **Disk space** | 💚 2 GB | 🔴 10-40 GB |
| **Code analysis** | ⭐⭐⭐⭐⭐ (ReSharper built-in) | ⭐⭐⭐ (cần mua ReSharper) |
| **Database tools** | ⭐⭐⭐⭐⭐ (DataGrip built-in) | ⭐⭐⭐ (SSMS riêng biệt) |
| **Git integration** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Refactoring** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Search/Navigation** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **UI/UX** | ⭐⭐⭐⭐⭐ (Modern) | ⭐⭐⭐⭐ |
| **Cross-platform** | ✅ Win/Mac/Linux | ⚠️ Windows only |
| **Price** | 💰 $149/year (Free for students) | 🆓 Community free |
| **Learning curve** | 📘 Trung bình | 📕 Dễ hơn (nếu quen Windows) |

### Khi nào nên dùng Visual Studio?

✅ **Nên dùng Visual Studio nếu:**
- Máy mạnh (16GB+ RAM)
- Chỉ dùng Windows
- Cần WinForms, WPF (desktop apps)
- Làm game với Unity (VS tích hợp tốt hơn)
- Không muốn trả phí (dùng Community)

✅ **Nên dùng Rider nếu:**
- Máy yếu/trung bình (8GB RAM)
- Dùng Mac hoặc Linux
- Muốn code analysis mạnh (ReSharper)
- Thích UI hiện đại, workflow nhanh
- Có license sinh viên miễn phí

### Migration từ Visual Studio sang Rider

**Shortcuts tương đương:**

| Action | Visual Studio | Rider |
|--------|---------------|-------|
| Go to definition | F12 | Ctrl+B / Cmd+B |
| Find usages | Shift+F12 | Alt+F7 / Opt+F7 |
| Rename | Ctrl+R,R / F2 | Ctrl+R,R / F2 |
| Run | F5 | Shift+F10 / Ctrl+R |
| Debug | F5 | Shift+F9 / Ctrl+D |

**Import settings từ VS:**
- Rider có thể import keymaps từ Visual Studio
- **File** → **Manage IDE Settings** → **Import Settings**

---

## 📚 TÀI LIỆU THAM KHẢO

### Official Documentation

- **.NET Docs:** https://docs.microsoft.com/dotnet
- **Rider Docs:** https://www.jetbrains.com/help/rider
- **ASP.NET Core:** https://docs.microsoft.com/aspnet/core
- **Entity Framework Core:** https://docs.microsoft.com/ef/core

### Video Tutorials

- **JetBrains Rider Tips:** https://www.youtube.com/@JetBrainsTV
- **Rider for Unity:** https://www.youtube.com/watch?v=6r3vQKUZnRE
- **.NET 8 Tutorial:** https://www.youtube.com/dotnet

### Community

- **Rider Forum:** https://rider-support.jetbrains.com/
- **Stack Overflow:** https://stackoverflow.com/questions/tagged/rider
- **.NET Discord:** https://discord.gg/dotnet

---

## ✅ CHECKLIST CÀI ĐẶT

In ra và check từng mục:

```
□ .NET SDK 8.0 installed
□ dotnet --version works in terminal
□ JetBrains account registered (student license)
□ Rider 2024.3 installed and activated
□ Rider detects .NET SDK
□ Database installed (SQL Server / PostgreSQL / LocalDB)
□ Database connection tested
□ Git installed and configured
□ Postman installed
□ Created first Web API project
□ Project runs successfully (Swagger UI works)
□ Database tools in Rider configured
□ Keyboard shortcuts learned
```

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ môi trường đã sẵn sàng, hãy:

1. **Làm quen với Rider:**
   - Xem video: "Rider in 10 minutes" trên YouTube
   - Thực hành shortcuts: `Double Shift`, `Ctrl+B`, `Alt+Enter`

2. **Tạo Hello World API:**
   - File → New Project → ASP.NET Core Web API
   - Chạy và test với Swagger

3. **Học C# cơ bản:**
   - Đọc: `phan_0_csharp_basic/01_syntax_variables.md`
   - Làm bài tập trong tài liệu

4. **Follow tutorial đầu tiên:**
   - `examples/HelloAPI/README.md`
   - Tạo API đơn giản với CRUD operations

5. **Tham gia cộng đồng:**
   - Join .NET Discord
   - Follow r/dotnet trên Reddit
   - Subscribe JetBrains TV

---

## 💡 TIPS & TRICKS

### Rider Productivity Tips

1. **Double Shift = Search Everywhere**
   - Tìm files, classes, methods, settings
   - Nhanh nhất để navigate

2. **Alt + Enter = Show Context Actions**
   - Quick fixes
   - Refactoring suggestions
   - Import namespaces

3. **Ctrl + Shift + A = Find Action**
   - Tìm commands
   - Không cần nhớ shortcuts

4. **Alt + F12 = Terminal**
   - Chạy `dotnet` commands
   - Git commands

5. **Ctrl + E = Recent Files**
   - Quay lại files vừa mở

### Performance Tips

1. **Exclude folders không cần thiết:**
   ```
   bin/
   obj/
   node_modules/
   .vs/
   ```

2. **Power Save Mode** khi máy yếu:
   - **File** → **Power Save Mode**
   - Tắt code analysis real-time

3. **Close unused solutions**
   - Chỉ mở 1 solution tại một thời điểm

### Learning Resources

- **Rider Blog:** https://blog.jetbrains.com/dotnet/
- **Weekly .NET Tips:** Subscribe newsletter
- **GitHub Student Pack:** https://education.github.com/pack

---

**Chúc bạn cài đặt thành công và học tốt! 🎉**

*Nếu gặp vấn đề, tham khảo phần Troubleshooting hoặc liên hệ giảng viên.*
