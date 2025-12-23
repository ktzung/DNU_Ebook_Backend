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

### 5.1. LINQ nâng cao (nhóm, join, tổng hợp, phân trang)

```csharp
// Dữ liệu mẫu
var orders = new List<Order>
{
    new(1, customerId: 10, total: 120m),
    new(2, customerId: 11, total: 80m),
    new(3, customerId: 10, total: 200m),
};
var customers = new List<Customer>
{
    new(10, "An"),
    new(11, "Binh"),
};

// Nhóm theo customerId và tính tổng
var totalsByCustomer = orders
    .GroupBy(o => o.CustomerId)
    .Select(g => new { CustomerId = g.Key, Total = g.Sum(x => x.Total) })
    .OrderByDescending(x => x.Total)
    .ToList();

// Join orders với customers
var orderViews = from o in orders
                 join c in customers on o.CustomerId equals c.Id
                 select new { o.Id, Customer = c.Name, o.Total };

// Phân trang (paging)
int page = 1, pageSize = 10;
var pageItems = orders
    .OrderBy(o => o.Id)
    .Skip((page - 1) * pageSize)
    .Take(pageSize)
    .ToList();

// Tập hợp: Distinct/Union/Intersect/Except
var a = new[] {1,2,3};
var b = new[] {3,4,5};
var union = a.Union(b).ToList();        // 1,2,3,4,5
var intersect = a.Intersect(b).ToList(); // 3
var except = a.Except(b).ToList();       // 1,2
```

### 5.2. SelectMany (flatten), Any/All, ToDictionary

```csharp
var coursesByStudent = new[]
{
    new { Name = "An", Courses = new[] {"Math","CS"} },
    new { Name = "Binh", Courses = new[] {"CS"} },
};

// Flatten danh sách khóa học
var allCourses = coursesByStudent.SelectMany(s => s.Courses)
                                 .Distinct()
                                 .ToList();

// Kiểm tra điều kiện tồn tại
bool hasExpensive = numbers.Any(n => n > 100);
bool allPositive = numbers.All(n => n > 0);

// Chuyển sang Dictionary
var dictById = orders.ToDictionary(o => o.Id, o => o);
```

### 5.3. IEnumerable vs IQueryable (quan trọng khi dùng EF Core)

- `IEnumerable<T>`: Thực thi trong bộ nhớ (in‑memory), các toán tử LINQ chạy trên dữ liệu đã nạp.
- `IQueryable<T>`: Biểu diễn cây biểu thức; EF Core dịch thành SQL và chạy tại database.

```csharp
// EF Core ví dụ
IQueryable<Product> query = db.Products
    .Where(p => p.Price > 100)
    .OrderBy(p => p.Name)
    .Select(p => new ProductDto { Id = p.Id, Name = p.Name });

// Thực thi truy vấn (gửi SQL xuống DB)
var pageData = await query.Skip(20).Take(10).ToListAsync();
```

Lưu ý: Gọi phương thức không thể dịch (ví dụ hàm tự viết) trong `IQueryable` sẽ ném lỗi hoặc kéo dữ liệu về bộ nhớ; hãy cố gắng dùng các toán tử LINQ/Method có thể dịch sang SQL.

### 5.4. Deferred Execution & Pitfalls

- LINQ đa số là deferred: chỉ thực thi khi cần kết quả (`ToList`, `Count`, `First`, `Any`...).
- Tránh gọi `ToList()` quá sớm trước khi hoàn thành pipeline (gây nạp dữ liệu thừa).
- Với tập dữ liệu lớn, ưu tiên `Select` chỉ các trường cần thiết, kết hợp `Skip/Take` để phân trang.

### 5.5. Hiệu năng & Kỹ thuật nâng cao

- Dùng `AsNoTracking()` khi chỉ đọc dữ liệu trong EF Core để giảm overhead.
- Sử dụng `Distinct`/`GroupBy` ở phía DB (IQueryable) thay vì lọc ở memory.
- Cân nhắc `Dapper` cho một số truy vấn read‑only tốc độ cao.
- Với mảng/chuỗi lớn, cân nhắc `Span<T>` để thao tác hiệu quả.

## ❌ 6. CÁC LỖI THƯỜNG GẶP

### ❌ Lỗi 1: Index out of range

