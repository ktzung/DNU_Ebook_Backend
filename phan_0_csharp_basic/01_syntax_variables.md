# 🟦 CHƯƠNG 01
# **CÚ PHÁP & BIẾN TRONG C#**

## 📖 1. Giới thiệu C#

**C# (C-Sharp)** là ngôn ngữ lập trình hướng đối tượng do Microsoft phát triển. Nó mạnh mẽ, an toàn kiểu dữ liệu (type-safe) và chạy trên nền tảng .NET.

### Cấu trúc cơ bản của một chương trình C#:
```csharp
using System;

namespace HelloWorld
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Xin chào DNU!");
        }
    }
}
```
- `using System;`: Khai báo sử dụng thư viện chuẩn.
- `namespace`: Gom nhóm các class liên quan.
- `static void Main`: Điểm bắt đầu của chương trình.

---

## 📦 2. Biến & Kiểu dữ liệu

### 2.1. Khai báo biến
```csharp
int age = 20;               // Số nguyên
double gpa = 3.5;           // Số thực
char grade = 'A';           // Ký tự (nháy đơn)
string name = "Nguyen Van A"; // Chuỗi (nháy kép)
bool isStudent = true;      // Logic (true/false)
```

### 2.2. Var keyword (Type Inference)
C# có thể tự suy luận kiểu dữ liệu:
```csharp
var city = "Da Nang"; // Trình biên dịch hiểu đây là string
var year = 2024;      // Trình biên dịch hiểu đây là int
```

---

## 🧮 3. Toán tử (Operators)

- **Toán học**: `+`, `-`, `*`, `/`, `%` (chia lấy dư)
- **So sánh**: `==`, `!=`, `>`, `<`, `>=`, `<=`
- **Logic**: `&&` (AND), `||` (OR), `!` (NOT)
- **Gán**: `=`, `+=`, `-=`, `++`, `--`

```csharp
int x = 10;
x++; // x thành 11
int y = x + 5; // y = 16
bool check = (x > 5) && (y < 20); // true
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

## 🧪 6. Bài tập thực hành

### Bài 1: Tính điểm trung bình
Viết chương trình nhập vào tên, điểm Toán, Lý, Hóa. Tính điểm trung bình và xếp loại.

### Bài 2: Kiểm tra số nguyên tố
Viết chương trình nhập vào một số nguyên n. Kiểm tra xem n có phải là số nguyên tố không.

### Bài 3: Bảng cửu chương
In ra bảng cửu chương từ 2 đến 9.

---

## 💡 Mẹo nhỏ
> [!TIP]
> Sử dụng `Console.ReadLine()` để đọc dữ liệu từ bàn phím và `int.Parse()` hoặc `Convert.ToInt32()` để chuyển đổi từ chuỗi sang số.

> [!NOTE]
> C# phân biệt chữ hoa chữ thường (Case-sensitive). `age` và `Age` là hai biến khác nhau.
