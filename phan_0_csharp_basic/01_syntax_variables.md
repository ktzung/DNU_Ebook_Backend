# 🟦 CHƯƠNG 01
# **CÚ PHÁP & BIẾN TRONG C#**

## 📖 1. Giới thiệu C#

**C# (C-Sharp)** là ngôn ngữ lập trình hướng đối tượng do Microsoft phát triển. Nó mạnh mẽ, an toàn kiểu dữ liệu (type-safe) và chạy trên nền tảng .NET.

### 🧠 Tại sao cần học C#?

C# là một trong những ngôn ngữ phổ biến nhất cho:
- **Backend Development**: Xây dựng Web API, Microservices với ASP.NET Core
- **Enterprise Applications**: Ứng dụng doanh nghiệp lớn
- **Game Development**: Unity game engine sử dụng C#
- **Cross-platform**: Chạy trên Windows, Linux, macOS

### 🎒 Ví dụ đời sống

Hãy tưởng tượng C# như **ngôn ngữ giao tiếp với máy tính**:
- Giống như bạn học tiếng Anh để giao tiếp với người nước ngoài
- C# giúp bạn "nói chuyện" với máy tính để tạo ra các ứng dụng

### Cấu trúc cơ bản của một chương trình C#:
```csharp
using System;  // Import thư viện chuẩn

namespace HelloWorld  // Tên không gian, giống như "thư mục"
{
    class Program  // Class chứa code
    {
        static void Main(string[] args)  // Hàm chính, chạy đầu tiên
        {
            Console.WriteLine("Xin chào DNU!");  // In ra màn hình
        }
    }
}
```

**Giải thích chi tiết:**
- `using System;`: Khai báo sử dụng thư viện chuẩn (giống `import` trong Java/Python)
- `namespace`: Gom nhóm các class liên quan, tránh xung đột tên
- `class Program`: Định nghĩa một class tên Program
- `static void Main`: Điểm bắt đầu của chương trình, bắt buộc phải có
- `string[] args`: Tham số dòng lệnh (command line arguments)

---

## 📦 2. Biến & Kiểu dữ liệu

### 🧠 Biến là gì?

**Biến** giống như **hộp đựng đồ** trong thực tế:
- Mỗi hộp có **tên** (tên biến)
- Mỗi hộp có **loại** (kiểu dữ liệu)
- Mỗi hộp chứa **giá trị** (value)

Ví dụ: Hộp tên "age" loại "số nguyên" chứa giá trị 20.

### 2.1. Khai báo biến

```csharp
// Cú pháp: [kiểu dữ liệu] [tên biến] = [giá trị];

int age = 20;                    // Số nguyên (32-bit)
long bigNumber = 1000000000L;    // Số nguyên lớn (64-bit)
float height = 1.75f;            // Số thực (32-bit, cần hậu tố 'f')
double gpa = 3.5;                // Số thực (64-bit, chính xác hơn)
decimal price = 99.99m;           // Số thực (128-bit, dùng cho tiền tệ, cần 'm')
char grade = 'A';                // Ký tự đơn (nháy đơn)
string name = "Nguyen Van A";    // Chuỗi ký tự (nháy kép)
bool isStudent = true;           // Logic (true/false)
```

**Bảng kiểu dữ liệu cơ bản:**

| Kiểu | Mô tả | Kích thước | Ví dụ |
|------|-------|------------|-------|
| `int` | Số nguyên | 32-bit | `int age = 20;` |
| `long` | Số nguyên lớn | 64-bit | `long id = 123456789L;` |
| `float` | Số thực | 32-bit | `float pi = 3.14f;` |
| `double` | Số thực chính xác | 64-bit | `double gpa = 3.75;` |
| `decimal` | Số thực rất chính xác | 128-bit | `decimal price = 99.99m;` |
| `char` | Ký tự đơn | 16-bit | `char letter = 'A';` |
| `string` | Chuỗi ký tự | Dynamic | `string name = "An";` |
| `bool` | Logic | 8-bit | `bool isActive = true;` |

### 2.2. Var keyword (Type Inference)

C# có thể tự suy luận kiểu dữ liệu từ giá trị:

```csharp
// Cách cũ: Khai báo rõ kiểu
string city = "Da Nang";
int year = 2024;

// Cách mới: Dùng var (trình biên dịch tự suy luận)
var city = "Da Nang";  // Trình biên dịch hiểu đây là string
var year = 2024;       // Trình biên dịch hiểu đây là int
var isActive = true;   // Trình biên dịch hiểu đây là bool
```