```csharp
// ❌ Vấn đề: Truy cập index không tồn tại
List<int> numbers = new List<int> { 1, 2, 3 };
int value = numbers[10]; // IndexOutOfRangeException

// ✅ Giải pháp: Kiểm tra index hoặc dùng Count
if (index >= 0 && index < numbers.Count)
{
    int value = numbers[index];
}

// Hoặc dùng TryGetValue cho Dictionary
Dictionary<string, int> dict = new();
if (dict.TryGetValue("key", out int value))
{
    Console.WriteLine(value);
}
```

**🔍 Giải thích:** Luôn kiểm tra index/Count trước khi truy cập phần tử trong collection.

---

### ❌ Lỗi 2: Thay đổi collection trong khi đang duyệt

```csharp
// ❌ Vấn đề: Thay đổi collection trong foreach
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
foreach (var num in numbers)
{
    if (num % 2 == 0)
        numbers.Remove(num); // InvalidOperationException
}

// ✅ Giải pháp: Duyệt ngược hoặc tạo collection mới
// Cách 1: Duyệt ngược
for (int i = numbers.Count - 1; i >= 0; i--)
{
    if (numbers[i] % 2 == 0)
        numbers.RemoveAt(i);
}

// Cách 2: Tạo collection mới
var filtered = numbers.Where(n => n % 2 != 0).ToList();
```

**🔍 Giải thích:** Không thể thay đổi collection trong khi đang duyệt bằng foreach.

---

### ❌ Lỗi 3: Key không tồn tại trong Dictionary

```csharp
// ❌ Vấn đề: Key không tồn tại
Dictionary<string, int> dict = new();
int value = dict["nonexistent"]; // KeyNotFoundException

// ✅ Giải pháp: Kiểm tra ContainsKey hoặc TryGetValue
if (dict.ContainsKey("key"))
{
    int value = dict["key"];
}

// Hoặc dùng TryGetValue
if (dict.TryGetValue("key", out int value))
{
    Console.WriteLine(value);
}
```

**🔍 Giải thích:** Luôn kiểm tra key tồn tại trước khi truy cập Dictionary.

---

### ❌ Lỗi 4: Gọi ToList() quá sớm với IQueryable

```csharp
// ❌ Vấn đề: ToList() quá sớm, không tận dụng SQL
var products = db.Products.ToList(); // Load tất cả vào memory
var filtered = products.Where(p => p.Price > 100).ToList(); // Filter ở memory

// ✅ Giải pháp: Giữ IQueryable, filter ở DB
var filtered = db.Products
    .Where(p => p.Price > 100) // Filter ở SQL
    .ToList(); // Chỉ load kết quả đã filter
```

**🔍 Giải thích:** Với EF Core, giữ IQueryable để filter ở database, không filter ở memory.

---

### ❌ Lỗi 5: Quên null check với LINQ

```csharp
// ❌ Vấn đề: Có thể null
List<string> names = null;
var count = names.Count(); // NullReferenceException

// ✅ Giải pháp: Kiểm tra null hoặc dùng null-conditional
if (names != null)
{
    var count = names.Count();
}

// Hoặc
var count = names?.Count() ?? 0;
```

**🔍 Giải thích:** Luôn kiểm tra null trước khi dùng LINQ trên collection.

---

### ❌ Lỗi 6: Dùng First() thay vì FirstOrDefault()

```csharp
// ❌ Vấn đề: First() throw exception nếu không có phần tử
List<int> numbers = new List<int>();
int first = numbers.First(); // InvalidOperationException

// ✅ Giải pháp: Dùng FirstOrDefault()
int? first = numbers.FirstOrDefault(); // Trả về default (0) nếu không có
if (first.HasValue)
{
    Console.WriteLine(first.Value);
}
```

**🔍 Giải thích:** `First()` throw exception nếu không có phần tử, `FirstOrDefault()` trả về default.

---

### ❌ Lỗi 7: Không hiểu Deferred Execution

```csharp
// ❌ Vấn đề: Tưởng đã thực thi nhưng chưa
var query = numbers.Where(n => n > 5); // Chưa thực thi
numbers.Add(10); // Thêm sau
var result = query.ToList(); // Kết quả có 10 (không như mong đợi)

// ✅ Giải pháp: Hiểu rõ deferred execution
var query = numbers.Where(n => n > 5).ToList(); // Thực thi ngay
numbers.Add(10); // Thêm sau
// result không có 10
```

