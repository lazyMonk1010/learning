# ASP.NET Core Middlewares

A guide to understanding, configuring, and building with the ASP.NET Core request pipeline—from built-in middleware to a full Todo API with custom middleware.

---

## What Are Middlewares?

**Middlewares** are components that process HTTP requests and responses in ASP.NET Core. They sit between the incoming request and your application logic (controllers), so you can:

- **Inspect** or **modify** the request before it hits your controllers  
- **Inspect** or **modify** the response after your code runs  
- **Short-circuit** the pipeline (e.g., return a response without reaching the controller)

Think of them as a chain of handlers: each one can do work, then either call the next handler or stop and send a response.

---

## Why Use Middlewares?

| Use Case | Purpose |
|----------|--------|
| **Authentication & Authorization** | Verify identity and permissions |
| **Logging** | Record requests, responses, and errors |
| **Caching** | Serve cached responses and reduce load |
| **Error Handling** | Centralized exception handling and error pages |
| **Compression** | Compress responses (e.g., gzip) |
| **Security** | HTTPS, headers, CORS, etc. |

---

## How the Pipeline Works

1. Each middleware receives **`HttpContext`**, does its work, then either:
   - Calls **`next(context)`** to pass the request to the next middleware, or  
   - **Short-circuits** by writing a response and not calling `next`.
2. **Order matters**: middlewares run in the order they are registered in `Program.cs`.

Example order:

```text
app.UseMiddleware<Middleware1>();
app.UseMiddleware<Middleware2>();
app.UseMiddleware<Middleware3>();
```

Execution: **Middleware1 → Middleware2 → Middleware3** (request), then back up the chain (response).

---

## Types of Middlewares

- **Built-in** – Provided by ASP.NET Core (routing, auth, CORS, exception handling, etc.).  
- **Custom** – Middleware you write for your app’s specific needs.

---

## Built-in Middlewares (Overview)

| Middleware | Purpose |
|------------|--------|
| **Exception handling** | Catches unhandled exceptions and can show custom error pages or API responses. |
| **Routing** | Maps URLs to endpoints (controllers, minimal APIs). |
| **Authentication** | Identifies the user (e.g., cookies, JWT). |
| **Authorization** | Checks if the user is allowed to access the resource. |
| **CORS** | Controls which origins can call your API from the browser. |
| **Request logging** | Logs each HTTP request (e.g., with Serilog). |

---

## Custom Middleware

A custom middleware must:

1. Accept a **`RequestDelegate`** in its constructor (the “next” step in the pipeline).  
2. Expose **`Invoke`** or **`InvokeAsync`** that takes **`HttpContext`** and processes the request.

### Example: Simple logging middleware

```csharp
public class CustomMiddleware
{
    private readonly RequestDelegate _next;

    public CustomMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        Console.WriteLine("Custom Middleware: Request started");
        await _next(context);
        Console.WriteLine("Custom Middleware: Response sent");
    }
}
```

### Register in `Program.cs`

```csharp
app.UseMiddleware<CustomMiddleware>();
```

---

## Project: Todo API with Middlewares

This section walks through:

1. A simple Todo Web API with a database  
2. Adding built-in middlewares  
3. Adding a custom middleware  

---

### Prerequisites

- .NET SDK  
- SQL Server (local or Express)  
- Visual Studio or VS Code  

---

### Step 1: Create the project

1. **File → New → Project → ASP.NET Core Web API**  
2. Project name: **TodoApp**  
3. Remove default files:
   - Delete `Controllers/WeatherForecastController.cs`
   - Delete `WeatherForecast.cs`

---

### Step 2: Install packages

In Package Manager Console (or `dotnet add package`):

```powershell
Install-Package Microsoft.EntityFrameworkCore
Install-Package Microsoft.EntityFrameworkCore.SqlServer
Install-Package Microsoft.EntityFrameworkCore.Tools
Install-Package Swashbuckle.AspNetCore
```

---

### Step 3: Database setup

**1. Create the database (SQL Server):**

```sql
CREATE DATABASE Tododb;
```

**2. Create the model** – Add a `Models` folder and `Todo.cs`:

```csharp
public class Todo
{
    public int Id { get; set; }
    public required string Title { get; set; }
}
```