**Lưu ý:**
- `var` phải có giá trị khởi tạo: `var x;` ❌ (sai)
- `var` phải có giá trị rõ ràng: `var x = null;` ❌ (sai, không biết kiểu)
- Dùng `var` khi kiểu rõ ràng từ context: `var list = new List<int>();` ✅

### 2.3. Hằng số (Constants)

Dùng `const` cho giá trị không đổi:

```csharp
const double PI = 3.14159;
const int MAX_STUDENTS = 100;
const string APP_NAME = "MyApp";

// ❌ Không thể thay đổi sau khi khai báo
// PI = 3.14; // Lỗi compile!
```

### 2.4. Naming Conventions (Quy ước đặt tên)

```csharp
// ✅ Đúng: PascalCase cho class, method, property
class Student { }
void CalculateTotal() { }
public string FirstName { get; set; }

// ✅ Đúng: camelCase cho biến local, tham số
int studentCount = 10;
void ProcessOrder(int orderId) { }

// ✅ Đúng: UPPER_CASE cho constants
const int MAX_SIZE = 100;
const string API_KEY = "secret";

// ❌ Sai: Tên không rõ ràng
int x = 10;  // Không biết x là gì
int temp = 5; // temp là gì?
```

---

## 🧮 3. Toán tử (Operators)

### 3.1. Toán tử toán học

```csharp
int a = 10, b = 3;

int sum = a + b;        // 13: Cộng
int diff = a - b;       // 7: Trừ
int product = a * b;    // 30: Nhân
int quotient = a / b;   // 3: Chia (lấy phần nguyên)
int remainder = a % b;   // 1: Chia lấy dư (modulo)

// Toán tử gán kết hợp
a += 5;  // Tương đương: a = a + 5
a -= 2;  // Tương đương: a = a - 2
a *= 3;  // Tương đương: a = a * 3
a /= 2;  // Tương đương: a = a / 2
a %= 3;  // Tương đương: a = a % 3
```

### 3.2. Toán tử tăng/giảm

```csharp
int x = 10;

x++;  // Tăng sau (post-increment): x = 10, sau đó x = 11
++x;  // Tăng trước (pre-increment): x tăng lên 12 ngay

int y = x++;  // y = 12, x = 13 (gán trước, tăng sau)
int z = ++x;  // x = 14, z = 14 (tăng trước, gán sau)

x--;  // Giảm sau
--x;  // Giảm trước
```

### 3.3. Toán tử so sánh

```csharp
int a = 10, b = 5;

bool equal = (a == b);        // false: Bằng
bool notEqual = (a != b);     // true: Khác
bool greater = (a > b);       // true: Lớn hơn
bool less = (a < b);          // false: Nhỏ hơn
bool greaterEqual = (a >= b); // true: Lớn hơn hoặc bằng
bool lessEqual = (a <= b);    // false: Nhỏ hơn hoặc bằng
```

### 3.4. Toán tử logic

```csharp
bool isStudent = true;
bool hasDiscount = false;
int age = 20;

// AND (&&): Cả hai điều kiện đều đúng
bool canGetDiscount = isStudent && (age < 25); // true && true = true

// OR (||): Một trong hai điều kiện đúng
bool canEnter = isStudent || hasDiscount; // true || false = true

// NOT (!): Phủ định
bool isNotStudent = !isStudent; // !true = false

// Kết hợp nhiều điều kiện
bool complex = (age >= 18) && (age <= 65) && isStudent; // true
```

**Lưu ý quan trọng:**
- `&&` và `||` có **short-circuit evaluation**: Dừng ngay khi biết kết quả
- `&` và `|` (bitwise) luôn đánh giá cả hai vế

```csharp
// Short-circuit: Nếu a == 0, không cần kiểm tra b
if (a != 0 && (b / a > 5)) { } // ✅ An toàn

// Không short-circuit: Luôn chia, có thể lỗi
if (a != 0 & (b / a > 5)) { } // ❌ Nguy hiểm nếu a == 0
```

### 3.5. Toán tử điều kiện (Ternary Operator)