**🔍 Giải thích:** LINQ có deferred execution, chỉ thực thi khi cần (ToList, Count, First, etc.).

---

### ❌ Lỗi 8: So sánh reference thay vì value với Dictionary key

```csharp
// ❌ Vấn đề: Key là object, so sánh reference
Dictionary<Student, int> scores = new();
var s1 = new Student("An", 20);
var s2 = new Student("An", 20);
scores[s1] = 100;
int score = scores[s2]; // KeyNotFoundException (khác reference)

// ✅ Giải pháp: Dùng record hoặc override Equals/GetHashCode
public record Student(string Name, int Age);
// Hoặc
public class Student
{
    public override bool Equals(object obj) { /* ... */ }
    public override int GetHashCode() { /* ... */ }
}
```

**🔍 Giải thích:** Dictionary dùng GetHashCode và Equals để tìm key, phải override nếu dùng custom class.

---

## 🎯 7. CASE STUDY / VÍ DỤ THỰC TẾ

### Case Study 1: Hệ thống quản lý đơn hàng với LINQ

**Yêu cầu:** Quản lý đơn hàng, tính tổng tiền, tìm đơn hàng lớn nhất.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

public class Order
{
    public int Id { get; set; }
    public string CustomerName { get; set; }
    public decimal Total { get; set; }
    public DateTime OrderDate { get; set; }
    public string Status { get; set; }
}

public class OrderManager
{
    private List<Order> _orders = new();
    
    public void AddOrder(Order order)
    {
        _orders.Add(order);
    }
    
    // Tìm đơn hàng lớn nhất
    public Order GetLargestOrder()
    {
        return _orders.OrderByDescending(o => o.Total).FirstOrDefault();
    }
    
    // Tính tổng tiền theo khách hàng
    public Dictionary<string, decimal> GetTotalByCustomer()
    {
        return _orders
            .GroupBy(o => o.CustomerName)
            .ToDictionary(g => g.Key, g => g.Sum(o => o.Total));
    }
    
    // Lấy đơn hàng trong tháng
    public List<Order> GetOrdersThisMonth()
    {
        var now = DateTime.Now;
        return _orders
            .Where(o => o.OrderDate.Year == now.Year && 
                       o.OrderDate.Month == now.Month)
            .OrderByDescending(o => o.OrderDate)
            .ToList();
    }
    
    // Thống kê theo trạng thái
    public void DisplayStatistics()
    {
        var stats = _orders
            .GroupBy(o => o.Status)
            .Select(g => new { Status = g.Key, Count = g.Count() })
            .ToList();
        
        Console.WriteLine("=== THỐNG KÊ ĐƠN HÀNG ===");
        foreach (var stat in stats)
        {
            Console.WriteLine($"{stat.Status}: {stat.Count} đơn");
        }
        
        Console.WriteLine($"\nTổng số đơn: {_orders.Count}");
        Console.WriteLine($"Tổng giá trị: {_orders.Sum(o => o.Total):C}");
    }
}

