# 🟦 CHƯƠNG 02
# **LẬP TRÌNH HƯỚNG ĐỐI TƯỢNG (OOP)**

## 📖 1. Giới thiệu OOP

**OOP (Object-Oriented Programming)** là mô hình lập trình dựa trên khái niệm "đối tượng" chứa dữ liệu (thuộc tính) và mã (phương thức).

### 🧠 Tại sao cần học OOP?

OOP giúp:
- **Tổ chức code tốt hơn**: Chia nhỏ thành các class, dễ quản lý
- **Tái sử dụng code**: Kế thừa giúp tránh lặp lại
- **Bảo mật dữ liệu**: Encapsulation che giấu chi tiết bên trong
- **Dễ bảo trì**: Thay đổi một class không ảnh hưởng class khác

### 🎒 Ví dụ đời sống

Hãy tưởng tượng **Class** như **bản thiết kế xe hơi**:
- **Class Car** = Bản thiết kế (có thuộc tính: màu, số chỗ, động cơ)
- **Object** = Chiếc xe cụ thể được sản xuất từ bản thiết kế
- **Method** = Chức năng của xe (khởi động, tăng tốc, phanh)

### 4 Tính chất cơ bản của OOP:
1. **Đóng gói (Encapsulation)**: Che giấu dữ liệu bên trong, chỉ cho phép truy cập qua methods.
2. **Kế thừa (Inheritance)**: Class con kế thừa từ Class cha, tránh lặp code.
3. **Đa hình (Polymorphism)**: Một hành động có thể thực hiện theo nhiều cách khác nhau.
4. **Trừu tượng (Abstraction)**: Ẩn chi tiết cài đặt phức tạp, chỉ hiện interface đơn giản.

---

## 🏗️ 2. Class và Object

- **Class**: Bản thiết kế (Blueprint).
- **Object**: Thực thể cụ thể được tạo từ Class.

```csharp
public class Student
{
    // Fields (Thuộc tính)
    public string Name;
    public int Age;

    // Constructor (Hàm khởi tạo)
    public Student(string name, int age)
    {
        Name = name;
        Age = age;
    }

    // Method (Phương thức)
    public void Introduce()
    {
        Console.WriteLine($"Tôi tên là {Name}, {Age} tuổi.");
    }
}

// Sử dụng
Student s1 = new Student("Nam", 20);
s1.Introduce();
```

---

## 🔒 3. Tính đóng gói (Encapsulation)

Sử dụng **Access Modifiers** (public, private, protected) và **Properties** (get/set) để bảo vệ dữ liệu.

```csharp
public class BankAccount
{
    private decimal _balance; // Biến private, không thể truy cập từ ngoài

    public decimal Balance // Property công khai
    {
        get { return _balance; }
        // Chỉ cho phép đọc, không cho set từ ngoài (read-only)
    }

    public void Deposit(decimal amount)
    {
        if (amount > 0)
        {
            _balance += amount;
        }
    }
}
```

---

## 🧬 4. Tính kế thừa (Inheritance)

Dùng dấu hai chấm `:` để kế thừa.

```csharp
// Class cha
public class Animal
{
    public void Eat()
    {
        Console.WriteLine("Đang ăn...");
    }
}

// Class con
public class Dog : Animal
{
    public void Bark()
    {
        Console.WriteLine("Gâu gâu!");
    }
}

// Sử dụng
Dog milu = new Dog();
milu.Eat(); // Kế thừa từ Animal
milu.Bark(); // Của riêng Dog
```

---

## 🎭 5. Tính đa hình (Polymorphism)

### 5.1. Overriding (Ghi đè)
Dùng từ khóa `virtual` ở cha và `override` ở con.

```csharp
public class Animal
{
    public virtual void MakeSound()
    {
        Console.WriteLine("Tiếng kêu động vật");
    }
}

public class Cat : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Meo meo");
    }
}
```

### 5.2. Overloading (Nạp chồng)
Cùng tên hàm nhưng khác tham số.

