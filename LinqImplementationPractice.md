# C# LINQ Implementation Practice — Learning Guide

A structured reference for LINQ in C#. Each section maps to a concrete exercise you can implement. All questions are covered; none are skipped.

---

## Table of Contents

1. [Query and Method Syntax Basics](#1-query-and-method-syntax-basics)
2. [Where and Filtering](#2-where-and-filtering)
3. [Select and Projection](#3-select-and-projection)
4. [SelectMany and Flattening](#4-selectmany-and-flattening)
5. [Ordering](#5-ordering)
6. [Take, Skip, and Pagination](#6-take-skip-and-pagination)
7. [First, Last, Single, and Exception Behavior](#7-first-last-single-and-exception-behavior)
8. [Any and All](#8-any-and-all)
9. [Count and LongCount](#9-count-and-longcount)
10. [Sum, Average, Min, Max, and Aggregate](#10-sum-average-min-max-and-aggregate)
11. [Distinct](#11-distinct)
12. [GroupBy and ToLookup](#12-groupby-and-tolookup)
13. [Join, GroupJoin, Left Outer Join, Cross Join](#13-join-groupjoin-left-outer-join-cross-join)
14. [Union, Intersect, Except, Concat](#14-union-intersect-except-concat)
15. [SequenceEqual, Zip, DefaultIfEmpty](#15-sequenceequal-zip-defaultifempty)
16. [Cast and OfType](#16-cast-and-oftype)
17. [Materialization: List, Array, Dictionary](#17-materialization-list-array-dictionary)
18. [Deferred vs Immediate Execution](#18-deferred-vs-immediate-execution)
19. [AsEnumerable and AsQueryable](#19-asenumerable-and-asqueryable)
20. [Custom Extension Methods and IEnumerable](#20-custom-extension-methods-and-ienumerable)
21. [IEnumerable vs IQueryable](#21-ienumerable-vs-iqueryable)
22. [LINQ on Arrays, Lists, Dictionaries, Strings](#22-linq-on-arrays-lists-dictionaries-strings)
23. [Query Syntax: Let, Multiple from, into](#23-query-syntax-let-multiple-from-into)
24. [Null Handling and Optimization](#24-null-handling-and-optimization)
25. [TakeWhile, SkipWhile, Reverse, ElementAt](#25-takewhile-skipwhile-reverse-elementat)
26. [LINQ vs Traditional Loops Performance](#26-linq-vs-traditional-loops-performance)

---

## 1. Query and Method Syntax Basics

### 1.1 Write a LINQ query using query syntax to select all elements from a list

```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5 };

var query = from n in numbers
            select n;

foreach (var n in query)
    Console.WriteLine(n);  // 1, 2, 3, 4, 5
```

### 1.2 Write the same query using method syntax

```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5 };

var query = numbers.Select(n => n);

foreach (var n in query)
    Console.WriteLine(n);  // 1, 2, 3, 4, 5
```

---

## 2. Where and Filtering

### 2.1 Use Where to filter elements based on a condition

```csharp
var numbers = new[] { 1, 2, 3, 4, 5, 6 };

var evens = numbers.Where(n => n % 2 == 0);
foreach (var n in evens)
    Console.WriteLine(n);  // 2, 4, 6
```

### 2.2 Use multiple Where clauses in a single LINQ query

```csharp
var numbers = new[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Method syntax: chain Where
var result = numbers
    .Where(n => n > 2)
    .Where(n => n < 8);

// Query syntax: multiple where
var result2 = from n in numbers
              where n > 2
              where n < 8
              select n;

foreach (var n in result)
    Console.WriteLine(n);  // 3, 4, 5, 6, 7
```

---

## 3. Select and Projection

### 3.1 Use Select to project elements into a new shape

```csharp
var numbers = new[] { 1, 2, 3 };
var squared = numbers.Select(n => n * n);
foreach (var n in squared)
    Console.WriteLine(n);  // 1, 4, 9

// Project to a different type
var people = new[] { ("Alice", 30), ("Bob", 25) };
var names = people.Select(p => p.Item1);
```

### 3.2 Use Select to project into an anonymous type

```csharp
var numbers = new[] { 1, 2, 3 };
var projected = numbers.Select(n => new { Number = n, Squared = n * n });

foreach (var item in projected)
    Console.WriteLine($"{item.Number} -> {item.Squared}");
// 1 -> 1, 2 -> 4, 3 -> 9
```

### 3.3 Filter data using Where before Select and after Select

```csharp
var numbers = new[] { 1, 2, 3, 4, 5 };

// Where then Select: filter first, then project
var a = numbers.Where(n => n > 2).Select(n => n * 10);  // 30, 40, 50

// Select then Where: project first, then filter on projected values
var b = numbers.Select(n => n * 10).Where(n => n > 25);  // 30, 40, 50

// Same result here; order matters when projection changes shape or cost
```

---

## 4. SelectMany and Flattening

### 4.1 Use SelectMany to flatten a collection of collections

```csharp
var matrix = new[] { new[] { 1, 2 }, new[] { 3, 4 }, new[] { 5 } };
var flat = matrix.SelectMany(row => row);
foreach (var n in flat)
    Console.WriteLine(n);  // 1, 2, 3, 4, 5
```

---

## 5. Ordering

### 5.1 Use OrderBy to sort elements ascending

```csharp
var numbers = new[] { 3, 1, 4, 1, 5 };
var ascending = numbers.OrderBy(n => n);
Console.WriteLine(string.Join(", ", ascending));  // 1, 1, 3, 4, 5
```

### 5.2 Use OrderByDescending to sort elements descending

```csharp
var numbers = new[] { 3, 1, 4, 1, 5 };
var descending = numbers.OrderByDescending(n => n);
Console.WriteLine(string.Join(", ", descending));  // 5, 4, 3, 1, 1
```

### 5.3 Use ThenBy and ThenByDescending for secondary sorting

```csharp
var people = new[] { ("Alice", 30), ("Bob", 25), ("Alice", 20) };

var sorted = people
    .OrderBy(p => p.Item1)           // Primary: name
    .ThenBy(p => p.Item2);           // Secondary: age ascending

var sorted2 = people
    .OrderBy(p => p.Item1)
    .ThenByDescending(p => p.Item2); // Secondary: age descending

foreach (var p in sorted)
    Console.WriteLine($"{p.Item1}, {p.Item2}");
// Alice 20, Alice 30, Bob 25
```

---

## 6. Take, Skip, and Pagination

### 6.1 Use Take to fetch first N elements

```csharp
var numbers = new[] { 1, 2, 3, 4, 5 };
var firstThree = numbers.Take(3);
Console.WriteLine(string.Join(", ", firstThree));  // 1, 2, 3
```

### 6.2 Use Skip to skip first N elements

```csharp
var numbers = new[] { 1, 2, 3, 4, 5 };
var afterTwo = numbers.Skip(2);
Console.WriteLine(string.Join(", ", afterTwo));  // 3, 4, 5
```

### 6.3 Combine Skip and Take for pagination

```csharp
var numbers = Enumerable.Range(1, 100).ToArray();
int pageSize = 10;
int page = 2;  // zero-based page index

var pageItems = numbers.Skip(page * pageSize).Take(pageSize);
foreach (var n in pageItems)
    Console.WriteLine(n);  // 11, 12, ..., 20
```

---

## 7. First, Last, Single, and Exception Behavior

### 7.1 Use First and FirstOrDefault

```csharp
var numbers = new[] { 10, 20, 30 };
int first = numbers.First();           // 10
int firstMatch = numbers.First(n => n > 15);  // 20
int firstOrDefault = new int[0].FirstOrDefault();  // 0 (default)
int firstOrDefaultMatch = numbers.FirstOrDefault(n => n > 100);  // 0
```

### 7.2 Use Last and LastOrDefault

```csharp
var numbers = new[] { 10, 20, 30 };
int last = numbers.Last();            // 30
int lastMatch = numbers.Last(n => n < 25);  // 20
int lastOrDefault = new int[0].LastOrDefault();  // 0
```

### 7.3 Use Single and SingleOrDefault

```csharp
var one = new[] { 42 };
int single = one.Single();  // 42

var numbers = new[] { 10, 20, 30 };
int singleMatch = numbers.Single(n => n == 20);  // 20

var empty = new int[0];
int singleOrDefault = empty.SingleOrDefault();  // 0
int singleOrDefaultMatch = numbers.SingleOrDefault(n => n > 100);  // 0
```

### 7.4 Demonstrate exception behavior of First, Single, and Last

```csharp
var empty = Array.Empty<int>();
var multiple = new[] { 1, 2, 3 };

// First() on empty -> InvalidOperationException
// empty.First();

// FirstOrDefault() on empty -> returns default, no exception
_ = empty.FirstOrDefault();

// Last() on empty -> InvalidOperationException
// empty.Last();

// Single() on empty -> InvalidOperationException
// empty.Single();

// Single() on sequence with more than one element -> InvalidOperationException
// multiple.Single();

// Single(n => true) on multiple -> InvalidOperationException
// multiple.Single(n => n > 0);

// SingleOrDefault() on multiple (no predicate match) -> returns default
_ = multiple.SingleOrDefault(n => n > 10);
```

---

## 8. Any and All

### 8.1 Use Any to check existence of elements

```csharp
var numbers = new[] { 1, 2, 3, 4, 5 };
bool hasAny = numbers.Any();                    // true
bool hasEven = numbers.Any(n => n % 2 == 0);   // true
bool hasNegative = numbers.Any(n => n < 0);     // false

var empty = Array.Empty<int>();
bool emptyAny = empty.Any();  // false
```

### 8.2 Use All to validate a condition for all elements

```csharp
var numbers = new[] { 2, 4, 6, 8 };
bool allEven = numbers.All(n => n % 2 == 0);   // true
bool allPositive = numbers.All(n => n > 0);    // true
bool allGreaterThan5 = numbers.All(n => n > 5); // false

var empty = Array.Empty<int>();
bool emptyAll = empty.All(n => n > 10);  // true (vacuous)
```

---

## 9. Count and LongCount

### 9.1 Use Count with and without predicate

```csharp
var numbers = new[] { 1, 2, 3, 4, 5 };
int total = numbers.Count();              // 5
int evenCount = numbers.Count(n => n % 2 == 0);  // 2
```

### 9.2 Use LongCount

```csharp
var numbers = Enumerable.Range(0, 1_000_000);
long total = numbers.LongCount();  // 1000000 (use when count can exceed int.MaxValue)
long evenCount = numbers.LongCount(n => n % 2 == 0);
```

---

## 10. Sum, Average, Min, Max, and Aggregate

### 10.1 Use Sum, Average, Min, and Max

```csharp
var numbers = new[] { 10, 20, 30, 40, 50 };
int sum = numbers.Sum();           // 150
double avg = numbers.Average();    // 30
int min = numbers.Min();           // 10
int max = numbers.Max();           // 50

// With selector
var items = new[] { ("a", 10), ("b", 20) };
int sumOfValues = items.Sum(x => x.Item2);   // 30
double avgOfValues = items.Average(x => x.Item2);
```

### 10.2 Use Aggregate to implement custom aggregation

```csharp
var numbers = new[] { 1, 2, 3, 4, 5 };
int product = numbers.Aggregate(1, (acc, n) => acc * n);  // 120
string concatenated = numbers.Aggregate("", (acc, n) => acc + n.ToString());  // "12345"

// Seed, accumulator, result selector
int sumWithResult = numbers.Aggregate(0, (acc, n) => acc + n, total => total * 2);  // 30
```

---

## 11. Distinct

### 11.1 Use Distinct on primitive types

```csharp
var numbers = new[] { 1, 2, 2, 3, 3, 3, 4 };
var distinct = numbers.Distinct();
Console.WriteLine(string.Join(", ", distinct));  // 1, 2, 3, 4
```

### 11.2 Use Distinct on custom objects using IEquatable<T>

```csharp
public class Person : IEquatable<Person>
{
    public int Id { get; set; }
    public string Name { get; set; } = "";

    public bool Equals(Person? other) => other is not null && Id == other.Id;
    public override bool Equals(object? obj) => Equals(obj as Person);
    public override int GetHashCode() => Id.GetHashCode();
}

var people = new List<Person>
{
    new Person { Id = 1, Name = "Alice" },
    new Person { Id = 1, Name = "Alice" },
    new Person { Id = 2, Name = "Bob" }
};
var distinctPeople = people.Distinct();  // Uses IEquatable; two items (Id 1, Id 2)
```

### 11.3 Use Distinct with a custom IEqualityComparer<T>

```csharp
public class IgnoreCaseComparer : IEqualityComparer<string>
{
    public bool Equals(string? x, string? y) =>
        string.Equals(x, y, StringComparison.OrdinalIgnoreCase);
    public int GetHashCode(string obj) => obj.ToUpperInvariant().GetHashCode();
}

var words = new[] { "apple", "Apple", "APPLE", "banana" };
var distinctWords = words.Distinct(new IgnoreCaseComparer());
Console.WriteLine(string.Join(", ", distinctWords));  // apple, banana
```

---

## 12. GroupBy and ToLookup

### 12.1 Use GroupBy with a single key

```csharp
var numbers = new[] { 1, 2, 3, 4, 5, 6 };
var groups = numbers.GroupBy(n => n % 2);  // 0 -> {2,4,6}, 1 -> {1,3,5}
foreach (var g in groups)
{
    Console.WriteLine($"Key {g.Key}: {string.Join(", ", g)}");
}
```

### 12.2 Use GroupBy with composite keys

```csharp
var people = new[] { ("Alice", "HR"), ("Bob", "IT"), ("Charlie", "HR") };
var byDept = people.GroupBy(p => p.Item2);
var byComposite = people.GroupBy(p => (p.Item2, p.Item1.Length));
foreach (var g in byComposite)
    Console.WriteLine($"{g.Key.Item1}, NameLen={g.Key.Item2}: {string.Join(", ", g.Select(x => x.Item1))}");
```

### 12.3 Use GroupBy followed by aggregation

```csharp
var sales = new[] { ("North", 100), ("South", 150), ("North", 200) };
var totalByRegion = sales
    .GroupBy(s => s.Item1)
    .Select(g => new { Region = g.Key, Total = g.Sum(s => s.Item2) });
foreach (var x in totalByRegion)
    Console.WriteLine($"{x.Region}: {x.Total}");  // North: 300, South: 150
```

### 12.4 Use ToLookup and compare it with GroupBy

```csharp
var numbers = new[] { 1, 2, 3, 4, 5, 6 };
// GroupBy: deferred; re-enumerating re-groups
var groupBy = numbers.GroupBy(n => n % 2);

// ToLookup: immediate; built once, then indexed
var lookup = numbers.ToLookup(n => n % 2);
IEnumerable<int> evens = lookup[0];  // 2, 4, 6 — direct key access
IEnumerable<int> odds = lookup[1];   // 1, 3, 5
bool hasKey = lookup.Contains(0);   // true
```

---

## 13. Join, GroupJoin, Left Outer Join, Cross Join

### 13.1 Use Join to perform inner join between two collections

```csharp
var orders = new[] { (1, "O1"), (2, "O2"), (3, "O3") };
var customers = new[] { (1, "Alice"), (2, "Bob") };

var inner = orders.Join(customers,
    o => o.Item1,
    c => c.Item1,
    (o, c) => new { OrderId = o.Item2, CustomerName = c.Item2 });
foreach (var x in inner)
    Console.WriteLine($"{x.OrderId} -> {x.CustomerName}");
// O1 -> Alice, O2 -> Bob (O3 has no matching customer)
```

### 13.2 Use GroupJoin to perform left join–like behavior

```csharp
var customers = new[] { (1, "Alice"), (2, "Bob") };
var orders = new[] { (1, 100), (1, 200), (2, 150) };

var groupJoin = customers.GroupJoin(orders,
    c => c.Item1,
    o => o.Item1,
    (c, ordersForC) => new { Customer = c.Item2, Orders = ordersForC });
foreach (var x in groupJoin)
    Console.WriteLine($"{x.Customer}: {string.Join(", ", x.Orders.Select(o => o.Item2))}");
// Alice: 100, 200 — Bob: 150
```

### 13.3 Implement left outer join using GroupJoin and SelectMany

```csharp
var customers = new[] { (1, "Alice"), (2, "Bob"), (3, "Charlie") };
var orders = new[] { (1, 100), (1, 200), (2, 150) };

var leftOuter = from c in customers
                join o in orders on c.Item1 equals o.Item1 into ordersForC
                from o in ordersForC.DefaultIfEmpty()
                select new { Customer = c.Item2, OrderAmount = o.Item2 };

foreach (var x in leftOuter)
    Console.WriteLine($"{x.Customer}: {x.OrderAmount}");
// Alice: 100, Alice: 200, Bob: 150, Charlie: 0 (default)
```

### 13.4 Implement cross join using LINQ

```csharp
var colors = new[] { "Red", "Green" };
var sizes = new[] { "S", "M", "L" };

var cross = from c in colors
            from s in sizes
            select new { Color = c, Size = s };

// Or: colors.SelectMany(c => sizes.Select(s => new { Color = c, Size = s }));
foreach (var x in cross)
    Console.WriteLine($"{x.Color} - {x.Size}");
// Red-S, Red-M, Red-L, Green-S, Green-M, Green-L
```

---

## 14. Union, Intersect, Except, Concat

### 14.1 Use Union, Intersect, and Except

```csharp
var a = new[] { 1, 2, 3 };
var b = new[] { 2, 3, 4 };

var union = a.Union(b);         // 1, 2, 3, 4 (distinct)
var intersect = a.Intersect(b); // 2, 3
var except = a.Except(b);       // 1
var exceptReverse = b.Except(a); // 4
```

### 14.2 Use Concat and compare it with Union

```csharp
var a = new[] { 1, 2, 3 };
var b = new[] { 2, 3, 4 };

var concat = a.Concat(b);  // 1, 2, 3, 2, 3, 4 — preserves duplicates, order
var union = a.Union(b);    // 1, 2, 3, 4 — distinct set
```

---

## 15. SequenceEqual, Zip, DefaultIfEmpty

### 15.1 Use SequenceEqual to compare two sequences

```csharp
var a = new[] { 1, 2, 3 };
var b = new[] { 1, 2, 3 };
var c = new[] { 1, 2, 4 };
bool ab = a.SequenceEqual(b);  // true
bool ac = a.SequenceEqual(c);  // false
```

### 15.2 Use Zip to combine two sequences element-wise

```csharp
var numbers = new[] { 1, 2, 3 };
var letters = new[] { "a", "b", "c" };
var zipped = numbers.Zip(letters, (n, l) => $"{n}{l}");
Console.WriteLine(string.Join(", ", zipped));  // 1a, 2b, 3c

// Shorter sequence limits result
var shortLetters = new[] { "x", "y" };
var zipped2 = numbers.Zip(shortLetters);  // (1,"x"), (2,"y")
```

### 15.3 Use DefaultIfEmpty to handle empty sequences

```csharp
var numbers = new[] { 10, 20 };
var withDefault = numbers.DefaultIfEmpty();  // 10, 20
var empty = Array.Empty<int>();
var emptyWithDefault = empty.DefaultIfEmpty();  // 0 (single element)
var emptyWithCustom = empty.DefaultIfEmpty(-1);  // -1
```

---

## 16. Cast and OfType

### 16.1 Use Cast to cast elements in a non-generic collection

```csharp
var list = new System.Collections.ArrayList { 1, 2, 3 };
var cast = list.Cast<int>();
Console.WriteLine(string.Join(", ", cast));  // 1, 2, 3
// Cast throws if any element is not int
```

### 16.2 Use OfType to filter elements by type

```csharp
var mixed = new object[] { 1, "two", 3, "four", 5 };
var integers = mixed.OfType<int>();
Console.WriteLine(string.Join(", ", integers));  // 1, 3, 5
var strings = mixed.OfType<string>();
Console.WriteLine(string.Join(", ", strings));  // two, four
```

---

## 17. Materialization: List, Array, Dictionary

### 17.1 Convert LINQ result to List, Array, and Dictionary

```csharp
var numbers = new[] { 1, 2, 3 };
List<int> list = numbers.Select(n => n * 2).ToList();
int[] array = numbers.Select(n => n * 2).ToArray();

var pairs = new[] { (1, "one"), (2, "two") };
Dictionary<int, string> dict = pairs.ToDictionary(p => p.Item1, p => p.Item2);
```

### 17.2 Use ToDictionary with key collision handling

```csharp
var items = new[] { ("a", 1), ("b", 2), ("a", 3) };
// ToDictionary throws on duplicate key:
// var bad = items.ToDictionary(x => x.Item1, x => x.Item2);

// Option 1: GroupBy then take first (or last) per key
var dict = items.GroupBy(x => x.Item1).ToDictionary(g => g.Key, g => g.First().Item2);

// Option 2: ToLookup for multiple values per key
var lookup = items.ToLookup(x => x.Item1, x => x.Item2);
foreach (var key in lookup["a"])
    Console.WriteLine(key);  // 1, 3
```

---

## 18. Deferred vs Immediate Execution

### 18.1 Demonstrate deferred execution in LINQ

```csharp
var numbers = new List<int> { 1, 2, 3 };
var query = numbers.Where(n => { Console.WriteLine($"Filter {n}"); return n > 1; });
// Nothing printed yet — query not executed
foreach (var n in query)
    Console.WriteLine($"Result {n}");
// Filter 1, Filter 2, Result 2, Filter 3, Result 3 — executes as we iterate
```

### 18.2 Demonstrate immediate execution using ToList or ToArray

```csharp
var numbers = new List<int> { 1, 2, 3 };
var materialized = numbers.Where(n => n > 1).ToList();
// All filtering runs here; list is built
foreach (var n in materialized)
    Console.WriteLine(n);
// Second foreach does not re-run the Where; uses cached list
```

### 18.3 Show how modifying source collection affects deferred queries

```csharp
var numbers = new List<int> { 1, 2, 3 };
var query = numbers.Where(n => n > 1);
numbers.Add(4);
var result = query.ToList();  // 2, 3, 4 — query uses current state when executed
numbers.Clear();
var result2 = query.ToList(); // empty — deferred query sees updated source
```

---

## 19. AsEnumerable and AsQueryable

### 19.1 Use AsEnumerable to force LINQ-to-Objects execution

```csharp
// On IQueryable (e.g. EF DbSet), AsEnumerable() switches to client-side LINQ
IQueryable<int> queryable = new[] { 1, 2, 3 }.AsQueryable();
var asEnum = queryable.AsEnumerable();  // Further operators use IEnumerable overloads
var list = asEnum.Where(x => x > 1).ToList();  // Executed in memory
```

### 19.2 Use AsQueryable and explain its effect via implementation

```csharp
// AsQueryable wraps IEnumerable<T> in IQueryable<T>; expression tree is built
// but typically executed in memory (provider-dependent)
IEnumerable<int> source = new[] { 1, 2, 3 };
IQueryable<int> q = source.AsQueryable();
var filtered = q.Where(x => x > 1);  // Expression tree; when enumerated, runs in memory
Console.WriteLine(string.Join(", ", filtered));
```

---

## 20. Custom Extension Methods and IEnumerable

### 20.1 Write a custom LINQ extension method

```csharp
public static class MyLinq
{
    public static IEnumerable<T> WhereNotNull<T>(this IEnumerable<T> source) where T : class
    {
        foreach (var item in source)
        {
            if (item != null)
                yield return item;
        }
    }
}

var withNulls = new string?[] { "a", null, "b", null, "c" };
var noNulls = withNulls.WhereNotNull();
Console.WriteLine(string.Join(", ", noNulls));  // a, b, c
```

### 20.2 Implement IEnumerable<T> and use LINQ on it

```csharp
public class CountUpTo : IEnumerable<int>
{
    private readonly int _max;
    public CountUpTo(int max) => _max = max;
    public IEnumerator<int> GetEnumerator() => new CountUpToEnumerator(_max);
    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() => GetEnumerator();
}

public class CountUpToEnumerator : IEnumerator<int>
{
    private readonly int _max;
    private int _current;
    public CountUpToEnumerator(int max) { _max = max; _current = 0; }
    public bool MoveNext() => ++_current <= _max;
    public int Current => _current;
    object System.Collections.IEnumerator.Current => Current;
    public void Reset() => _current = 0;
    public void Dispose() { }
}

var seq = new CountUpTo(5);
var doubled = seq.Select(n => n * 2);
Console.WriteLine(string.Join(", ", doubled));  // 2, 4, 6, 8, 10
```

### 20.3 Implement IEnumerator<T> manually

See `CountUpToEnumerator` above: it implements `IEnumerator<int>` with `MoveNext`, `Current`, `Reset`, and `Dispose`.

### 20.4 Demonstrate how yield return works with LINQ

```csharp
static IEnumerable<int> Generate(int max)
{
    for (int i = 1; i <= max; i++)
        yield return i;
}

var seq = Generate(5);
var filtered = seq.Where(n => n % 2 == 0);  // Composes with LINQ
Console.WriteLine(string.Join(", ", filtered));  // 2, 4
// Each MoveNext in filtered pulls from Generate; lazy pipeline
```

---

## 21. IEnumerable vs IQueryable

### 21.1 Compare IEnumerable<T> vs IQueryable<T> using code

```csharp
// IEnumerable: operators run in memory (delegate-based)
IEnumerable<int> ie = new[] { 1, 2, 3 };
var ieQuery = ie.Where(x => x > 1);  // Compiler uses Func<int, bool>

// IQueryable: operators build expression tree; execution can be remote (e.g. SQL)
IQueryable<int> iq = new[] { 1, 2, 3 }.AsQueryable();
var iqQuery = iq.Where(x => x > 1);  // Expression<Func<int, bool>>
Console.WriteLine(iqQuery.Expression);  // Tree representation
var result = iqQuery.ToList();  // When enumerated, provider executes (here in-memory)
```

---

## 22. LINQ on Arrays, Lists, Dictionaries, Strings

### 22.1 Use LINQ with arrays, lists, and dictionaries

```csharp
int[] arr = { 1, 2, 3 };
var fromArray = arr.Where(x => x > 1).ToList();

List<string> list = new List<string> { "a", "bb", "ccc" };
var fromList = list.Where(s => s.Length > 1).Select(s => s.Length);

var dict = new Dictionary<int, string> { [1] = "one", [2] = "two" };
var fromDict = dict.Where(kv => kv.Key > 1).Select(kv => kv.Value);
var keys = dict.Keys.Where(k => k > 1);
var values = dict.Values.Where(v => v.Length > 3);
```

### 22.2 Use LINQ on strings and characters

```csharp
string text = "Hello World";
var chars = text.Where(c => char.IsLetter(c));
var upper = text.Where(c => char.IsUpper(c));
var letters = text.Select(c => c);
var distinctChars = text.Distinct().OrderBy(c => c);
Console.WriteLine(string.Join("", distinctChars));
```

---

## 23. Query Syntax: Let, Multiple from, into

### 23.1 Use Let keyword in query syntax

```csharp
var numbers = new[] { 1, 2, 3, 4, 5 };
var query = from n in numbers
            let squared = n * n
            let cubed = squared * n
            where cubed > 10
            select new { n, squared, cubed };
foreach (var x in query)
    Console.WriteLine($"{x.n} -> {x.squared}, {x.cubed}");
```

### 23.2 Use multiple from clauses in query syntax

```csharp
var rows = new[] { 1, 2 };
var cols = new[] { 10, 20 };
var product = from r in rows
              from c in cols
              select r * c;
Console.WriteLine(string.Join(", ", product));  // 10, 20, 20, 40
```

### 23.3 Use into keyword to continue a query

```csharp
var numbers = new[] { 1, 2, 3, 4, 5 };
var query = from n in numbers
            where n > 1
            select n * 2 into doubled
            where doubled > 4
            select doubled;
Console.WriteLine(string.Join(", ", query));  // 6, 8, 10
```

---

## 24. Null Handling and Optimization

### 24.1 Handle null values safely in LINQ queries

```csharp
var items = new string?[] { "a", null, "b", null };
var nonNull = items.Where(x => x != null);
var withDefault = items.Select(x => x ?? "");
var firstOrNull = items.FirstOrDefault();  // null if empty
// Nullable reference types and ?? help avoid NRE
```

### 24.2 Optimize LINQ queries to avoid multiple enumerations

```csharp
var source = Enumerable.Range(1, 100);
// Bad: multiple enumerations (e.g. Count() then Sum() each enumerate)
// int c = source.Count(); int s = source.Sum();

// Good: materialize once if you need to iterate multiple times
var list = source.ToList();
int c = list.Count;
int s = list.Sum();
// Or use a single pass with Aggregate if possible
```

---

## 25. TakeWhile, SkipWhile, Reverse, ElementAt

### 25.1 Use TakeWhile and SkipWhile

```csharp
var numbers = new[] { 1, 2, 3, 4, 5, 1, 2 };
var takeWhile = numbers.TakeWhile(n => n < 4);   // 1, 2, 3
var skipWhile = numbers.SkipWhile(n => n < 4);   // 4, 5, 1, 2
Console.WriteLine(string.Join(", ", takeWhile));
Console.WriteLine(string.Join(", ", skipWhile));
```

### 25.2 Use Reverse and observe execution behavior

```csharp
var numbers = new[] { 1, 2, 3, 4, 5 };
var reversed = numbers.Reverse();  // Deferred; buffers when enumerated
Console.WriteLine(string.Join(", ", reversed));  // 5, 4, 3, 2, 1
// On non-indexed sequences, Reverse must buffer the whole sequence
```

### 25.3 Use ElementAt and ElementAtOrDefault

```csharp
var numbers = new[] { 10, 20, 30, 40 };
int at2 = numbers.ElementAt(2);     // 30
int at10 = numbers.ElementAtOrDefault(10);  // 0 (default)
// ElementAt(10) would throw ArgumentOutOfRangeException
```

---

## 26. LINQ vs Traditional Loops Performance

### 26.1 Compare performance of LINQ vs traditional loops via implementation

```csharp
var numbers = Enumerable.Range(1, 1_000_000).ToArray();
var sw = System.Diagnostics.Stopwatch.StartNew();

// LINQ
var linqSum = numbers.Where(n => n % 2 == 0).Sum();
sw.Stop();
var linqMs = sw.ElapsedMilliseconds;

sw.Restart();
// Traditional loop
int loopSum = 0;
foreach (var n in numbers)
    if (n % 2 == 0) loopSum += n;
sw.Stop();
var loopMs = sw.ElapsedMilliseconds;

Console.WriteLine($"LINQ: {linqSum} in {linqMs}ms");
Console.WriteLine($"Loop: {loopSum} in {loopMs}ms");
// Loop is often faster due to no delegate/iterator overhead; LINQ is more readable.
// For complex pipelines or IQueryable (e.g. DB), LINQ can be better (single query).
```

---

## Quick Reference

| Need | Use |
|------|-----|
| All elements | `from x in source select x` or `source.Select(x => x)` |
| Filter | `Where(predicate)` |
| Project | `Select(selector)` |
| Anonymous type | `Select(x => new { A = x.Foo, B = x.Bar })` |
| Flatten | `SelectMany(collectionSelector)` |
| Sort | `OrderBy` / `OrderByDescending`, `ThenBy` / `ThenByDescending` |
| Page | `Skip(n).Take(pageSize)` |
| First/Last/Single | `First`, `FirstOrDefault`, `Last`, `LastOrDefault`, `Single`, `SingleOrDefault` |
| Existence | `Any()`, `Any(predicate)` |
| All match | `All(predicate)` |
| Count | `Count()`, `Count(predicate)`, `LongCount()` |
| Aggregate | `Sum`, `Average`, `Min`, `Max`, `Aggregate` |
| Distinct | `Distinct()`, `Distinct(IEqualityComparer<T>)` |
| Group | `GroupBy(keySelector)` → deferred; `ToLookup` → immediate |
| Inner join | `Join(inner, outerKey, innerKey, resultSelector)` |
| Left outer | `GroupJoin` + `SelectMany` + `DefaultIfEmpty` |
| Cross join | `from a in A from b in B select ...` |
| Set ops | `Union`, `Intersect`, `Except`, `Concat` (no dedup) |
| Compare | `SequenceEqual` |
| Pairwise | `Zip(first, second, resultSelector)` |
| Empty handling | `DefaultIfEmpty()`, `DefaultIfEmpty(defaultValue)` |
| Type filter | `OfType<T>()`; `Cast<T>()` (throws on invalid) |
| Materialize | `ToList()`, `ToArray()`, `ToDictionary()`, `ToLookup()` |
| Deferred | Most operators; execution on enumeration |
| Immediate | `ToList`, `ToArray`, `Count`, `Sum`, etc. |
| Force in-memory | `AsEnumerable()` on IQueryable |
| Custom | Extension method on `IEnumerable<T>` with `yield return` |

---

*Use this guide as a checklist: implement each subsection in a small console app to solidify your understanding. Every question listed is covered above.*