// Sử dụng
class Program
{
    static void Main()
    {
        var manager = new OrderManager();
        
        manager.AddOrder(new Order 
        { 
            Id = 1, 
            CustomerName = "Nguyen Van A", 
            Total = 500000, 
            OrderDate = DateTime.Now,
            Status = "Completed"
        });
        
        manager.AddOrder(new Order 
        { 
            Id = 2, 
            CustomerName = "Tran Thi B", 
            Total = 1200000, 
            OrderDate = DateTime.Now.AddDays(-5),
            Status = "Pending"
        });
        
        manager.DisplayStatistics();
        
        var largest = manager.GetLargestOrder();
        Console.WriteLine($"\nĐơn hàng lớn nhất: {largest?.Total:C}");
    }
}
```

**Giải thích:**
- Sử dụng LINQ để query dữ liệu phức tạp
- GroupBy để thống kê
- OrderBy để sắp xếp
- ToDictionary để chuyển đổi format

---

### Case Study 2: Hệ thống từ điển Anh-Việt

**Yêu cầu:** Tra cứu từ, thêm từ mới, tìm từ gần giống.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

public class Dictionary
{
    private Dictionary<string, string> _words = new();
    
    public void AddWord(string english, string vietnamese)
    {
        if (_words.ContainsKey(english.ToLower()))
        {
            Console.WriteLine($"Từ '{english}' đã tồn tại!");
            return;
        }
        _words[english.ToLower()] = vietnamese;
    }
    
    public string Lookup(string english)
    {
        if (_words.TryGetValue(english.ToLower(), out string meaning))
        {
            return meaning;
        }
        return "Không tìm thấy trong từ điển";
    }
    
    // Tìm từ gần giống (fuzzy search)
    public List<string> FindSimilar(string searchTerm)
    {
        var lower = searchTerm.ToLower();
        return _words.Keys
            .Where(word => word.Contains(lower) || lower.Contains(word))
            .Take(5) // Lấy 5 từ đầu tiên
            .ToList();
    }
    
    // Liệt kê tất cả từ
    public void ListAllWords()
    {
        Console.WriteLine("=== TỪ ĐIỂN ANH-VIỆT ===\n");
        foreach (var word in _words.OrderBy(w => w.Key))
        {
            Console.WriteLine($"{word.Key}: {word.Value}");
        }
    }
    
    // Thống kê
    public void DisplayStats()
    {
        Console.WriteLine($"Tổng số từ: {_words.Count}");
        var avgLength = _words.Values.Average(v => v.Length);
        Console.WriteLine($"Độ dài trung bình nghĩa: {avgLength:F1} ký tự");
    }
}

// Sử dụng
class Program
{
    static void Main()
    {
        var dict = new Dictionary();
        
        dict.AddWord("Hello", "Xin chào");
        dict.AddWord("World", "Thế giới");
        dict.AddWord("Computer", "Máy tính");
        dict.AddWord("Program", "Chương trình");
        
        Console.WriteLine(dict.Lookup("Hello"));
        Console.WriteLine(dict.Lookup("Unknown"));
        
        Console.WriteLine("\nTừ tương tự 'pro':");
        foreach (var word in dict.FindSimilar("pro"))
        {
            Console.WriteLine($"- {word}");
        }
        
        dict.DisplayStats();
    }
}
```

**Best practices:**
- Dùng Dictionary cho tra cứu nhanh O(1)
- Case-insensitive search
- Fuzzy search với LINQ
- Validation trước khi thêm

---

### Case Study 3: Hệ thống quản lý điểm số sinh viên

**Yêu cầu:** Tính điểm trung bình, xếp hạng, thống kê.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

public class Student
{
    public string Name { get; set; }
    public string StudentId { get; set; }
    public List<double> Scores { get; set; } = new();
    
    public double AverageScore => Scores.Any() ? Scores.Average() : 0;
}

public class GradeManager
{
    private List<Student> _students = new();
    
    public void AddStudent(Student student)
    {
        _students.Add(student);
    }
    
    // Xếp hạng sinh viên
    public List<Student> GetRankedStudents()
    {
        return _students
            .OrderByDescending(s => s.AverageScore)
            .ToList();
    }
    
    // Tìm sinh viên có điểm cao nhất
    public Student GetTopStudent()
    {
        return _students
            .OrderByDescending(s => s.AverageScore)
            .FirstOrDefault();
    }
    
    // Thống kê điểm
    public void DisplayStatistics()
    {
        if (!_students.Any())
        {
            Console.WriteLine("Chưa có dữ liệu");
            return;
        }
        
        var allScores = _students.SelectMany(s => s.Scores).ToList();
        
        Console.WriteLine("=== THỐNG KÊ ĐIỂM ===");
        Console.WriteLine($"Tổng số sinh viên: {_students.Count}");
        Console.WriteLine($"Điểm cao nhất: {allScores.Max():F2}");
        Console.WriteLine($"Điểm thấp nhất: {allScores.Min():F2}");
        Console.WriteLine($"Điểm trung bình: {allScores.Average():F2}");
        
        // Phân loại
        var excellent = _students.Count(s => s.AverageScore >= 8.0);
        var good = _students.Count(s => s.AverageScore >= 6.5 && s.AverageScore < 8.0);
        var average = _students.Count(s => s.AverageScore >= 5.0 && s.AverageScore < 6.5);
        var poor = _students.Count(s => s.AverageScore < 5.0);
        
        Console.WriteLine($"\nPhân loại:");
        Console.WriteLine($"  Giỏi (>=8.0): {excellent}");
        Console.WriteLine($"  Khá (6.5-8.0): {good}");
        Console.WriteLine($"  Trung bình (5.0-6.5): {average}");
        Console.WriteLine($"  Yếu (<5.0): {poor}");
    }
    
