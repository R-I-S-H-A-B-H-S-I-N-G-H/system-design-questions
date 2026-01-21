# Core Java Interview Questions & Answers

This document contains in-depth answers to Core Java interview questions extracted from the provided PDF.

## Table of Contents
1. [Basic Core Java (Multithreading, Collections, Java 8)](#basic-core-java)
2. [Advanced Core Java (JVM, Memory, Reflection)](#advanced-core-java)

---

## Basic Core Java

### 1. What is multithreading? Lifecycle of a thread?
**Answer:**
Multithreading is a Java feature that allows concurrent execution of two or more parts of a program for maximum utilization of the CPU. Each part of such a program is called a thread. So, threads are light-weight processes within a process.

**Lifecycle of a Thread:**
A thread can be in one of the following states (defined in `Thread.State` enum):
1.  **New:** The thread is in this state when an instance of the `Thread` class is created but `start()` has not been invoked yet.
2.  **Runnable:** After `start()` is called, the thread becomes runnable. It may be running or waiting for the CPU.
3.  **Blocked:** The thread is waiting to acquire a lock to enter a synchronized block/method.
4.  **Waiting:** The thread is waiting indefinitely for another thread to perform a particular action (e.g., `wait()`, `join()`).
5.  **Timed Waiting:** The thread is waiting for another thread for a specified waiting time (e.g., `sleep(1000)`).
6.  **Terminated:** The thread has finished its execution.

### 2. Difference between Runnable and Thread class?
**Answer:**
*   **Extending `Thread` class:**
    *   Cannot extend any other class (Java doesn't support multiple inheritance).
    *   Each thread creates a unique object.
    *   `class MyThread extends Thread { public void run() { ... } }`
*   **Implementing `Runnable` interface:**
    *   Can extend another class.
    *   Promotes code reusability (separating task from runner).
    *   Multiple threads can share the same runnable instance.
    *   `class MyTask implements Runnable { public void run() { ... } }`
    *   **Verdict:** Implementing `Runnable` is preferred for flexibility.

### 3. What are thread priorities?
**Answer:**
Thread priority signifies the importance of a thread to the scheduler.
*   Range: 1 (MIN) to 10 (MAX). Default is 5 (NORM).
*   Set using `thread.setPriority(int)`.
*   **Note:** Priority behavior is platform-dependent and not guaranteed.

### 4. What is thread synchronization?
**Answer:**
Synchronization controls access of multiple threads to any shared resource to prevent consistency errors.
*   **Synchronized Method:** Locks the object (instance) or class (static).
*   **Synchronized Block:** Locks a specific object. Preferred for performance (critical section only).

### 5. Difference between wait(), notify(), and notifyAll()?
**Answer:**
Methods of `Object` class used for inter-thread communication. Must be used in synchronized context.
*   `wait()`: Releases lock and waits until notified.
*   `notify()`: Wakes up a single waiting thread (arbitrary choice).
*   `notifyAll()`: Wakes up all waiting threads.

### 6. What is ExecutorService in Java?
**Answer:**
A framework in `java.util.concurrent` to manage thread pools and asynchronous task execution.
*   Decouples task submission from execution.
*   Implementations: `ThreadPoolExecutor`, `ScheduledThreadPoolExecutor`.
*   Example: `ExecutorService es = Executors.newFixedThreadPool(5);`

### 7. What is Callable vs Runnable?
**Answer:**
*   **Runnable:** `void run()`. Cannot return value or throw checked exception. (Java 1.0)
*   **Callable:** `V call()`. Returns result and can throw exception. Used with `ExecutorService` to get a `Future`. (Java 5.0)

### 8. Difference between ConcurrentHashMap and HashMap?
**Answer:**
*   **HashMap:** Not thread-safe. Allows null key/values. Throws `ConcurrentModificationException`.
*   **ConcurrentHashMap:** Thread-safe. No nulls. High concurrency via bucket locking (CAS/synchronized) rather than full map locking.

### 9. What is thread-safe collection?
**Answer:**
Guarantees data integrity under concurrent access.
*   `ConcurrentHashMap`, `CopyOnWriteArrayList`, `BlockingQueue`.
*   `Collections.synchronizedList(...)` (Older wrapper approach).

### 10. Difference between ReentrantLock and synchronized?
**Answer:**
*   **synchronized:** Implicit, block-scoped, automatic release.
*   **ReentrantLock:** Explicit (`lock()`, `unlock()`), supports fairness, `tryLock()` (timeout), and interruptible locking.

### 11. What is volatile keyword?
**Answer:**
`volatile` indicates that a variable's value will be modified by different threads.
*   Guarantees **visibility**: Changes made by one thread are immediately visible to others (bypassing CPU cache).
*   Prevents instruction reordering.
*   Does **not** guarantee atomicity (e.g., `count++` is not atomic even with volatile).

### 12. What are atomic variables?
**Answer:**
Classes in `java.util.concurrent.atomic` (e.g., `AtomicInteger`, `AtomicReference`).
*   Provide lock-free thread-safety for single variables.
*   Use low-level CPU operations like CAS (Compare-And-Swap).
*   Example: `AtomicInteger count = new AtomicInteger(0); count.incrementAndGet();`

### 13. Difference between synchronized block and method?
**Answer:**
*   **Method:** Locks the whole method. `this` (instance) or `Class` (static).
*   **Block:** Locks only a specific section. Can lock on any object. generally preferred for reducing lock contention scope.

### 14. What is deadlock? How to prevent it?
**Answer:**
**Deadlock:** Two or more threads are waiting for each other to release locks forever.
**Prevention:**
1.  **Lock Ordering:** Always acquire locks in a consistent order.
2.  **Lock Timeout:** Use `tryLock()` with timeout.
3.  **Reduce Scope:** Minimize synchronized block size.

### 15. What is livelock?
**Answer:**
Threads are not blocked, but they are too busy responding to each other to resume work (e.g., two people trying to pass each other in a hall and constantly moving to the same side).


### 16. What is starvation in threads?
**Answer:**
Starvation describes a situation where a thread is unable to gain regular access to shared resources and is unable to make progress. This happens when shared resources are made unavailable for long periods by "greedy" threads (e.g., high priority threads constantly utilizing CPU).

### 17. Difference between yield() and sleep()?
**Answer:**
*   **`sleep(t)`:** Causes the thread to pause execution for a defined time `t`. It does not release any locks. State changes to Timed Waiting.
*   **`yield()`:** Hints the scheduler that the current thread is willing to yield its current use of a processor. The scheduler is free to ignore this hint. It puts the thread back to Runnable state to compete with other threads of same priority.

### 18. Difference between join() and wait()?
**Answer:**
*   **`join()`:** Method of `Thread` class. Allows one thread to wait for the completion of another. `t1.join()` means current thread waits until `t1` dies.
*   **`wait()`:** Method of `Object` class. Used for inter-thread synchronization/communication via monitors.

### 19. What is daemon thread?
**Answer:**
A daemon thread is a service thread that provides services to user threads (e.g., Garbage Collector, Finalizer).
*   Its life depends on user threads.
*   The JVM exits when the only threads running are all daemon threads.
*   Created by calling `thread.setDaemon(true)` before start.

### 20. Difference between user thread and daemon thread?
**Answer:**
*   **User Thread:** High priority. JVM waits for them to finish before terminating.
*   **Daemon Thread:** Low priority. JVM does not wait for them; it kills them if all user threads are done.

### 21. What is thread pool?
**Answer:**
A thread pool is a group of pre-instantiated, idle threads that stand ready to be given work.
*   **Advantages:** Reduces overhead of creating/destroying threads; controls maximum number of concurrent threads (preventing resource exhaustion).

### 22. Difference between fixed and cached thread pool?
**Answer:**
*   **Fixed (`newFixedThreadPool(n)`):** Reuses a fixed number of threads. If all are busy, new tasks wait in a queue. Good for steady load.
*   **Cached (`newCachedThreadPool()`):** Creates new threads as needed, but reuses previously constructed threads when available. Threads that have not been used for 60 seconds are terminated. Good for many short-lived tasks.

### 23. What is Future and CompletableFuture?
**Answer:**
*   **`Future`:** Represents the result of an asynchronous computation. It provides methods to check if computation is complete, wait for completion (`get()`), and retrieve result. Limitations: Cannot be manually completed, cannot chain actions easily, blocking `get()`.
*   **`CompletableFuture` (Java 8):** Implements `Future` and `CompletionStage`. Supports non-blocking callbacks (`thenApply`, `thenAccept`), chaining, and combining multiple futures.

### 24. Difference between synchronous and asynchronous execution?
**Answer:**
*   **Synchronous:** Tasks are executed one after another. The caller waits for the method to return before proceeding.
*   **Asynchronous:** The caller does not wait. The task runs in the background (e.g., separate thread), and the caller is notified via callback, Future, or event when done.

### 25. What is fork/join framework?
**Answer:**
Introduced in Java 7 to speed up parallel processing by attempting to use all available processor cores.
*   **Work-Stealing Algorithm:** Idle threads steal tasks from busy threads' queues.
*   **`ForkJoinPool`:** The specialized thread pool.
*   **`RecursiveTask` / `RecursiveAction`:** Base classes for tasks.

### 26. Difference between parallel and sequential streams?
**Answer:**
*   **Sequential Stream:** Operations are performed on a single thread (the current one). Order is guaranteed.
*   **Parallel Stream:** Splits the source data into chunks and processes them on multiple threads (ForkJoinPool common pool). Order not guaranteed.
*   **Usage:** Use parallel only for large datasets where processing is independent and CPU-intensive.

### 27. How to use Java 8 Stream API?
**Answer:**
The Stream API is used to process collections of objects. A stream is a sequence of objects that supports various methods which can be pipelined to produce the desired result.
*   **Source:** `list.stream()`
*   **Intermediate Ops:** `filter`, `map`, `sorted` (Lazy)
*   **Terminal Ops:** `collect`, `forEach`, `reduce` (Eager)
*   *Example:* `list.stream().filter(s -> s.startsWith("A")).map(String::toUpperCase).collect(Collectors.toList());`

### 28. What are functional interfaces?
**Answer:**
An interface with exactly one abstract method. They can be implemented using Lambda expressions.
*   Marked with `@FunctionalInterface` (optional but recommended).
*   Examples: `Runnable`, `Callable`, `Comparator`, `Function`, `Predicate`, `Consumer`, `Supplier`.

### 29. Difference between lambda expressions and anonymous classes?
**Answer:**
*   **Syntax:** Lambda is concise (`(args) -> body`). Anonymous class is verbose (`new Interface() { ... }`).
*   **`this` keyword:** In lambda, `this` refers to the enclosing class. In anonymous class, `this` refers to the anonymous class instance itself.
*   **Compilation:** Lambda uses `invokedynamic` bytecode instruction (more efficient). Anonymous class creates a separate `.class` file.

### 30. What are default and static methods in interface?
**Answer:**
*   **Default Method:** Allows adding new methods to interfaces without breaking existing implementations. Uses `default` keyword. Can be overridden.
*   **Static Method:** Utility methods associated with the interface. Cannot be overridden. Called via `InterfaceName.method()`.

### 31. What is method reference?
**Answer:**
Shorthand notation of a lambda expression to call a method. Operator: `::`.
*   Types:
    *   Static method: `ClassName::methodName`
    *   Instance method of object: `instance::methodName`
    *   Instance method of arbitrary object of particular type: `ClassName::methodName`
    *   Constructor: `ClassName::new`

### 32. What is Optional class and how to use it?
**Answer:**
`Optional<T>` is a container object used to contain not-null objects. It avoids null checks and `NullPointerException`.
*   Creation: `Optional.of(val)`, `Optional.ofNullable(val)`, `Optional.empty()`.
*   Usage: `opt.orElse("default")`, `opt.ifPresent(val -> ...)`
*   *Bad Practice:* Using `get()` without `isPresent()`.

### 33. Difference between Map, Filter, and Reduce in streams?
**Answer:**
*   **Filter:** Selects elements based on a predicate (returns boolean). `Stream<T> -> Stream<T>` (subset).
*   **Map:** Transforms elements. `Stream<T> -> Stream<R>`.
*   **Reduce:** Aggregates elements to a single value. `Stream<T> -> T` (e.g., sum, max).

### 34. What are Java Generics?
**Answer:**
Generics allow types (classes and interfaces) to be parameters when defining classes, interfaces and methods.
*   **Benefit:** Stronger type checks at compile time (eliminates ClassCastException) and code reuse.
*   Example: `List<String>` ensures list contains only strings.

### 35. Difference between List extends Object?
**Answer:**
Actually, `List<Object>` vs `List<?>`:
*   `List<Object>`: A list that can hold any object. You can add anything to it.
*   `List<?>`: A list of unknown type. You can read from it (as Object), but you cannot add anything (except null) because the type is not known.

### 36. What are wildcards in Java Generics?
**Answer:**
The question mark `?` represents an unknown type.
*   **Unbounded:** `List<?>`
*   **Upper Bounded:** `List<? extends Number>` (Unknown type that is Number or subclass). Read-only for Number.
*   **Lower Bounded:** `List<? super Integer>` (Unknown type that is Integer or superclass). Can write Integers.

### 37. Difference between checked and unchecked exceptions?
**Answer:**
*   **Checked (Compile-time):** Extend `Exception` (but not `RuntimeException`). Must be handled (`try-catch`) or declared (`throws`). Examples: `IOException`, `SQLException`.
*   **Unchecked (Runtime):** Extend `RuntimeException`. Compiler doesn't force handling. Represent programming errors. Examples: `NullPointerException`, `ArrayIndexOutOfBoundsException`.

### 38. How to create a custom exception?
**Answer:**
Extend `Exception` (for checked) or `RuntimeException` (for unchecked).
```java
public class MyCustomException extends Exception {
    public MyCustomException(String message) {
        super(message);
    }
}
```

### 39. Difference between final, finally, and finalize?
**Answer:**
*   **`final`:** Keyword. Variable (constant), Method (cannot override), Class (cannot extend).
*   **`finally`:** Block used with `try-catch`. Executed always (cleanup code).
*   **`finalize`:** Method in `Object` class. Called by Garbage Collector before reclaiming object memory. Deprecated in Java 9.

### 40. Difference between shallow copy and deep copy?
**Answer:**
*   **Shallow Copy:** Creates a new object, but inserts references to the objects found in the original. Changes to mutable fields in the copy affect the original. Default `clone()`.
*   **Deep Copy:** Creates a new object and recursively copies everything. The copy and original are fully independent.

### 41. How to serialize and deserialize objects?
**Answer:**
*   **Serialization:** Converting an object state into a byte stream. Class must implement `Serializable` interface. Use `ObjectOutputStream.writeObject()`.
*   **Deserialization:** Recreating object from byte stream. Use `ObjectInputStream.readObject()`.

### 42. What is transient keyword?
**Answer:**
Used in serialization. If a field is declared `transient`, it is ignored during the serialization process. Its value will be default (null/0) upon deserialization.

### 43. Difference between Comparable and Comparator?
**Answer:**
*   **`Comparable` (`java.lang`):** Defines "natural ordering" for a class. Implement `compareTo(Object o)`. String, Integer implement this.
*   **`Comparator` (`java.util`):** Defines "custom ordering". Implement `compare(Object o1, Object o2)`. Useful when you can't modify the class or want multiple sorting strategies.

### 44. How to sort a list using Comparator?
**Answer:**
```java
Collections.sort(list, new Comparator<MyClass>() {
    public int compare(MyClass o1, MyClass o2) {
        return o1.age - o2.age;
    }
});
// Java 8
list.sort((o1, o2) -> o1.age - o2.age);
```

### 45. What is Java Reflection API?
**Answer:**
Allows inspection and modification of classes, interfaces, fields, and methods at runtime, even if they are private.
*   Classes: `Class`, `Method`, `Field`, `Constructor`.
*   Usage: Frameworks (Spring, Hibernate), Testing tools.

### 46. Difference between Class.forName() and object.getClass()?
**Answer:**
*   **`Class.forName("className")`:** Static method. Loads the class dynamically by name. Useful when class name is known only at runtime.
*   **`object.getClass()`:** Instance method. Returns the class of the given object. Requires an instance.

### 47. What are Java annotations?
**Answer:**
Metadata provides data about a program that is not part of the program itself.
*   Built-in: `@Override`, `@Deprecated`.
*   Custom: Defined using `@interface`.
*   Processed at compile-time or runtime (via Reflection).

### 48. What is Java Memory Model?
**Answer:**
Describes how threads interact through memory. Defines the rules for visibility of changes to shared variables (happens-before relationship).
*   Divides memory into **Thread Stack** (local variables) and **Heap** (shared objects).

### 49. Difference between stack, heap, and metaspace?
**Answer:**
*   **Stack:** Per-thread. Stores local primitives and object references. LIFO. Auto-cleaned.
*   **Heap:** Shared. Stores Objects. Managed by Garbage Collector.
*   **Metaspace (Java 8+):** Stores Class metadata (static variables, methods). Uses native memory (outside heap). Replaced PermGen.

### 50. Difference between String, StringBuilder, and StringBuffer?
**Answer:**
*   **`String`:** Immutable. Slow for concatenations (creates new objects). Thread-safe (read-only).
*   **`StringBuilder`:** Mutable. Fast. Not thread-safe. (Java 5).
*   **`StringBuffer`:** Mutable. Slower than Builder (synchronized methods). Thread-safe. (Legacy).

---

## Advanced Core Java

### 1. What is JVM architecture?
**Answer:**
JVM (Java Virtual Machine) consists of:
1.  **Class Loader Subsystem:** Loads, links, and initializes class files.
2.  **Runtime Data Areas:** Method Area, Heap, Stack, PC Register, Native Method Stack.
3.  **Execution Engine:** Interpreter, JIT Compiler, Garbage Collector.
4.  **Native Interface (JNI):** Interacts with native libraries.

### 2. Difference between JIT and AOT compilation?
**Answer:**
*   **JIT (Just-In-Time):** Compiles bytecode to native machine code *at runtime*. Optimizes "hot spots" based on execution profile. Standard in HotSpot JVM.
*   **AOT (Ahead-Of-Time):** Compiles bytecode to native code *before execution* (build time). Faster startup, lower memory footprint, but less peak optimization potential. Used in GraalVM Native Image.

### 3. What is class loading mechanism in Java?
**Answer:**
Process of loading class files into memory. Phases:
1.  **Loading:** finding binary representation of class.
2.  **Linking:**
    *   *Verification:* Ensure bytecode validity.
    *   *Preparation:* Allocate memory for static variables (default values).
    *   *Resolution:* Replace symbolic references with direct references.
3.  **Initialization:** Execute static blocks and assign static variables.

### 4. Difference between static and dynamic class loading?
**Answer:**
*   **Static:** Classes are loaded via `new` operator. compiled time dependency.
*   **Dynamic:** Classes are loaded at runtime using `Class.forName()`.

### 5. What are Java bytecode instructions?
**Answer:**
The opcode set for the JVM. 1-byte opcodes followed by operands.
Examples: `aload_0` (load reference), `iadd` (integer add), `invokevirtual` (method call), `new` (create object).

### 6. Difference between stack frame and stack memory?
**Answer:**
*   **Stack Memory:** The entire memory area allocated to a thread.
*   **Stack Frame:** A block within the stack memory created for *each method call*. It contains local variables, operand stack, and frame data (return address). Pushed on enter, popped on exit.

### 7. What is method area in JVM?
**Answer:**
Shared memory area that stores class-level data: class structures, field/method data, runtime constant pool, and code for methods. Logically part of Heap (in PermGen/Metaspace).

### 8. What is metaspace in Java 8+?
**Answer:**
Replaced PermGen. Stores class metadata.
*   **Location:** Native Memory (outside Heap).
*   **Size:** limited only by available system RAM (unlike PermGen which had fixed max size). avoiding `OutOfMemoryError: PermGen space`.

### 9. What is PermGen in earlier Java versions?
**Answer:**
Permanent Generation. Part of Heap. Stored class metadata. prone to OOM errors if too many classes were loaded (e.g., in heavy application servers).

### 10. How does garbage collector work in JVM?
**Answer:**
Automatic memory management.
1.  **Mark:** Traverse object graph from GC Roots (stack variables, statics) to identify live objects.
2.  **Sweep:** Reclaim memory occupied by unvisited (dead) objects.
3.  **Compact (Optional):** Move objects to contiguous memory to reduce fragmentation.

### 11. Difference between minor GC and major GC?
**Answer:**
*   **Minor GC:** Collects from Young Generation (Eden + Survivor). Fast. Triggered when Eden is full.
*   **Major/Full GC:** Collects from Old (Tenured) Generation (and usually Young too). Slower. "Stop-the-world" pause.

### 12. What is GC tuning?
**Answer:**
Optimizing JVM flags to improve performance (throughput/latency).
*   Heap Sizing: `-Xms`, `-Xmx`.
*   Collector Selection: `-XX:+UseG1GC`.
*   Log analysis to minimize Full GC frequency and pause times.

### 13. Difference between Serial, Parallel, CMS, and G1 garbage collectors?
**Answer:**
*   **Serial:** Single-threaded. Small apps.
*   **Parallel:** Multi-threaded throughput collector. High throughput, longer pauses.
*   **CMS (Concurrent Mark Sweep):** Low latency, minimizes pauses. Deprecated.
*   **G1 (Garbage First):** Splits heap into regions. Balances throughput and latency. Default in Java 9+.

### 14. What is escape analysis?
**Answer:**
Compiler optimization. Analyzes scope of a new object. If an object is allocated in a method and *never escapes* (not returned or assigned to static), JVM can optimize it via **Stack Allocation** or **Scalar Replacement**.

### 15. What is scalar replacement?
**Answer:**
If an object doesn't escape, JVM breaks it into its primitive fields ("scalars") and stores them in registers/stack, avoiding heap allocation entirely.

### 16. How does Java allocate objects in Eden, Survivor, and Tenured spaces?
**Answer:**
1.  New objects -> **Eden**.
2.  Minor GC -> Live objects from Eden -> **Survivor S0**.
3.  Next GC -> Live from Eden + S0 -> **Survivor S1** (age incremented).
4.  Swapping S0/S1.
5.  If age > Threshold (default 15) -> Promoted to **Tenured (Old)**.

### 17. What is memory leak in Java?
**Answer:**
Objects are no longer needed by the application but are still referenced (e.g., in a static Map), preventing GC from reclaiming them. Eventually causes `OutOfMemoryError`.

### 18. How to detect memory leaks?
**Answer:**
*   Tools: VisualVM, Eclipse Memory Analyzer (MAT), JProfiler.
*   Technique: Take **Heap Dump**. Analyze "Dominator Tree" to find large objects or static collections holding references.

### 19. Difference between stack, heap, and metaspace?
(See Basic Q49. Stack=Methods/Locals, Heap=Objects, Metaspace=Class Metadata).

### 20. How are local variables stored in stack memory?
**Answer:**
Stored in the "Local Variable Array" of the Stack Frame. Primitives are stored directly. Objects are stored as *references* (pointers) to the Heap.

### 21. How are objects stored in heap memory?
**Answer:**
Objects are stored as a contiguous block of memory containing:
*   **Object Header:** Mark Word (hashcode, lock state, GC age) + Class Pointer.
*   **Instance Data:** Fields.
*   **Padding:** To align to 8-byte boundary.

### 22. Difference between == and equals() in custom objects?
**Answer:**
*   `==`: Checks reference equality (memory address).
*   `equals()`: Default implementation in Object uses `==`. Must be overridden to check logical/content equality.

### 23. How to create objects dynamically using reflection?
**Answer:**
1.  `Class<?> clazz = Class.forName("MyClass");`
2.  `Object obj = clazz.getDeclaredConstructor().newInstance();`

### 24. How to access private fields using reflection?
**Answer:**
```java
Field field = clazz.getDeclaredField("privateField");
field.setAccessible(true); // Disables security check
Object value = field.get(instance);
```

### 25. What is annotations processing?
**Answer:**
A hook in the compilation process (JSR 269) where the compiler scans for annotations and allows "Processors" to generate additional source files (e.g., Lombok generates getters/setters, Dagger generates DI code). It does *not* modify existing code.

### 26. Difference between compile-time and runtime annotations?
**Answer:**
Defined by `@Retention`:
*   `RetentionPolicy.SOURCE`: Discarded by compiler (e.g., `@Override`).
*   `RetentionPolicy.CLASS`: Recorded in .class but ignored by JVM.
*   `RetentionPolicy.RUNTIME`: Available at runtime via Reflection (e.g., `@Autowired`, `@Entity`).

### 27. What is bootstrap, extension, and application class loader?
**Answer:**
1.  **Bootstrap:** Loads core Java classes (`rt.jar`, `java.lang.*`). Written in native code (C++). Parent of all.
2.  **Extension (Platform):** Loads extensions from `jre/lib/ext`.
3.  **Application (System):** Loads classes from classpath (`-cp`).

### 28. How to implement custom ClassLoader?
**Answer:**
Extend `java.lang.ClassLoader` and override `findClass(String name)`. Read byte array from source (file/network) and call `defineClass()`.

### 29. What is dynamic proxy in Java?
**Answer:**
A mechanism to create an implementation of a list of interfaces at runtime without defining a class.
*   Used by Spring AOP, Hibernate (Lazy loading).
*   Requires `InvocationHandler` to intercept method calls.

### 30. How to use Proxy.newProxyInstance()?
**Answer:**
```java
MyInterface proxy = (MyInterface) Proxy.newProxyInstance(
    classLoader,
    new Class<?>[]{MyInterface.class},
    (proxyObj, method, args) -> {
        System.out.println("Before");
        return method.invoke(target, args);
    }
);
```

### 31. Difference between static proxy and dynamic proxy?
**Answer:**
*   **Static:** Programmer creates proxy class manually implementing interface. Tight coupling.
*   **Dynamic:** Generated by Java Reflection API at runtime. Generic handling.

### 32. Difference between Optional.of(), Optional.ofNullable(), and Optional.empty()?
**Answer:**
*   `of(val)`: Throws NPE if `val` is null. Use when you are sure it's not null.
*   `ofNullable(val)`: Returns `Optional` with value if present, otherwise empty `Optional` if null. Safe.
*   `empty()`: Returns an empty instance.

### 33. How to avoid NullPointerException using Optional?
**Answer:**
Avoid `if (x != null)`. Instead:
`Optional.ofNullable(x).map(X::getProp).orElse("default");`

### 34. How to combine multiple CompletableFutures?
**Answer:**
*   `CompletableFuture.allOf(f1, f2)`: Returns void future when all complete.
*   `CompletableFuture.anyOf(f1, f2)`: Returns result of first completed.
*   `f1.thenCombine(f2, (res1, res2) -> combine(res1, res2))`.

### 35. Difference between thenApply(), thenAccept(), thenCompose()?
**Answer:**
*   `thenApply(Function)`: Transform result (`T -> R`). Returns `Future<R>`. (Like Map).
*   `thenAccept(Consumer)`: Consume result (`T -> void`). Returns `Future<Void>`.
*   `thenCompose(Function)`: FlatMap. Function returns another `Future`. Returns `Future<R>` (prevents nested `Future<Future<R>>`).

### 36. Difference between RecursiveTask and RecursiveAction?
**Answer:**
Used in Fork/Join Framework.
*   `RecursiveTask<V>`: Computes a value (returns result).
*   `RecursiveAction`: Runs an action (void).

### 37. What is ReadWriteLock?
**Answer:**
Allows multiple readers to access a resource concurrently, but only one writer.
*   Performance boost for read-heavy workloads.
*   Implementation: `ReentrantReadWriteLock`.

### 38. What is StampedLock?
**Answer:**
Introduced in Java 8. Faster than `ReadWriteLock`.
*   Uses "optimistic locking" (returns a stamp/token).
*   Does not support reentrancy.

### 39. Difference between fair and unfair locks?
**Answer:**
*   **Fair:** Threads acquire lock in order (FIFO). No starvation, lower throughput.
*   **Unfair:** Threads can "barge" in. Higher throughput, potential starvation. Default in `ReentrantLock`.

### 40. How to use CountDownLatch?
**Answer:**
A synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes.
*   Initialized with count `N`.
*   Tasks call `latch.countDown()`.
*   Main thread calls `latch.await()` (blocks until count is 0).
*   **One-time use.**

### 41. How to use CyclicBarrier?
**Answer:**
Allows a set of threads to all wait for each other to reach a common barrier point.
*   Initialized with `parties` (number of threads).
*   Threads call `barrier.await()`.
*   **Reusable** (Cyclic).

### 42. What is Semaphore in Java?
**Answer:**
Maintains a set of permits. Used to restrict the number of threads accessing a resource (Rate Limiting).
*   `acquire()`: Takes a permit (blocks if none).
*   `release()`: Returns a permit.

### 43. What is BlockingQueue?
**Answer:**
A Queue that supports operations that wait for the queue to become non-empty when retrieving an element, and wait for space to become available when storing an element.
*   Implementations: `ArrayBlockingQueue`, `LinkedBlockingQueue`, `PriorityBlockingQueue`.
*   Used heavily in Producer-Consumer pattern.

### 44. Difference between ArrayBlockingQueue and LinkedBlockingQueue?
**Answer:**
*   **ArrayBlockingQueue:** Bounded, backed by array. Lower overhead.
*   **LinkedBlockingQueue:** Optionally bounded (default Integer.MAX_VALUE), backed by linked nodes. Higher throughput (separate locks for head/tail).

### 45. Difference between AtomicInteger and synchronized integer?
**Answer:**
*   **AtomicInteger:** Uses CAS (CPU instruction), non-blocking, faster for simple counters.
*   **Synchronized:** Uses locks (OS mutex), blocking, context switch overhead.

### 46. How to implement thread-safe Singleton?
**Answer:**
Using "Double-Checked Locking":
```java
public class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
*   `volatile` is crucial to prevent instruction reordering.

### 47. Difference between volatile and synchronized?
**Answer:**
*   **volatile:** Visibility only. No atomicity. No blocking.
*   **synchronized:** Visibility + Atomicity. Blocking.

### 48. Difference between Java Heap and Native Memory?
**Answer:**
*   **Heap:** Managed by JVM GC. Stores Java objects.
*   **Native Memory:** Managed by OS. Used by Metaspace, JNI, Direct ByteBuffers (NIO), Thread Stacks. Not garbage collected by JVM (requires explicit release or OS cleanup).

### 49. How to profile Java application memory and CPU usage?
**Answer:**
*   **JVisualVM / JConsole:** Bundled with JDK. Connect to running process.
*   **Java Flight Recorder (JFR):** Low overhead profiling event recorder.
*   **Command Line:** `jmap -heap` (memory), `jstack` (threads), `top -H` (CPU).

### 50. What is Phaser?
**Answer:**
A flexible synchronization barrier, similar to `CyclicBarrier` and `CountDownLatch`, but more dynamic.
*   Allows the number of registered parties to change over time.
*   Supports multiple phases.

### 51. How to implement Producer-Consumer problem?
**Answer:**
Best way is using `BlockingQueue`:
```java
BlockingQueue<Integer> queue = new LinkedBlockingQueue<>(10);
// Producer
Runnable producer = () -> { while(true) queue.put(generate()); };
// Consumer
Runnable consumer = () -> { while(true) process(queue.take()); };
```

### 52. What is Memory Consistency Error?
**Answer:**
Occurs when different threads have inconsistent views of what should be the same data.
*   Happens when there is no "happens-before" relationship between operations.
*   Fixed by using `synchronized`, `volatile`, or atomic variables.

### 53. Difference between RecursiveTask and RecursiveAction?
**Answer:**
*   **RecursiveTask:** Returns a result (like `Callable`).
*   **RecursiveAction:** Does not return a result (like `Runnable`).
*   Both are used in `ForkJoinPool`.