```csharp
int score = 85;
string grade = (score >= 50) ? "Đậu" : "Rớt";
// Cú pháp: [điều kiện] ? [nếu đúng] : [nếu sai]

// Tương đương với:
string grade;
if (score >= 50)
    grade = "Đậu";
else
    grade = "Rớt";
```

### 3.6. Toán tử null-coalescing

```csharp
string name = null;
string displayName = name ?? "Khách"; // Nếu name null thì dùng "Khách"

// Null-coalescing assignment (C# 8+)
List<int> numbers = null;
numbers ??= new List<int>(); // Nếu null thì tạo mới
```

---

## 🚦 4. Cấu trúc điều khiển

### 4.1. Câu lệnh If-Else
```csharp
int score = 85;

if (score >= 90)
{
    Console.WriteLine("Xuất sắc");
}
else if (score >= 70)
{
    Console.WriteLine("Khá");
}
else
{
    Console.WriteLine("Cần cố gắng");
}
```

### 4.2. Switch-Case
```csharp
int day = 3;
switch (day)
{
    case 2:
        Console.WriteLine("Thứ Hai");
        break;
    case 3:
        Console.WriteLine("Thứ Ba");
        break;
    default:
        Console.WriteLine("Ngày khác");
        break;
}
```

---

## 🔄 5. Vòng lặp (Loops)

### 5.1. Vòng lặp For
Dùng khi biết trước số lần lặp.
```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine($"Lần lặp thứ {i}");
}
```

### 5.2. Vòng lặp While
Dùng khi lặp dựa trên điều kiện.
```csharp
int count = 0;
while (count < 5)
{
    Console.WriteLine(count);
    count++;
}
```

### 5.3. Vòng lặp Do-While
Chạy ít nhất 1 lần trước khi kiểm tra điều kiện.
```csharp
do
{
    Console.WriteLine("Chạy ít nhất 1 lần");
} while (false);
```

---

## ❌ 6. CÁC LỖI THƯỜNG GẶP

### ❌ Lỗi 1: Không khởi tạo biến trước khi dùng

```csharp
// ❌ Vấn đề: Biến chưa có giá trị
int age;
Console.WriteLine(age); // Lỗi: Use of unassigned local variable

// ✅ Giải pháp: Khởi tạo giá trị
int age = 0;
Console.WriteLine(age); // OK
```

**🔍 Giải thích:** C# yêu cầu biến phải được khởi tạo trước khi sử dụng để tránh lỗi logic.

---

### ❌ Lỗi 2: Nhầm lẫn giữa `=` và `==`

```csharp
// ❌ Vấn đề: Dùng = thay vì == trong điều kiện
int x = 10;
if (x = 5) // Lỗi: Cannot implicitly convert int to bool
{
    Console.WriteLine("x bằng 5");
}

// ✅ Giải pháp: Dùng == để so sánh
if (x == 5)
{
    Console.WriteLine("x bằng 5");
}
```

**🔍 Giải thích:** `=` là toán tử gán, `==` là toán tử so sánh. Trong điều kiện phải dùng `==`.

---

### ❌ Lỗi 3: Chia cho 0

```csharp
// ❌ Vấn đề: Chia cho 0 gây lỗi runtime
int a = 10, b = 0;
int result = a / b; // DivideByZeroException

// ✅ Giải pháp: Kiểm tra trước khi chia
if (b != 0)
{
    int result = a / b;
}
else
{
    Console.WriteLine("Không thể chia cho 0!");
}
```

**🔍 Giải thích:** Chia số nguyên cho 0 sẽ ném `DivideByZeroException`. Luôn kiểm tra mẫu số trước.

---

### ❌ Lỗi 4: Quên break trong switch-case

```csharp
// ❌ Vấn đề: Quên break, code chạy tiếp
int day = 1;
switch (day)
{
    case 1:
        Console.WriteLine("Thứ Hai");
        // Quên break!
    case 2:
        Console.WriteLine("Thứ Ba");
        break;
}
// Output: "Thứ Hai" và "Thứ Ba" (sai!)

// ✅ Giải pháp: Thêm break hoặc dùng switch expression
switch (day)
{
    case 1:
        Console.WriteLine("Thứ Hai");
        break; // ✅
    case 2:
        Console.WriteLine("Thứ Ba");
        break;
}

// Hoặc dùng switch expression (C# 8+)
string dayName = day switch
{
    1 => "Thứ Hai",
    2 => "Thứ Ba",
    _ => "Khác"
};
```