    // Tìm sinh viên có điểm >= ngưỡng
    public List<Student> GetStudentsAbove(double threshold)
    {
        return _students
            .Where(s => s.AverageScore >= threshold)
            .OrderByDescending(s => s.AverageScore)
            .ToList();
    }
}

// Sử dụng
class Program
{
    static void Main()
    {
        var manager = new GradeManager();
        
        manager.AddStudent(new Student 
        { 
            Name = "Nguyen Van A", 
            StudentId = "SV001",
            Scores = new List<double> { 8.5, 9.0, 8.0, 7.5 }
        });
        
        manager.AddStudent(new Student 
        { 
            Name = "Tran Thi B", 
            StudentId = "SV002",
            Scores = new List<double> { 6.0, 7.0, 6.5, 7.5 }
        });
        
        manager.DisplayStatistics();
        
        Console.WriteLine("\n=== XẾP HẠNG ===");
        var ranked = manager.GetRankedStudents();
        for (int i = 0; i < ranked.Count; i++)
        {
            Console.WriteLine($"{i + 1}. {ranked[i].Name}: {ranked[i].AverageScore:F2}");
        }
    }
}
```

**Giải thích:**
- LINQ để tính toán phức tạp
- SelectMany để flatten nested collections
- GroupBy và Count để thống kê
- OrderBy để sắp xếp

---

## ✅ 8. BEST PRACTICES

### 8.1. Chọn đúng Collection type

```csharp
// ✅ List<T> cho danh sách có thể thêm/xóa
List<string> names = new();

// ✅ Dictionary<TKey, TValue> cho tra cứu nhanh
Dictionary<string, int> scores = new();

// ✅ HashSet<T> cho tập hợp không trùng
HashSet<int> uniqueNumbers = new();

// ✅ Queue<T> cho FIFO
Queue<string> tasks = new();

// ✅ Stack<T> cho LIFO
Stack<string> history = new();
```

### 8.2. Sử dụng LINQ hiệu quả

```csharp
// ❌ Nhiều vòng lặp
var result = new List<int>();
foreach (var n in numbers)
{
    if (n > 5)
        result.Add(n);
}
result.Sort();

// ✅ LINQ ngắn gọn
var result = numbers.Where(n => n > 5).OrderBy(n => n).ToList();
```

### 8.3. Tránh gọi ToList() quá sớm

```csharp
// ❌ Load tất cả vào memory
var all = db.Products.ToList();
var filtered = all.Where(p => p.Price > 100).ToList();

// ✅ Filter ở database
var filtered = db.Products
    .Where(p => p.Price > 100)
    .ToList();
```

### 8.4. Dùng TryGetValue cho Dictionary

```csharp
// ❌ Kiểm tra 2 lần
if (dict.ContainsKey("key"))
{
    var value = dict["key"];
}

// ✅ Hiệu quả hơn
if (dict.TryGetValue("key", out var value))
{
    // Dùng value
}
```

### 8.5. Sử dụng null-conditional với collections

```csharp
// ❌ Có thể null
int count = list.Count; // NullReferenceException nếu list null

// ✅ An toàn
int count = list?.Count ?? 0;
```

---

## 🧪 9. BÀI TẬP THỰC HÀNH

### Bài 1: Quản lý điểm số

Tạo một `List<double>` chứa điểm số của 10 sinh viên.

- Tìm điểm cao nhất, thấp nhất.
- Tính điểm trung bình.
- Tìm tất cả sinh viên có điểm >= 5.

<details>
<summary>💡 Đáp án</summary>

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        var scores = new List<double> { 8.5, 7.0, 6.5, 9.0, 5.5, 4.0, 8.0, 7.5, 6.0, 9.5 };
        
        double max = scores.Max();
        double min = scores.Min();
        double average = scores.Average();
        var passed = scores.Where(s => s >= 5.0).ToList();
        
        Console.WriteLine($"Điểm cao nhất: {max}");
        Console.WriteLine($"Điểm thấp nhất: {min}");
        Console.WriteLine($"Điểm trung bình: {average:F2}");
        Console.WriteLine($"Số sinh viên đậu (>=5): {passed.Count}");
        Console.WriteLine($"Danh sách điểm đậu: {string.Join(", ", passed)}");
    }
}
```
</details>

---

### Bài 2: Từ điển Anh-Việt