```csharp
public class Calculator
{
    public int Add(int a, int b) => a + b;
    public double Add(double a, double b) => a + b;
    public int Add(int a, int b, int c) => a + b + c;
}
```

---

## 🧩 6. Interface và Abstract Class

### 6.1. Abstract Class
Lớp trừu tượng, không thể tạo object, có thể chứa hàm chưa cài đặt.

```csharp
public abstract class Shape
{
    public abstract double GetArea(); // Hàm trừu tượng
}
```

### 6.2. Interface
Bản hợp đồng, chỉ chứa định nghĩa hàm, không chứa code cài đặt (trước C# 8).

```csharp
public interface IMovable
{
    void Move();
}

public class Car : IMovable
{
    public void Move()
    {
        Console.WriteLine("Xe chạy...");
    }
}
```

---

## ❌ 7. CÁC LỖI THƯỜNG GẶP

### ❌ Lỗi 1: Quên khởi tạo object trước khi dùng

```csharp
// ❌ Vấn đề: Object chưa được tạo
Student student;
student.Name = "An"; // Lỗi: Use of unassigned local variable

// ✅ Giải pháp: Tạo object trước
Student student = new Student("An", 20);
// Hoặc
Student student = new Student { Name = "An", Age = 20 };
```

**🔍 Giải thích:** Phải tạo object bằng `new` trước khi sử dụng các thuộc tính/phương thức.

---

### ❌ Lỗi 2: Nhầm lẫn giữa `virtual` và `override`

```csharp
// ❌ Vấn đề: Quên virtual ở class cha
public class Animal
{
    public void MakeSound() { } // Thiếu virtual
}

public class Dog : Animal
{
    public override void MakeSound() { } // Lỗi: Cannot override non-virtual method
}

// ✅ Giải pháp: Thêm virtual ở class cha
public class Animal
{
    public virtual void MakeSound() { } // ✅
}

public class Dog : Animal
{
    public override void MakeSound() { } // ✅
}
```

**🔍 Giải thích:** Muốn override method, class cha phải có `virtual` hoặc `abstract`.

---

### ❌ Lỗi 3: Truy cập private field từ bên ngoài

```csharp
// ❌ Vấn đề: Truy cập private field
public class BankAccount
{
    private decimal _balance;
}

var account = new BankAccount();
account._balance = 1000; // Lỗi: Cannot access private field

// ✅ Giải pháp: Dùng public property hoặc method
public class BankAccount
{
    private decimal _balance;
    
    public decimal Balance { get; private set; } // ✅
    public void Deposit(decimal amount) { _balance += amount; } // ✅
}
```

**🔍 Giải thích:** Private fields chỉ có thể truy cập trong cùng class. Dùng properties hoặc methods để truy cập từ bên ngoài.

---

### ❌ Lỗi 4: Quên implement interface methods

```csharp
// ❌ Vấn đề: Class không implement đầy đủ interface
public interface IShape
{
    double GetArea();
    double GetPerimeter();
}

public class Rectangle : IShape
{
    public double GetArea() { return 10; }
    // Quên GetPerimeter() → Lỗi compile
}

// ✅ Giải pháp: Implement tất cả methods
public class Rectangle : IShape
{
    public double GetArea() { return 10; }
    public double GetPerimeter() { return 20; } // ✅
}
```

**🔍 Giải thích:** Class phải implement TẤT CẢ methods của interface.

---

### ❌ Lỗi 5: Tạo object từ abstract class

```csharp
// ❌ Vấn đề: Không thể tạo object từ abstract class
public abstract class Shape
{
    public abstract double GetArea();
}

var shape = new Shape(); // Lỗi: Cannot create instance of abstract class

// ✅ Giải pháp: Tạo object từ class con
public class Rectangle : Shape
{
    public override double GetArea() { return 10; }
}

var rect = new Rectangle(); // ✅
```

**🔍 Giải thích:** Abstract class không thể tạo object trực tiếp, phải tạo từ class con.

---

### ❌ Lỗi 6: Nhầm lẫn giữa `base` và `this`

```csharp
// ❌ Vấn đề: Dùng this thay vì base
public class Animal
{
    public virtual void Eat() { Console.WriteLine("Animal eating"); }
}

public class Dog : Animal
{
    public override void Eat()
    {
        this.Eat(); // Vòng lặp vô hạn! Gọi chính nó
    }
}

// ✅ Giải pháp: Dùng base để gọi method của class cha
public class Dog : Animal
{
    public override void Eat()
    {
        base.Eat(); // ✅ Gọi method của Animal
        Console.WriteLine("Dog eating");
    }
}
```

**🔍 Giải thích:** `base` gọi method của class cha, `this` gọi method của chính class hiện tại.

---

### ❌ Lỗi 7: Không kiểm tra null trước khi dùng

```csharp
// ❌ Vấn đề: Có thể null
Animal animal = GetAnimal(); // Có thể trả về null
animal.MakeSound(); // NullReferenceException nếu null

// ✅ Giải pháp: Kiểm tra null
Animal animal = GetAnimal();
if (animal != null)
{
    animal.MakeSound();
}

// Hoặc dùng null-conditional operator
animal?.MakeSound(); // ✅ An toàn
```

**🔍 Giải thích:** Luôn kiểm tra null trước khi gọi method trên object.

---

### ❌ Lỗi 8: Nhầm lẫn giữa `==` và `Equals()` cho object

```csharp
// ❌ Vấn đề: == so sánh reference, không phải value
Student s1 = new Student("An", 20);
Student s2 = new Student("An", 20);
if (s1 == s2) // false! (khác reference)

// ✅ Giải pháp: Override Equals() hoặc dùng record
public class Student
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    public override bool Equals(object obj)
    {
        if (obj is Student other)
            return Name == other.Name && Age == other.Age;
        return false;
    }
}

if (s1.Equals(s2)) // ✅ true

// Hoặc dùng record (tự động có value equality)
public record Student(string Name, int Age);
```

**🔍 Giải thích:** `==` so sánh reference (địa chỉ bộ nhớ), `Equals()` có thể so sánh value nếu được override.

---

## 🎯 8. CASE STUDY / VÍ DỤ THỰC TẾ

### Case Study 1: Hệ thống quản lý thư viện

**Yêu cầu:** Quản lý sách, người mượn, và thông tin mượn trả.

```csharp
using System;
using System.Collections.Generic;

// Interface cho các items có thể mượn
public interface IBorrowable
{
    string Title { get; }
    bool IsAvailable { get; }
    void Borrow();
    void Return();
}

// Class Book
public class Book : IBorrowable
{
    public string Title { get; set; }
    public string Author { get; set; }
    public string ISBN { get; set; }
    public bool IsAvailable { get; private set; } = true;
    
    public void Borrow()
    {
        if (!IsAvailable)
            throw new InvalidOperationException("Sách đã được mượn");
        IsAvailable = false;
    }
    
    public void Return()
    {
        IsAvailable = true;
    }
}

// Class Member
public class Member
{
    public string Name { get; set; }
    public string MemberId { get; set; }
    private List<IBorrowable> _borrowedItems = new();
    
    public void BorrowItem(IBorrowable item)
    {
        if (_borrowedItems.Count >= 5)
            throw new InvalidOperationException("Đã mượn tối đa 5 items");
        
        item.Borrow();
        _borrowedItems.Add(item);
    }
    
    public void ReturnItem(IBorrowable item)
    {
        if (_borrowedItems.Remove(item))
        {
            item.Return();
        }
    }
    
    public int GetBorrowedCount() => _borrowedItems.Count;
}

// Sử dụng
class Program
{
    static void Main()
    {
        var book = new Book 
        { 
            Title = "Clean Code", 
            Author = "Robert Martin",
            ISBN = "978-0132350884"
        };
        
        var member = new Member 
        { 
            Name = "Nguyen Van A", 
            MemberId = "M001" 
        };
        
        member.BorrowItem(book);
        Console.WriteLine($"Đã mượn: {member.GetBorrowedCount()} items");
        Console.WriteLine($"Sách còn lại: {book.IsAvailable}");
    }
}
```

**Giải thích:**
- Sử dụng interface `IBorrowable` để đảm bảo tính đa hình
- Encapsulation: `IsAvailable` là private set, chỉ thay đổi qua methods
- Validation: Kiểm tra điều kiện trước khi cho mượn

---

### Case Study 2: Hệ thống tính lương nhân viên

**Yêu cầu:** Tính lương cho các loại nhân viên khác nhau (Full-time, Part-time, Manager).

```csharp
using System;
using System.Collections.Generic;

// Abstract class cho Employee
public abstract class Employee
{
    public string Name { get; set; }
    public string EmployeeId { get; set; }
    
    // Abstract method - class con phải implement
    public abstract decimal CalculateSalary();
    
    // Virtual method - class con có thể override
    public virtual string GetEmployeeType()
    {
        return "Employee";
    }
}

// Full-time Employee
public class FullTimeEmployee : Employee
{
    public decimal MonthlySalary { get; set; }
    
    public override decimal CalculateSalary()
    {
        return MonthlySalary;
    }
    
    public override string GetEmployeeType()
    {
        return "Full-time";
    }
}

// Part-time Employee
public class PartTimeEmployee : Employee
{
    public decimal HourlyRate { get; set; }
    public int HoursWorked { get; set; }
    
    public override decimal CalculateSalary()
    {
        return HourlyRate * HoursWorked;
    }
    
    public override string GetEmployeeType()
    {
        return "Part-time";
    }
}

// Manager (kế thừa FullTimeEmployee)
public class Manager : FullTimeEmployee
{
    public decimal Bonus { get; set; }
    
    public override decimal CalculateSalary()
    {
        return base.CalculateSalary() + Bonus; // Lương cơ bản + Bonus
    }
    
    public override string GetEmployeeType()
    {
        return "Manager";
    }
}

// Payroll System
public class PayrollSystem
{
    private List<Employee> _employees = new();
    
    public void AddEmployee(Employee employee)
    {
        _employees.Add(employee);
    }
    
    public void ProcessPayroll()
    {
        Console.WriteLine("=== BẢNG LƯƠNG ===\n");
        decimal totalPayroll = 0;
        
        foreach (var emp in _employees)
        {
            decimal salary = emp.CalculateSalary();
            totalPayroll += salary;
            
            Console.WriteLine($"{emp.Name} ({emp.GetEmployeeType()}): {salary:C}");
        }
        
        Console.WriteLine($"\nTổng lương: {totalPayroll:C}");
    }
}

// Sử dụng
class Program
{
    static void Main()
    {
        var payroll = new PayrollSystem();
        
        payroll.AddEmployee(new FullTimeEmployee 
        { 
            Name = "Nguyen Van A", 
            MonthlySalary = 10000000 
        });
        
        payroll.AddEmployee(new PartTimeEmployee 
        { 
            Name = "Tran Thi B", 
            HourlyRate = 50000, 
            HoursWorked = 80 
        });
        
        payroll.AddEmployee(new Manager 
        { 
            Name = "Le Van C", 
            MonthlySalary = 15000000, 
            Bonus = 5000000 
        });
        
        payroll.ProcessPayroll();
    }
}
```

**Best practices áp dụng:**
- Abstract class để định nghĩa cấu trúc chung
- Polymorphism: Mỗi loại employee tính lương khác nhau
- Inheritance: Manager kế thừa FullTimeEmployee
- Virtual/Override để tùy biến hành vi

---

### Case Study 3: Hệ thống hình học với Interface

**Yêu cầu:** Tính diện tích và chu vi các hình khác nhau.

```csharp
using System;
using System.Collections.Generic;

// Interface cho các hình
public interface IShape
{
    double CalculateArea();
    double CalculatePerimeter();
    string GetShapeName();
}

// Rectangle
public class Rectangle : IShape
{
    public double Width { get; set; }
    public double Height { get; set; }
    
    public double CalculateArea() => Width * Height;
    
    public double CalculatePerimeter() => 2 * (Width + Height);
    
    public string GetShapeName() => "Hình chữ nhật";
}

// Circle
public class Circle : IShape
{
    public double Radius { get; set; }
    private const double PI = 3.14159;
    
    public double CalculateArea() => PI * Radius * Radius;
    
    public double CalculatePerimeter() => 2 * PI * Radius;
    
    public string GetShapeName() => "Hình tròn";
}

// Triangle
public class Triangle : IShape
{
    public double SideA { get; set; }
    public double SideB { get; set; }
    public double SideC { get; set; }
    
    public double CalculateArea()
    {
        // Heron's formula
        double s = CalculatePerimeter() / 2;
        return Math.Sqrt(s * (s - SideA) * (s - SideB) * (s - SideC));
    }
    
    public double CalculatePerimeter() => SideA + SideB + SideC;
    
    public string GetShapeName() => "Hình tam giác";
}

// Shape Calculator
public class ShapeCalculator
{
    private List<IShape> _shapes = new();
    
    public void AddShape(IShape shape)
    {
        _shapes.Add(shape);
    }
    
    public void DisplayAllShapes()
    {
        Console.WriteLine("=== DANH SÁCH HÌNH ===\n");
        
        foreach (var shape in _shapes)
        {
            Console.WriteLine($"{shape.GetShapeName()}:");
            Console.WriteLine($"  Diện tích: {shape.CalculateArea():F2}");
            Console.WriteLine($"  Chu vi: {shape.CalculatePerimeter():F2}\n");
        }
        
        double totalArea = 0;
        foreach (var shape in _shapes)
        {
            totalArea += shape.CalculateArea();
        }
        
        Console.WriteLine($"Tổng diện tích: {totalArea:F2}");
    }
}

// Sử dụng
class Program
{
    static void Main()
    {
        var calculator = new ShapeCalculator();
        
        calculator.AddShape(new Rectangle { Width = 10, Height = 5 });
        calculator.AddShape(new Circle { Radius = 7 });
        calculator.AddShape(new Triangle { SideA = 3, SideB = 4, SideC = 5 });
        
        calculator.DisplayAllShapes();
    }
}
```

**Giải thích:**
- Interface `IShape` đảm bảo tất cả hình đều có methods cần thiết
- Polymorphism: Mỗi hình tính diện tích/chu vi khác nhau
- Dễ mở rộng: Thêm hình mới chỉ cần implement interface

---

## ✅ 9. BEST PRACTICES

### 9.1. Ưu tiên Composition over Inheritance

```csharp
// ❌ Inheritance có thể dẫn đến hierarchy phức tạp
public class Car : Vehicle { }
public class Truck : Vehicle { }
public class ElectricCar : Car { } // Hierarchy sâu

// ✅ Composition linh hoạt hơn
public class Car
{
    private Engine _engine; // Composition
    public Car(Engine engine) { _engine = engine; }
}
```

### 9.2. Sử dụng Interface cho Loose Coupling

```csharp
// ❌ Phụ thuộc vào class cụ thể
public class OrderService
{
    private SqlDatabase _db; // Phụ thuộc chặt
}

// ✅ Phụ thuộc vào interface
public class OrderService
{
    private IDatabase _db; // Loose coupling
    public OrderService(IDatabase db) { _db = db; }
}
```

### 9.3. Encapsulation: Luôn dùng Properties thay vì public fields

```csharp
// ❌ Public field
public class Student
{
    public int Age; // Không kiểm soát được
}

// ✅ Property với validation
public class Student
{
    private int _age;
    public int Age
    {
        get => _age;
        set
        {
            if (value < 0 || value > 150)
                throw new ArgumentException("Age invalid");
            _age = value;
        }
    }
}
```

### 9.4. Single Responsibility Principle

```csharp
// ❌ Class làm quá nhiều việc
public class Student
{
    public void Study() { }
    public void SaveToDatabase() { } // Không nên ở đây
    public void SendEmail() { } // Không nên ở đây
}

// ✅ Tách thành nhiều class
public class Student { public void Study() { } }
public class StudentRepository { public void Save(Student s) { } }
public class EmailService { public void Send(Student s) { } }
```

### 9.5. Dùng `sealed` khi không cần kế thừa

```csharp
// ✅ Sealed class để tránh kế thừa không cần thiết
public sealed class Configuration
{
    // Class này không nên được kế thừa
}
```

---

## 🧪 10. BÀI TẬP THỰC HÀNH

### Bài 1: Quản lý nhân viên
Tạo class `Employee` (Name, Salary).
Tạo class `Manager` kế thừa `Employee`, thêm thuộc tính `Bonus`.
Tính tổng lương của Manager = Salary + Bonus.

<details>
<summary>💡 Đáp án</summary>

```csharp
public class Employee
{
    public string Name { get; set; }
    public decimal Salary { get; set; }
    
    public Employee(string name, decimal salary)
    {
        Name = name;
        Salary = salary;
    }
    
    public virtual decimal CalculateTotalSalary()
    {
        return Salary;
    }
}

public class Manager : Employee
{
    public decimal Bonus { get; set; }
    
    public Manager(string name, decimal salary, decimal bonus) 
        : base(name, salary)
    {
        Bonus = bonus;
    }
    
    public override decimal CalculateTotalSalary()
    {
        return base.CalculateTotalSalary() + Bonus;
    }
}

// Sử dụng
var manager = new Manager("Nguyen Van A", 10000000, 2000000);
Console.WriteLine($"Tổng lương: {manager.CalculateTotalSalary():C}");
```
</details>

---

### Bài 2: Hình học
Tạo interface `IShape` có hàm `CalculateArea()`.
Tạo class `Rectangle` và `Circle` thực thi interface này.
Tạo một danh sách các hình và tính tổng diện tích.

<details>
<summary>💡 Đáp án</summary>

```csharp
public interface IShape
{
    double CalculateArea();
}

public class Rectangle : IShape
{
    public double Width { get; set; }
    public double Height { get; set; }
    
    public double CalculateArea() => Width * Height;
}

public class Circle : IShape
{
    public double Radius { get; set; }
    
    public double CalculateArea() => Math.PI * Radius * Radius;
}

// Sử dụng
var shapes = new List<IShape>
{
    new Rectangle { Width = 10, Height = 5 },
    new Circle { Radius = 7 },
    new Rectangle { Width = 3, Height = 4 }
};

double totalArea = 0;
foreach (var shape in shapes)
{
    totalArea += shape.CalculateArea();
}

Console.WriteLine($"Tổng diện tích: {totalArea:F2}");
```
</details>

---

### Bài 3: Hệ thống đăng ký khóa học
Tạo abstract class `Course` với method `CalculateFee()`.
Tạo class `OnlineCourse` và `OfflineCourse` kế thừa `Course`.
Mỗi loại khóa học có cách tính phí khác nhau.

<details>
<summary>💡 Đáp án</summary>

```csharp
public abstract class Course
{
    public string Name { get; set; }
    public int Duration { get; set; } // Số giờ
    
    public abstract decimal CalculateFee();
}

public class OnlineCourse : Course
{
    private const decimal RATE_PER_HOUR = 50000;
    
    public override decimal CalculateFee()
    {
        return Duration * RATE_PER_HOUR;
    }
}

public class OfflineCourse : Course
{
    private const decimal RATE_PER_HOUR = 100000;
    public decimal MaterialFee { get; set; } = 500000;
    
    public override decimal CalculateFee()
    {
        return (Duration * RATE_PER_HOUR) + MaterialFee;
    }
}

// Sử dụng
var online = new OnlineCourse { Name = "C# Basics", Duration = 40 };
var offline = new OfflineCourse { Name = "ASP.NET Core", Duration = 60 };

Console.WriteLine($"Online: {online.CalculateFee():C}");
Console.WriteLine($"Offline: {offline.CalculateFee():C}");
```
</details>

---

## 🧪 11. MINI TEST

1. **Tính chất nào của OOP cho phép che giấu dữ liệu?**
   - A. Inheritance
   - B. Polymorphism
   - C. Encapsulation ✅
   - D. Abstraction

2. **C# cho phép một class kế thừa từ bao nhiêu class cha?**
   - A. Không giới hạn
   - B. 2
   - C. 1 ✅
   - D. 0

3. **Từ khóa nào dùng để override method?**
   - A. `virtual`
   - B. `override` ✅
   - C. `abstract`
   - D. `sealed`

4. **Interface khác Abstract Class như thế nào?**
   - A. Interface có thể chứa code implementation
   - B. Class có thể kế thừa nhiều interface ✅
   - C. Interface có thể tạo object
   - D. Không khác gì

5. **Khi nào nên dùng `sealed`?**
   - A. Khi muốn class có thể kế thừa
   - B. Khi muốn ngăn kế thừa ✅
   - C. Khi muốn override method
   - D. Không bao giờ dùng

<details>
<summary>💡 Đáp án</summary>

1. **C** - Encapsulation che giấu dữ liệu bên trong class
2. **C** - C# chỉ cho phép single inheritance (1 class cha)
3. **B** - `override` dùng để ghi đè method từ class cha
4. **B** - Class có thể implement nhiều interface, nhưng chỉ kế thừa 1 class
5. **B** - `sealed` ngăn class khác kế thừa
</details>

---

## 📝 12. QUICK NOTES

### 4 Tính chất OOP:
- **Encapsulation**: Che giấu dữ liệu, dùng private + properties
- **Inheritance**: Class con kế thừa class cha (`: BaseClass`)
- **Polymorphism**: Một method có nhiều implementation (`virtual`/`override`)
- **Abstraction**: Ẩn chi tiết, dùng abstract class hoặc interface

### Access Modifiers:
- `public`: Truy cập từ mọi nơi
- `private`: Chỉ trong cùng class
- `protected`: Trong class và class con
- `internal`: Trong cùng assembly

### Keywords quan trọng:
- `virtual`: Cho phép override
- `override`: Ghi đè method từ class cha
- `abstract`: Method/class chưa có implementation
- `sealed`: Ngăn kế thừa
- `base`: Gọi method/property của class cha
- `this`: Tham chiếu đến object hiện tại

### Interface vs Abstract Class:
- **Interface**: Chỉ định nghĩa contract, không có implementation (trước C# 8)
- **Abstract Class**: Có thể có implementation, nhưng không thể tạo object
- Class có thể implement nhiều interface, nhưng chỉ kế thừa 1 class

### Best Practices:
- ✅ Dùng Properties thay vì public fields
- ✅ Ưu tiên Composition over Inheritance
- ✅ Sử dụng Interface cho Loose Coupling
- ✅ Single Responsibility Principle
- ✅ Dùng `sealed` khi không cần kế thừa

---

## 💡 Mẹo nhỏ
> [!TIP]
> Ưu tiên sử dụng **Interface** để giảm sự phụ thuộc (Loose Coupling) giữa các thành phần.

> [!IMPORTANT]
> Trong C#, một class chỉ có thể kế thừa từ **1 class cha**, nhưng có thể thực thi **nhiều interface**.

> [!WARNING]
> Luôn kiểm tra null trước khi gọi method trên object để tránh NullReferenceException.

---

## ✨ Điểm nhấn OOP C# hiện đại so với Java

Mục này tập trung các khác biệt quan trọng trong mô hình OOP. Xem chi tiết đầy đủ tại: `00_so_sanh_csharp_vs_java.md`.

### 1) Properties, Init‑only, Expression‑bodied Members

Trong C#, `property` là khái niệm ngôn ngữ (không cần viết getter/setter thủ công):

```csharp
public class Student
{
    public string Name { get; init; } // init‑only setter (immutable sau khởi tạo)
    public int Age { get; set; }
    public override string ToString() => $"{Name} ({Age})"; // expression‑bodied
}

var s = new Student { Name = "Nam", Age = 20 };
```

Java thường dùng getter/setter thủ công hoặc Lombok/record để rút gọn; không có `init` tương đương.

#### Tổng quan Getter/Setter trong C# (đầy đủ các trường hợp thường gặp)

```csharp
public class BankAccount
{
    // 1) Auto-property đơn giản
    public string Owner { get; set; } = string.Empty;

    // 2) Auto-property chỉ đọc (immutable sau khởi tạo)
    public string AccountNumber { get; } = Guid.NewGuid().ToString();

    // 3) Init-only setter (thiết lập trong object initializer, sau đó chỉ-read)
    public string Currency { get; init; } = "VND";

    // 4) Private setter (chỉ class nội bộ được quyền thay đổi)
    public decimal Balance { get; private set; }

    // 5) Property với backing field + validation logic
    private decimal _dailyLimit = 5_000_000m;
    public decimal DailyLimit
    {
        get => _dailyLimit;
        set
        {
            if (value < 0) throw new ArgumentOutOfRangeException(nameof(DailyLimit));
            _dailyLimit = value;
        }
    }

    // 6) Computed property (chỉ get, tính từ dữ liệu khác)
    public bool IsVip => Balance >= 100_000_000m;

    // 7) Expression-bodied getter/setter
    private string _alias = string.Empty;
    public string Alias
    {
        get => _alias;
        set => _alias = value?.Trim() ?? string.Empty;
    }

    // 8) Required members (C# 11) – buộc phải set khi khởi tạo
    public required string Branch { get; init; }

    // 9) Indexer – cho phép truy cập như mảng theo key (minh họa)
    private readonly Dictionary<DateOnly, decimal> _history = new();
    public decimal this[DateOnly date]
    {
        get => _history.TryGetValue(date, out var v) ? v : 0m;
        set => _history[date] = value;
    }

    // Hành vi nghiệp vụ cập nhật số dư
    public void Deposit(decimal amount)
    {
        if (amount <= 0) throw new ArgumentOutOfRangeException(nameof(amount));
        Balance += amount;
    }
}
```

Ghi chú:
- Dùng `init` cho dữ liệu chỉ được set khi khởi tạo, đảm bảo bất biến.
- `required` buộc người dùng class phải cung cấp giá trị khi tạo object.
- `private set` bảo vệ tính toàn vẹn dữ liệu (chỉ phương thức trong class được phép thay đổi).
- Indexer giúp API tự nhiên khi tra cứu theo key.

### 2) Records và `with` (Value‑based Equality)

```csharp
public record User(string Name, int Age);
var u1 = new User("An", 20);
var u2 = u1 with { Age = 21 }; // clone + thay đổi một phần
```

- C# có cả `record class` (reference) và `record struct` (value type) cho equality theo giá trị.
- Java có `record` (reference) equality theo state nhưng không có value‑type cho record.

### 3) Struct (Value Type) vs Java (Reference‑only)

```csharp
public readonly struct Point(int x, int y)
{
    public int X { get; } = x;
    public int Y { get; } = y;
}
```

Struct giúp giảm allocation/GC trong đường nóng hiệu năng. Java chưa có user‑defined value types (Project Valhalla đang phát triển).

### 4) Delegates & Events (Mô hình sự kiện native)

```csharp
public delegate void Notifier(string message);

public class Clock
{
    public event Notifier? Ticked;
    public void Tick() => Ticked?.Invoke("tick");
}
```

Java không có `delegate/event` ở mức ngôn ngữ; thường dùng interface functional (`Consumer<T>`) và listener patterns.

### 5) Pattern Matching cho OOP

```csharp
string Describe(object o) => o switch
{
    Student { Age: >= 18 } s => $"Adult student: {s.Name}",
    null => "null",
    _ => "other"
};
```

Java 21 có pattern matching cho `switch`/`instanceof`, nhưng C# hiện hỗ trợ sâu các property/list patterns và `switch` expression.

### 6) Nullable Reference Types trong thiết kế API

```csharp
#nullable enable
public void SetEmail(string? email)
{
    if (string.IsNullOrWhiteSpace(email))
        throw new ArgumentException("Email invalid");
}
```

Cảnh báo null ở compile‑time giúp API an toàn hơn. Java chủ yếu dựa vào annotation `@Nullable`/`@NonNull` và kiểm tra runtime.