**🔍 Giải thích:** Trong switch truyền thống, nếu không có `break`, code sẽ "fall through" sang case tiếp theo.

---

### ❌ Lỗi 5: Vòng lặp vô hạn

```csharp
// ❌ Vấn đề: Quên tăng biến đếm
int i = 0;
while (i < 10)
{
    Console.WriteLine(i);
    // Quên i++ → vòng lặp vô hạn!
}

// ✅ Giải pháp: Nhớ tăng/giảm biến đếm
int i = 0;
while (i < 10)
{
    Console.WriteLine(i);
    i++; // ✅
}
```

**🔍 Giải thích:** Vòng lặp `while` cần điều kiện thay đổi để có thể kết thúc.

---

### ❌ Lỗi 6: So sánh string bằng `==` thay vì `Equals()`

```csharp
// ⚠️ Vấn đề: == có thể không hoạt động đúng với string null
string name1 = null;
string name2 = "An";

if (name1 == name2) // OK, nhưng...
{
}

// ✅ Giải pháp: Dùng Equals() hoặc string.IsNullOrEmpty()
if (string.Equals(name1, name2, StringComparison.OrdinalIgnoreCase))
{
}

// Hoặc dùng null-safe comparison
if (name1?.Equals(name2) == true)
{
}
```

**🔍 Giải thích:** Với string, `==` thường OK, nhưng `Equals()` rõ ràng hơn và hỗ trợ so sánh không phân biệt hoa thường.

---

### ❌ Lỗi 7: Không kiểm tra null trước khi dùng

```csharp
// ❌ Vấn đề: Có thể null
string name = GetName(); // Có thể trả về null
int length = name.Length; // NullReferenceException nếu name = null

// ✅ Giải pháp: Kiểm tra null
string name = GetName();
if (name != null)
{
    int length = name.Length;
}

// Hoặc dùng null-conditional operator (C# 6+)
int? length = name?.Length; // Trả về null nếu name null
```

**🔍 Giải thích:** Luôn kiểm tra null trước khi truy cập thuộc tính/phương thức của object.

---

### ❌ Lỗi 8: Ép kiểu không an toàn

```csharp
// ❌ Vấn đề: Ép kiểu có thể mất dữ liệu
double price = 99.99;
int intPrice = (int)price; // 99 (mất phần thập phân)

// Hoặc ép kiểu không hợp lệ
object obj = "Hello";
int number = (int)obj; // InvalidCastException

// ✅ Giải pháp: Dùng TryParse hoặc kiểm tra kiểu
double price = 99.99;
int intPrice = (int)price; // OK nếu chấp nhận mất phần thập phân

// Với string sang số
string input = "123";
if (int.TryParse(input, out int number))
{
    Console.WriteLine(number); // 123
}
else
{
    Console.WriteLine("Không phải số hợp lệ");
}
```

**🔍 Giải thích:** Ép kiểu có thể mất dữ liệu hoặc gây lỗi. Luôn kiểm tra hoặc dùng `TryParse`.

---

## 🎯 7. CASE STUDY / VÍ DỤ THỰC TẾ

### Case Study 1: Chương trình tính lương nhân viên

**Yêu cầu:** Tính lương thực nhận dựa trên lương cơ bản, số ngày làm việc, và phụ cấp.

```csharp
using System;

class Program
{
    static void Main()
    {
        // Nhập dữ liệu
        Console.Write("Nhập lương cơ bản: ");
        double baseSalary = double.Parse(Console.ReadLine());
        
        Console.Write("Nhập số ngày làm việc: ");
        int workingDays = int.Parse(Console.ReadLine());
        
        Console.Write("Nhập phụ cấp: ");
        double allowance = double.Parse(Console.ReadLine());
        
        // Tính toán
        double dailyRate = baseSalary / 30; // Giả sử 1 tháng = 30 ngày
        double salaryByDays = dailyRate * workingDays;
        double totalSalary = salaryByDays + allowance;
        
        // Hiển thị kết quả
        Console.WriteLine($"\n=== KẾT QUẢ ===");
        Console.WriteLine($"Lương theo ngày: {dailyRate:C}");
        Console.WriteLine($"Lương theo số ngày: {salaryByDays:C}");
        Console.WriteLine($"Phụ cấp: {allowance:C}");
        Console.WriteLine($"TỔNG LƯƠNG: {totalSalary:C}");
        
        // Xếp loại
        if (totalSalary >= 10000000)
            Console.WriteLine("Mức lương: CAO");
        else if (totalSalary >= 5000000)
            Console.WriteLine("Mức lương: TRUNG BÌNH");
        else
            Console.WriteLine("Mức lương: THẤP");
    }
}
```

