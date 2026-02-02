# C# OOP Implementation Practice — Learning Guide

A structured reference for object-oriented programming in C#. Each section maps to a concrete exercise you can implement.

---

## Table of Contents

1. [Encapsulation](#1-encapsulation)
2. [Inheritance](#2-inheritance)
3. [Polymorphism](#3-polymorphism)
4. [Abstract Classes and Interfaces](#4-abstract-classes-and-interfaces)
5. [Sealed Classes and Methods](#5-sealed-classes-and-methods)
6. [Readonly and Immutability](#6-readonly-and-immutability)
7. [Static Members and Constructors](#7-static-members-and-constructors)
8. [Dependency Injection and Coupling](#8-dependency-injection-and-coupling)
9. [Copy Constructor and ICloneable](#9-copy-constructor-and-icloneable)
10. [Equals, GetHashCode, and Operator Overloading](#10-equals-gethashcode-and-operator-overloading)
11. [Custom Exceptions](#11-custom-exceptions)
12. [Type Checking, Casting, and Pattern Matching](#12-type-checking-casting-and-pattern-matching)
13. [Composition and Aggregation](#13-composition-and-aggregation)
14. [Partial Classes and Methods](#14-partial-classes-and-methods)
15. [Nested Classes](#15-nested-classes)
16. [Object Initializers and Constructor Overloading](#16-object-initializers-and-constructor-overloading)
17. [Destructor and IDisposable](#17-destructor-and-idisposable)
18. [Boxing and Unboxing](#18-boxing-and-unboxing)

---

## 1. Encapsulation

### 1.1 Create a class with private fields and expose them using public properties

```csharp
public class Person
{
    private string _name;
    private int _age;

    public string Name
    {
        get => _name;
        set => _name = value;
    }

    public int Age
    {
        get => _age;
        set => _age = value;
    }
}

// Usage
var p = new Person();
p.Name = "Alice";
p.Age = 30;
Console.WriteLine($"{p.Name}, {p.Age}");
```

### 1.2 Implement encapsulation using getter and setter with validation logic

```csharp
public class BankAccount
{
    private decimal _balance;

    public decimal Balance
    {
        get => _balance;
        set
        {
            if (value < 0)
                throw new ArgumentException("Balance cannot be negative.");
            _balance = value;
        }
    }

    private string _accountNumber = "";
    public string AccountNumber
    {
        get => _accountNumber;
        set
        {
            if (string.IsNullOrWhiteSpace(value) || value.Length != 10)
                throw new ArgumentException("Account number must be 10 characters.");
            _accountNumber = value;
        }
    }
}
```

---

## 2. Inheritance

### 2.1 Create a base class and derive a child class using inheritance

```csharp
public class Animal
{
    public string Name { get; set; } = "";

    public void Eat()
    {
        Console.WriteLine($"{Name} is eating.");
    }
}

public class Dog : Animal
{
    public void Bark()
    {
        Console.WriteLine($"{Name} says woof.");
    }
}

// Usage
var dog = new Dog { Name = "Rex" };
dog.Eat();
dog.Bark();
```

### 2.2 Call a base class constructor from a derived class

```csharp
public class Vehicle
{
    public string Brand { get; }
    public Vehicle(string brand) => Brand = brand;
}

public class Car : Vehicle
{
    public int Doors { get; }

    public Car(string brand, int doors) : base(brand)
    {
        Doors = doors;
    }
}

// Usage: base(brand) runs first, then Car constructor body.
var car = new Car("Toyota", 4);
Console.WriteLine($"{car.Brand}, {car.Doors} doors");
```

### 2.3 Override a virtual method in a derived class

```csharp
public class Shape
{
    public virtual double GetArea() => 0;
}

public class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }

    public override double GetArea() => Width * Height;
}

// Usage
Shape s = new Rectangle { Width = 5, Height = 3 };
Console.WriteLine(s.GetArea()); // 15 — polymorphic call
```

### 2.4 Demonstrate method hiding using the new keyword

```csharp
public class Base
{
    public void DoWork() => Console.WriteLine("Base.DoWork");
}

public class Derived : Base
{
    public new void DoWork() => Console.WriteLine("Derived.DoWork");
}

// Usage
Base b = new Derived();
b.DoWork();           // "Base.DoWork" — non-virtual, so base method runs
((Derived)b).DoWork(); // "Derived.DoWork"
```

### 2.5 Access base class members using the base keyword

```csharp
public class Animal
{
    public virtual void Speak() => Console.WriteLine("...");
}

public class Cat : Animal
{
    public override void Speak()
    {
        base.Speak(); // Call Animal.Speak first
        Console.WriteLine("Meow");
    }
}
```

---

## 3. Polymorphism

### 3.1 Implement runtime polymorphism using a base class reference pointing to a derived object

```csharp
public abstract class Payment
{
    public abstract void Process(decimal amount);
}

public class CardPayment : Payment
{
    public override void Process(decimal amount) =>
        Console.WriteLine($"Card payment: {amount:C}");
}

public class CashPayment : Payment
{
    public override void Process(decimal amount) =>
        Console.WriteLine($"Cash payment: {amount:C}");
}

// Runtime polymorphism: actual type decides which override runs
Payment p1 = new CardPayment();
Payment p2 = new CashPayment();
p1.Process(100); // Card payment: $100.00
p2.Process(50);  // Cash payment: $50.00
```

### 3.2 Demonstrate compile-time polymorphism using method overloading

```csharp
public class Calculator
{
    public int Add(int a, int b) => a + b;
    public double Add(double a, double b) => a + b;
    public int Add(int a, int b, int c) => a + b + c;
}

var calc = new Calculator();
Console.WriteLine(calc.Add(1, 2));      // 3 — int overload
Console.WriteLine(calc.Add(1.5, 2.5));  // 4 — double overload
Console.WriteLine(calc.Add(1, 2, 3));   // 6 — three-arg overload
```

### 3.3 Override ToString() in a custom class

```csharp
public class Product
{
    public string Name { get; set; } = "";
    public decimal Price { get; set; }

    public override string ToString() => $"{Name} - {Price:C}";
}

var p = new Product { Name = "Widget", Price = 9.99m };
Console.WriteLine(p);        // "Widget - $9.99"
Console.WriteLine(p.ToString());
```

---

## 4. Abstract Classes and Interfaces

### 4.1 Create an abstract class with abstract and non-abstract methods

```csharp
public abstract class Shape
{
    public string Name { get; set; } = "";

    public abstract double GetArea();

    public void PrintInfo()
    {
        Console.WriteLine($"{Name}: Area = {GetArea()}");
    }
}

public class Circle : Shape
{
    public double Radius { get; set; }
    public override double GetArea() => Math.PI * Radius * Radius;
}
```

### 4.2 Implement an abstract class in a derived class

```csharp
public abstract class Logger
{
    public abstract void Log(string message);
}

public class FileLogger : Logger
{
    public override void Log(string message) =>
        Console.WriteLine($"[File] {message}");
}
```

### 4.3 Create an interface and implement it in a class

```csharp
public interface IDrawable
{
    void Draw();
}

public class Square : IDrawable
{
    public void Draw() => Console.WriteLine("Drawing a square.");
}
```

### 4.4 Implement multiple interfaces in a single class

```csharp
public interface IReadable { string Read(); }
public interface IWritable { void Write(string data); }

public class DataStore : IReadable, IWritable
{
    private string _data = "";

    public string Read() => _data;
    public void Write(string data) => _data = data;
}
```

### 4.5 Demonstrate interface-based polymorphism

```csharp
public interface ILogger
{
    void Log(string msg);
}

public class ConsoleLogger : ILogger
{
    public void Log(string msg) => Console.WriteLine($"[Console] {msg}");
}

// Polymorphism via interface
ILogger logger = new ConsoleLogger();
logger.Log("Hello");
```

### 4.6 Use explicit interface implementation

```csharp
public interface IFoo { void Do(); }
public interface IBar { void Do(); }

public class Multi : IFoo, IBar
{
    void IFoo.Do() => Console.WriteLine("IFoo.Do");
    void IBar.Do() => Console.WriteLine("IBar.Do");
}

// Usage
var m = new Multi();
((IFoo)m).Do(); // IFoo.Do
((IBar)m).Do(); // IBar.Do
// m.Do();      // Compile error: must cast to interface
```

### 4.7 Demonstrate the difference between abstract classes and interfaces via implementation

- **Abstract class**: can have fields, constructors, non-abstract methods, and access modifiers. Single inheritance.
- **Interface**: contract only (no implementation until default interface members in C# 8+). Multiple inheritance. No fields or constructors.

```csharp
// Abstract class: shared state and behavior
public abstract class Animal
{
    protected string Name { get; }
    protected Animal(string name) => Name = name;
    public abstract void Speak();
    public void Sleep() => Console.WriteLine($"{Name} is sleeping.");
}

// Interface: capability contract only
public interface IFlyable
{
    void Fly();
}

public class Bird : Animal, IFlyable
{
    public Bird(string name) : base(name) { }
    public override void Speak() => Console.WriteLine($"{Name} chirps.");
    public void Fly() => Console.WriteLine($"{Name} flies.");
}
```

---

## 5. Sealed Classes and Methods

### 5.1 Create a sealed class and attempt to inherit from it

```csharp
public sealed class SealedClass
{
    public int Value { get; set; }
}

// This does NOT compile:
// public class Derived : SealedClass { }
```

### 5.2 Create a sealed method and attempt to override it

```csharp
public class Base
{
    public virtual void Method() => Console.WriteLine("Base");
}

public class Derived : Base
{
    public sealed override void Method() => Console.WriteLine("Derived");
}

// This does NOT compile:
// public class Further : Derived
// {
//     public override void Method() { }  // Cannot override sealed method
// }
```

---

## 6. Readonly and Immutability

### 6.1 Implement a readonly field and initialize it via constructor

```csharp
public class Config
{
    private readonly string _connectionString;

    public Config(string connectionString)
    {
        _connectionString = connectionString;
    }

    public string ConnectionString => _connectionString;
}
```

### 6.2 Implement immutable class design

```csharp
public sealed class ImmutablePoint
{
    public int X { get; }
    public int Y { get; }

    public ImmutablePoint(int x, int y)
    {
        X = x;
        Y = y;
    }

    public ImmutablePoint WithX(int x) => new ImmutablePoint(x, Y);
    public ImmutablePoint WithY(int y) => new ImmutablePoint(X, y);
}
```

---

## 7. Static Members and Constructors

### 7.1 Create a static class with static members only

```csharp
public static class MathHelper
{
    public static int Square(int x) => x * x;
    public static double Pi => 3.14159;
}

Console.WriteLine(MathHelper.Square(5));  // 25
Console.WriteLine(MathHelper.Pi);
```

### 7.2 Use static constructors and demonstrate their execution order

```csharp
public class Example
{
    public static int StaticField;

    static Example()
    {
        StaticField = 42;
        Console.WriteLine("Static constructor ran.");
    }

    public Example()
    {
        Console.WriteLine("Instance constructor ran.");
    }
}

// First use of Example triggers static ctor, then instance ctor
var a = new Example();  // "Static constructor ran." then "Instance constructor ran."
var b = new Example();  // Only "Instance constructor ran."
```

---

## 8. Dependency Injection and Coupling

### 8.1 Demonstrate the difference between const and readonly

- **const**: compile-time constant; must be initialized at declaration; value is baked into IL.
- **readonly**: set once (typically in constructor); can be different per instance or computed.

```csharp
public class Demo
{
    public const int MaxItems = 100;
    public readonly int Id;

    public Demo(int id) => Id = id;
}

Console.WriteLine(Demo.MaxItems);  // 100, no instance needed
var d = new Demo(1);
Console.WriteLine(d.Id);          // 1
```

### 8.2 Implement dependency injection using constructor injection

```csharp
public interface IEmailService
{
    void Send(string to, string body);
}

public class SmtpEmailService : IEmailService
{
    public void Send(string to, string body) =>
        Console.WriteLine($"Sending email to {to}: {body}");
}

public class OrderService
{
    private readonly IEmailService _emailService;

    public OrderService(IEmailService emailService)
    {
        _emailService = emailService ?? throw new ArgumentNullException(nameof(emailService));
    }

    public void PlaceOrder(string customerEmail)
    {
        // ... place order logic
        _emailService.Send(customerEmail, "Order confirmed.");
    }
}

// Usage: inject dependency
var orderService = new OrderService(new SmtpEmailService());
orderService.PlaceOrder("user@example.com");
```

### 8.3 Implement dependency injection using interface abstraction

Same as above: depend on `IEmailService` (interface abstraction) and inject a concrete implementation via constructor. Callers depend on the abstraction, not on `SmtpEmailService`.

### 8.4 Demonstrate tight coupling vs loose coupling via implementation

```csharp
// Tight coupling: direct dependency on concrete class
public class TightOrderService
{
    private readonly SmtpEmailService _email = new SmtpEmailService();
    public void Notify(string to) => _email.Send(to, "Done");
}

// Loose coupling: depend on interface, implementation injected
public class LooseOrderService
{
    private readonly IEmailService _email;
    public LooseOrderService(IEmailService email) => _email = email;
    public void Notify(string to) => _email.Send(to, "Done");
}
```

---

## 9. Copy Constructor and ICloneable

### 9.1 Implement a copy constructor

```csharp
public class Student
{
    public string Name { get; set; } = "";
    public int Age { get; set; }

    public Student() { }

    public Student(Student other)
    {
        Name = other.Name;
        Age = other.Age;
    }
}

var original = new Student { Name = "Bob", Age = 20 };
var copy = new Student(original);
```

### 9.2 Implement the ICloneable interface

```csharp
public class Item : ICloneable
{
    public string Code { get; set; } = "";
    public decimal Price { get; set; }

    public object Clone() => new Item { Code = Code, Price = Price };
}

var item = new Item { Code = "X1", Price = 10m };
var cloned = (Item)item.Clone();
```

---

## 10. Equals, GetHashCode, and Operator Overloading

### 10.1 Override Equals() and GetHashCode() correctly

```csharp
public class Person
{
    public int Id { get; set; }
    public string Name { get; set; } = "";

    public override bool Equals(object? obj) =>
        obj is Person other && Id == other.Id;

    public override int GetHashCode() => Id.GetHashCode();
}
```

### 10.2 Overload the == and != operators

```csharp
public class Money
{
    public decimal Amount { get; set; }
    public string Currency { get; set; } = "USD";

    public static bool operator ==(Money? a, Money? b)
    {
        if (a is null) return b is null;
        if (b is null) return false;
        return a.Amount == b.Amount && a.Currency == b.Currency;
    }

    public static bool operator !=(Money? a, Money? b) => !(a == b);

    public override bool Equals(object? obj) =>
        obj is Money m && this == m;

    public override int GetHashCode() =>
        HashCode.Combine(Amount, Currency);
}
```

---

## 11. Custom Exceptions

### 11.1 Implement custom exception class by inheriting from Exception

```csharp
public class InvalidAccountException : Exception
{
    public string AccountId { get; }

    public InvalidAccountException(string accountId)
        : base($"Invalid account: {accountId}")
    {
        AccountId = accountId;
    }

    public InvalidAccountException(string accountId, Exception inner)
        : base($"Invalid account: {accountId}", inner)
    {
        AccountId = accountId;
    }
}

// Usage
throw new InvalidAccountException("ACC-001");
```

---

## 12. Type Checking, Casting, and Pattern Matching

### 12.1 Use is, as, and pattern matching with objects

```csharp
object obj = "hello";

// is — type check
if (obj is string)
    Console.WriteLine("It's a string");

// as — safe cast (null if wrong type)
string? s = obj as string;

// Pattern matching
if (obj is string str)
    Console.WriteLine(str.Length);

// Switch pattern
Console.WriteLine(obj switch
{
    string x => $"String: {x}",
    int n => $"Int: {n}",
    _ => "Other"
});
```

### 12.2 Demonstrate upcasting and downcasting

```csharp
Dog dog = new Dog();
Animal animal = dog;           // Upcasting: always safe, implicit

Animal a = new Dog();
Dog d = (Dog)a;                // Downcasting: explicit, can throw InvalidCastException
Dog? d2 = a as Dog;            // Safe downcast: null if not Dog
if (a is Dog d3)               // Pattern matching: safe and concise
    d3.Bark();
```

### 12.3 Use typeof and GetType() and compare results

```csharp
// typeof — operator, works on type name (compile-time)
Type t1 = typeof(Dog);

// GetType() — method on instance (runtime)
Dog dog = new Dog();
Type t2 = dog.GetType();

Console.WriteLine(t1 == t2);        // True
Console.WriteLine(t1.Name);         // "Dog"
Console.WriteLine(typeof(Animal).IsAssignableFrom(t2)); // True
```

---

## 13. Composition and Aggregation

### 13.1 Implement composition over inheritance using classes

Composition: the contained object’s lifetime is owned by the container. When the container is destroyed, the part is destroyed.

```csharp
public class Engine
{
    public void Start() => Console.WriteLine("Engine started.");
}

public class Car
{
    private readonly Engine _engine = new Engine();

    public void StartCar()
    {
        _engine.Start();
        Console.WriteLine("Car started.");
    }
}
```

### 13.2 Demonstrate aggregation vs composition

- **Composition**: child cannot exist without parent (e.g. `Car` owns `Engine`; when `Car` is gone, `Engine` is gone).
- **Aggregation**: child can exist independently; parent holds a reference (e.g. `Department` has `Employee[]`; employees can exist without the department).

```csharp
// Composition: Engine is created and owned by Car
public class Car
{
    private readonly Engine _engine = new Engine();
}

// Aggregation: Wheel can exist independently; Car holds reference
public class Wheel { }
public class CarWithWheels
{
    private readonly List<Wheel> _wheels = new List<Wheel>();
    public void AddWheel(Wheel w) => _wheels.Add(w);
}
```

---

## 14. Partial Classes and Methods

### 14.1 Implement partial classes across multiple files

**File: Person.cs**
```csharp
public partial class Person
{
    public string Name { get; set; } = "";
}
```

**File: Person.Other.cs**
```csharp
public partial class Person
{
    public int Age { get; set; }
    public void Print() => Console.WriteLine($"{Name}, {Age}");
}
```

### 14.2 Implement partial methods

```csharp
public partial class DataProcessor
{
    partial void OnProcessingStarted();

    public void Process()
    {
        OnProcessingStarted(); // No-op if not implemented
        Console.WriteLine("Processing...");
    }
}

// In another file: implementation is optional
public partial class DataProcessor
{
    partial void OnProcessingStarted() =>
        Console.WriteLine("Started.");
}
```

---

## 15. Nested Classes

### 15.1 Use nested classes and control their accessibility

```csharp
public class Outer
{
    private int _privateField = 1;

    public class PublicNested
    {
        public void UseOuter(Outer o) => Console.WriteLine(o._privateField);
    }

    private class PrivateNested
    {
        public void UseOuter(Outer o) => Console.WriteLine(o._privateField);
    }

    public void CreateNested()
    {
        var p = new PublicNested();
        var pr = new PrivateNested();
    }
}

// Usage
var outer = new Outer();
var nested = new Outer.PublicNested();
nested.UseOuter(outer);
// Outer.PrivateNested — not visible outside Outer
```

---

## 16. Object Initializers and Constructor Overloading

### 16.1 Demonstrate object initialization using object initializers

```csharp
public class Person
{
    public string Name { get; set; } = "";
    public int Age { get; set; }
}

var p = new Person
{
    Name = "Alice",
    Age = 30
};
```

### 16.2 Demonstrate constructor overloading

```csharp
public class Product
{
    public string Name { get; set; } = "";
    public decimal Price { get; set; }

    public Product() { }

    public Product(string name)
    {
        Name = name;
    }

    public Product(string name, decimal price) : this(name)
    {
        Price = price;
    }
}
```

---

## 17. Destructor and IDisposable

### 17.1 Implement a destructor (finalizer) and observe GC behavior

```csharp
public class ResourceHolder
{
    ~ResourceHolder()
    {
        Console.WriteLine("Finalizer called.");
    }
}

// Object becomes eligible for GC when no longer referenced.
// Finalizer runs at some point during GC (non-deterministic).
var r = new ResourceHolder();
r = null;
GC.Collect();
GC.WaitForPendingFinalizers();
```

### 17.2 Use the IDisposable interface and implement the dispose pattern

```csharp
public class ManagedResource : IDisposable
{
    private bool _disposed;

    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;
        if (disposing)
        {
            // Dispose managed resources
        }
        // Free unmanaged resources
        _disposed = true;
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
}

// Usage
using (var r = new ManagedResource())
{
    // use r
}
// Dispose called automatically
```

---

## 18. Boxing and Unboxing

### 18.1 Demonstrate boxing and unboxing with value types

```csharp
int value = 42;

// Boxing: value type -> object (heap)
object boxed = value;

// Unboxing: object -> value type (must be exact type)
int unboxed = (int)boxed;

Console.WriteLine(boxed);   // 42
Console.WriteLine(unboxed); // 42

// Invalid unboxing throws InvalidCastException
// long wrong = (long)boxed; // Runtime error
```

---

## Quick Reference

| Concept | C# construct |
|--------|----------------|
| Encapsulation | private fields + public properties (with validation in setters) |
| Inheritance | `class Child : Base`, `base()` constructor, `base.Member` |
| Override | `virtual` in base, `override` in derived |
| Hide | `new` in derived (non-polymorphic) |
| Compile-time polymorphism | Method overloading |
| Runtime polymorphism | `virtual`/`override` or interface |
| Abstract | `abstract` class/method; must override in non-abstract derived |
| Interface | `interface`, multiple implementation, explicit: `IFoo.Member` |
| Sealed | `sealed` class (no inheritance) or `sealed override` (no further override) |
| Immutable | init-only or get-only properties, set in constructor |
| Static | `static` class/member, static constructor runs once per type |
| const vs readonly | const = compile-time; readonly = set once (e.g. in ctor) |
| DI | Constructor injection with interface abstraction |
| Copy | Copy constructor or `ICloneable.Clone()` |
| Equality | Override `Equals` + `GetHashCode`; overload `==`/`!=` consistently |
| Custom exception | Inherit `Exception`, pass message/InnerException |
| Type checks | `is`, `as`, pattern matching; `typeof()` vs `GetType()` |
| Composition vs aggregation | Owned lifetime vs shared reference |
| Partial | `partial class`, `partial method` (optional implementation) |
| Nested | Inner class; control with `public`/`private` |
| Cleanup | Finalizer (non-deterministic) vs `IDisposable` + `using` |
| Boxing/unboxing | value → object (box), (T)object (unbox) |

---

*Use this guide as a checklist: implement each subsection in a small console app to solidify your understanding.*
