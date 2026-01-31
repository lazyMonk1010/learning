# Singleton Pattern

## Introduction

Singleton Pattern is a design pattern that ensures a class has only **one instance** and provides a **global point of access** to it.

**Key Points:**
- Global point of access to the instance of the class
- Ensure that a class has only one instance throughout the application
- Used when we need only one instance of a class throughout the application

**Common Examples:**
- Database connection
- Logger
- Configuration
- Thread pool
- Cache

---

## Video Tutorial

Watch this video for a detailed implementation walkthrough:

[Singleton Pattern](https://youtu.be/7qUVUFE4_Sc?si=wSWYF-G5FIk-3hHt)

## Basic Implementation

In singleton pattern, you cannot create an instance manually. You can only get the instance from the Singleton class.

### Step-by-Step Implementation

```csharp
class Singleton {
    // Step 1: Private static variable to store the instance of the class.
    private static Singleton instance = null;

    // Step 2: Private constructor to prevent instantiation of the class.
    private Singleton() {
        // Constructor logic
    }   

    // Step 3: Public static method to get the instance of the class.
    public static Singleton GetInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

### Normal Class vs Singleton Pattern

**Normal Class Usage:**

Generally, what we do is we create a class and where we need to use the instance of the class, we instantiate the class and use the instance of the class.

For example:

```csharp
class Employee {
    public void Work() {
        // Employee logic
    }
}

var employee = new Employee();
employee.Work();
```

**Singleton Pattern Usage:**

But in singleton pattern, we don't instantiate the class. We only get the instance from the Singleton class.

```csharp
var singleton = Singleton.GetInstance();
singleton.DoSomething();
```

**Key Difference:**
- ❌ **Normal class:** `var obj = new ClassName();` - Can create multiple instances
- ✅ **Singleton:** `var obj = ClassName.GetInstance();` - Always returns the same single instance

---

## Thread Safety in Singleton Pattern

### The Problem

Now the above example is **not thread safe**. If multiple threads can access the same instance of the class simultaneously, it can cause issues.

**What can go wrong:**
- If one thread is creating the instance and another thread is trying to get the instance at the same time, it can cause issues
- Multiple threads might create multiple instances, breaking the singleton pattern

### The Solution

We can use the `lock` keyword to ensure that only one thread can access the instance of the class simultaneously.

```csharp
class Singleton {
    // Step 1: Private static variable to store the instance of the class.
    private static Singleton instance = null;
    private static readonly object lockObject = new object();

    // Step 2: Private constructor to prevent instantiation of the class.
    private Singleton() {
        // Constructor logic
    }   

    // Step 3: Public static method to get the instance of the class.
    public static Singleton GetInstance() {
        if (instance == null) {
            lock (lockObject) { // Double Check Locking
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

### How It Works

Basically, what we do is we lock the object and check if the instance is null. If it is null, we create the instance. If it is not null, we return the instance.

**Double-Check Locking Pattern:**
1. First check `if (instance == null)` - Quick check without locking (performance optimization)
2. `lock (lockObject)` - Only one thread can enter this block at a time
3. Second check `if (instance == null)` - Check again inside the lock (another thread might have created it while waiting)
4. Create instance if needed

This ensures that only one thread can access the instance of the class simultaneously. Now the above example is thread safe. If multiple threads can access the same instance of the class simultaneously, it will not cause issues.

---

## Lazy Initialization in Singleton Pattern

### What is Lazy Initialization?

Lazy Initialization is a technique of delaying the initialization of the instance of the class until the instance is needed. This is useful when the instance of the class is not needed immediately and we want to save the memory. This is also known as lazy loading.

### Implementation Using Lazy<T>

C# provides a built-in `Lazy<T>` class that makes lazy initialization easy and thread-safe:

```csharp
public sealed class Singleton {
    private static readonly Lazy<Singleton> instance =
        new Lazy<Singleton>(() => new Singleton());

    private Singleton() {
        // Constructor logic
    }

    public static Singleton GetInstance() {
        return instance.Value;
    }
}
```

### Benefits of Lazy<T>

- ✅ **Thread-safe by default** - No need for manual locking
- ✅ **Lazy creation** - Instance is created only when `instance.Value` is first accessed
- ✅ **Simpler code** - Less boilerplate than manual locking
- ✅ **Recommended approach** - Modern C# best practice

**Note:** The `sealed` keyword prevents inheritance, which is often desired for singleton classes.

---

## Real Life Examples of Singleton Pattern

### 1. Database Connection

Database connections are expensive to create. Using a singleton ensures you reuse the same connection throughout your application.

```csharp
class DatabaseConnection {
    private static DatabaseConnection instance = null;
    private static readonly object lockObject = new object();

    private DatabaseConnection() {
        // Constructor logic - Initialize database connection
    }

    public static DatabaseConnection GetInstance() {
        if (instance == null) {
            lock (lockObject) {
                if (instance == null) {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }

    public void Connect() {
        // Connection logic
    }
}
```

### 2. Logger

A logger should write all logs to the same file or output. Singleton ensures consistency.

```csharp
class Logger {
    private static Logger instance = null;
    private static readonly object lockObject = new object();

    private Logger() {
        // Constructor logic - Initialize logger
    }

    public static Logger GetInstance() {
        if (instance == null) {
            lock (lockObject) {
                if (instance == null) {
                    instance = new Logger();
                }
            }
        }
        return instance;
    }

    public void Log(string message) {
        // Logging logic
    }
}
```

### 3. Configuration

Application settings should be loaded once and accessed globally.

```csharp
class Configuration {
    private static Configuration instance = null;
    private static readonly object lockObject = new object();

    private Configuration() {
        // Constructor logic - Load configuration from file
    }

    public static Configuration GetInstance() {
        if (instance == null) {
            lock (lockObject) {
                if (instance == null) {
                    instance = new Configuration();
                }
            }
        }
        return instance;
    }

    public string GetSetting(string key) {
        // Get configuration setting
    }
}
```

### 4. Thread Pool

A thread pool should be managed as a single instance to efficiently handle concurrent tasks.

```csharp
class ThreadPool {
    private static ThreadPool instance = null;
    private static readonly object lockObject = new object();

    private ThreadPool() {
        // Constructor logic - Initialize thread pool
    }

    public static ThreadPool GetInstance() {
        if (instance == null) {
            lock (lockObject) {
                if (instance == null) {
                    instance = new ThreadPool();
                }
            }
        }
        return instance;
    }

    public void ExecuteTask(Action task) {
        // Execute task using thread pool
    }
}
```

### 5. Cache

A cache should be shared across the entire application to avoid duplicate data.

```csharp
class Cache {
    private static Cache instance = null;
    private static readonly object lockObject = new object();

    private Cache() {
        // Constructor logic - Initialize cache
    }

    public static Cache GetInstance() {
        if (instance == null) {
            lock (lockObject) {
                if (instance == null) {
                    instance = new Cache();
                }
            }
        }
        return instance;
    }

    public void Set(string key, object value) {
        // Store value in cache
    }

    public object Get(string key) {
        // Retrieve value from cache
    }
}
```

---

## Summary

### Key Points

- ✅ Singleton Pattern ensures only **one instance** of a class exists
- ✅ Provides a **global point of access** to that instance
- ✅ Use **private constructor** to prevent direct instantiation
- ✅ Use **GetInstance()** method to get the instance
- ✅ Use **locks** or **Lazy<T>** for thread safety
- ✅ Use **Lazy<T>** for modern, simpler implementation

### When to Use

- When you need exactly one instance throughout the application
- When the instance needs to be globally accessible
- When creating multiple instances would be wasteful or problematic

### Common Use Cases

- Database connections
- Logger services
- Configuration managers
- Thread pools
- Cache systems