**Giải thích từng bước:**
1. Nhập dữ liệu từ người dùng bằng `Console.ReadLine()`
2. Chuyển đổi string sang số bằng `Parse()`
3. Tính toán theo công thức nghiệp vụ
4. Hiển thị kết quả với format tiền tệ (`:C`)
5. Xếp loại dựa trên điều kiện

---

### Case Study 2: Hệ thống chấm điểm học sinh

**Yêu cầu:** Nhập điểm các môn, tính điểm trung bình, xếp loại học lực.

```csharp
using System;

class StudentGradeCalculator
{
    static void Main()
    {
        Console.WriteLine("=== HỆ THỐNG CHẤM ĐIỂM ===\n");
        
        // Nhập thông tin
        Console.Write("Tên học sinh: ");
        string studentName = Console.ReadLine();
        
        Console.Write("Điểm Toán: ");
        double math = double.Parse(Console.ReadLine());
        
        Console.Write("Điểm Lý: ");
        double physics = double.Parse(Console.ReadLine());
        
        Console.Write("Điểm Hóa: ");
        double chemistry = double.Parse(Console.ReadLine());
        
        // Tính điểm trung bình
        double average = (math + physics + chemistry) / 3;
        
        // Xếp loại
        string grade;
        if (average >= 8.0)
            grade = "GIỎI";
        else if (average >= 6.5)
            grade = "KHÁ";
        else if (average >= 5.0)
            grade = "TRUNG BÌNH";
        else
            grade = "YẾU";
        
        // Hiển thị kết quả
        Console.WriteLine("\n=== KẾT QUẢ ===");
        Console.WriteLine($"Học sinh: {studentName}");
        Console.WriteLine($"Điểm Toán: {math:F2}");
        Console.WriteLine($"Điểm Lý: {physics:F2}");
        Console.WriteLine($"Điểm Hóa: {chemistry:F2}");
        Console.WriteLine($"Điểm trung bình: {average:F2}");
        Console.WriteLine($"Xếp loại: {grade}");
        
        // Kiểm tra đậu/rớt
        if (average >= 5.0)
            Console.WriteLine("KẾT QUẢ: ĐẬU ✅");
        else
            Console.WriteLine("KẾT QUẢ: RỚT ❌");
    }
}
```

**Best practices áp dụng:**
- Tên biến rõ ràng (`studentName`, `average`)
- Format số thập phân (`:F2` cho 2 chữ số)
- Cấu trúc if-else rõ ràng
- Thông báo kết quả dễ hiểu

---

### Case Study 3: Máy tính đơn giản

**Yêu cầu:** Thực hiện các phép toán cơ bản (+, -, *, /).

```csharp
using System;

class SimpleCalculator
{
    static void Main()
    {
        Console.WriteLine("=== MÁY TÍNH ĐƠN GIẢN ===\n");
        
        // Nhập số thứ nhất
        Console.Write("Nhập số thứ nhất: ");
        double num1 = double.Parse(Console.ReadLine());
        
        // Nhập phép toán
        Console.Write("Nhập phép toán (+, -, *, /): ");
        char operation = char.Parse(Console.ReadLine());
        
        // Nhập số thứ hai
        Console.Write("Nhập số thứ hai: ");
        double num2 = double.Parse(Console.ReadLine());
        
        // Tính toán
        double result = 0;
        bool isValid = true;
        
        switch (operation)
        {
            case '+':
                result = num1 + num2;
                break;
            case '-':
                result = num1 - num2;
                break;
            case '*':
                result = num1 * num2;
                break;
            case '/':
                if (num2 != 0)
                    result = num1 / num2;
                else
                {
                    Console.WriteLine("Lỗi: Không thể chia cho 0!");
                    isValid = false;
                }
                break;
            default:
                Console.WriteLine("Lỗi: Phép toán không hợp lệ!");
                isValid = false;
                break;
        }
        
        // Hiển thị kết quả
        if (isValid)
        {
            Console.WriteLine($"\nKết quả: {num1} {operation} {num2} = {result:F2}");
        }
    }
}
```

