# C# Threading & Concurrency — Learning Guide

A structured reference for .NET threading, tasks, synchronization, and async patterns. Each section maps to a concrete exercise you can implement.

---

## Table of Contents

1. [Basic Threading](#1-basic-threading)
2. [Thread Synchronization](#2-thread-synchronization)
3. [Producer–Consumer Patterns](#3-producerconsumer-patterns)
4. [ThreadPool](#4-threadpool)
5. [Timers](#5-timers)
6. [Task-Based Asynchronous Programming](#6-task-based-asynchronous-programming)
7. [async/await and Thread Behavior](#7-asyncawait-and-thread-behavior)
8. [Parallel Loops](#8-parallel-loops)
9. [Concurrent Collections](#9-concurrent-collections)
10. [Cancellation](#10-cancellation)
11. [Thread-Safe Singleton](#11-thread-safe-singleton)
12. [Semaphores and Async Locks](#12-semaphores-and-async-locks)
13. [Thread Starvation](#13-thread-starvation)

---

## 1. Basic Threading

### 1.1 Create and start a thread that executes a method and prints the current thread ID

```csharp
void PrintThreadId()
{
    Console.WriteLine($"Thread ID: {Environment.CurrentManagedThreadId}");
}

var thread = new Thread(PrintThreadId);
thread.Start();
thread.Join();
```

### 1.2 Create a thread using `ParameterizedThreadStart` and pass an integer

```csharp
void PrintNumber(object? obj)
{
    if (obj is int n)
        Console.WriteLine($"Received: {n}, Thread ID: {Environment.CurrentManagedThreadId}");
}

var thread = new Thread(PrintNumber);
thread.Start(42);
thread.Join();
```

### 1.3 Create multiple threads and wait for all of them using `Join`

```csharp
var threads = new List<Thread>();
for (int i = 0; i < 5; i++)
{
    int id = i;
    var t = new Thread(() => Console.WriteLine($"Thread {id}: {Environment.CurrentManagedThreadId}"));
    threads.Add(t);
    t.Start();
}
foreach (var t in threads)
    t.Join();
Console.WriteLine("All threads finished.");
```

### 1.4 Demonstrate the difference between foreground and background threads

- **Foreground threads** keep the process alive until they finish.
- **Background threads** do not; when all foreground threads end, the process exits and background threads are stopped.

```csharp
var bg = new Thread(() => { Thread.Sleep(5000); Console.WriteLine("Background"); })
    { IsBackground = true };
var fg = new Thread(() => { Thread.Sleep(2000); Console.WriteLine("Foreground"); });

bg.Start();
fg.Start();
fg.Join();
// Process may exit before "Background" is printed, because bg is background.
```

### 1.5 Read and print the `ThreadState` of a running thread

```csharp
var thread = new Thread(() => Thread.Sleep(2000));
Console.WriteLine(thread.ThreadState); // Unstarted
thread.Start();
Console.WriteLine(thread.ThreadState); // Running (or WaitSleepJoin)
thread.Join();
Console.WriteLine(thread.ThreadState); // Stopped
```

---

## 2. Thread Synchronization

### 2.1 Create a shared counter accessed by multiple threads and show a race condition

```csharp
int counter = 0;
var threads = new List<Thread>();
for (int i = 0; i < 10; i++)
{
    var t = new Thread(() =>
    {
        for (int j = 0; j < 1000; j++)
            counter++; // Race: read-modify-write is not atomic
    });
    threads.Add(t);
    t.Start();
}
foreach (var t in threads) t.Join();
Console.WriteLine($"Counter (expected 10000): {counter}"); // Often less than 10000
```

### 2.2 Fix the race condition using the `lock` keyword

```csharp
int counter = 0;
object gate = new object();
for (int i = 0; i < 10; i++)
{
    var t = new Thread(() =>
    {
        for (int j = 0; j < 1000; j++)
        {
            lock (gate)
                counter++;
        }
    });
    t.Start();
    t.Join();
}
Console.WriteLine($"Counter: {counter}"); // 10000
```

### 2.3 Implement a scenario that causes a deadlock using two locks

```csharp
object lockA = new object(), lockB = new object();
var t1 = new Thread(() =>
{
    lock (lockA)
    {
        Thread.Sleep(100);
        lock (lockB) { }
    }
});
var t2 = new Thread(() =>
{
    lock (lockB)
    {
        Thread.Sleep(100);
        lock (lockA) { }
    }
});
t1.Start(); t2.Start();
t1.Join(); t2.Join(); // Deadlock: each holds one lock and waits for the other
```

### 2.4 Fix the deadlock by enforcing a consistent lock order

Always acquire locks in the same order (e.g. A then B) so no cycle can form.

```csharp
object lockA = new object(), lockB = new object();
var t1 = new Thread(() =>
{
    lock (lockA) { lock (lockB) { } }
});
var t2 = new Thread(() =>
{
    lock (lockA) { lock (lockB) { } } // Same order as t1
});
t1.Start(); t2.Start();
t1.Join(); t2.Join();
```

### 2.5 Increment a shared variable using `Interlocked.Increment`

```csharp
int counter = 0;
Parallel.For(0, 10000, _ => Interlocked.Increment(ref counter));
Console.WriteLine(counter); // 10000
```

### 2.6 Compare `lock` vs `Interlocked` by implementing both for a counter

- **Interlocked**: best for a single shared variable (increment, compare-exchange). No blocking, very fast.
- **lock**: for multiple operations that must be atomic together, or when you need to protect more than one field.

```csharp
// Interlocked — use for single counter
int c1 = 0;
Parallel.For(0, 100000, _ => Interlocked.Increment(ref c1));

// lock — use when you need multiple statements atomic
int c2 = 0;
object gate = new object();
Parallel.For(0, 100000, _ => { lock (gate) c2++; });
```

### 2.7 Demonstrate the use of the `volatile` keyword for a stop flag between threads

`volatile` ensures reads/writes are not cached in a register and are visible across threads (visibility, not atomicity).

```csharp
private static volatile bool _stop = false;

var worker = new Thread(() =>
{
    while (!_stop)
    {
        // do work
    }
});
worker.Start();
Thread.Sleep(100);
_stop = true;
worker.Join();
```

---

## 3. Producer–Consumer Patterns

### 3.1 Implement producer–consumer using `Monitor.Wait` and `Monitor.Pulse`

```csharp
object gate = new object();
Queue<int> queue = new Queue<int>();
const int Capacity = 5;

void Producer()
{
    for (int i = 0; i < 10; i++)
    {
        lock (gate)
        {
            while (queue.Count >= Capacity)
                Monitor.Wait(gate);
            queue.Enqueue(i);
            Monitor.Pulse(gate);
        }
    }
}

void Consumer()
{
    for (int i = 0; i < 10; i++)
    {
        lock (gate)
        {
            while (queue.Count == 0)
                Monitor.Wait(gate);
            int item = queue.Dequeue();
            Monitor.Pulse(gate);
        }
    }
}

var p = new Thread(Producer);
var c = new Thread(Consumer);
p.Start(); c.Start();
p.Join(); c.Join();
```

### 3.2 Implement producer–consumer using `BlockingCollection<T>`

See [Section 9.3](#93-implement-producerconsumer-using-blockingcollectiont).

---

## 4. ThreadPool

### 4.1 Queue multiple work items using `ThreadPool.QueueUserWorkItem`

```csharp
for (int i = 0; i < 10; i++)
{
    int id = i;
    ThreadPool.QueueUserWorkItem(_ =>
    {
        Console.WriteLine($"Work item {id} on thread {Environment.CurrentManagedThreadId}");
    });
}
Thread.Sleep(500); // Give pool threads time to run
```

### 4.2 Print thread IDs of ThreadPool threads to show reuse

Queue many items and print thread IDs; you’ll see the same IDs reused.

```csharp
var threadIds = new ConcurrentDictionary<int, int>();
for (int i = 0; i < 100; i++)
{
    ThreadPool.QueueUserWorkItem(_ =>
    {
        int tid = Environment.CurrentManagedThreadId;
        threadIds.AddOrUpdate(tid, 1, (_, count) => count + 1);
    });
}
Thread.Sleep(2000);
foreach (var kv in threadIds)
    Console.WriteLine($"Thread {kv.Key} executed {kv.Value} work items");
```

---

## 5. Timers

### 5.1 Create a `System.Threading.Timer` that runs a callback periodically

```csharp
var timer = new Timer(_ =>
{
    Console.WriteLine($"Tick at {DateTime.Now:HH:mm:ss} on thread {Environment.CurrentManagedThreadId}");
}, null, TimeSpan.Zero, TimeSpan.FromSeconds(1));
Thread.Sleep(5000);
timer.Dispose();
```

### 5.2 Stop a running timer after a fixed number of executions

```csharp
int count = 0;
Timer? timer = null;
timer = new Timer(_ =>
{
    if (Interlocked.Increment(ref count) >= 5)
    {
        timer?.Dispose();
        return;
    }
    Console.WriteLine($"Execution {count}");
}, null, TimeSpan.Zero, TimeSpan.FromSeconds(1));
Thread.Sleep(10000);
```

---

## 6. Task-Based Asynchronous Programming

### 6.1 Create and start a task using `Task.Run`

```csharp
var task = Task.Run(() =>
{
    Console.WriteLine($"Task on thread {Environment.CurrentManagedThreadId}");
});
task.Wait();
```

### 6.2 Create a task using `new Task()` and start it manually

```csharp
var task = new Task(() => Console.WriteLine("Started manually"));
task.Start();
task.Wait();
```

### 6.3 Return a value from a `Task<T>` and read the result

```csharp
var task = Task.Run(() =>
{
    Thread.Sleep(100);
    return 42;
});
Console.WriteLine(task.Result); // 42 (blocks until complete)
```

### 6.4 Throw an exception inside a task and handle it in the calling code

```csharp
var task = Task.Run(() => throw new InvalidOperationException("Task failed"));
try
{
    task.Wait();
}
catch (AggregateException ex)
{
    Console.WriteLine(ex.InnerException?.Message);
}
```

### 6.5 Handle multiple task exceptions using `AggregateException`

```csharp
var t1 = Task.Run(() => throw new Exception("One"));
var t2 = Task.Run(() => throw new Exception("Two"));
try
{
    Task.WaitAll(t1, t2);
}
catch (AggregateException ae)
{
    foreach (var e in ae.InnerExceptions)
        Console.WriteLine(e.Message);
}
```

### 6.6 Chain tasks using `ContinueWith`

```csharp
Task.Run(() => 1)
    .ContinueWith(t => t.Result + 1)
    .ContinueWith(t => Console.WriteLine(t.Result)) // 2
    .Wait();
```

### 6.7 Execute a continuation only when the previous task completes successfully

```csharp
var task = Task.Run(() => 42);
task.ContinueWith(t =>
{
    if (!t.IsFaulted)
        Console.WriteLine(t.Result);
}, TaskContinuationOptions.OnlyOnRanToCompletion);
task.Wait();
```

### 6.8 Run multiple tasks in parallel and wait for all using `Task.WhenAll`

```csharp
var tasks = Enumerable.Range(0, 5)
    .Select(i => Task.Run(() => { Thread.Sleep(100); return i; }))
    .ToArray();
int[] results = await Task.WhenAll(tasks);
Console.WriteLine(string.Join(", ", results));
```

### 6.9 Run multiple tasks and continue when any one completes using `Task.WhenAny`

```csharp
var tasks = new[]
{
    Task.Delay(3000).ContinueWith(_ => 1),
    Task.Delay(1000).ContinueWith(_ => 2),
    Task.Delay(2000).ContinueWith(_ => 3),
};
var first = await Task.WhenAny(tasks);
Console.WriteLine($"First result: {first.Result}");
```

---

## 7. async/await and Thread Behavior

### 7.1 Demonstrate thread behavior before and after `await`

Before `await`: runs on the calling synchronization context (e.g. UI thread). After `await`: by default continues on that same context unless you use `ConfigureAwait(false)`.

```csharp
async Task DemonstrateContext()
{
    Console.WriteLine($"Before await: {Environment.CurrentManagedThreadId}");
    await Task.Delay(100);
    Console.WriteLine($"After await (same context): {Environment.CurrentManagedThreadId}");
}
```

### 7.2 Compare a CPU-bound task using `Task.Run` with an IO-bound task using `Task.Delay`

- **CPU-bound**: offload to thread pool with `Task.Run(() => HeavyComputation())`.
- **IO-bound**: use async APIs and `await` (e.g. `await Task.Delay(100)` or `await httpClient.GetAsync(...)`), no thread blocked.

```csharp
// CPU-bound — use thread pool
var result = await Task.Run(() => Enumerable.Range(0, 1000000).Sum());

// IO-bound — no dedicated thread
await Task.Delay(1000);
```

### 7.3 Use `ConfigureAwait(false)` and observe context behavior

In library or non-UI code, `ConfigureAwait(false)` avoids capturing the synchronization context, so the continuation can run on any thread (often a thread-pool thread).

```csharp
async Task LibraryMethodAsync()
{
    Console.WriteLine(Environment.CurrentManagedThreadId);
    await Task.Delay(100).ConfigureAwait(false);
    Console.WriteLine(Environment.CurrentManagedThreadId); // May differ
}
```

---

## 8. Parallel Loops

### 8.1 Parallelize a loop using `Parallel.For`

```csharp
int sum = 0;
Parallel.For(0, 1000, i => Interlocked.Add(ref sum, i));
Console.WriteLine(sum);
```

### 8.2 Use `Parallel.ForEach` to process a collection concurrently

```csharp
var list = Enumerable.Range(0, 100).ToList();
Parallel.ForEach(list, item =>
{
    Console.WriteLine($"Processing {item} on thread {Environment.CurrentManagedThreadId}");
});
```

### 8.3 Modify a shared collection inside `Parallel.ForEach` and fix it with locking

```csharp
// Wrong: List<T> is not thread-safe
// var results = new List<int>();

var results = new List<int>();
object lockObj = new object();
Parallel.ForEach(Enumerable.Range(0, 100), item =>
{
    int value = item * item;
    lock (lockObj)
        results.Add(value);
});
```

---

## 9. Concurrent Collections

### 9.1 Use `ConcurrentDictionary` to safely add and update values from multiple threads

```csharp
var dict = new ConcurrentDictionary<int, int>();
Parallel.For(0, 100, i => dict.TryAdd(i, i * i));
Parallel.For(0, 100, i => dict.AddOrUpdate(i, 1, (k, v) => v + 1));
```

### 9.2 Use `ConcurrentQueue` for multi-threaded enqueue and dequeue

```csharp
var queue = new ConcurrentQueue<int>();
Parallel.For(0, 100, queue.Enqueue);
int count = 0;
while (queue.TryDequeue(out int item))
    count++;
Console.WriteLine(count);
```

### 9.3 Implement producer–consumer using `BlockingCollection<T>`

```csharp
using var collection = new BlockingCollection<int>(boundedCapacity: 10);

var producer = Task.Run(() =>
{
    for (int i = 0; i < 20; i++)
        collection.Add(i);
    collection.CompleteAdding();
});

var consumer = Task.Run(() =>
{
    foreach (int item in collection.GetConsumingEnumerable())
        Console.WriteLine(item);
});

Task.WaitAll(producer, consumer);
```

---

## 10. Cancellation

### 10.1 Cancel a running task using `CancellationToken`

```csharp
var cts = new CancellationTokenSource();
var task = Task.Run(() =>
{
    while (!cts.Token.IsCancellationRequested)
    {
        Thread.Sleep(100);
    }
}, cts.Token);
cts.CancelAfter(500);
try { task.Wait(); } catch (AggregateException) { }
```

### 10.2 Gracefully stop a long-running loop using cancellation tokens

```csharp
var cts = new CancellationTokenSource();
var task = Task.Run(() =>
{
    for (int i = 0; i < 1000 && !cts.Token.IsCancellationRequested; i++)
    {
        cts.Token.ThrowIfCancellationRequested();
        Thread.Sleep(10);
    }
}, cts.Token);
cts.Cancel();
```

### 10.3 Implement a task timeout using `Task.WhenAny`

```csharp
var work = Task.Run(() => { Thread.Sleep(5000); return 42; });
var timeout = Task.Delay(1000).ContinueWith(_ => throw new TimeoutException());
var result = await Task.WhenAny(work, timeout);
if (result == work)
    Console.WriteLine(work.Result);
else
    Console.WriteLine("Timed out");
```

---

## 11. Thread-Safe Singleton

### 11.1 Implement a thread-safe singleton using `lock`

```csharp
public sealed class Singleton
{
    private static Singleton? _instance;
    private static readonly object _lock = new object();

    public static Singleton Instance
    {
        get
        {
            lock (_lock)
            {
                return _instance ??= new Singleton();
            }
        }
    }

    private Singleton() { }
}
```

### 11.2 Implement a thread-safe singleton using `Lazy<T>`

```csharp
public sealed class Singleton
{
    private static readonly Lazy<Singleton> _lazy = new Lazy<Singleton>(() => new Singleton());
    public static Singleton Instance => _lazy.Value;
    private Singleton() { }
}
```

---

## 12. Semaphores and Async Locks

### 12.1 Use `SemaphoreSlim` to limit concurrent access to a resource

```csharp
var semaphore = new SemaphoreSlim(3, 3); // Max 3 concurrent
Parallel.For(0, 10, async i =>
{
    await semaphore.WaitAsync();
    try
    {
        Console.WriteLine($"{i} acquired");
        await Task.Delay(1000);
    }
    finally
    {
        semaphore.Release();
    }
});
```

### 12.2 Implement an async-compatible lock using `SemaphoreSlim`

```csharp
public class AsyncLock
{
    private readonly SemaphoreSlim _sem = new SemaphoreSlim(1, 1);

    public async Task<IDisposable> AcquireAsync()
    {
        await _sem.WaitAsync();
        return new Releaser(_sem);
    }

    private class Releaser : IDisposable
    {
        private readonly SemaphoreSlim _sem;
        public Releaser(SemaphoreSlim sem) => _sem = sem;
        public void Dispose() => _sem.Release();
    }
}
```

### 12.3 Compare `lock` and `SemaphoreSlim` in synchronous vs async code

- **lock**: synchronous only; cannot use with `await` inside the locked region.
- **SemaphoreSlim**: supports both `Wait()` (sync) and `WaitAsync()` (async); use in async code when you need mutual exclusion across awaits.

---

## 13. Thread Starvation

### 13.1 Demonstrate thread starvation using excessive locking

If one thread holds a lock for a long time or very frequently, other threads waiting on that lock get little CPU time (starvation).

```csharp
object gate = new object();
var greedy = new Thread(() =>
{
    while (true)
    {
        lock (gate)
        {
            Thread.Sleep(10); // Holds lock often
        }
    }
});
var starved = new Thread(() =>
{
    for (int i = 0; i < 5; i++)
    {
        lock (gate) { Console.WriteLine("Starved thread got lock"); }
        Thread.Sleep(100);
    }
});
greedy.Start();
starved.Start();
starved.Join();
greedy.Join(0); // Don't wait forever
```

Mitigations: shorten critical sections, use fair locking where needed, avoid nested locks, consider reader-writer locks or other structures.

---

## Quick Reference

| Need | Use |
|------|-----|
| Simple background work | `Thread` or `Task.Run` |
| Pass data to thread | `ParameterizedThreadStart` or closure |
| Atomic single variable | `Interlocked` |
| Multiple statements atomic | `lock` |
| Async mutual exclusion | `SemaphoreSlim(1,1)` or async lock |
| Bounded producer–consumer | `BlockingCollection<T>` or `Monitor.Wait/Pulse` |
| Thread-safe collections | `ConcurrentDictionary`, `ConcurrentQueue`, etc. |
| Cancel work | `CancellationToken` |
| Lazy thread-safe singleton | `Lazy<T>` |
| Limit concurrency | `SemaphoreSlim` |

---

*Use this guide as a checklist: implement each subsection in a small console app to solidify your understanding.*
