# 🟦 CHƯƠNG 03
# **COLLECTIONS & GENERICS**

## 📖 1. Giới thiệu

Trong thực tế, ta thường làm việc với một nhóm đối tượng (danh sách sinh viên, danh sách sản phẩm) thay vì từng biến lẻ tẻ. C# cung cấp **Collections** để giải quyết việc này.

---

## 📦 2. Mảng (Array)

Mảng có kích thước cố định, phải khai báo số lượng phần tử ngay từ đầu.

```csharp
// Khai báo mảng số nguyên có 5 phần tử
int[] numbers = new int[5];
numbers[0] = 10;
numbers[1] = 20;

// Khai báo và khởi tạo
string[] fruits = { "Apple", "Banana", "Orange" };

// Duyệt mảng
foreach (var fruit in fruits)
{
    Console.WriteLine(fruit);
}
```
**Nhược điểm**: Không thể thêm/bớt phần tử sau khi tạo.

---

## 📋 3. List<T> (Generic List)

`List<T>` là collection phổ biến nhất, kích thước động (có thể co giãn).
Nằm trong namespace `System.Collections.Generic`.

```csharp
List<string> names = new List<string>();

// Thêm phần tử
names.Add("An");
names.Add("Binh");

// Chèn vào vị trí
names.Insert(1, "Cuong");

// Xóa phần tử
names.Remove("An");

// Truy cập
Console.WriteLine(names[0]); // Cuong
```

### Tại sao gọi là Generic?
`<T>` là tham số kiểu. `List<int>` chỉ chứa số, `List<string>` chỉ chứa chuỗi. Giúp an toàn kiểu dữ liệu và hiệu năng cao hơn `ArrayList` cũ.

---

## 🔑 4. Dictionary<TKey, TValue>

Lưu trữ dữ liệu dưới dạng cặp **Key - Value**. Key là duy nhất.

```csharp
Dictionary<string, string> phones = new Dictionary<string, string>();

// Thêm
phones.Add("Alice", "0901234567");
phones.Add("Bob", "0987654321");

// Truy cập theo Key
if (phones.ContainsKey("Alice"))
{
    Console.WriteLine(phones["Alice"]);
}

// Duyệt
foreach (var item in phones)
{
    Console.WriteLine($"{item.Key}: {item.Value}");
}
```

---

## 🔄 5. LINQ (Language Integrated Query)

LINQ giúp truy vấn dữ liệu từ Collections cực kỳ mạnh mẽ (giống SQL).

```csharp
List<int> numbers = new List<int> { 1, 5, 8, 10, 3, 7 };

// Lấy các số chẵn và lớn hơn 5
var result = numbers.Where(n => n % 2 == 0 && n > 5).ToList();
// Kết quả: 8, 10

// Sắp xếp giảm dần
var sorted = numbers.OrderByDescending(n => n).ToList();

// Lấy số đầu tiên
var first = numbers.FirstOrDefault();
```

---

## 🧪 6. Bài tập thực hành

### Bài 1: Quản lý điểm số
Tạo một `List<double>` chứa điểm số của 10 sinh viên.
- Tìm điểm cao nhất, thấp nhất.
- Tính điểm trung bình.
- Tìm tất cả sinh viên có điểm >= 5.

### Bài 2: Từ điển Anh-Việt
Tạo chương trình từ điển đơn giản dùng `Dictionary`.
- Nhập từ tiếng Anh -> Trả về nghĩa tiếng Việt.
- Nếu không tìm thấy -> Thông báo "Chưa có trong từ điển".

---

## 💡 Mẹo nhỏ
> [!TIP]
> Luôn sử dụng `List<T>` thay vì Array trừ khi bạn biết chắc chắn số lượng phần tử không đổi và cần tối ưu bộ nhớ cực đại.

> [!NOTE]
> LINQ là công cụ cực kỳ quan trọng trong .NET Core, đặc biệt khi làm việc với Database (Entity Framework). Hãy luyện tập kỹ phần này.
