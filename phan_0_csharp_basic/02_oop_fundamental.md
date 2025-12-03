# 🟦 CHƯƠNG 02
# **LẬP TRÌNH HƯỚNG ĐỐI TƯỢNG (OOP)**

## 📖 1. Giới thiệu OOP

**OOP (Object-Oriented Programming)** là mô hình lập trình dựa trên khái niệm "đối tượng" chứa dữ liệu (thuộc tính) và mã (phương thức).

### 4 Tính chất cơ bản của OOP:
1. **Đóng gói (Encapsulation)**: Che giấu dữ liệu bên trong.
2. **Kế thừa (Inheritance)**: Class con kế thừa từ Class cha.
3. **Đa hình (Polymorphism)**: Một hành động có thể thực hiện theo nhiều cách.
4. **Trừu tượng (Abstraction)**: Ẩn chi tiết cài đặt, chỉ hiện tính năng.

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

## 🧪 7. Bài tập thực hành

### Bài 1: Quản lý nhân viên
Tạo class `Employee` (Name, Salary).
Tạo class `Manager` kế thừa `Employee`, thêm thuộc tính `Bonus`.
Tính tổng lương của Manager = Salary + Bonus.

### Bài 2: Hình học
Tạo interface `IShape` có hàm `CalculateArea()`.
Tạo class `Rectangle` và `Circle` thực thi interface này.
Tạo một danh sách các hình và tính tổng diện tích.

---

## 💡 Mẹo nhỏ
> [!TIP]
> Ưu tiên sử dụng **Interface** để giảm sự phụ thuộc (Loose Coupling) giữa các thành phần.

> [!IMPORTANT]
> Trong C#, một class chỉ có thể kế thừa từ **1 class cha**, nhưng có thể thực thi **nhiều interface**.
