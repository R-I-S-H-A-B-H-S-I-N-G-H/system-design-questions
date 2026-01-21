# Core Java Interview Questions & Answers (In-Depth Guide)

This document provides comprehensive, deep-dive answers to Core Java interview questions. It covers internal implementations, best practices, code examples, and tricky interview scenarios.

## Table of Contents
1.  [Java Basics & OOPs](#java-basics)
2.  [Java Collections Framework (Internals)](#collections)
3.  [Multithreading & Concurrency (Advanced)](#multithreading)
4.  [Java 8+ Features (Streams, Lambdas)](#java-8)
5.  [JVM Architecture & Garbage Collection](#jvm)

---

## <a name="java-basics"></a>1. Java Basics & OOPs

### Q1: What is the difference between `==` and `equals()`? Explain the contract.
**Deep Dive:**
*   **`==` Operator:**
    *   This is a reference comparison operator. It checks if two variables point to the *exact same memory location* (heap address).
    *   For primitives (`int`, `char`, etc.), it compares the actual values.
    *   For objects, it returns `true` only if both references point to the same object instance.
*   **`equals()` Method:**
    *   Defined in `java.lang.Object`. The default implementation in `Object` uses `==` (reference equality).
    *   Classes are expected to override `equals()` to provide "logical equality" (e.g., two Strings are equal if their character sequences are identical, even if they are different objects in memory).

**The `equals()` and `hashCode()` Contract:**
If you override `equals()`, you **must** override `hashCode()`.
1.  **Reflexive:** `x.equals(x)` must be true.
2.  **Symmetric:** If `x.equals(y)` is true, `y.equals(x)` must be true.
3.  **Transitive:** If `x.equals(y)` and `y.equals(z)`, then `x.equals(z)`.
4.  **Consistent:** Multiple invocations return the same result if objects haven't changed.
5.  **Null Check:** `x.equals(null)` must return false.
6.  **HashCode Rule:** If `x.equals(y)` is true, then `x.hashCode()` **must** be equal to `y.hashCode()`. (Note: The reverse is not required; collision is possible).

**Interview Tip:**
A common trap is defining `equals` but forgetting `hashCode`. If you do this, storing the object in a `HashMap` or `HashSet` will fail because the collection uses the hash code to find the bucket first.

---

### Q2: Why is String immutable in Java?
**Detailed Explanation:**
String immutability means once a `String` object is created, its state cannot be changed. This is achieved by making the `String` class `final` and storing the character array (or byte array in Java 9+) as `private final`.

**Key Reasons for Immutability:**
1.  **String Constant Pool (SCP):** Java saves memory by reusing String literals. If Strings were mutable, changing one reference would affect all other references pointing to the same literal in the pool.
2.  **Security:** Strings are widely used for parameters like database usernames, passwords, network hostnames, and file paths. If mutable, a malicious thread could change the path after security checks but before the OS call (Time-of-Check to Time-of-Use vulnerability).
3.  **Thread Safety:** Immutable objects are inherently thread-safe. They can be shared across threads without synchronization, improving performance.
4.  **Caching HashCode:** Since the content doesn't change, `String` caches its hash code after the first calculation. This makes it extremely fast as a `HashMap` key.

---

### Q3: What is Java Reflection API? How does it break encapsulation?
**Deep Dive:**
Reflection is a powerful API (`java.lang.reflect`) that allows a Java program to inspect and manipulate the internal properties of classes, interfaces, fields, and methods at runtime.

**Capabilities:**
*   Instantiate objects (`Class.forName("...").newInstance()`).
*   Invoke methods dynamically.
*   Access private fields and methods.

**Breaking Encapsulation (Code Example):**
```java
public class Secret {
    private String password = "TopSecret";
}

// In another class:
Secret secret = new Secret();
Field field = Secret.class.getDeclaredField("password");
field.setAccessible(true); // <--- This breaks encapsulation!
String value = (String) field.get(secret);
System.out.println(value); // Prints "TopSecret"
```
**Why allow this?** Frameworks like **Spring** (Dependency Injection), **Hibernate** (ORM mapping), and testing libraries (**JUnit/Mockito**) rely heavily on reflection to inject dependencies and inspect annotations, even on private fields.

**Downsides:**
1.  **Performance Overhead:** Reflection involves dynamic type resolving, which is slower than direct calls (though JVM optimizations have reduced this gap).
2.  **Security Risks:** Can expose internals.
3.  **Compile-time Safety:** Errors (like method not found) happen at runtime, not compile time.

---

## <a name="collections"></a>2. Java Collections Framework (Internals)

### Q4: How does `HashMap` work internally? (Most Popular Question)
**In-Depth Architecture:**
`HashMap` uses an array of "Nodes" (or buckets) and linked lists (or Red-Black Trees) to store data.
`Map<K, V> map = new HashMap<>();`

**1. Hashing:**
*   When you call `put(key, value)`, `HashMap` calls `key.hashCode()`.
*   It applies a supplementary hash function (XOR shifting) to ensure bits are spread evenly.
*   **Index Calculation:** `index = (n - 1) & hash`. This is a bitwise AND operation (efficient modulo) to find the bucket index.

**2. Collision Handling:**
*   If two keys end up at the same index, a **Collision** occurs.
*   **Before Java 8:** It used a LinkedList. The new node was added to the list. Searching was `O(n)`.
*   **Java 8+ Improvement:** If the bucket contains more than **8 nodes** (TREEIFY_THRESHOLD) and the array size is at least 64, the LinkedList converts into a **Red-Black Tree**.
*   **Performance:** This improves worst-case lookup from `O(n)` to `O(log n)`.

**3. `get(key)` Workflow:**
1.  Calculate hash of key.
2.  Go to the specific bucket index.
3.  Traverse the List/Tree: Compare `hash` and call `equals()` on the key.
4.  Return value if match found.

**4. Resizing:**
*   When the number of entries exceeds `Capacity * LoadFactor` (default 16 * 0.75 = 12), the map resizes.
*   It doubles the array size (16 -> 32).
*   **Rehashing:** All existing entries are recalculated and moved to new buckets. This is an expensive operation.

---

### Q5: Difference between `HashMap` and `ConcurrentHashMap`?
**Deep Dive:**

1.  **`HashMap`:**
    *   **Locking:** None. Not thread-safe.
    *   **Nulls:** Allows 1 null key and multiple null values.
    *   **Fail-Fast:** Iterators throw `ConcurrentModificationException` if modified during iteration.

2.  **`ConcurrentHashMap` (CHM):**
    *   **Java 7 Architecture:** Used **Segment Locking**. The map was divided into 16 segments (default concurrency level). Only the segment being written to was locked.
    *   **Java 8+ Architecture (Major Change):**
        *   Removed Segments.
        *   Uses **CAS (Compare-And-Swap)** for optimistic updates (inserting into empty bucket).
        *   Uses **`synchronized`** on the specific Node (bucket head) for collisions/updates.
        *   This provides extremely fine-grained locking. Two threads can write to different buckets simultaneously without blocking each other.
    *   **Nulls:** Neither keys nor values can be null (throws NPE).
    *   **Iterators:** Weakly Consistent (reflects state at some point, won't throw exception).

**Code Example (Atomic Update):**
```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
// Thread-safe increment without explicit synchronization
map.compute("key", (k, v) -> (v == null) ? 1 : v + 1);
```

---

### Q6: How does `ArrayList` vs `LinkedList` work? When to use which?
**Internal Implementation:**
*   **`ArrayList`:**
    *   Backed by a **dynamic array**.
    *   **Access:** `O(1)` (Random access via index).
    *   **Insert/Delete:** `O(n)` (Needs to shift elements).
    *   **Resizing:** When full, grows by **50%** (new capacity = `old * 1.5`). Copies old array to new array.
*   **`LinkedList`:**
    *   Backed by a **Doubly Linked List**. Each node stores data, `prev`, and `next` pointers.
    *   **Access:** `O(n)` (Must traverse from head/tail).
    *   **Insert/Delete:** `O(1)` (Just update pointers), but finding the position still takes `O(n)` unless you have an iterator.

**Memory Overhead:**
`LinkedList` consumes more memory because every node requires extra space for `prev` and `next` references (typically 16-24 bytes extra per element). `ArrayList` is cache-friendly (contiguous memory).

**Verdict:** Use `ArrayList` for 99% of cases. Use `LinkedList` *only* if you frequently insert/delete at the *beginning* or *middle* and rarely access elements by index.

---

### Q7: What is `HashSet` internally?
**Answer:**
Surprisingly, `HashSet` is just a wrapper around `HashMap`.
*   When you call `set.add(value)`, it essentially calls `map.put(value, PRESENT)`.
*   `PRESENT` is a static final dummy Object used as the value for all keys.
*   This explains why `HashSet` allows one null (HashMap allows one null key) and ensures uniqueness (HashMap keys are unique).

---

## <a name="multithreading"></a>3. Multithreading & Concurrency (Advanced)

### Q8: What is the `volatile` keyword? How is it different from `synchronized`?
**Deep Dive:**
`volatile` is a lightweight synchronization mechanism that addresses the **Visibility Problem**.

**The Visibility Problem:**
In multi-core processors, each thread may run on a different core with its own CPU cache (L1/L2). If Thread A updates a variable, it might write to its local cache but not flush it to main RAM immediately. Thread B might read the stale value from its own cache.

**How `volatile` works:**
1.  **Visibility Guarantee:** Declaring a variable `volatile` guarantees that any write to it is immediately flushed to main memory, and any read bypasses the cache and goes directly to main memory.
2.  **Happens-Before Relationship:** It establishes a memory barrier. Writes to a volatile variable *happen-before* any subsequent reads of that variable.
3.  **Prevention of Reordering:** It prevents the compiler/CPU from reordering instructions involving that variable.

**Difference from `synchronized`:**
*   **Atomicity:** `volatile` does **NOT** guarantee atomicity.
    *   Example: `count++` is three steps (Read -> Modify -> Write). Even if `count` is volatile, two threads can overwrite each other's increment. `synchronized` guarantees atomicity.
*   **Blocking:** `volatile` is non-blocking. `synchronized` blocks threads.

**Use Case:** Boolean flags (e.g., `private volatile boolean running = true;`) used to stop threads.

---

### Q9: Explain `ThreadPoolExecutor` internal parameters?
**Architecture:**
When you use `Executors.newFixedThreadPool(10)`, you are actually creating an instance of `ThreadPoolExecutor`.
Constructor:
```java
public ThreadPoolExecutor(
    int corePoolSize,
    int maximumPoolSize,
    long keepAliveTime,
    TimeUnit unit,
    BlockingQueue<Runnable> workQueue,
    ThreadFactory threadFactory,
    RejectedExecutionHandler handler
)
```

**Workflow logic:**
1.  **New Task Arrives:**
2.  If `runningThreads < corePoolSize`: Create new thread and run task.
3.  Else: Put task into `workQueue`.
4.  If `workQueue` is FULL and `runningThreads < maximumPoolSize`: Create new "emergency" thread.
5.  If `workQueue` is FULL and `runningThreads == maximumPoolSize`: Reject task (throw `RejectedExecutionException`).

**Common Interview Scenario:**
"I created a pool with core=5, max=10, queue=LinkedBlockingQueue (unbounded). How many threads will be created under heavy load?"
**Answer:** Only **5**. Since the queue is unbounded, it never gets full. The `maximumPoolSize` of 10 is effectively ignored because step 4 is never triggered.

---

### Q10: What is `ThreadLocal`? Why is it a source of memory leaks?
**Concept:**
`ThreadLocal` allows you to store data that is accessible *only* by a specific thread. It provides thread safety by isolation rather than synchronization.
*   Common use: Storing User Context in a web request, `SimpleDateFormat` (which is not thread-safe).

**Internals:**
Every `Thread` object has a field `ThreadLocalMap`. This map stores the data.
Key = `WeakReference<ThreadLocal>`, Value = `YourObject`.

**The Memory Leak Issue:**
*   In app servers (Tomcat), threads are pooled and reused.
*   If you set a `ThreadLocal` value but forget to call `.remove()`, the value remains attached to the thread even after the request finishes.
*   Since the thread goes back to the pool (it doesn't die), the object stays in memory, potentially holding references to ClassLoaders, leading to `OutOfMemoryError: PermGen/Metaspace`.
*   **Best Practice:** Always use `try-finally` block to call `threadLocal.remove()`.

---

### Q11: Explain `CompletableFuture` (Java 8) vs `Future`?
**Limitations of `Future` (Java 5):**
1.  **Blocking:** calling `.get()` blocks the main thread until computation is done.
2.  **No Chaining:** You cannot say "Do Step A, then take result and do Step B".
3.  **No Exception Handling:** Hard to manage exceptions inside the async task.

**Advantages of `CompletableFuture`:**
It implements `CompletionStage` allowing functional composition.
```java
CompletableFuture.supplyAsync(() -> fetchOrder(orderId)) // Run in background
    .thenApply(order -> enrichOrder(order))              // Transform result
    .thenAccept(order -> sendEmail(order))               // Consume result
    .exceptionally(ex -> { log.error(ex); return null; }); // Handle error
```
It is non-blocking (callback style) and supports combining multiple futures (`allOf`, `anyOf`).

---

## <a name="java-8"></a>4. Java 8+ Features

### Q12: What is a Functional Interface? What is `@FunctionalInterface`?
**Definition:**
An interface with **exactly one abstract method**.
*   It can have any number of `default` or `static` methods.
*   It can override methods from `java.lang.Object` (like `toString`).

**Purpose:**
They enable Lambda Expressions. A lambda is essentially an inline implementation of a functional interface.
Examples: `Runnable` (run), `Callable` (call), `Comparator` (compare), `Predicate` (test), `Function` (apply).

**`@FunctionalInterface` Annotation:**
It is optional but recommended. It forces the compiler to check that the interface strictly adheres to the rule (only one abstract method). If you add a second abstract method, compilation fails.

### Q13: How does Stream API work internally? (Lazy Evaluation)
**Concept:**
Streams are pipelines of operations.
`source -> intermediate ops (filter, map) -> terminal op (collect)`

**Lazy Evaluation:**
*   Intermediate operations are **lazy**. When you call `.filter()`, it doesn't process data immediately. It just creates a new Stream description.
*   Processing only starts when a **Terminal Operation** (like `collect`, `count`, `forEach`) is invoked.
*   **Loop Fusion:** The JVM optimizes the pipeline. It doesn't iterate the list 3 times for `filter`, `map`, and `sorted`. It fuses them into a single pass where possible.

**Parallel Streams:**
`list.parallelStream()` splits the data into chunks (using Spliterators) and processes them on the common `ForkJoinPool`. Use with caution: it can slow down simple tasks due to thread management overhead or cause issues if tasks are blocking (DB calls).

---

## <a name="jvm"></a>5. JVM & Garbage Collection

### Q14: Explain the JVM Memory Model (Stack vs Heap).
**1. Stack Memory:**
*   **Scope:** Per-Thread. Each thread has its own stack.
*   **Contents:** Stores primitive local variables (`int x = 5`) and **references** to objects (`String s = ...`).
*   **Structure:** LIFO (Last In First Out). Managed via Stack Frames (one frame per method call).
*   **Error:** `StackOverflowError` (e.g., infinite recursion).

**2. Heap Memory:**
*   **Scope:** Shared across the application.
*   **Contents:** Stores all **Objects** (`new Employee()`). Even if an object is created locally in a method, the object lives in Heap; only the reference is in Stack.
*   **Management:** Managed by Garbage Collector.
*   **Error:** `OutOfMemoryError: Java heap space`.

**3. Metaspace (Java 8+): Replaces PermGen.**
*   Stores Class metadata (static methods, static variables, constant pool).
*   Uses native OS memory (not Heap). Grows automatically.

---

### Q15: How does Garbage Collection (GC) work? What is "Stop-the-World"?
**Generational Hypothesis:** "Most objects die young."
Based on this, Heap is divided into:
1.  **Young Generation (Eden + Survivor S0/S1):** New objects go here. Minor GC runs here frequently and is fast.
2.  **Old Generation (Tenured):** Objects that survive multiple GCs are moved here. Major/Full GC runs here.

**The Process (Mark and Sweep):**
1.  **Mark:** GC identifies "Live" objects by traversing the graph starting from **GC Roots** (Active threads, Static variables).
2.  **Sweep:** Deletes unreachable objects.
3.  **Compact:** Defragments memory (optional, depends on collector).

**Stop-the-World (STW):**
During GC (especially Full GC), the JVM **pauses all application threads**. No code runs.
*   If STW is too long (e.g., 5 seconds), the app becomes unresponsive.
*   **Goal of Tuning:** Minimize STW pauses.

**GC Algorithms:**
*   **G1 GC (Default in Java 9+):** Divides heap into small regions. Cleans regions with most garbage first. Predictable pause times.
*   **ZGC / Shenandoah:** Low-latency collectors (sub-millisecond pauses) for huge heaps.

---

### Q16: What is a ClassLoader? Explain Delegation Model.
**Role:** Loading `.class` files from disk/network into JVM memory.

**Hierarchy:**
1.  **Bootstrap ClassLoader:** Loads core Java classes (`rt.jar`, `java.lang.*`). Written in C++.
2.  **Platform/Extension ClassLoader:** Loads extensions (`lib/ext`).
3.  **App/System ClassLoader:** Loads classes from your Classpath (`-cp`).

**Delegation Principle:**
When a class is needed:
1.  App ClassLoader asks Parent (Platform).
2.  Platform asks Parent (Bootstrap).
3.  If Bootstrap finds it, good. If not, it delegates back down.
4.  Platform tries. If not found, App ClassLoader tries.
5.  If none find it -> `ClassNotFoundException`.

**Why?** Security. It prevents you from replacing core classes (e.g., writing your own `java.lang.String`) because Bootstrap will always load the trusted JDK version first.


---

### Q17: What is the difference between `Checked` and `Unchecked` Exceptions?
**Deep Dive:**
*   **Checked Exceptions:**
    *   Inherit from `Exception` (but not `RuntimeException`).
    *   **Enforced at Compile Time:** You *must* handle them (try-catch) or declare them (`throws`).
    *   **Use Case:** Recoverable conditions (e.g., `FileNotFoundException`, `SQLException`). The caller is expected to recover (e.g., ask user for new file path).
*   **Unchecked Exceptions:**
    *   Inherit from `RuntimeException`.
    *   **Runtime:** Compiler does not check them.
    *   **Use Case:** Programming errors (e.g., `NullPointerException`, `IllegalArgumentException`). The application usually cannot recover, and these indicate bugs.

**Best Practice:** Use Checked exceptions for business logic failures the client can handle. Use Unchecked for everything else. Spring Framework prefers Unchecked exceptions (e.g., `DataAccessException`) to avoid cluttering code with try-catch blocks.

---

### Q18: Explain the `try-with-resources` statement.
**Concept:**
Introduced in Java 7 to manage resources (files, sockets, DB connections) automatically.
It ensures that each resource is closed at the end of the statement, eliminating the need for a `finally` block with nested null checks.

**Internal Mechanism:**
Any class implementing `AutoCloseable` or `Closeable` can be used.
```java
try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    return br.readLine();
} catch (IOException e) {
    // handle
}
// br.close() is called automatically here, even if exception occurs.
```
**Suppressed Exceptions:** If an exception is thrown in the `try` block AND in the `close()` method, the `close()` exception is "suppressed" and added to the primary exception. You can retrieve it via `e.getSuppressed()`.

---

### Q19: What is the difference between `Comparator` and `Comparable`?
**Deep Dive:**
*   **`Comparable<T>` (Natural Ordering):**
    *   Implemented by the class itself (e.g., `String`, `Integer`, `Date`).
    *   Method: `public int compareTo(T o)`.
    *   Modifies the original class. Only one logic possible.
    *   Usage: `Collections.sort(list)`.
*   **`Comparator<T>` (Custom Ordering):**
    *   Implemented by a separate class or lambda.
    *   Method: `public int compare(T o1, T o2)`.
    *   Allows multiple sorting strategies (Sort by Name, Sort by Age).
    *   Usage: `Collections.sort(list, new AgeComparator())` or `list.sort(Comparator.comparing(User::getAge))`.

---

### Q20: What is Serialization? What is `serialVersionUID`?
**Concept:**
Converting an object's state into a byte stream (to save to file or send over network). The class must implement `java.io.Serializable` (marker interface).

**`serialVersionUID`:**
A unique identifier version ID for the class.
*   **Purpose:** Verification during Deserialization. The sender and receiver must have the class with the *same* `serialVersionUID`.
*   **Mismatch:** If you modify the class (add field) and don't update the ID (or let Java generate it), deserialization fails with `InvalidClassException`.
*   **Best Practice:** Always declare it explicitly (`private static final long serialVersionUID = 1L;`) to ensure compatibility across different compiler implementations.

**`transient` Keyword:** Fields marked `transient` are skipped during serialization (e.g., passwords).

---

### Q21: Difference between `String`, `StringBuilder`, and `StringBuffer`?
**Internals:**
1.  **`String`:** Immutable. Stored in String Pool. Modification creates new objects. Thread-safe.
2.  **`StringBuffer`:** Mutable. Synchronized (Thread-safe). Slow due to lock overhead. Legacy.
3.  **`StringBuilder`:** Mutable. Not Synchronized. Fast. Introduced in Java 5.

**Interview Scenario:** "I need to concatenate strings in a loop."
**Answer:** Use `StringBuilder`. Using `String` + operator in a loop creates N objects (O(N^2) complexity due to copying). `StringBuilder` resizes its internal char array (doubling capacity) only when needed (O(N)).

---

### Q22: What is the Java Memory Model (JMM) "Happens-Before" relationship?
**Deep Dive:**
The JMM defines how threads interact through memory. "Happens-Before" is a guarantee that memory writes by one specific statement are visible to another specific statement.

**Rules:**
1.  **Program Order Rule:** Each action in a thread happens-before every later action in that thread.
2.  **Monitor Lock Rule:** An unlock on a monitor happens-before every subsequent lock on that monitor.
3.  **Volatile Variable Rule:** A write to a `volatile` field happens-before every subsequent read of that field.
4.  **Thread Start Rule:** `thread.start()` happens-before any action in the started thread.
5.  **Thread Join Rule:** All actions in a thread happen-before any other thread successfully returns from a `join()` on that thread.

Without these rules, the JVM/CPU is free to reorder instructions for performance, leading to race conditions.

---

### Q23: Explain `ForkJoinPool` and Work Stealing.
**Architecture:**
Designed for recursive divide-and-conquer tasks (e.g., Parallel Streams).
*   **Deque:** Each worker thread has its own double-ended queue (Deque) of tasks.
*   **Push/Pop:** A thread pushes new subtasks to the *head* of its own deque and pops from the *head* (LIFO).
*   **Work Stealing:** If Thread A runs out of tasks, it looks at Thread B's deque and "steals" a task from the *tail* (FIFO).
*   **Benefit:** Reduces contention. The owner works at the head, thieves work at the tail.

**Classes:**
*   `RecursiveTask<V>`: Returns a result.
*   `RecursiveAction`: No result (void).

---

### Q24: What are the different types of References in Java?
**1. Strong Reference:** Standard assignment (`Object o = new Object()`). Never collected by GC until nullified.
**2. Soft Reference:** (`SoftReference<T>`). Collected only if JVM is running low on memory. Good for Caching.
**3. Weak Reference:** (`WeakReference<T>`). Collected eagerly on the next GC cycle. Used in `WeakHashMap`.
**4. Phantom Reference:** (`PhantomReference<T>`). Used to schedule post-mortem cleanup actions. Object is already finalized but memory not reclaimed.

---

### Q25: How does `Collections.synchronizedMap` differ from `ConcurrentHashMap`?
**Internals:**
*   **`synchronizedMap`:** Wraps a standard map. All methods (`get`, `put`, `size`) are synchronized on a single object lock (`this`).
    *   **Concurrency:** Very poor. Only one thread can access the map at a time.
*   **`ConcurrentHashMap`:**
    *   **Java 8:** Uses CAS and synchronized blocks on individual bucket heads.
    *   **Concurrency:** High. Multiple threads can read/write simultaneously.
    *   **Reads:** Non-blocking (no locks).

---

### Q26: What is the difference between `final`, `finally`, and `finalize`?
**Quick Recap:**
1.  **`final`:** Modifier.
    *   Variable: Constant.
    *   Method: Cannot override.
    *   Class: Cannot inherit (e.g., `String`).
2.  **`finally`:** Block. Always executes after try-catch (unless JVM exits `System.exit(0)`). Used for cleanup.
3.  **`finalize()`:** Method.
    *   **Deprecated (Java 9).**
    *   Called by GC before reclaiming memory.
    *   **Issues:** Unpredictable, performance drag, can resurrect objects. Use `Cleaner` or `PhantomReference` instead.

---

### Q27: What is Classloading? What are the delegation models?
*(Covered in previous batch under Q16, expanding here)*
**Class Loading Phases:**
1.  **Loading:** Read binary data -> Method Area.
2.  **Linking:**
    *   **Verify:** Bytecode verification.
    *   **Prepare:** Allocate static variables (default values).
    *   **Resolve:** Symbolic refs -> Direct refs.
3.  **Initialization:** Run `static { ... }` blocks.

**Context ClassLoader:**
Used by frameworks (Tomcat, Spring) to break the parent-delegation model.
`Thread.currentThread().getContextClassLoader()`. Allows core code to load classes from the webapp level.

---

### Q28: How do you identify and fix a Deadlock?
**Identification:**
1.  **Thread Dump:** Run `jstack <pid>`. Look for "Found one Java-level deadlock".
2.  **VisualVM:** Auto-detects deadlocks.

**Fixing:**
1.  **Lock Ordering:** Ensure all threads acquire locks in the exact same order.
    *   Bad: A locks (1,2), B locks (2,1).
    *   Good: Both lock (1) then (2).
2.  **Lock Timeout:** Use `ReentrantLock.tryLock(timeout)`. If lock not acquired, back off and retry.

---

### Q29: What is `CompletableFuture` chaining?
**Scenario:**
Fetch User -> Then Fetch Orders -> Then Send Email.
```java
CompletableFuture.supplyAsync(() -> userService.getUser(id))
    .thenCompose(user -> orderService.getOrders(user)) // returns another Future
    .thenAccept(orders -> emailService.send(orders));  // consumes result
```
*   `thenApply`: Map `T -> R`.
*   `thenCompose`: FlatMap `T -> Future<R>`.
*   `thenAccept`: Consumer `T -> Void`.

---

### Q30: What is Type Erasure in Generics?
**Concept:**
Java Generics are a compile-time feature. The JVM does not know about generic types.
`List<String>` becomes `List` (raw type) at runtime.
*   The compiler inserts casts: `String s = (String) list.get(0)`.
*   **Implication:** You cannot do `new T()`, `instanceof T`, or create arrays of T.
*   **Bridge Methods:** Generated by compiler to preserve polymorphism in extended generic classes.


### Q31: What is a `Record` in Java 14+? How does it differ from a Class?
**Concept:**
Records are immutable data carriers. They drastically reduce boilerplate for POJOs (Plain Old Java Objects) that just hold data.
```java
public record User(String name, int age) {}
```
**Internals:**
1.  **Immutable:** All fields are `private final`.
2.  **Auto-Generated:**
    *   Constructor (Canonical constructor).
    *   Getters (named `name()`, `age()` - no `get` prefix).
    *   `equals()`, `hashCode()`, `toString()`.
3.  **Inheritance:** Records cannot extend other classes (they implicitly extend `java.lang.Record`). They are `final` (cannot be extended).

**Use Case:** DTOs (Data Transfer Objects), Map keys, Stream processing intermediate results.

---

### Q32: What is Sealed Class (Java 17)?
**Concept:**
Sealed classes allow you to restrict which other classes may extend them. It gives better control over the inheritance hierarchy.
```java
public sealed class Shape permits Circle, Square {}
public final class Circle extends Shape {}
public final class Square extends Shape {}
```
**Benefit:**
*   **Security:** Prevents unauthorized subclassing.
*   **Pattern Matching:** The compiler knows *exactly* all possible subclasses, enabling exhaustive switch expressions without a `default` case.

---

### Q33: Explain `ArrayList` vs `Vector` vs `CopyOnWriteArrayList`?
**Comparison:**
1.  **`ArrayList`:** Fast, Not Thread-Safe. `O(1)` read.
2.  **`Vector`:** Thread-Safe (Synchronized methods). Slow. Legacy (Java 1.0).
3.  **`CopyOnWriteArrayList` (COWAL):**
    *   **Thread-Safe:** Designed for high-concurrency **Read-Heavy** scenarios.
    *   **Mechanism:** Every time you modify it (`add`, `set`), it copies the **entire underlying array** to a new array. Reads happen on the old array (lock-free).
    *   **Trade-off:** Writes are expensive (`O(n)` memory copy). Reads are extremely fast.

---

### Q34: How does `String.intern()` work?
**Mechanism:**
When `intern()` is called on a String object:
1.  JVM checks the **String Constant Pool** (a special HashTable in Heap).
2.  If an equal String exists, it returns the reference to the pooled String.
3.  If not, it adds this String to the pool and returns the reference.

**Use Case:**
Saving memory when you have millions of duplicate strings (e.g., parsing XML/JSON keys).
```java
String s1 = new String("hello");
String s2 = "hello";
s1 == s2; // false
s1.intern() == s2; // true
```

---

### Q35: What is the difference between `Stream.map()` and `Stream.flatMap()`?
**Deep Dive:**
*   **`map()`:** Transforms one element to one element.
    *   Signature: `Function<T, R>`
    *   Input: `Stream<String>`. Output: `Stream<Integer>` (lengths).
*   **`flatMap()`:** Transforms one element to *multiple* elements (flattening).
    *   Signature: `Function<T, Stream<R>>`
    *   Input: `Stream<List<String>>` (List of sentences).
    *   Output: `Stream<String>` (Stream of all words combined).
    *   It "flattens" nested structures.

---

### Q36: What are `default` methods in Interfaces? Why were they introduced?
**History:**
Before Java 8, adding a method to an interface broke all implementing classes.
To support **Lambdas and Streams**, Java needed to add `stream()` method to the `Collection` interface. Doing this without default methods would have broken every custom Collection implementation in the world.

**Solution:**
`default` methods allow interfaces to provide an implementation.
```java
public interface Collection<E> {
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
}
```
**Diamond Problem:** If a class implements Interface A and B, and both have the same default method, the class **must** override it to resolve ambiguity.

---

### Q37: What is `try-with-resources`? How does it handle suppressed exceptions?
**(Already covered in Q18 - skipping duplicate concept but expanding on "Suppressed")**
**Suppressed Exceptions Internals:**
If the `try` block throws `Exception A`, and the implicit `close()` call throws `Exception B`:
*   In classic `try-finally`: `Exception B` swallows `Exception A`. You lose the original error.
*   In `try-with-resources`: `Exception A` is thrown. `Exception B` is added to A's "suppressed list". You can access it via `A.getSuppressed()`.

---

### Q38: What is `Optional`? Is it Serializable?
**Concept:**
A container object which may or may not contain a non-null value. It forces you to handle the "empty" case, reducing NPEs.

**Is it Serializable? NO.**
`Optional` does not implement `Serializable`.
**Why?** It was designed as a **return type** for methods, not as a field type for POJOs. Using `Optional` fields in a class is considered an anti-pattern because it adds memory overhead and breaks serialization.

---

### Q39: What is `Predicate`, `Consumer`, `Supplier`, `Function`?
**Core Functional Interfaces (Java 8):**
1.  **`Predicate<T>`:** `T -> boolean`. Checks condition. (Used in `.filter()`).
2.  **`Consumer<T>`:** `T -> void`. Performs action. (Used in `.forEach()`).
3.  **`Supplier<T>`:** `void -> T`. Provides value. (Used in `Optional.orElseGet()`).
4.  **`Function<T, R>`:** `T -> R`. Transforms value. (Used in `.map()`).

---

### Q40: What is the purpose of `WeakHashMap`?
**Mechanism:**
A Map where keys are **WeakReferences**.
*   In a normal `HashMap`, putting an object as a Key creates a Strong Reference. Even if you set the original variable to `null`, the Map holds it, so GC cannot collect it.
*   In `WeakHashMap`, if the key is no longer referenced anywhere else in the application (outside the map), the entry is automatically removed during the next GC cycle.
**Use Case:** Caching metadata about objects where the metadata should vanish when the object dies.

---

### Q41: Explain the `finalize()` method. Why is it deprecated?
**Issues with Finalizers:**
1.  **Unpredictable:** No guarantee when (or if) it runs.
2.  **Performance:** Adds significant overhead to GC (objects take at least 2 GC cycles to die).
3.  **Security:** Vulnerable to "Finalizer attacks".
4.  **Resurrection:** An object can assign `this` to a static variable inside `finalize()`, preventing GC.

**Replacement:** Use `java.lang.ref.Cleaner` (Java 9) or `AutoCloseable`.

---

### Q42: What is the difference between `Process` and `Thread`?
**Deep Dive:**
*   **Process:** An execution instance of a program (e.g., JVM). Has its own independent memory space (Heap). Communication requires IPC (Sockets/Pipes). Context switching is heavy.
*   **Thread:** A lightweight path of execution within a Process. Shares the same memory (Heap) but has its own Stack. Context switching is fast.

---

### Q43: What is Context Switching?
**Concept:**
The CPU saving the state (registers, program counter) of the currently running thread and loading the state of the next thread.
**Cost:**
It is expensive. It invalidates CPU caches (L1/L2), leading to cache misses. High thread contention causes excessive context switching, killing performance (Thrashing).

---

### Q44: What is `ReentrantLock`? How does it differ from `synchronized`?
**Architecture:**
Based on **AQS (AbstractQueuedSynchronizer)**.
**Advantages over `synchronized`:**
1.  **Timeout:** `tryLock(1, TimeUnit.SECONDS)`. Avoids infinite deadlocks.
2.  **Fairness:** `new ReentrantLock(true)` grants lock to the longest-waiting thread. (`synchronized` is unfair/random).
3.  **Interruptible:** Can be interrupted while waiting for lock (`lockInterruptibly()`).
4.  **Condition Variables:** Supports multiple conditions (`newCondition()`) for one lock (e.g., "NotFull", "NotEmpty"), whereas `synchronized` only has one wait-set.

---

### Q45: What is `Thread.join()`?
**Mechanism:**
It puts the current thread into `WAITING` state until the thread on which `join()` is called terminates.
**Internals:**
It uses `wait()` loop.
```java
while (isAlive()) {
    wait(0);
}
```
When the target thread dies, the JVM calls `notify_all` (in C++ code), waking up the joining thread.

---

### Q46: What is a Daemon Thread?
**(Covered in Basic Q19/20 - Adding implementation detail)**
**JVM Shutdown Rule:**
The JVM continues running as long as there is at least one **Non-Daemon** (User) thread running. If only Daemon threads remain, the JVM kills them abruptly and exits.
**Warning:** Do not do I/O (write to file) in a Daemon thread's `finally` block. It might never execute!

---

### Q47: What is `yield()`?
**Hint to Scheduler:**
It tells the OS: "I am willing to give up my current CPU timeslice".
The OS moves the thread from **RUNNING** to **READY** state.
**Unreliable:** The scheduler is free to ignore it. The thread might run again immediately. Rarely used in production code.

---

### Q48: What is `ThreadLocalRandom`?
**Java 7 Improvement:**
`java.util.Random` is thread-safe but uses a single AtomicLong seed. In high concurrency, all threads contend to update this seed (CAS loop), causing performance bottleneck.
`ThreadLocalRandom.current()` stores the seed *inside the Thread object itself*. No contention. Much faster for parallel systems.

---

### Q49: What is the `transient` keyword?
**Serialization Control:**
Fields marked `transient` are ignored by `ObjectOutputStream`.
**Use Case:**
1.  **Security:** Passwords / API Keys.
2.  **Derived Data:** Fields that can be calculated from others (e.g., `age` calculated from `birthDate`).
3.  **Non-Serializable Objects:** Connections/Sockets.

---

### Q50: How do you stop a thread in Java?
**The Right Way:**
1.  **Do NOT use `stop()`:** It is deprecated and unsafe (leaves objects in inconsistent state/corrupted).
2.  **Use Interrupts:**
    *   Call `workerThread.interrupt()`.
    *   Inside the worker: Check `Thread.currentThread().isInterrupted()`.
    *   Handle `InterruptedException` properly (clean up and exit).
3.  **Use Volatile Flag:** `volatile boolean running = false;`.


---

## <a name="collections-advanced"></a>Java Collections (Advanced)

### Q51: How does `LinkedHashMap` maintain insertion order?
**Internals:**
It extends `HashMap`. It adds a **Doubly Linked List** running through all its entries.
*   Entry class has `before` and `after` pointers (in addition to `next` for the hash bucket).
*   **Access Order:** `new LinkedHashMap<>(cap, load, true)` creates an **LRU Cache** (Least Recently Used). Accessing a key moves it to the end of the list.

### Q52: What is `TreeMap`? How does it work?
**Internals:**
Based on **Red-Black Tree** (Self-balancing Binary Search Tree).
*   **Ordering:** Sorts keys based on Natural Ordering (`Comparable`) or custom `Comparator`.
*   **Complexity:** `O(log n)` for `get`, `put`, `remove`.
*   **Nulls:** Does NOT allow null keys (throws NPE) because it needs to compare keys.

### Q53: Difference between `Iterator` and `ListIterator`?
1.  **Iterator:**
    *   Traverse forward only (`next()`).
    *   Works on Set, List, Map (keySet).
2.  **ListIterator:**
    *   Traverse forward and backward (`previous()`).
    *   Can add/set elements during iteration.
    *   Works only on `List`.

### Q54: What happens if you modify a collection while iterating? (`ConcurrentModificationException`)
**Fail-Fast Mechanism:**
Collections maintain a `modCount` variable.
*   Iterator records `expectedModCount = modCount` at creation.
*   Every `next()` call checks `if (modCount != expectedModCount) throw CME`.
**Solution:**
1.  Use `Iterator.remove()` (safe).
2.  Use `ConcurrentHashMap` or `CopyOnWriteArrayList` (Fail-Safe iterators).

### Q55: How to sort a Map by Value?
**Approach:**
Maps cannot be sorted directly.
1.  Get `entrySet()`.
2.  Convert to `List<Entry>`.
3.  Sort list using `Comparator.comparing(Entry::getValue)`.
4.  Put back into a `LinkedHashMap` (to preserve order).

---

## <a name="jvm-advanced"></a>JVM & Performance Tuning

### Q56: What is the purpose of the Survivor Spaces (S0, S1)?
**Aging Objects:**
If we promoted objects directly from Eden to Old Gen, the Old Gen would fill up quickly, causing frequent Full GCs (slow).
**Mechanism:**
Objects "ping-pong" between S0 and S1 after every Minor GC.
Each copy increments the "Object Age". Only after age 15 (default) do they move to Old Gen. This filters out short-lived objects.

### Q57: What is `OutOfMemoryError: Metaspace`?
**Cause:**
Metaspace stores Class Metadata.
This error happens if you load too many classes dynamically (e.g., using Reflection, CGLIB proxies in Spring, JSP compilation) and exhaust the Metaspace limit.
**Fix:** Increase `-XX:MaxMetaspaceSize`.

### Q58: What is JIT (Just-In-Time) Compiler?
**Optimization:**
JVM interprets bytecode initially (slow).
JIT monitors "Hot Spots" (frequently executed code).
It compiles these hot spots into optimized **Native Machine Code**.
*   **C1 Compiler (Client):** Fast compilation, basic optimization.
*   **C2 Compiler (Server):** Slow compilation, aggressive optimization (Inlining, Loop unrolling).

### Q59: Explain `invokedynamic` instruction?
**Java 7 Feature:**
Before Java 7, all method calls were linked at compile time.
`invokedynamic` allows delaying the linking of a method call until runtime.
**Use Case:** It is the foundation of **Lambdas** in Java 8. It allows the JVM to choose the strategy for implementing lambdas efficiently without creating anonymous inner classes at compile time.

### Q60: What are Phantom References used for?
**Deep Dive:**
Unlike WeakReferences, PhantomReferences are **not** automatically cleared by GC. They are enqueued in a `ReferenceQueue`.
**Usage:**
Replacing `finalize()`. By polling the queue, you know *exactly* when an object has been removed from memory, allowing you to cleanup native resources (like off-heap memory in `DirectByteBuffer`) safely.

---

## <a name="java-threads-advanced"></a>Concurrency Patterns

### Q61: What is the "Producer-Consumer" problem? Solution?
**Pattern:**
Producer adds data to buffer. Consumer takes data.
**Issues:**
*   Buffer Full: Producer must wait.
*   Buffer Empty: Consumer must wait.
**Solution (BlockingQueue):**
`ArrayBlockingQueue` handles all the wait/notify logic internally.
```java
queue.put(x); // Blocks if full
x = queue.take(); // Blocks if empty
```

### Q62: What is "Double-Checked Locking" in Singleton?
**Code:**
```java
if (instance == null) { // 1st check
    synchronized (Singleton.class) {
        if (instance == null) { // 2nd check
            instance = new Singleton();
        }
    }
}
```
**Why Volatile?**
`instance` must be `volatile`. Without it, due to instruction reordering, another thread might see a partially constructed object (reference assigned before constructor finishes).

### Q63: What is `AtomicInteger`? How does it work?
**CAS (Compare-And-Swap):**
It uses a CPU instruction (hardware support).
`compareAndSwap(expectedValue, newValue)`
*   Reads current value.
*   If current == expected, update to new.
*   If not, retry (loop).
*   **Lock-Free:** No OS-level suspension of threads. Faster than `synchronized`.

### Q64: What is `LongAdder`? (Java 8)
**Scaling AtomicLong:**
Under very high contention, `AtomicLong` suffers because all threads fight to update one variable (CAS failures).
**LongAdder:**
Maintains a set of variables (cells) internally. Threads update different cells to reduce contention. When you call `sum()`, it adds all cells together.
**Verdict:** Use `LongAdder` for statistics/metrics.

### Q65: What is `CopyOnWriteArraySet`?
**Internals:**
Backed by `CopyOnWriteArrayList`.
Efficient for small sets where reads vastly outnumber writes (e.g., storing Listeners/Subscribers).

### Q66: What is `CountDownLatch` vs `CyclicBarrier`?
**Difference:**
*   **Latch:** One-time use. "Wait for N tasks to finish." (e.g., Server waits for cache to warm up).
*   **Barrier:** Reusable. "Wait for N threads to reach a meeting point, then continue together." (e.g., Parallel computation steps).

### Q67: What is `Exchanger`?
**Concept:**
Synchronization point where two threads can swap objects.
Thread A calls `exchange(objA)` and waits.
Thread B calls `exchange(objB)`.
Returns swapped objects. Useful for buffer swapping (Filling vs Emptying).

### Q68: What is `Semaphore`?
**Rate Limiting:**
Maintains a set of permits.
`acquire()`: decrements. Blocks if 0.
`release()`: increments.
**Usage:** Limiting DB connections or API calls.

### Q69: What is `Thread.sleep(0)`?
**Optimization:**
It triggers a yield. It tells the OS to check if any other threads of equal priority are waiting. Useful in spin-loops to prevent 100% CPU usage.

### Q70: What is False Sharing?
**Hardware Cache Issue:**
CPU caches work in "Lines" (64 bytes).
If `Var A` and `Var B` are adjacent in memory, they load into the same Cache Line.
Thread 1 updates A. Thread 2 reads B.
Because A changed, the *entire cache line* is marked invalid. Thread 2 must reload from RAM.
**Solution:** Padding variables (`@Contended` annotation in Java 8).

---

## <a name="misc-advanced"></a>Miscellaneous Advanced

### Q71: Explain `System.identityHashCode()`?
**Concept:**
Returns the hash code that would be returned by default `hashCode()` (memory address based), even if the object overrides `hashCode()`.
**Usage:** `IdentityHashMap` uses this to compare keys by reference (`==`), not by logic (`equals`).

### Q72: What is `Floating Point` determinism (`strictfp`)?
**Issue:** Different CPUs (Intel vs AMD) might calculate float results slightly differently (precision).
**`strictfp`:** Forces strict adherence to IEEE 754 standard, ensuring identical results on all platforms. (Obsolete in Java 17+ as strict semantics are now required).

### Q73: What is the difference between `Error` and `Exception`?
*   **Exception:** Recoverable. App should handle it.
*   **Error:** Serious JVM problem (OOM, StackOverflow, LinkageError). App typically cannot recover.

### Q74: What is `ShutdownHook`?
**Graceful Shutdown:**
A thread registered with `Runtime.getRuntime().addShutdownHook(thread)`.
Runs when JVM receives termination signal (SIGTERM/Ctrl+C).
**Usage:** Close DB connections, save state, release file locks.

### Q75: How does `String` concatenation work internally?
**Optimizations:**
*   **Java 8:** Compiled to `StringBuilder`.
*   **Java 9+:** Uses `invokedynamic` to call `StringConcatFactory`. This allows JVM to switch strategies (e.g., perform size calculation upfront and allocate exact byte array) without recompiling code.