Tạo chương trình từ điển đơn giản dùng `Dictionary`.

- Nhập từ tiếng Anh -> Trả về nghĩa tiếng Việt.
- Nếu không tìm thấy -> Thông báo "Chưa có trong từ điển".

<details>
<summary>💡 Đáp án</summary>

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var dictionary = new Dictionary<string, string>
        {
            { "hello", "Xin chào" },
            { "world", "Thế giới" },
            { "computer", "Máy tính" },
            { "program", "Chương trình" }
        };
        
        Console.Write("Nhập từ tiếng Anh: ");
        string word = Console.ReadLine()?.ToLower() ?? "";
        
        if (dictionary.TryGetValue(word, out string meaning))
        {
            Console.WriteLine($"Nghĩa: {meaning}");
        }
        else
        {
            Console.WriteLine("Chưa có trong từ điển");
        }
    }
}
```
</details>

---

### Bài 3: Thống kê từ trong câu

Nhập một câu, đếm số lần xuất hiện của mỗi từ.

<details>
<summary>💡 Đáp án</summary>

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        Console.Write("Nhập câu: ");
        string sentence = Console.ReadLine() ?? "";
        
        var words = sentence
            .ToLower()
            .Split(new[] { ' ', '.', ',', '!', '?' }, StringSplitOptions.RemoveEmptyEntries);
        
        var wordCount = words
            .GroupBy(w => w)
            .ToDictionary(g => g.Key, g => g.Count());
        
        Console.WriteLine("\n=== THỐNG KÊ TỪ ===");
        foreach (var item in wordCount.OrderByDescending(w => w.Value))
        {
            Console.WriteLine($"{item.Key}: {item.Value} lần");
        }
    }
}
```
</details>

---

## 🧪 10. MINI TEST

1. **Collection nào phù hợp để tra cứu nhanh theo key?**
   - A. `List<T>`
   - B. `Dictionary<TKey, TValue>` ✅
   - C. `Array`
   - D. `Queue<T>`

2. **LINQ method nào để lọc dữ liệu?**
   - A. `Select`
   - B. `Where` ✅
   - C. `OrderBy`
   - D. `GroupBy`

3. **Kết quả của `new List<int>().First()` là gì?**
   - A. 0
   - B. null
   - C. Throw exception ✅
   - D. -1

4. **IQueryable khác IEnumerable như thế nào?**
   - A. IQueryable nhanh hơn
   - B. IQueryable có thể dịch sang SQL ✅
   - C. Không khác gì
   - D. IEnumerable tốt hơn

5. **Method nào an toàn để lấy phần tử đầu tiên?**
   - A. `First()`
   - B. `FirstOrDefault()` ✅
   - C. `Take(1)`
   - D. `Single()`

<details>
<summary>💡 Đáp án</summary>

1. **B** - Dictionary có O(1) lookup time theo key
2. **B** - `Where` dùng để lọc dữ liệu theo điều kiện
3. **C** - `First()` throw exception nếu collection rỗng
4. **B** - IQueryable có thể dịch LINQ sang SQL (EF Core)
5. **B** - `FirstOrDefault()` trả về default nếu không có phần tử
</details>

---

## 📝 11. QUICK NOTES

### Collections phổ biến:
- **List<T>**: Danh sách động, có thể thêm/xóa
- **Dictionary<TKey, TValue>**: Key-value pairs, tra cứu nhanh
- **HashSet<T>**: Tập hợp không trùng
- **Queue<T>**: FIFO (First In First Out)
- **Stack<T>**: LIFO (Last In First Out)

### LINQ Methods quan trọng:
- **Filtering**: `Where`, `OfType`
- **Projection**: `Select`, `SelectMany`
- **Sorting**: `OrderBy`, `OrderByDescending`, `ThenBy`
- **Grouping**: `GroupBy`
- **Aggregation**: `Count`, `Sum`, `Average`, `Min`, `Max`
- **Element**: `First`, `FirstOrDefault`, `Last`, `Single`
- **Partitioning**: `Skip`, `Take`
- **Set**: `Distinct`, `Union`, `Intersect`, `Except`

### IEnumerable vs IQueryable:
- **IEnumerable**: In-memory, thực thi ngay
- **IQueryable**: Deferred, có thể dịch sang SQL