**3. Create the DbContext** – Add a `Data` folder and `TodoDbContext.cs`:

```csharp
using Microsoft.EntityFrameworkCore;

public class TodoDbContext : DbContext
{
    public TodoDbContext(DbContextOptions<TodoDbContext> options) : base(options) { }

    public DbSet<Todo> Todos { get; set; }
}
```

**4. Connection string** – In `appsettings.json`:

```json
"ConnectionStrings": {
  "DefaultConnection": "Server=localhost;Database=Tododb;Trusted_Connection=True;TrustServerCertificate=True;"
}
```

**5. Register DbContext** – In `Program.cs`:

```csharp
builder.Services.AddDbContext<TodoDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

**6. Run migrations:**

```powershell
Add-Migration InitialCreate
Update-Database
```

---

### Step 4: Todo controller

In `Controllers/TodoController.cs` (use **TodoDbContext** to match the DbContext name above):

```csharp
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class TodoController : ControllerBase
{
    private readonly TodoDbContext _context;

    public TodoController(TodoDbContext context)
    {
        _context = context;
    }

    [HttpGet]
    public ActionResult<IEnumerable<Todo>> GetAll()
    {
        return _context.Todos.ToList();
    }

    [HttpPost]
    public IActionResult Create([FromBody] string title)
    {
        var todo = new Todo { Title = title };
        _context.Todos.Add(todo);
        _context.SaveChanges();
        return Ok(todo);
    }

    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        var todo = _context.Todos.Find(id);
        if (todo == null)
            return NotFound();

        _context.Todos.Remove(todo);
        _context.SaveChanges();
        return NoContent();
    }
}
```

---

### Step 5: Add built-in middlewares (Program.cs)

Add these in **logical order** (exception handling first, then routing, then auth, etc.):

```csharp
// 1. Exception handling – catch errors and re-execute error route
app.UseExceptionHandler("/Error");
app.UseStatusCodePagesWithReExecute("/Error/{0}");

// 2. CORS – allow cross-origin requests (tighten in production)
app.UseCors(options =>
    options.AllowAnyOrigin()
           .AllowAnyMethod()
           .AllowAnyHeader());

// 3. Routing
app.UseRouting();

// 4. Authentication & Authorization (order is important)
app.UseAuthentication();
app.UseAuthorization();

// 5. Request logging (if using Serilog)
// app.UseSerilogRequestLogging();
```

**Notes:**

- **Exception handling**: Unhandled exceptions are sent to `/Error`; status code pages use `/Error/{0}` (e.g. 404).  
- **CORS**: `AllowAnyOrigin()` is for development; restrict origins in production.  
- **Serilog**: Add the Serilog packages and configure it, then uncomment `UseSerilogRequestLogging()` if you use it.

---

### Step 6: Custom middleware – request logging

Create `Middleware/RequestLoggingMiddleware.cs`:

```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var path = context.Request.Path;
        var method = context.Request.Method;
        _logger.LogInformation("Incoming request: {Method} {Path}", method, path);

        var stopwatch = System.Diagnostics.Stopwatch.StartNew();
        await _next(context);
        stopwatch.Stop();

        _logger.LogInformation("Response completed: {StatusCode} in {ElapsedMs}ms",
            context.Response.StatusCode, stopwatch.ElapsedMilliseconds);
    }
}
```

Register it early in the pipeline (e.g. after exception handling):

```csharp
app.UseMiddleware<RequestLoggingMiddleware>();
```

---

## Recommended middleware order (summary)

1. Exception handling  
2. HTTPS redirection (if needed)  
3. Static files (if needed)  
4. Routing  
5. CORS  
6. Authentication  
7. Authorization  
8. Custom middlewares (e.g. request logging)  
9. `app.MapControllers()` or endpoint mapping  

---

## Quick reference

| Task | Code |
|------|------|
| Use custom middleware | `app.UseMiddleware<YourMiddleware>();` |
| Use exception handler | `app.UseExceptionHandler("/Error");` |
| Use CORS | `app.UseCors(...);` |
| Use auth | `app.UseAuthentication(); app.UseAuthorization();` |
| Use routing | `app.UseRouting();` |

---

## License

Use and adapt as needed for your projects.
