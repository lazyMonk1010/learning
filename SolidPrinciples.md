# SOLID Principles

## Introduction

SOLID is an acronym for five object-oriented design principles that help developers write maintainable, scalable, and robust code. These principles were introduced by Robert C. Martin (Uncle Bob) and have become fundamental guidelines in software development.

**SOLID stands for:**
- **S** - Single Responsibility Principle
- **O** - Open/Closed Principle
- **L** - Liskov Substitution Principle
- **I** - Interface Segregation Principle
- **D** - Dependency Inversion Principle

---

## Video Tutorial

Watch this video for a detailed implementation walkthrough:

[Solid Principles ](https://youtu.be/5999cgzA95A?si=CfCcWAImP5DUXleY)

---

## Table of Contents

1. [Single Responsibility Principle](#1-single-responsibility-principle)
2. [Open/Closed Principle](#2-openclosed-principle)
3. [Liskov Substitution Principle](#3-liskov-substitution-principle)
4. [Interface Segregation Principle](#4-interface-segregation-principle)
5. [Dependency Inversion Principle](#5-dependency-inversion-principle)

---

## 1. Single Responsibility Principle

### Definition
**A class should have only one reason to change, meaning it should have only one job or responsibility.**

### Why It Matters
When a class has multiple responsibilities, it becomes harder to:
- Understand what the class does
- Test individual functionalities
- Maintain and modify code
- Reuse code in different contexts

### Bad Example ❌

```csharp
class Employee {
    public string Name { get; set; }
    public string Department { get; set; }

    // Data persistence responsibility
    public void addEmployee(string name, string department) {}
    public void removeEmployee(string name) {}
    
    // Business logic responsibility
    public void processSalary(string name) {}
    
    // Another business logic responsibility
    public void processLeave(string name) {}
}
```

**Problem:** This class is doing too much! It's handling:
- Employee data storage
- Salary processing
- Leave processing

If we need to change how employees are stored (e.g., switch from database to file), we'd have to modify this class. If we need to change salary calculation logic, we'd also modify this same class.

### Good Example ✅

```csharp
// Employee class only holds data
class Employee {
    public string Name { get; set; }
    public string Department { get; set; }
}

// Separate class for data persistence
class EmployeeRepository {
    public void addEmployee(string name, string department) {}
    public void removeEmployee(string name) {}
}

// Separate class for salary processing
class EmployeeSalary {
    public void processSalary(string name) {}
}

// Separate class for leave processing
class EmployeeLeave {
    public void processLeave(string name) {}
}
```

**Benefits:**
- ✅ **Readability** - Each class has a clear, single purpose
- ✅ **Maintainability** - Changes to one responsibility don't affect others
- ✅ **Testability** - Each class can be tested independently
- ✅ **Scalability** - Easy to add new features without touching existing code
- ✅ **Reusability** - Classes can be reused in different contexts

---

## 2. Open/Closed Principle

### Definition
**Software entities (classes, modules, functions) should be open for extension but closed for modification.**

### Why It Matters
- **Open for extension:** You should be able to add new functionality easily
- **Closed for modification:** You shouldn't need to change existing, working code

This principle helps prevent bugs in existing code when adding new features.

### Bad Example ❌

```csharp
class DiscountCalculator {
    public double GetDiscount(string type) {
        if (type == "Regular") return 0.1;
        if (type == "Premium") return 0.2;
        return 0;
    }
}
```

**Problem:** To add a new discount type (e.g., "VIP"), you must modify this class. This violates the "closed for modification" part of the principle and risks breaking existing functionality.

### Good Example ✅

```csharp
// Abstract interface for discounts
interface IDiscount {
    double GetDiscount();
}

// Specific discount implementations
class RegularDiscount : IDiscount {
    public double GetDiscount() => 0.1;
}

class PremiumDiscount : IDiscount {
    public double GetDiscount() => 0.2;
}

// Calculator that works with any discount type
class DiscountCalculator {
    public double GetDiscount(IDiscount discount) {
        return discount.GetDiscount();
    }
}
```

**How It Works:**
- **Open for extension:** To add a new discount type, simply create a new class implementing `IDiscount` (e.g., `VIPDiscount`)
- **Closed for modification:** The `DiscountCalculator` class doesn't need to change when adding new discount types

**Example Usage:**
```csharp
var calculator = new DiscountCalculator();
var regularDiscount = new RegularDiscount();
var premiumDiscount = new PremiumDiscount();

// Works with any discount type without modifying the calculator
double discount1 = calculator.GetDiscount(regularDiscount);
double discount2 = calculator.GetDiscount(premiumDiscount);
```

---

## 3. Liskov Substitution Principle

### Definition
**Objects of a superclass should be replaceable with objects of its subclasses without breaking the application. In other words, derived classes must be substitutable for their base classes.**

### Why It Matters
This principle ensures that inheritance is used correctly. If a subclass can't be used where its parent class is expected, the inheritance relationship is broken.

### Bad Example ❌

```csharp
class Employee {
    public void ProcessSalary() {}
    public void ProcessLeave() {}
}

class Manager : Employee {
    public void ProcessSalary() {
        throw new UnsupportedOperationException(); 
    }
    public void ProcessLeave() {
        throw new UnsupportedOperationException();
    }
}

// Usage
Employee employee = new Manager();
employee.ProcessSalary(); // Throws exception! ❌
```

**Problem:** A `Manager` cannot be used as an `Employee` because it throws exceptions for methods that should work. This breaks the substitution principle - you can't replace `Employee` with `Manager` without breaking the code.

### Good Example ✅

**Approach 1: Separate Interfaces**

```csharp
interface IEmployee {
    void ProcessSalary();
}

interface IManager {
    void ProcessLeave();
}

class Employee : IEmployee {
    public void ProcessSalary() {}
}

class Manager : IManager {
    public void ProcessLeave() {}
}
```

**Approach 2: Composition Over Inheritance**

```csharp
class Employee {
    public void ProcessSalary() {}
    public void ProcessLeave() {}
}

class Manager {
    private Employee _employee = new Employee();
    
    public void ProcessLeave() {
        _employee.ProcessLeave();
    }
}
```

**Approach 3: Proper Polymorphism**

```csharp
class Employee {
    public virtual void ProcessSalary() {}
    public virtual void ProcessLeave() {}
}

class Manager : Employee {
    public override void ProcessLeave() {
        // Manager-specific leave processing
    }
    // ProcessSalary() is inherited and works correctly
}
```

**Key Takeaway:** Subclasses should enhance or specialize behavior, not remove or break it.

---

## 4. Interface Segregation Principle

### Definition
**Clients should not be forced to depend on interfaces they do not use. Many client-specific interfaces are better than one general-purpose interface.**

### Why It Matters
When a class is forced to implement methods it doesn't need, it leads to:
- Unnecessary code (empty method implementations)
- Confusion about what the class actually does
- Tight coupling between unrelated functionalities

### Bad Example ❌

```csharp
interface IEmployeeActions {
    void work();
    void manage();
    void attendMeeting();
}

class Employee : IEmployeeActions {
    public void work() {}
    public void manage() {} // Employee doesn't manage! ❌
    public void attendMeeting() {}
}

class Manager : IEmployeeActions {
    public void work() {} // Manager might not do regular work! ❌
    public void manage() {}
    public void attendMeeting() {}
}
```

**Problem:** Both `Employee` and `Manager` are forced to implement methods they don't actually need. An employee shouldn't have to implement `manage()`, and a manager might not need `work()`.

### Good Example ✅

```csharp
// Segregated interfaces - each with a specific purpose
interface IEmployeeWork {
    void work();
}

interface IEmployeeManage {
    void manage();
}

interface IEmployeeAttendMeeting {
    void attendMeeting();
}

// Employee implements only what it needs
class Employee : IEmployeeWork, IEmployeeAttendMeeting {
    public void work() {}
    public void attendMeeting() {}
}

// Manager implements only what it needs
class Manager : IEmployeeManage, IEmployeeAttendMeeting {
    public void manage() {}
    public void attendMeeting() {}
}
```

**Benefits:**
- ✅ Classes only implement methods they actually use
- ✅ No empty or unnecessary method implementations
- ✅ Clearer intent - you can see what each class does by looking at its interfaces
- ✅ Easier to maintain and understand

---

## 5. Dependency Inversion Principle

### Definition
**High-level modules should not depend on low-level modules. Both should depend on abstractions (interfaces). Abstractions should not depend on details. Details should depend on abstractions.**

### Why It Matters
This principle promotes:
- **Loose coupling** - Classes depend on abstractions, not concrete implementations
- **Flexibility** - Easy to swap implementations (e.g., switch from email to SMS)
- **Testability** - Easy to inject mock objects for testing

### Bad Example ❌

```csharp
class EmailService {
    public void SendEmail(string message) {}
}

class OrderService {
    private EmailService _email = new EmailService(); // Direct dependency ❌
    
    public void ProcessOrder() {
        // Process order logic
        _email.SendEmail("Order confirmed");
    }
}
```

**Problem:** 
- `OrderService` is tightly coupled to `EmailService`
- Can't easily switch to SMS or push notifications
- Hard to test (can't mock `EmailService`)
- Violates dependency inversion - high-level `OrderService` depends on low-level `EmailService`

### Good Example ✅

```csharp
// Abstraction (interface)
interface INotificationService {
    void SendNotification(string message);
}

// Concrete implementation
class EmailService : INotificationService {
    public void SendNotification(string message) {
        // Email sending logic
    }
}

// Another concrete implementation
class SMSService : INotificationService {
    public void SendNotification(string message) {
        // SMS sending logic
    }
}

// High-level module depends on abstraction
class OrderService {
    private readonly INotificationService _notification;
    
    // Dependency injection through constructor
    public OrderService(INotificationService notification) {
        _notification = notification;
    }
    
    public void ProcessOrder() {
        // Process order logic
        _notification.SendNotification("Order confirmed");
    }
}
```

**Benefits:**
- ✅ **Flexibility** - Can easily switch between email, SMS, or any notification service
- ✅ **Testability** - Can inject a mock `INotificationService` for testing
- ✅ **Loose coupling** - `OrderService` doesn't know about specific implementations
- ✅ **Extensibility** - Add new notification types without changing `OrderService`

**Usage Example:**
```csharp
// Can use email
var emailService = new EmailService();
var orderService1 = new OrderService(emailService);

// Or use SMS
var smsService = new SMSService();
var orderService2 = new OrderService(smsService);
```

---

## Summary

| Principle | Key Idea | Benefit |
|-----------|----------|---------|
| **Single Responsibility** | One class, one job | Easier to understand and maintain |
| **Open/Closed** | Open for extension, closed for modification | Add features without breaking existing code |
| **Liskov Substitution** | Subclasses must be usable as their base class | Correct inheritance relationships |
| **Interface Segregation** | Many specific interfaces > one general interface | Classes only implement what they need |
| **Dependency Inversion** | Depend on abstractions, not concretions | Flexible and testable code |