### Best Practices:
- ✅ Chọn đúng collection type cho use case
- ✅ Dùng LINQ thay vì vòng lặp thủ công
- ✅ Tránh gọi ToList() quá sớm với IQueryable
- ✅ Dùng TryGetValue cho Dictionary
- ✅ Kiểm tra null trước khi dùng LINQ
- ✅ Dùng FirstOrDefault thay vì First khi có thể null

### Lỗi thường gặp:
- ❌ Index out of range
- ❌ Thay đổi collection trong foreach
- ❌ Key không tồn tại trong Dictionary
- ❌ Gọi ToList() quá sớm
- ❌ Quên null check

---

## 💡 Mẹo nhỏ

> [!TIP]
> Luôn sử dụng `List<T>` thay vì Array trừ khi bạn biết chắc chắn số lượng phần tử không đổi và cần tối ưu bộ nhớ cực đại.

> [!NOTE]
> LINQ là công cụ cực kỳ quan trọng trong .NET Core, đặc biệt khi làm việc với Database (Entity Framework). Hãy luyện tập kỹ phần này.

> [!WARNING]
> Không thay đổi collection trong khi đang duyệt bằng foreach. Dùng for loop ngược hoặc tạo collection mới.

---

## ✨ Điểm nhấn Collections & Generics: C# vs Java

Tập trung vào reified generics, LINQ, và khả năng biểu đạt cao của C#. Xem thêm: `00_so_sanh_csharp_vs_java.md`.

### 1) Generics: Reified (C#) vs Erasure (Java)

- C# giữ thông tin kiểu cho closed generics và hỗ trợ ràng buộc mạnh `where`:

```csharp
public T CreateDefault<T>() where T : class, new() => new T();

bool IsListOfInt(object obj)
    => obj?.GetType() == typeof(List<int>); // Phân biệt List<int> và List<string>
```

- Java xóa kiểu (type erasure), không thể phân biệt `List<Integer>` vs `List<String>` lúc runtime và không thể `new T()` trực tiếp.

### 2) LINQ vs Stream API

- C# LINQ hỗ trợ cả method chain và query syntax, tích hợp sâu với `IEnumerable<T>` và `IQueryable<T>` (biểu thức có thể dịch sang SQL bởi EF Core):

```csharp
// LINQ method chain
var names = users.Where(u => u.Age >= 18)
                 .OrderBy(u => u.Name)
                 .Select(u => u.Name)
                 .ToList();

// LINQ query syntax
var teenNames = from u in users
                where u.Age is >= 13 and <= 19
                orderby u.Name
                select u.Name;
```

- Java Stream mạnh về xử lý pipeline nhưng không có biểu thức truy vấn tích hợp; ORM thường dùng JPQL/HQL (chuỗi) hoặc Criteria API (code verbose) thay vì biểu thức lambda có thể dịch như `IQueryable`.

### 3) Indexer, Deconstruction, Records trong Collections

- C# hỗ trợ indexer cho kiểu tự định nghĩa (Java không có cú pháp indexer):

```csharp
public class Bag<T>
{
    private readonly List<T> _items = new();
    public T this[int i] => _items[i];
}
```

- Deconstruction/Tuple giúp viết code truy vấn gọn:

```csharp
var (min, max) = (numbers.Min(), numbers.Max());
```

- `record`/value equality hữu ích khi dùng làm key (với `record struct` có thể là value type):

```csharp
public readonly record struct Coord(int X, int Y);
var dict = new Dictionary<Coord, string>();
dict[new(1,2)] = "A";
```

### 4) Hiệu năng & Bộ nhớ

- C# có `Span<T>`/`Memory<T>` để thao tác vùng nhớ liên tục, giảm allocation khi xử lý mảng/chuỗi lớn (Java tương đương thường phải dùng `ByteBuffer`/off‑heap libs):

```csharp
ReadOnlySpan<char> s = "abcdef";
var slice = s.Slice(1, 3); // "bcd" mà không cấp phát chuỗi mới
```

### 5) Bất đồng bộ với Collections

- C# hỗ trợ `IAsyncEnumerable<T>` giúp duyệt dữ liệu bất đồng bộ (streaming) tự nhiên:

```csharp
await foreach (var item in repo.StreamAllAsync())
{
    Console.WriteLine(item);
}
```

- Java có thể dùng reactive streams (`Flow`, Project Reactor, RxJava) hoặc `CompletableFuture` kết hợp, nhưng không có cú pháp `await foreach` tích hợp ngôn ngữ.