**Giải thích:**
- Sử dụng `switch-case` để xử lý nhiều trường hợp
- Kiểm tra chia cho 0 trước khi thực hiện
- Xử lý trường hợp phép toán không hợp lệ
- Format kết quả với 2 chữ số thập phân

---

## ✅ 8. BEST PRACTICES

### 8.1. Đặt tên biến rõ ràng

```csharp
// ❌ Tên không rõ ràng
int x = 10;
int temp = 5;
string s = "Hello";

// ✅ Tên mô tả rõ mục đích
int studentCount = 10;
int totalPrice = 5;
string welcomeMessage = "Hello";
```

### 8.2. Sử dụng const cho giá trị không đổi

```csharp
// ❌ Magic numbers
double total = price * 1.1; // 1.1 là gì?

// ✅ Dùng const
const double VAT_RATE = 1.1;
double total = price * VAT_RATE;
```

### 8.3. Luôn khởi tạo biến

```csharp
// ❌ Chưa khởi tạo
int count;
if (someCondition)
    count = 10;
Console.WriteLine(count); // Lỗi nếu someCondition = false

// ✅ Luôn khởi tạo
int count = 0;
if (someCondition)
    count = 10;
Console.WriteLine(count); // An toàn
```

### 8.4. Kiểm tra điều kiện trước khi thao tác

```csharp
// ❌ Không kiểm tra
int result = a / b; // Có thể lỗi nếu b = 0

// ✅ Kiểm tra trước
if (b != 0)
{
    int result = a / b;
}
else
{
    Console.WriteLine("Không thể chia cho 0");
}
```

### 8.5. Sử dụng string interpolation thay vì concatenation

```csharp
// ❌ Nối chuỗi bằng +
string message = "Xin chào " + name + ", bạn " + age + " tuổi";

// ✅ String interpolation
string message = $"Xin chào {name}, bạn {age} tuổi";
```

---

## 🧪 9. BÀI TẬP THỰC HÀNH

### Bài 1: Tính điểm trung bình
Viết chương trình nhập vào tên, điểm Toán, Lý, Hóa. Tính điểm trung bình và xếp loại.

<details>
<summary>💡 Đáp án</summary>

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.Write("Tên học sinh: ");
        string name = Console.ReadLine();
        
        Console.Write("Điểm Toán: ");
        double math = double.Parse(Console.ReadLine());
        
        Console.Write("Điểm Lý: ");
        double physics = double.Parse(Console.ReadLine());
        
        Console.Write("Điểm Hóa: ");
        double chemistry = double.Parse(Console.ReadLine());
        
        double average = (math + physics + chemistry) / 3;
        
        Console.WriteLine($"\nHọc sinh: {name}");
        Console.WriteLine($"Điểm trung bình: {average:F2}");
        
        if (average >= 8.0)
            Console.WriteLine("Xếp loại: GIỎI");
        else if (average >= 6.5)
            Console.WriteLine("Xếp loại: KHÁ");
        else if (average >= 5.0)
            Console.WriteLine("Xếp loại: TRUNG BÌNH");
        else
            Console.WriteLine("Xếp loại: YẾU");
    }
}
```
</details>

---

### Bài 2: Kiểm tra số nguyên tố
Viết chương trình nhập vào một số nguyên n. Kiểm tra xem n có phải là số nguyên tố không.

<details>
<summary>💡 Đáp án</summary>

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.Write("Nhập số nguyên n: ");
        int n = int.Parse(Console.ReadLine());
        
        if (n < 2)
        {
            Console.WriteLine($"{n} không phải số nguyên tố");
            return;
        }
        
        bool isPrime = true;
        for (int i = 2; i <= Math.Sqrt(n); i++)
        {
            if (n % i == 0)
            {
                isPrime = false;
                break;
            }
        }
        
        if (isPrime)
            Console.WriteLine($"{n} là số nguyên tố");
        else
            Console.WriteLine($"{n} không phải số nguyên tố");
    }
}
```
</details>

---

### Bài 3: Bảng cửu chương
In ra bảng cửu chương từ 2 đến 9.

<details>
<summary>💡 Đáp án</summary>

