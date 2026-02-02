# SOLID, Clean Code, Singleton & Factory — Implementation Practice

A structured reference for SOLID principles, Clean Code (DRY, KISS, YAGNI), Singleton, and Factory patterns in C#. Each section maps to a concrete exercise you can implement. All questions are covered; none are skipped.

---

## Table of Contents

1. [DRY (Don't Repeat Yourself)](#1-dry-dont-repeat-yourself)
2. [KISS (Keep It Simple, Stupid)](#2-kiss-keep-it-simple-stupid)
3. [YAGNI (You Aren't Gonna Need It)](#3-yagni-you-arent-gonna-need-it)
4. [Single Responsibility Principle](#4-single-responsibility-principle)
5. [Open/Closed Principle](#5-openclosed-principle)
6. [Liskov Substitution Principle](#6-liskov-substitution-principle)
7. [Interface Segregation Principle](#7-interface-segregation-principle)
8. [Dependency Inversion Principle](#8-dependency-inversion-principle)
9. [Singleton Pattern](#9-singleton-pattern)
10. [Factory Pattern](#10-factory-pattern)
11. [Factory Method and Abstract Factory](#11-factory-method-and-abstract-factory)
12. [Factory with DI and SOLID](#12-factory-with-di-and-solid)
13. [Unit Tests for Singleton and Factory](#13-unit-tests-for-singleton-and-factory)

---

## 1. DRY (Don't Repeat Yourself)

### 1.1 Refactor duplicate code to follow the DRY principle

```csharp
// Before: duplicate logic
public void ProcessOrder(Order order)
{
    ValidateOrder(order);
    Log($"Processing order {order.Id}");
    // ... process
    Log($"Order {order.Id} completed");
}
public void ProcessRefund(Refund refund)
{
    ValidateRefund(refund);
    Log($"Processing refund {refund.Id}");
    // ... process
    Log($"Refund {refund.Id} completed");
}

// After: DRY
public void ProcessOrder(Order order)
{
    ValidateOrder(order);
    ProcessWithLog(order.Id, "order", () => DoProcessOrder(order));
}
public void ProcessRefund(Refund refund)
{
    ValidateRefund(refund);
    ProcessWithLog(refund.Id, "refund", () => DoProcessRefund(refund));
}
private void ProcessWithLog(int id, string type, Action work)
{
    Log($"Processing {type} {id}");
    work();
    Log($"{type} {id} completed");
}
```

### 1.2 Identify and remove hidden duplication across methods

```csharp
// Before: same validation + formatting repeated
public string FormatCustomerReport(Customer c)
{
    if (c == null || string.IsNullOrEmpty(c.Name)) throw new ArgumentException("Invalid customer");
    return $"{c.Name}: {c.Email}";
}
public string FormatSupplierReport(Supplier s)
{
    if (s == null || string.IsNullOrEmpty(s.Name)) throw new ArgumentException("Invalid supplier");
    return $"{s.Name}: {s.Contact}";
}

// After: shared validation and formatting
private static void ValidateName(object entity, string name, string entityType)
{
    if (entity == null || string.IsNullOrEmpty(name))
        throw new ArgumentException($"Invalid {entityType}");
}
public string FormatCustomerReport(Customer c)
{
    ValidateName(c, c?.Name ?? "", "customer");
    return $"{c!.Name}: {c.Email}";
}
public string FormatSupplierReport(Supplier s)
{
    ValidateName(s, s?.Name ?? "", "supplier");
    return $"{s!.Name}: {s.Contact}";
}
```

### 1.3 Apply DRY by extracting common logic into a helper method

```csharp
// Before: repeated null check + trim + max length
public void SetTitle(string? value) => _title = (value ?? "").Trim().Length > 100 ? (value ?? "").Trim()[..100] : (value ?? "").Trim();
public void SetDescription(string? value) => _description = (value ?? "").Trim().Length > 500 ? (value ?? "").Trim()[..500] : (value ?? "").Trim();

// After: helper
private static string NormalizeString(string? value, int maxLength)
{
    var s = (value ?? "").Trim();
    return s.Length <= maxLength ? s : s[..maxLength];
}
public void SetTitle(string? value) => _title = NormalizeString(value, 100);
public void SetDescription(string? value) => _description = NormalizeString(value, 500);
```

### 1.4 Apply DRY by creating a reusable class

```csharp
// Before: same parsing logic in multiple places
// OrderService: ParseDate(order.DateString) ...
// ReportService: ParseDate(report.DateString) ...

// After: reusable helper class
public static class DateParser
{
    private static readonly string[] Formats = { "yyyy-MM-dd", "dd/MM/yyyy", "MM-dd-yyyy" };
    public static DateTime? TryParse(string? value)
    {
        if (string.IsNullOrWhiteSpace(value)) return null;
        return DateTime.TryParseExact(value.Trim(), Formats, CultureInfo.InvariantCulture, DateTimeStyles.None, out var dt) ? dt : null;
    }
}
// Usage: DateParser.TryParse(order.DateString)
```

---

## 2. KISS (Keep It Simple, Stupid)

### 2.1 Refactor a complex method to follow the KISS principle

```csharp
// Before: one complex method
public decimal CalculatePrice(Order order)
{
    decimal total = 0;
    for (int i = 0; i < order.Items.Count; i++)
    {
        var item = order.Items[i];
        decimal lineTotal = item.Quantity * item.UnitPrice;
        if (order.Customer.IsPremium) lineTotal *= 0.9m;
        if (item.Quantity > 10) lineTotal *= 0.95m;
        total += lineTotal;
    }
    if (total > 1000) total *= 0.98m;
    return Math.Round(total, 2);
}

// After: KISS — smaller, named steps
public decimal CalculatePrice(Order order)
{
    var lineTotals = order.Items.Select(GetLineTotal).ToList();
    decimal total = lineTotals.Sum();
    total = ApplyDiscount(total, order.Customer.IsPremium);
    total = ApplyBulkDiscount(total, order.Items.Sum(i => i.Quantity));
    return Math.Round(total, 2);
}
private decimal GetLineTotal(OrderItem item) => item.Quantity * item.UnitPrice;
private decimal ApplyDiscount(decimal total, bool isPremium) => isPremium ? total * 0.9m : total;
private decimal ApplyBulkDiscount(decimal total, int totalQty) => totalQty > 10 ? total * 0.95m : total;
```

### 2.2 Replace nested conditionals with simpler logic (KISS)

```csharp
// Before: nested conditionals
public string GetStatus(Order order)
{
    if (order == null) return "Unknown";
    else
    {
        if (order.ShippedDate.HasValue) return "Shipped";
        else if (order.PaymentDate.HasValue) return "Paid";
        else if (order.CreatedDate > DateTime.UtcNow.AddDays(-1)) return "New";
        else return "Pending";
    }
}

// After: flat, early returns
public string GetStatus(Order order)
{
    if (order == null) return "Unknown";
    if (order.ShippedDate.HasValue) return "Shipped";
    if (order.PaymentDate.HasValue) return "Paid";
    if (order.CreatedDate > DateTime.UtcNow.AddDays(-1)) return "New";
    return "Pending";
}
```

### 2.3 Break a large method into smaller methods to follow KISS

```csharp
// Before: one 80-line method doing validation, load, transform, save, log
public void ImportOrders(string path) { /* 80 lines */ }

// After: small methods, single responsibility each
public void ImportOrders(string path)
{
    ValidatePath(path);
    var rows = ReadCsv(path);
    var orders = MapToOrders(rows);
    SaveOrders(orders);
    LogImportComplete(orders.Count);
}
private void ValidatePath(string path) { /* ... */ }
private List<CsvRow> ReadCsv(string path) { /* ... */ }
private List<Order> MapToOrders(List<CsvRow> rows) { /* ... */ }
private void SaveOrders(List<Order> orders) { /* ... */ }
private void LogImportComplete(int count) { /* ... */ }
```

### 2.4 Remove unnecessary abstractions to follow KISS

```csharp
// Before: abstraction for one implementation
public interface IOrderIdGenerator { string Generate(); }
public class OrderIdGenerator : IOrderIdGenerator { public string Generate() => Guid.NewGuid().ToString("N")[..8]; }
// Injected everywhere, only one implementation ever

// After: use concrete type or static helper until you have a real need to swap
public static class OrderIdGenerator
{
    public static string Generate() => Guid.NewGuid().ToString("N")[..8];
}
// Or keep interface only when you have tests/multiple implementations
```

---

## 3. YAGNI (You Aren't Gonna Need It)

### 3.1 Identify premature features violating YAGNI

```csharp
// YAGNI violation: generic repository, UoW, and 5 layers when you only need one API and one DB call
public interface IRepository<T> { ... }
public interface IUnitOfWork { ... }
public class OrderRepository : IRepository<Order> { ... }
public class OrderService { ... }
public class OrderController { ... }
// If the app only reads orders and displays them, start with a single service + DbContext.

// YAGNI: "future" parameters and options
public void ExportOrders(string path, bool includeDeleted = false, string format = "csv", bool compress = false);
// Add includeDeleted, format, compress only when a real requirement appears.
```

### 3.2 Remove unused code that violates YAGNI

```csharp
// Before: unused overloads and "just in case" code
public void Save(Order order) { ... }
public void Save(Order order, bool validateOnly) { }  // Never called
public void SaveBatch(IEnumerable<Order> orders) { }   // Never called
private void ValidateForFutureMultiCurrency(Order order) { }  // Never used

// After: keep only what is used
public void Save(Order order) { ... }
```

### 3.3 Refactor code to eliminate over-engineering (YAGNI)

```csharp
// Before: plugin system for two fixed exporters
public interface IExportStrategy { void Export(Order order, Stream stream); }
public class CsvExportStrategy : IExportStrategy { ... }
public class JsonExportStrategy : IExportStrategy { ... }
public class ExportStrategyFactory { ... }
public class ExportService { ... }

// After: simple API until you need extensibility
public class OrderExporter
{
    public void ExportCsv(Order order, Stream stream) { ... }
    public void ExportJson(Order order, Stream stream) { ... }
}
// Introduce strategy/factory when you have a third format or runtime selection.
```

---

## 4. Single Responsibility Principle

### 4.1 Implement a class that follows Single Responsibility Principle

```csharp
// Single responsibility: only validates order data
public class OrderValidator
{
    public bool IsValid(Order order)
    {
        if (order == null) return false;
        if (order.Items.Count == 0) return false;
        if (order.Total <= 0) return false;
        return true;
    }
}

// Single responsibility: only persists orders
public class OrderRepository
{
    public void Save(Order order) { /* save to DB */ }
}
```

### 4.2 Refactor a class that violates Single Responsibility Principle

```csharp
// Before: one class validates, saves, sends email, logs
public class OrderProcessor
{
    public void Process(Order order)
    {
        if (order.Items.Count == 0) throw new ArgumentException();
        _db.Save(order);
        _emailService.Send(order.Customer.Email, "Order confirmed");
        _logger.Log($"Processed {order.Id}");
    }
}

// After: separate responsibilities
public class OrderValidator { public void Validate(Order order) { ... } }
public class OrderRepository { public void Save(Order order) { ... } }
public class OrderProcessor
{
    public void Process(Order order)
    {
        _validator.Validate(order);
        _repository.Save(order);
        _notificationService.SendOrderConfirmation(order);
    }
}
```

---

## 5. Open/Closed Principle

### 5.1 Implement Open/Closed Principle using inheritance

```csharp
public abstract class DiscountCalculator
{
    public abstract decimal Calculate(decimal amount);
}
public class NoDiscount : DiscountCalculator { public override decimal Calculate(decimal amount) => amount; }
public class PercentDiscount : DiscountCalculator
{
    private readonly decimal _percent;
    public PercentDiscount(decimal percent) => _percent = percent;
    public override decimal Calculate(decimal amount) => amount * (1 - _percent / 100);
}
// New discount type: add new class, no change to existing classes
```

### 5.2 Implement Open/Closed Principle using interfaces

```csharp
public interface IDiscountCalculator
{
    decimal Calculate(decimal amount);
}
public class PercentDiscount : IDiscountCalculator { ... }
public class FixedAmountDiscount : IDiscountCalculator { ... }

public class OrderPricing
{
    private readonly IDiscountCalculator _discount;
    public OrderPricing(IDiscountCalculator discount) => _discount = discount;
    public decimal GetTotal(decimal subtotal) => _discount.Calculate(subtotal);
}
// New behavior: implement IDiscountCalculator; OrderPricing stays unchanged
```

### 5.3 Refactor code to add new behavior without modifying existing code

```csharp
// New requirement: add "Buy 2 get 10% off" — add new class only
public class BuyTwoGetTenPercentOff : IDiscountCalculator
{
    public decimal Calculate(decimal amount) { /* new logic */ return amount; }
}
// OrderPricing and other calculators unchanged; register new type in DI
```

---

## 6. Liskov Substitution Principle

### 6.1 Demonstrate a violation of Liskov Substitution Principle

```csharp
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
    public int Area => Width * Height;
}
public class Square : Rectangle
{
    public override int Width { set { base.Width = value; base.Height = value; } }
    public override int Height { set { base.Height = value; base.Width = value; } }
}
// Violation: code that sets Width and Height independently expects Rectangle behavior.
// Square changes both dimensions; substitute Square for Rectangle breaks that expectation.
```

### 6.2 Fix Liskov Substitution Principle violation

```csharp
// Option 1: don't inherit; compose or use separate types
public interface IShape { int Area { get; } }
public class Rectangle : IShape { public int Width { get; set; } public int Height { get; set; } public int Area => Width * Height; }
public class Square : IShape { public int Side { get; set; } public int Area => Side * Side; }

// Option 2: if Square must substitute Rectangle, ensure setters preserve invariants
// and document that setting one dimension sets both (or make Square immutable from base view).
```

---

## 7. Interface Segregation Principle

### 7.1 Implement Interface Segregation Principle using multiple small interfaces

```csharp
public interface IReadable { Order GetById(int id); }
public interface IWritable { void Save(Order order); }
public interface IDeletable { void Delete(int id); }

public class OrderRepository : IReadable, IWritable, IDeletable
{
    public Order GetById(int id) { ... }
    public void Save(Order order) { ... }
    public void Delete(int id) { ... }
}
// Clients depend only on what they use: IReadable for reports, IWritable for import
```

### 7.2 Refactor a fat interface to follow Interface Segregation Principle

```csharp
// Before: fat interface
public interface IOrderService
{
    Order Create(Order order);
    void Update(Order order);
    void Delete(int id);
    Order GetById(int id);
    void SendInvoice(int orderId);
    void ExportToCsv(Stream stream);
}

// After: segregated
public interface IOrderCommand { Order Create(Order order); void Update(Order order); void Delete(int id); }
public interface IOrderQuery { Order GetById(int id); }
public interface IOrderNotification { void SendInvoice(int orderId); }
public interface IOrderExport { void ExportToCsv(Stream stream); }
// Implementations can implement one or more; callers depend on the smallest needed interface
```

---

## 8. Dependency Inversion Principle

### 8.1 Implement Dependency Inversion Principle using constructor injection

```csharp
public interface ILogger { void Log(string message); }
public interface IOrderRepository { void Save(Order order); }

public class OrderService
{
    private readonly IOrderRepository _repository;
    private readonly ILogger _logger;

    public OrderService(IOrderRepository repository, ILogger logger)
    {
        _repository = repository;
        _logger = logger;
    }

    public void PlaceOrder(Order order)
    {
        _repository.Save(order);
        _logger.Log($"Order {order.Id} placed");
    }
}
// High-level OrderService depends on abstractions; concrete implementations injected
```

### 8.2 Refactor tightly coupled code to follow Dependency Inversion Principle

```csharp
// Before: direct dependency on concrete classes
public class OrderService
{
    private readonly SqlOrderRepository _repository = new SqlOrderRepository();
    private readonly FileLogger _logger = new FileLogger();
    public void PlaceOrder(Order order) { _repository.Save(order); _logger.Log("..."); }
}

// After: depend on abstractions, inject via constructor
public class OrderService
{
    private readonly IOrderRepository _repository;
    private readonly ILogger _logger;
    public OrderService(IOrderRepository repository, ILogger logger) { _repository = repository; _logger = logger; }
    public void PlaceOrder(Order order) { _repository.Save(order); _logger.Log("..."); }
}
```

---

## 9. Singleton Pattern

### 9.1 Implement a basic Singleton pattern

```csharp
public sealed class Singleton
{
    private static Singleton? _instance;
    public static Singleton Instance => _instance ??= new Singleton();
    private Singleton() { }
}
```

### 9.2 Implement a thread-safe Singleton using lock

```csharp
public sealed class ThreadSafeSingleton
{
    private static ThreadSafeSingleton? _instance;
    private static readonly object _lock = new object();

    public static ThreadSafeSingleton Instance
    {
        get
        {
            lock (_lock)
                return _instance ??= new ThreadSafeSingleton();
        }
    }
    private ThreadSafeSingleton() { }
}
```

### 9.3 Implement a Singleton using double-check locking

```csharp
public sealed class DoubleCheckSingleton
{
    private static volatile DoubleCheckSingleton? _instance;
    private static readonly object _lock = new object();

    public static DoubleCheckSingleton Instance
    {
        get
        {
            if (_instance == null)
                lock (_lock)
                    _instance ??= new DoubleCheckSingleton();
            return _instance;
        }
    }
    private DoubleCheckSingleton() { }
}
```

### 9.4 Implement a Singleton using Lazy<T>

```csharp
public sealed class LazySingleton
{
    private static readonly Lazy<LazySingleton> _instance = new Lazy<LazySingleton>(() => new LazySingleton());
    public static LazySingleton Instance => _instance.Value;
    private LazySingleton() { }
}
```

### 9.5 Break a Singleton using reflection

```csharp
var instance = Singleton.Instance;
var type = typeof(Singleton);
var ctor = type.GetConstructor(BindingFlags.NonPublic | BindingFlags.Instance, null, Type.EmptyTypes, null);
var another = (Singleton)ctor!.Invoke(null);
Console.WriteLine(ReferenceEquals(instance, another));  // False — second instance created
```

### 9.6 Prevent breaking Singleton via reflection

```csharp
public sealed class Singleton
{
    private static readonly Lazy<Singleton> _instance = new Lazy<Singleton>(() => new Singleton());
    public static Singleton Instance => _instance.Value;

    private static bool _created;
    private Singleton()
    {
        if (_created)
            throw new InvalidOperationException("Use Singleton.Instance");
        _created = true;
    }
}
// Reflection calling ctor again sees _created and throws
```

---

## 10. Factory Pattern

### 10.1 Implement a basic Factory pattern

```csharp
public class Product { }
public class ProductA : Product { }
public class ProductB : Product { }

public static class ProductFactory
{
    public static Product Create(string type)
    {
        return type.ToUpperInvariant() switch
        {
            "A" => new ProductA(),
            "B" => new ProductB(),
            _ => throw new ArgumentException($"Unknown product: {type}")
        };
    }
}
```

### 10.2 Implement Factory pattern using interfaces

```csharp
public interface IProduct { string Name { get; } }
public class ProductA : IProduct { public string Name => "A"; }
public class ProductB : IProduct { public string Name => "B"; }

public interface IProductFactory
{
    IProduct Create(string type);
}
public class ProductFactory : IProductFactory
{
    public IProduct Create(string type) => type.ToUpperInvariant() switch
    {
        "A" => new ProductA(),
        "B" => new ProductB(),
        _ => throw new ArgumentException($"Unknown: {type}")
    };
}
```

### 10.3 Implement Factory pattern using enums

```csharp
public enum ProductType { A, B }
public interface IProduct { string Name { get; } }
public class ProductA : IProduct { public string Name => "A"; }
public class ProductB : IProduct { public string Name => "B"; }

public class ProductFactory
{
    public IProduct Create(ProductType type) => type switch
    {
        ProductType.A => new ProductA(),
        ProductType.B => new ProductB(),
        _ => throw new ArgumentOutOfRangeException(nameof(type))
    };
}
```

### 10.4 Refactor conditional object creation logic into a Factory

```csharp
// Before: creation logic scattered
if (type == "A") return new ProductA();
if (type == "B") return new ProductB();

// After: centralized in factory
return _productFactory.Create(type);
```

---

## 11. Factory Method and Abstract Factory

### 11.1 Implement Factory Method pattern

```csharp
public abstract class DocumentCreator
{
    public abstract IDocument CreateDocument();
    public void SaveDocument() { var doc = CreateDocument(); doc.Save(); }
}
public class PdfCreator : DocumentCreator
{
    public override IDocument CreateDocument() => new PdfDocument();
}
public class WordCreator : DocumentCreator
{
    public override IDocument CreateDocument() => new WordDocument();
}
```

### 11.2 Implement Abstract Factory pattern

```csharp
public interface IUiFactory
{
    IButton CreateButton();
    ITextBox CreateTextBox();
}
public class WinUiFactory : IUiFactory
{
    public IButton CreateButton() => new WinButton();
    public ITextBox CreateTextBox() => new WinTextBox();
}
public class MacUiFactory : IUiFactory
{
    public IButton CreateButton() => new MacButton();
    public ITextBox CreateTextBox() => new MacTextBox();
}
```

### 11.3 Compare Factory and Abstract Factory via implementation

```csharp
// Simple Factory: one method, creates one product type
var product = productFactory.Create("A");

// Abstract Factory: one factory creates a family of related products (Button + TextBox)
var factory = new WinUiFactory();
var button = factory.CreateButton();
var textBox = factory.CreateTextBox();
// Switching to Mac: use MacUiFactory; all products are from same family
```

---

## 12. Factory with DI and SOLID

### 12.1 Combine Factory with Dependency Injection

```csharp
// Register factory in DI
builder.Services.AddSingleton<IProductFactory, ProductFactory>();

// Consumer receives IProductFactory
public class OrderService
{
    private readonly IProductFactory _factory;
    public OrderService(IProductFactory factory) => _factory = factory;
    public IProduct CreateProduct(string type) => _factory.Create(type);
}
```

### 12.2 Replace new keyword usage using Factory

```csharp
// Before: new scattered in code
var product = type == "A" ? new ProductA() : new ProductB();

// After: factory encapsulates creation
var product = _productFactory.Create(type);
```

### 12.3 Demonstrate violation of SOLID in Factory implementation

```csharp
// Violations: factory depends on concrete types (DIP), adds new type by editing method (OCP)
public class BadFactory
{
    public IProduct Create(string type)
    {
        if (type == "A") return new ProductA();  // DIP: depends on concrete ProductA
        if (type == "B") return new ProductB();  // OCP: must modify to add ProductC
        return new DefaultProduct();
    }
}
```

### 12.4 Refactor Factory to comply with SOLID principles

```csharp
// Register mappings in DI; factory depends on abstraction; new types added by registration (OCP)
public interface IProductFactory
{
    IProduct Create(string type);
}
public class ProductFactory : IProductFactory
{
    private readonly IReadOnlyDictionary<string, Func<IProduct>> _creators;
    public ProductFactory(IEnumerable<KeyValuePair<string, Func<IProduct>>> creators)
        => _creators = creators.ToDictionary(x => x.Key, x => x.Value);
    public IProduct Create(string type) => _creators.TryGetValue(type, out var create) ? create() : throw new ArgumentException($"Unknown: {type}");
}
// Registration: builder.Services.AddSingleton<IProductFactory>(sp => new ProductFactory(new Dictionary<string, Func<IProduct>> { ["A"] = () => new ProductA(), ["B"] = () => new ProductB() }));
```

---

## 13. Unit Tests for Singleton and Factory

### 13.1 Implement unit tests for Singleton and Factory patterns

```csharp
[Fact]
public void Singleton_ReturnsSameInstance()
{
    var a = Singleton.Instance;
    var b = Singleton.Instance;
    Assert.Same(a, b);
}

[Fact]
public void Factory_Create_ReturnsCorrectType()
{
    var factory = new ProductFactory();
    var a = factory.Create("A");
    var b = factory.Create("B");
    Assert.IsType<ProductA>(a);
    Assert.IsType<ProductB>(b);
}

[Fact]
public void Factory_Create_UnknownType_Throws()
{
    var factory = new ProductFactory();
    Assert.Throws<ArgumentException>(() => factory.Create("X"));
}
```

---

## Quick Reference

| Topic | Guideline |
|-------|-----------|
| DRY | Extract repeated logic to helper or reusable class; remove hidden duplication |
| KISS | Small methods, flat conditionals, no unnecessary abstractions |
| YAGNI | Remove unused code; add features when required, not in advance |
| SRP | One class, one reason to change; split validation, persistence, notification |
| OCP | Extend via new types (inheritance/interfaces), avoid changing existing code |
| LSP | Subtypes must be substitutable for base; no surprising behavior |
| ISP | Small, focused interfaces; clients depend only on what they use |
| DIP | Depend on abstractions; inject concretions via constructor |
| Singleton | Single instance; use Lazy&lt;T&gt; or lock for thread safety |
| Factory | Centralize object creation; return interface type |
| Factory Method | Creator subclass defines which product to instantiate |
| Abstract Factory | Factory creates a family of related products |
| Factory + DI | Register factory; inject IProductFactory; register mappings for OCP |

---

*Use this guide as a checklist: implement each subsection in a small console or test project. Every question listed is covered above.*
