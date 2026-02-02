# EF Core Implementation Practice — Learning Guide

A structured reference for Entity Framework Core in C#. Each section maps to a concrete exercise you can implement. All questions are covered; none are skipped.

---

## Table of Contents

1. [DbContext Setup](#1-dbcontext-setup)
2. [Entity Mapping and Conventions](#2-entity-mapping-and-conventions)
3. [Fluent API: Table, Schema, Keys](#3-fluent-api-table-schema-keys)
4. [Fluent API: Properties and Value Generation](#4-fluent-api-properties-and-value-generation)
5. [Relationships: One-to-One, One-to-Many, Many-to-Many](#5-relationships-one-to-one-one-to-many-many-to-many)
6. [Foreign Keys and Cascade Delete](#6-foreign-keys-and-cascade-delete)
7. [Indexes](#7-indexes)
8. [Add and Save Entities](#8-add-and-save-entities)
9. [Update Entities](#9-update-entities)
10. [Delete Entities](#10-delete-entities)
11. [Entity States and Tracking](#11-entity-states-and-tracking)
12. [AsNoTracking and Find](#12-asnotracking-and-find)
13. [Loading Related Data: Include, Explicit, Lazy](#13-loading-related-data-include-explicit-lazy)
14. [Projection and LINQ in Queries](#14-projection-and-linq-in-queries)
15. [Raw SQL](#15-raw-sql)
16. [Pagination and Aggregates](#16-pagination-and-aggregates)
17. [Compiled Queries and Query Filters](#17-compiled-queries-and-query-filters)
18. [Soft Delete and Concurrency](#18-soft-delete-and-concurrency)
19. [Migrations](#19-migrations)
20. [Seeding and Owned Types](#20-seeding-and-owned-types)
21. [Value Converters, Enums, Complex Types](#21-value-converters-enums-complex-types)
22. [Logging and Performance](#22-logging-and-performance)
23. [N+1, SplitQuery, Batch Size](#23-n1-splitquery-batch-size)
24. [Transactions and Resiliency](#24-transactions-and-resiliency)
25. [Database Providers and Testing](#25-database-providers-and-testing)
26. [IQueryable vs IEnumerable and Client Evaluation](#26-iqueryable-vs-ienumerable-and-client-evaluation)

---

## 1. DbContext Setup

### 1.1 Create a DbContext with a single DbSet<T>

```csharp
using Microsoft.EntityFrameworkCore;

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
}

public class AppDbContext : DbContext
{
    public DbSet<Product> Products => Set<Product>();
}
```

### 1.2 Configure DbContext using OnConfiguring

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Product> Products => Set<Product>();

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        if (!optionsBuilder.IsConfigured)
            optionsBuilder.UseSqlite("Data Source=app.db");
    }
}
```

### 1.3 Configure DbContext using dependency injection

```csharp
// In Program.cs or Startup.ConfigureServices:
// builder.Services.AddDbContext<AppDbContext>(options =>
//     options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    public DbSet<Product> Products => Set<Product>();
}
```

---

## 2. Entity Mapping and Conventions

### 2.1 Create entity classes and map them using EF Core conventions

Conventions: class name → table name; `Id` or `ClassNameId` → primary key; navigation properties → relationships.

```csharp
public class Blog
{
    public int Id { get; set; }
    public string Title { get; set; } = "";
    public List<Post> Posts { get; set; } = new();
}

public class Post
{
    public int Id { get; set; }
    public string Title { get; set; } = "";
    public int BlogId { get; set; }
    public Blog Blog { get; set; } = null!;
}

public class AppDbContext : DbContext
{
    public DbSet<Blog> Blogs => Set<Blog>();
    public DbSet<Post> Posts => Set<Post>();
}
```

### 2.2 Configure entity mapping using Fluent API

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>(entity =>
    {
        entity.HasKey(e => e.Id);
        entity.Property(e => e.Title).IsRequired().HasMaxLength(200);
    });
}
```

---

## 3. Fluent API: Table, Schema, Keys

### 3.1 Configure table name and schema using Fluent API

```csharp
modelBuilder.Entity<Blog>(entity =>
{
    entity.ToTable("Blogs", "blogging");
});
```

### 3.2 Configure primary key using HasKey

```csharp
modelBuilder.Entity<Product>(entity =>
{
    entity.HasKey(e => e.Id);
});
```

### 3.3 Configure composite primary key

```csharp
public class OrderLine
{
    public int OrderId { get; set; }
    public int LineId { get; set; }
    public int Quantity { get; set; }
}

modelBuilder.Entity<OrderLine>(entity =>
{
    entity.HasKey(e => new { e.OrderId, e.LineId });
});
```

### 3.4 Configure alternate key

```csharp
modelBuilder.Entity<Customer>(entity =>
{
    entity.HasKey(e => e.Id);
    entity.HasAlternateKey(e => e.Email);  // Unique index on Email
});
```

---

## 4. Fluent API: Properties and Value Generation

### 4.1 Configure required and optional properties

```csharp
modelBuilder.Entity<Product>(entity =>
{
    entity.Property(e => e.Name).IsRequired();
    entity.Property(e => e.Description).IsRequired(false);
});
```

### 4.2 Configure column type and max length

```csharp
modelBuilder.Entity<Product>(entity =>
{
    entity.Property(e => e.Name).HasMaxLength(200);
    entity.Property(e => e.Price).HasColumnType("decimal(18,2)");
});
```

### 4.3 Configure default values for columns

```csharp
modelBuilder.Entity<Order>(entity =>
{
    entity.Property(e => e.CreatedAt).HasDefaultValueSql("GETUTCDATE()");
    entity.Property(e => e.Status).HasDefaultValue("Pending");
});
```

### 4.4 Configure shadow properties

```csharp
modelBuilder.Entity<Blog>(entity =>
{
    entity.Property<DateTime>("CreatedAt");
    entity.Property<string>("CreatedBy");
});
// Access: context.Entry(blog).Property<DateTime>("CreatedAt").CurrentValue = DateTime.UtcNow;
```

### 4.5 Configure value generation (ValueGeneratedOnAdd)

```csharp
modelBuilder.Entity<Product>(entity =>
{
    entity.Property(e => e.Id).ValueGeneratedOnAdd();
});
// Or use [DatabaseGenerated(DatabaseGeneratedOption.Identity)] on property
```

---

## 5. Relationships: One-to-One, One-to-Many, Many-to-Many

### 5.1 Configure one-to-one relationship

```csharp
public class Author
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public AuthorBio? Bio { get; set; }
}

public class AuthorBio
{
    public int Id { get; set; }
    public string Bio { get; set; } = "";
    public int AuthorId { get; set; }
    public Author Author { get; set; } = null!;
}

modelBuilder.Entity<Author>(entity =>
{
    entity.HasOne(a => a.Bio).WithOne(b => b.Author).HasForeignKey<AuthorBio>(b => b.AuthorId);
});
```

### 5.2 Configure one-to-many relationship

```csharp
modelBuilder.Entity<Blog>(entity =>
{
    entity.HasMany(b => b.Posts).WithOne(p => p.Blog).HasForeignKey(p => p.BlogId);
});
```

### 5.3 Configure many-to-many relationship using join entity

```csharp
public class Post
{
    public int Id { get; set; }
    public List<PostTag> PostTags { get; set; } = new();
}

public class Tag
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public List<PostTag> PostTags { get; set; } = new();
}

public class PostTag
{
    public int PostId { get; set; }
    public Post Post { get; set; } = null!;
    public int TagId { get; set; }
    public Tag Tag { get; set; } = null!;
}

modelBuilder.Entity<PostTag>(entity =>
{
    entity.HasKey(pt => new { pt.PostId, pt.TagId });
    entity.HasOne(pt => pt.Post).WithMany(p => p.PostTags).HasForeignKey(pt => pt.PostId);
    entity.HasOne(pt => pt.Tag).WithMany(t => t.PostTags).HasForeignKey(pt => pt.TagId);
});
```

### 5.4 Configure many-to-many relationship without explicit join entity (EF Core 5+)

```csharp
public class Post
{
    public int Id { get; set; }
    public List<Tag> Tags { get; set; } = new();
}

public class Tag
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public List<Post> Posts { get; set; } = new();
}

modelBuilder.Entity<Post>()
    .HasMany(p => p.Tags)
    .WithMany(t => t.Posts)
    .UsingEntity<Dictionary<string, object>>(
        "PostTag",
        j => j.HasOne<Tag>().WithMany().HasForeignKey("TagId"),
        j => j.HasOne<Post>().WithMany().HasForeignKey("PostId"));
```

### 5.5 Configure foreign keys explicitly

```csharp
modelBuilder.Entity<Post>(entity =>
{
    entity.HasOne(p => p.Blog).WithMany(b => b.Posts).HasForeignKey(p => p.BlogId);
    entity.Property(e => e.BlogId).HasColumnName("BlogFk");
});
```

---

## 6. Foreign Keys and Cascade Delete

### 6.1 Configure cascade delete behavior

```csharp
modelBuilder.Entity<Blog>(entity =>
{
    entity.HasMany(b => b.Posts).WithOne(p => p.Blog)
        .HasForeignKey(p => p.BlogId)
        .OnDelete(DeleteBehavior.Cascade);
});
```

### 6.2 Disable cascade delete

```csharp
modelBuilder.Entity<Blog>(entity =>
{
    entity.HasMany(b => b.Posts).WithOne(p => p.Blog)
        .HasForeignKey(p => p.BlogId)
        .OnDelete(DeleteBehavior.Restrict);
});
```

---

## 7. Indexes

### 7.1 Configure indexes using HasIndex

```csharp
modelBuilder.Entity<Product>(entity =>
{
    entity.HasIndex(e => e.Name);
});
```

### 7.2 Configure unique indexes

```csharp
modelBuilder.Entity<Customer>(entity =>
{
    entity.HasIndex(e => e.Email).IsUnique();
});
```

### 7.3 Configure filtered indexes

```csharp
modelBuilder.Entity<Post>(entity =>
{
    entity.HasIndex(e => e.PublishedAt).HasFilter("[PublishedAt] IS NOT NULL");
});
// SQL Server: HasFilter; other providers may use different syntax
```

---

## 8. Add and Save Entities

### 8.1 Add and save a single entity

```csharp
var product = new Product { Name = "Widget", Price = 9.99m };
context.Products.Add(product);
await context.SaveChangesAsync();
```

### 8.2 Add and save related entities

```csharp
var blog = new Blog { Title = "My Blog" };
blog.Posts.Add(new Post { Title = "First Post" });
blog.Posts.Add(new Post { Title = "Second Post" });
context.Blogs.Add(blog);
await context.SaveChangesAsync();
```

---

## 9. Update Entities

### 9.1 Update an entity using tracked state

```csharp
var product = await context.Products.FirstAsync(p => p.Id == 1);
product.Name = "Updated Name";
product.Price = 12.99m;
await context.SaveChangesAsync();  // EF detects changes; sets State = Modified
```

### 9.2 Update an entity using Attach

```csharp
var product = new Product { Id = 1, Name = "Updated", Price = 10m };
context.Products.Attach(product);
context.Entry(product).Property(p => p.Name).IsModified = true;
context.Entry(product).Property(p => p.Price).IsModified = true;
await context.SaveChangesAsync();
```

### 9.3 Update an entity using Entry.State

```csharp
var product = new Product { Id = 1, Name = "New Name", Price = 15m };
context.Entry(product).State = EntityState.Modified;
await context.SaveChangesAsync();
```

---

## 10. Delete Entities

### 10.1 Delete an entity using tracked state

```csharp
var product = await context.Products.FirstAsync(p => p.Id == 1);
context.Products.Remove(product);
await context.SaveChangesAsync();
```

### 10.2 Delete an entity without fetching it first

```csharp
var product = new Product { Id = 1 };
context.Products.Remove(product);
await context.SaveChangesAsync();
// Or: context.Entry(product).State = EntityState.Deleted;
```

---

## 11. Entity States and Tracking

### 11.1 Track entity states (Added, Modified, Deleted, Unchanged)

```csharp
var product = new Product { Name = "New" };
Console.WriteLine(context.Entry(product).State);  // Added

var existing = await context.Products.FirstAsync();
existing.Name = "Changed";
Console.WriteLine(context.Entry(existing).State);  // Modified

context.Products.Remove(existing);
Console.WriteLine(context.Entry(existing).State);   // Deleted

var untouched = await context.Products.AsNoTracking().FirstAsync();
context.Attach(untouched);
Console.WriteLine(context.Entry(untouched).State); // Unchanged
```

---

## 12. AsNoTracking and Find

### 12.1 Use AsNoTracking in queries

```csharp
var products = await context.Products.AsNoTracking().ToListAsync();
// Entities are not tracked; read-only scenario; better performance
```

### 12.2 Compare tracking vs no-tracking queries

```csharp
// Tracking: entities cached in context; change detection; more memory
var tracked = await context.Products.FirstAsync(p => p.Id == 1);
tracked.Name = "X";
await context.SaveChangesAsync();  // Update sent

// No-tracking: not cached; no change detection; less memory; faster for read-only
var untracked = await context.Products.AsNoTracking().FirstAsync(p => p.Id == 1);
untracked.Name = "Y";
await context.SaveChangesAsync();  // No update (entity not tracked)
```

### 12.3 Use Find vs FirstOrDefault

```csharp
// Find: uses primary key; checks local cache first; sync overload
var product = await context.Products.FindAsync(1);

// FirstOrDefault: always hits DB (unless from local)
var product2 = await context.Products.FirstOrDefaultAsync(p => p.Id == 1);
```

---

## 13. Loading Related Data: Include, Explicit, Lazy

### 13.1 Use Include to load related data

```csharp
var blogs = await context.Blogs.Include(b => b.Posts).ToListAsync();
```

### 13.2 Use ThenInclude for nested relationships

```csharp
var blogs = await context.Blogs
    .Include(b => b.Posts)
    .ThenInclude(p => p.Author)
    .ToListAsync();
```

### 13.3 Use explicit loading of navigation properties

```csharp
var blog = await context.Blogs.FirstAsync(b => b.Id == 1);
await context.Entry(blog).Collection(b => b.Posts).LoadAsync();
// Or: await context.Entry(blog).Reference(b => b.Owner).LoadAsync();
```

### 13.4 Use lazy loading with proxies

```csharp
// 1. Install: Microsoft.EntityFrameworkCore.Proxies
// 2. optionsBuilder.UseLazyLoadingProxies();
// 3. Navigation properties must be virtual
public class Blog
{
    public virtual ICollection<Post> Posts { get; set; } = new List<Post>();
}
// Access blog.Posts triggers load when first accessed
```

---

## 14. Projection and LINQ in Queries

### 14.1 Project query results using Select

```csharp
var names = await context.Products.Select(p => new { p.Id, p.Name }).ToListAsync();
var dtos = await context.Products.Select(p => new ProductDto { Id = p.Id, Name = p.Name }).ToListAsync();
```

### 14.2 Use LINQ filters in EF Core queries

```csharp
var active = await context.Products
    .Where(p => p.IsActive && p.Price > 0)
    .OrderBy(p => p.Name)
    .ToListAsync();
```

---

## 15. Raw SQL

### 15.1 Use raw SQL queries with FromSqlRaw

```csharp
var products = await context.Products
    .FromSqlRaw("SELECT * FROM Products WHERE Price > {0}", 10m)
    .ToListAsync();
```

### 15.2 Execute raw SQL commands using ExecuteSqlRaw

```csharp
await context.Database.ExecuteSqlRawAsync(
    "UPDATE Products SET Price = Price * 1.1 WHERE CategoryId = {0}", categoryId);
```

### 15.3 Prevent SQL injection in raw SQL queries

```csharp
// Use parameterized queries (interpolation in FromSqlRaw/ExecuteSqlRaw is parameterized)
await context.Products.FromSqlRaw("SELECT * FROM Products WHERE Id = {0}", id).ToListAsync();
// Or use FromSqlInterpolated
FormattableString sql = $"SELECT * FROM Products WHERE Id = {id}";
var list = await context.Products.FromSqlInterpolated(sql).ToListAsync();
// Never concatenate user input: "WHERE Id = " + id
```

---

## 16. Pagination and Aggregates

### 16.1 Implement pagination using Skip and Take

```csharp
int page = 1, pageSize = 10;
var paged = await context.Products
    .OrderBy(p => p.Id)
    .Skip((page - 1) * pageSize)
    .Take(pageSize)
    .ToListAsync();
```

### 16.2 Use Any, Count, and Sum in EF Core queries

```csharp
bool hasAny = await context.Products.AnyAsync();
int count = await context.Products.CountAsync();
int countFiltered = await context.Products.CountAsync(p => p.Price > 10);
decimal total = await context.Products.SumAsync(p => p.Price);
```

---

## 17. Compiled Queries and Query Filters

### 17.1 Use compiled queries

```csharp
private static readonly Func<AppDbContext, int, IAsyncEnumerable<Product>> GetByPrice =
    EF.CompileAsyncQuery((AppDbContext ctx, int minPrice) =>
        ctx.Products.Where(p => p.Price >= minPrice));

await foreach (var p in GetByPrice(context, 10))
    Console.WriteLine(p.Name);
```

### 17.2 Configure query filters (global filters)

```csharp
modelBuilder.Entity<Post>().HasQueryFilter(p => !p.IsDeleted);
// All queries on Posts automatically add WHERE IsDeleted = 0
// Ignore filter: context.Posts.IgnoreQueryFilters()
```

### 17.3 Implement soft delete using query filters

```csharp
modelBuilder.Entity<Post>().HasQueryFilter(p => !p.IsDeleted);
// Soft delete: set IsDeleted = true; filter hides deleted rows
public async Task SoftDelete(int id)
{
    var post = await context.Posts.IgnoreQueryFilters().FirstAsync(p => p.Id == id);
    post.IsDeleted = true;
    await context.SaveChangesAsync();
}
```

---

## 18. Soft Delete and Concurrency

### 18.1 Handle concurrency using concurrency tokens

```csharp
modelBuilder.Entity<Product>(entity =>
{
    entity.Property(e => e.RowVersion).IsConcurrencyToken();
});
// Update RowVersion on save; if value changed in DB, concurrency exception
```

### 18.2 Use RowVersion for optimistic concurrency

```csharp
public class Product
{
    public byte[] RowVersion { get; set; } = null!;
}
modelBuilder.Entity<Product>(entity =>
{
    entity.Property(e => e.RowVersion).IsRowVersion();
});
```

### 18.3 Handle DbUpdateConcurrencyException

```csharp
try
{
    await context.SaveChangesAsync();
}
catch (DbUpdateConcurrencyException ex)
{
    foreach (var entry in ex.Entries)
    {
        var dbValues = await entry.GetDatabaseValuesAsync();
        if (dbValues == null) { /* deleted */ continue; }
        // Refresh, merge, or prompt user; then retry
        entry.OriginalValues.SetValues(dbValues);
    }
    await context.SaveChangesAsync();
}
```

---

## 19. Migrations

### 19.1 Create and apply migrations

```bash
# Create migration
dotnet ef migrations add InitialCreate

# Apply to database
dotnet ef database update
```

### 19.2 Roll back a migration

```bash
# Roll back one migration
dotnet ef database update PreviousMigrationName

# Remove last migration (if not applied)
dotnet ef migrations remove
```

### 19.3 Generate SQL script from migrations

```bash
dotnet ef migrations script
dotnet ef migrations script FromMigration ToMigration -o script.sql
```

### 19.4 Update database using migrations

```bash
dotnet ef database update
# Or in code: context.Database.Migrate();
```

### 19.5 Use EnsureCreated vs Migrate

```csharp
// EnsureCreated: creates DB and schema; no migrations; for tests/prototyping
await context.Database.EnsureCreatedAsync();

// Migrate: applies pending migrations; use in production
await context.Database.MigrateAsync();
```

---

## 20. Seeding and Owned Types

### 20.1 Seed data using OnModelCreating

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>().HasData(
        new Blog { Id = 1, Title = "Seed Blog" });
}
// Add migration to capture seed data; then update database
```

### 20.2 Seed data using migrations

```csharp
// In migration Up():
migrationBuilder.InsertData(
    table: "Blogs",
    columns: new[] { "Id", "Title" },
    values: new object[] { 1, "Seed Blog" });
```

### 20.3 Configure owned entity types

```csharp
modelBuilder.Entity<Order>().OwnsOne(o => o.ShippingAddress);
public class Order
{
    public int Id { get; set; }
    public Address ShippingAddress { get; set; } = null!;
}
public class Address
{
    public string Street { get; set; } = "";
    public string City { get; set; } = "";
}
```

---

## 21. Value Converters, Enums, Complex Types

### 21.1 Configure value converters

```csharp
modelBuilder.Entity<Product>(entity =>
{
    entity.Property(e => e.Tags).HasConversion(
        v => string.Join(",", v),
        v => v.Split(",", StringSplitOptions.None).ToList());
});
```

### 21.2 Configure enums mapping

```csharp
public enum OrderStatus { Pending, Shipped, Delivered }
modelBuilder.Entity<Order>(entity =>
{
    entity.Property(e => e.Status).HasConversion<string>();  // or int (default)
});
```

### 21.3 Configure complex types/value objects (EF Core 8+)

```csharp
modelBuilder.Entity<Order>().ComplexProperty(o => o.ShippingAddress);
// Address is a value object; no own table; columns on Order table
```

---

## 22. Logging and Performance

### 22.1 Log generated SQL queries

```csharp
optionsBuilder.UseSqlServer(connectionString)
    .LogTo(Console.WriteLine, LogLevel.Information);
// Or use ILoggerFactory from DI
```

### 22.2 Enable sensitive data logging

```csharp
optionsBuilder.EnableSensitiveDataLogging();  // Log parameter values; dev only
```

### 22.3 Measure query performance

```csharp
var sw = Stopwatch.StartNew();
var list = await context.Products.ToListAsync();
sw.Stop();
Console.WriteLine(sw.ElapsedMilliseconds);
// Or use DiagnosticSource / Application Insights
```

---

## 23. N+1, SplitQuery, Batch Size

### 23.1 Avoid N+1 query problem

```csharp
// Bad: 1 + N queries
var blogs = await context.Blogs.ToListAsync();
foreach (var b in blogs)
    _ = b.Posts.Count;  // Triggers one query per blog

// Good: one query with Include
var blogsWithPosts = await context.Blogs.Include(b => b.Posts).ToListAsync();
```

### 23.2 Use SplitQuery and SingleQuery

```csharp
// Single query (default): one SQL with JOINs
var blogs = await context.Blogs.Include(b => b.Posts).ToListAsync();

// Split: multiple SQL; avoids Cartesian explosion with multiple collections
var blogsSplit = await context.Blogs.Include(b => b.Posts).AsSplitQuery().ToListAsync();
```

### 23.3 Configure batch size for SaveChanges

```csharp
optionsBuilder.UseSqlServer(connectionString, b => b.MaxBatchSize(100));
// Inserts/updates sent in batches of 100
```

---

## 24. Transactions and Resiliency

### 24.1 Use transactions explicitly

```csharp
using var transaction = await context.Database.BeginTransactionAsync();
try
{
    context.Products.Add(new Product { Name = "A" });
    await context.SaveChangesAsync();
    context.Products.Add(new Product { Name = "B" });
    await context.SaveChangesAsync();
    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

### 24.2 Use BeginTransaction and Commit

See 24.1: `BeginTransactionAsync()`, `CommitAsync()`, `RollbackAsync()`.

### 24.3 Use TransactionScope with EF Core

```csharp
using var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);
await context.SaveChangesAsync();
// ... other work ...
scope.Complete();
```

### 24.4 Handle connection resiliency and retries

```csharp
optionsBuilder.UseSqlServer(connectionString,
    b => b.EnableRetryOnFailure(maxRetryCount: 3));
```

### 24.5 Use execution strategies

```csharp
var strategy = context.Database.CreateExecutionStrategy();
await strategy.ExecuteAsync(async () =>
{
    using var transaction = await context.Database.BeginTransactionAsync();
    await context.SaveChangesAsync();
    await transaction.CommitAsync();
});
```

---

## 25. Database Providers and Testing

### 25.1 Configure database providers (SQL Server, SQLite)

```csharp
// SQL Server
optionsBuilder.UseSqlServer(connectionString);

// SQLite
optionsBuilder.UseSqlite("Data Source=app.db");
```

### 25.2 Use in-memory database for testing

```csharp
var options = new DbContextOptionsBuilder<AppDbContext>()
    .UseInMemoryDatabase(databaseName: "TestDb")
    .Options;
using var context = new AppDbContext(options);
```

### 25.3 Write unit tests for EF Core using in-memory provider

```csharp
[Fact]
public async Task AddProduct_SavesToDb()
{
    var options = new DbContextOptionsBuilder<AppDbContext>()
        .UseInMemoryDatabase("TestDb").Options;
    await using var context = new AppDbContext(options);
    context.Products.Add(new Product { Name = "Test", Price = 1m });
    await context.SaveChangesAsync();
    var count = await context.Products.CountAsync();
    Assert.Equal(1, count);
}
```

### 25.4 Mock DbContext and DbSet

```csharp
var mockSet = new Mock<DbSet<Product>>();
var data = new List<Product> { new Product { Id = 1, Name = "A" } }.AsQueryable();
mockSet.As<IQueryable<Product>>().Setup(m => m.Provider).Returns(data.Provider);
mockSet.As<IQueryable<Product>>().Setup(m => m.Expression).Returns(data.Expression);
mockSet.As<IQueryable<Product>>().Setup(m => m.ElementType).Returns(data.ElementType);
mockSet.As<IQueryable<Product>>().Setup(m => m.GetEnumerator()).Returns(data.GetEnumerator());
var mockContext = new Mock<AppDbContext>();
mockContext.Setup(c => c.Products).Returns(mockSet.Object);
```

---

## 26. IQueryable vs IEnumerable and Client Evaluation

### 26.1 Compare IQueryable vs IEnumerable in EF Core queries

```csharp
// IQueryable: translated to SQL; executed on server
IQueryable<Product> q = context.Products.Where(p => p.Price > 10);
var list = await q.ToListAsync();  // Single SQL query

// IEnumerable: after ToList(), further LINQ runs in memory
var inMemory = (await context.Products.ToListAsync()).Where(p => p.Price > 10);
```

### 26.2 Force client-side evaluation and observe behavior

```csharp
// Client evaluation: EF throws if unsupported operator used on IQueryable
// Example: .Where(p => SomeCSharpMethod(p.Name)) — SomeCSharpMethod cannot be translated
var bad = context.Products.Where(p => p.Name.ToLower().Contains("x"));  // OK, translated
var client = context.Products.AsEnumerable().Where(p => SomeLocalMethod(p));  // In memory
// AsEnumerable() pulls data; then Where runs in process
```

---

## Quick Reference

| Topic | Use |
|-------|-----|
| DbContext | Inherit `DbContext`; `DbSet<T>`; `OnConfiguring` or DI with `DbContextOptions<T>` |
| Conventions | `Id`/`ClassNameId` PK; navigation → FK; class name → table name |
| Fluent API | `OnModelCreating` → `modelBuilder.Entity<T>()` |
| Table/schema | `ToTable("Name", "Schema")` |
| Keys | `HasKey`, composite `HasKey(e => new { a, b })`, `HasAlternateKey` |
| Properties | `IsRequired`, `HasMaxLength`, `HasColumnType`, `HasDefaultValue`, shadow `Property<T>("Name")` |
| Value gen | `ValueGeneratedOnAdd()` |
| 1:1 | `HasOne().WithOne().HasForeignKey<T>()` |
| 1:many | `HasMany().WithOne().HasForeignKey()` |
| Many:many | Join entity + two `HasOne/WithMany` or `HasMany().WithMany()` (EF5+) |
| Cascade | `OnDelete(DeleteBehavior.Cascade|Restrict)` |
| Indexes | `HasIndex()`, `IsUnique()`, `HasFilter()` |
| Add | `Add()`, `AddRange()`; `SaveChangesAsync()` |
| Update | Modify tracked entity; or `Attach` + `IsModified`; or `Entry(e).State = Modified` |
| Delete | `Remove()` or `Entry(e).State = Deleted` |
| States | Added, Modified, Deleted, Unchanged |
| No tracking | `AsNoTracking()` |
| Include | `Include()`, `ThenInclude()` |
| Raw SQL | `FromSqlRaw`, `FromSqlInterpolated`; `ExecuteSqlRawAsync`; always parameterize |
| Pagination | `Skip().Take()` |
| Compiled | `EF.CompileAsyncQuery()` |
| Query filter | `HasQueryFilter()`; `IgnoreQueryFilters()` |
| Concurrency | Concurrency token; `IsRowVersion()`; handle `DbUpdateConcurrencyException` |
| Migrations | `dotnet ef migrations add`, `database update`, `migrations script` |
| EnsureCreated | No migrations; `Migrate()` for production |
| Seeding | `HasData()` in OnModelCreating; or in migration |
| Owned | `OwnsOne()`, `OwnsMany()` |
| Converters | `HasConversion()` |
| Enums | `HasConversion<string>()` or default int |
| Logging | `LogTo()`, `EnableSensitiveDataLogging()` |
| N+1 | Use `Include()` (or explicit load / lazy) |
| Split query | `AsSplitQuery()` |
| Batch size | `MaxBatchSize()` in provider options |
| Transactions | `BeginTransactionAsync()`, `CommitAsync()`, `RollbackAsync()` |
| Resiliency | `EnableRetryOnFailure()`; execution strategy |
| Testing | In-memory `UseInMemoryDatabase()`; or mock `DbSet`/`DbContext` |
| IQueryable | Server-side; keep as `IQueryable` until materialize |
| Client eval | `AsEnumerable()` or unsupported operator; runs in process after data loaded |

---

*Use this guide as a checklist: implement each subsection in a small app or test project. Every question listed is covered above.*