```csharp
using System;

class Program
{
    static void Main()
    {
        for (int i = 2; i <= 9; i++)
        {
            Console.WriteLine($"\nBảng cửu chương {i}:");
            for (int j = 1; j <= 10; j++)
            {
                Console.WriteLine($"{i} x {j} = {i * j}");
            }
        }
    }
}
```
</details>

---

### Bài 4: Tính tiền điện
Viết chương trình tính tiền điện dựa trên số kWh sử dụng:
- 0-50 kWh: 1,678 đ/kWh
- 51-100 kWh: 1,734 đ/kWh
- 101-200 kWh: 2,014 đ/kWh
- 201-300 kWh: 2,536 đ/kWh
- 301-400 kWh: 2,834 đ/kWh
- Trên 400 kWh: 2,927 đ/kWh

<details>
<summary>💡 Đáp án</summary>

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.Write("Nhập số kWh: ");
        int kwh = int.Parse(Console.ReadLine());
        
        double total = 0;
        
        if (kwh > 400)
        {
            total += (kwh - 400) * 2927;
            kwh = 400;
        }
        if (kwh > 300)
        {
            total += (kwh - 300) * 2834;
            kwh = 300;
        }
        if (kwh > 200)
        {
            total += (kwh - 200) * 2536;
            kwh = 200;
        }
        if (kwh > 100)
        {
            total += (kwh - 100) * 2014;
            kwh = 100;
        }
        if (kwh > 50)
        {
            total += (kwh - 50) * 1734;
            kwh = 50;
        }
        total += kwh * 1678;
        
        Console.WriteLine($"Tổng tiền điện: {total:N0} đ");
    }
}
```
</details>

---

### Bài 5: Kiểm tra năm nhuận
Viết chương trình kiểm tra một năm có phải là năm nhuận không.

<details>
<summary>💡 Đáp án</summary>

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.Write("Nhập năm: ");
        int year = int.Parse(Console.ReadLine());
        
        bool isLeapYear = (year % 4 == 0 && year % 100 != 0) || (year % 400 == 0);
        
        if (isLeapYear)
            Console.WriteLine($"{year} là năm nhuận");
        else
            Console.WriteLine($"{year} không phải năm nhuận");
    }
}
```
</details>

---

## 🧪 10. MINI TEST

1. **Biến nào sau đây khai báo đúng?**
   - A. `int x;` (chưa khởi tạo)
   - B. `var x = 10;` ✅
   - C. `string name = null;` (cần kiểm tra)
   - D. `const int MAX;` (const phải có giá trị)

2. **Kết quả của `10 / 3` trong C# là gì?**
   - A. 3.33
   - B. 3 ✅ (chia số nguyên lấy phần nguyên)
   - C. 3.333...
   - D. Lỗi

3. **Toán tử nào dùng để so sánh bằng?**
   - A. `=`
   - B. `==` ✅
   - C. `===`
   - D. `Equals()`

4. **Vòng lặp nào chạy ít nhất 1 lần?**
   - A. `for`
   - B. `while`
   - C. `do-while` ✅
   - D. `foreach`

5. **Câu lệnh nào đúng để khai báo hằng số?**
   - A. `var PI = 3.14;`
   - B. `const double PI = 3.14;` ✅
   - C. `readonly double PI = 3.14;`
   - D. `static double PI = 3.14;`

<details>
<summary>💡 Đáp án</summary>

1. **B** - `var x = 10;` là cách khai báo đúng với type inference
2. **B** - Chia số nguyên (`int / int`) cho kết quả là số nguyên (3)
3. **B** - `==` là toán tử so sánh bằng
4. **C** - `do-while` chạy ít nhất 1 lần trước khi kiểm tra điều kiện
5. **B** - `const` dùng để khai báo hằng số
</details>

---

## 📝 11. QUICK NOTES

### Cú pháp cơ bản:
```csharp
// Khai báo biến
[kiểu] [tên] = [giá trị];
int age = 20;

// Hằng số
const double PI = 3.14;

// Type inference
var name = "An";

// String interpolation
string msg = $"Xin chào {name}";
```

### Toán tử quan trọng:
- Toán học: `+`, `-`, `*`, `/`, `%`
- So sánh: `==`, `!=`, `>`, `<`, `>=`, `<=`
- Logic: `&&`, `||`, `!`
- Tăng/giảm: `++`, `--`
- Gán: `=`, `+=`, `-=`, `*=`, `/=`

### Cấu trúc điều khiển:
```csharp
// If-else
if (condition) { }
else if (condition) { }
else { }

// Switch
switch (value) {
    case 1: break;
    default: break;
}

// Vòng lặp
for (int i = 0; i < 10; i++) { }
while (condition) { }
do { } while (condition);
```

### Kiểu dữ liệu cơ bản:
- Số nguyên: `int`, `long`
- Số thực: `float`, `double`, `decimal`
- Ký tự: `char`, `string`
- Logic: `bool`

### Best Practices:
- ✅ Đặt tên biến rõ ràng, mô tả mục đích
- ✅ Luôn khởi tạo biến trước khi dùng
- ✅ Dùng `const` cho giá trị không đổi
- ✅ Kiểm tra điều kiện trước khi thao tác (chia cho 0, null, etc.)
- ✅ Dùng string interpolation thay vì nối chuỗi

---

## 💡 Mẹo nhỏ
> [!TIP]
> Sử dụng `Console.ReadLine()` để đọc dữ liệu từ bàn phím và `int.Parse()` hoặc `Convert.ToInt32()` để chuyển đổi từ chuỗi sang số. Dùng `TryParse()` để an toàn hơn.

> [!NOTE]
> C# phân biệt chữ hoa chữ thường (Case-sensitive). `age` và `Age` là hai biến khác nhau.

> [!WARNING]
> Luôn kiểm tra chia cho 0 và null trước khi thao tác để tránh runtime exceptions.

---

## ✨ Điểm nhấn C# hiện đại so với Java

Mục này tóm tắt các khác biệt quan trọng, thể hiện sự hiện đại và năng suất của C# trong những khái niệm cơ bản. Xem bản so sánh đầy đủ tại: `00_so_sanh_csharp_vs_java.md`.

### 1) Top‑level statements (C# 9+)

Trong C#, bạn có thể viết chương trình mà không cần bao bọc bởi `class Program` và `Main` (phù hợp cho minh họa, mẫu ngắn):

```csharp
// file: Program.cs
Console.WriteLine("Xin chào DNU!");
```

Java yêu cầu phương thức `main` trong một lớp (trừ khi dùng JShell). Điều này giúp C# gọn hơn cho ví dụ/hackathon.

### 2) String Interpolation và Raw Strings

- C# có nội suy chuỗi trực tiếp bằng `$"{expr}"` và hỗ trợ chuỗi raw `""" ... """` (C# 11) cùng chuỗi verbatim `@"C:\\Data\\file.txt"`:

```csharp
var name = "An";
var msg = $"Hello, {name}!"; // Nội suy

var path = @"C:\Users\Public\Documents"; // Verbatim string

var json = """
{
    "name": "An",
    "age": 20
}
"""; // Raw string literal
```

- Java dùng `String.format` hoặc từ Java 15 có Text Blocks `""" ... """` cho chuỗi nhiều dòng; nội suy trực tiếp là tính năng đang ở trạng thái preview (String Templates, JDK mới).

### 3) Pattern Matching & Switch Expression

C# có `switch` dạng biểu thức, pattern matching mạnh (quan hệ, property, list patterns):

```csharp
int score = 85;
string grade = score switch
{
        >= 90 => "A",
        >= 80 => "B",
        >= 70 => "C",
        _     => "D"
};
```

Java 21 có pattern matching cho `switch`, nhưng cú pháp và độ bao phủ pattern khác; C# hiện tích hợp rộng rãi trong ngôn ngữ và thư viện.

### 4) File‑scoped namespace (C# 10)

Giảm thụt dòng và nhiễu:

```csharp
namespace MyApp; // Thay vì mở/đóng block

class Program { }
```

Java dùng `package` ở đầu file (tương tự về ý nghĩa), nhưng không có dạng “file‑scoped” rút gọn khối như C#.

### 5) Nullable Reference Types (NRT)

C# bật cảnh báo `null` ở mức biên dịch, tăng an toàn:

```csharp
#nullable enable
string? maybeName = Console.ReadLine();
if (!string.IsNullOrWhiteSpace(maybeName))
{
        Console.WriteLine($"Hi {maybeName}");
}
```

Java không có null‑safety ở mức ngôn ngữ; thường dựa vào annotation (`@Nullable`) và kiểm tra runtime hoặc dùng `Optional<T>` trong một số trường hợp.

