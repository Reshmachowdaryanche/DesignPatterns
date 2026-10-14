# 1. Singleton Design Pattern

## 1. Overview

- **Category:** Creational Design Pattern
- **Purpose:** Ensure that a class has only one instance within a defined scope and provide a common access point to that instance.
- **Main focus:** Controlling object creation and sharing.

### Definition

Singleton is a creational design pattern that ensures a class has only one instance and provides a global access point to it.

## 2. Problem It Solves

Suppose multiple components in an application need to access the same configuration manager.

Without Singleton, each component might create a separate instance, resulting in duplicated state or unnecessary resource usage.

Singleton allows components to access the same shared instance.

```text
          AppConfig (one instance)
                    |
          +---------+---------+
          |         |         |
        Home     Settings   Profile
```

## 3. Real-World Analogy

Consider a central configuration manager used by different screens in an application.

Instead of creating a separate configuration manager for each screen, the application accesses one shared instance.

**Key idea:** One shared instance, multiple consumers.

## 4. Implementation in Kotlin

### Approach 1: Kotlin `object` (Preferred)

```kotlin
object AppConfig {

    var baseUrl: String = "https://api.example.com"

    fun printConfig() {
        println(baseUrl)
    }
}
```

Usage:

```kotlin
AppConfig.printConfig()

AppConfig.baseUrl = "https://qa.example.com"

AppConfig.printConfig()
```

**How it works:**

- Kotlin creates a single instance of the object declaration.
- Initialization is lazy and occurs when the object is first accessed.
- Initialization is thread-safe.
- No constructor call is required.
- The object can be accessed directly by its name.

For ordinary Kotlin singletons, `object` is generally the simplest implementation.

### Approach 2: Private Constructor and Companion Object

```kotlin
class AppConfig private constructor() {

    fun printConfig() {
        println("App configuration")
    }

    companion object {
        val instance: AppConfig by lazy {
            AppConfig()
        }
    }
}
```

Usage:

```kotlin
AppConfig.instance.printConfig()
```

**How it works:**

- `private constructor()` prevents external code from constructing the class directly.
- `companion object` provides the access point.
- `by lazy` initializes the instance on first access.
- Kotlin's default `lazy` initialization is thread-safe.

**Interview point:** Prefer `object` for a normal Kotlin singleton. Use a private constructor when you specifically need a class-based implementation.

## 5. Singleton in Android

### Example 1: Shared Configuration

```kotlin
object NetworkConfig {
    const val BASE_URL = "https://api.example.com"
}
```

Usage:

```kotlin
class UserRepository {
    fun getBaseUrl(): String {
        return NetworkConfig.BASE_URL
    }
}
```

Other components can access the same configuration without creating instances.

### Example 2: Singleton with Hilt

```kotlin
@Singleton
class TokenManager @Inject constructor(
    private val tokenStorage: TokenStorage
) {
    fun getToken(): String? {
        return tokenStorage.readToken()
    }
}
```

For constructor injection, Hilt uses `@Singleton` to scope the provided instance to the application's `SingletonComponent`, provided the class is injected through the configured Hilt graph.

**Important distinction:**

- Kotlin `object` implements a language-level singleton.
- Hilt `@Singleton` scopes an instance managed by dependency injection.
- Singleton is a design pattern; dependency injection is a technique for supplying dependencies.

Other possible applications include shared caches, application-wide configuration and shared services.

## 6. Advantages

1. **Single shared instance:** Avoids creating multiple instances when sharing is appropriate.
2. **Convenient access:** Provides a common access point.
3. **Resource reuse:** Can reduce repeated initialization costs.
4. **Consistent state:** Consumers can share the same state when that is the intended behavior.
5. **Kotlin support:** The `object` declaration makes implementation simple.

## 7. Disadvantages

1. **Global mutable state:** Changes in one component can unexpectedly affect others.
2. **Testing difficulties:** Tests can become dependent on shared state.
3. **Tight coupling:** Direct global access can hide dependencies.
4. **Memory leaks:** A long-lived singleton can retain an Activity or another short-lived object.
5. **Concurrency risks:** Shared mutable data may be accessed concurrently.
6. **Lifecycle concerns:** An object can live longer than the resources it manages.

## 8. Thread Safety

### Important concept

Kotlin guarantees thread-safe initialization of an `object` declaration. However, it does not automatically make all operations on that object's mutable state thread-safe.

Example:

```kotlin
object Counter {
    var count = 0

    fun increment() {
        count++
    }
}
```

If multiple threads call `increment()` concurrently, updates can be lost because `count++` is not an atomic operation.

Depending on the requirement, use synchronization, atomic types, a coroutine `Mutex`, or another appropriate concurrency mechanism.

**Interview takeaway:** One instance does not mean thread-safe state.

## 9. When to Use Singleton

Use Singleton when:

- One shared instance is genuinely required.
- The instance has a clearly defined lifecycle.
- Shared state is intentional and controlled.
- Repeated construction would be unnecessary or expensive.

Avoid Singleton when:

- Multiple independent instances are required.
- The object stores screen-specific state.
- Global state makes testing or maintenance difficult.
- It retains an Activity or other short-lived component.
- Dependency injection can manage dependencies and lifecycle more clearly.

**Android best practice:** Avoid storing an Activity reference in an application-wide singleton unless there is a compelling, lifecycle-safe reason.

## 10. Singleton vs Dependency Injection

| Singleton | Dependency Injection |
|---|---|
| Controls or represents shared instance creation | Supplies dependencies to consumers |
| Can be implemented using Kotlin `object` | Can be implemented manually or with Hilt/Dagger |
| Often accessed globally | Dependencies can be supplied explicitly |
| May introduce hidden dependencies | Makes dependencies more visible |
| Does not inherently provide test isolation | Makes substituting test dependencies easier |

These concepts are related but are not alternatives in every situation. Dependency injection can manage a singleton-scoped object.

## 11. Common Interview Questions

### Q1. What is the Singleton pattern?

A creational pattern that ensures one instance within a defined scope and provides a common access point.

### Q2. How do you implement Singleton in Kotlin?

Use an `object` declaration for a standard singleton. Alternatively, use a private constructor and a companion object.

### Q3. Is Kotlin `object` thread-safe?

Its initialization is thread-safe. Concurrent operations on mutable properties are not automatically thread-safe.

### Q4. Can Singleton cause a memory leak?

Yes. For example, an application-wide singleton retaining an Activity can prevent the Activity from being garbage-collected.

### Q5. Is Singleton always good for performance?

No. Although it can avoid repeated initialization, it can also introduce global state, contention and unnecessarily long object lifetimes.

### Q6. Is Hilt `@Singleton` the same as Kotlin `object`?

No. Hilt scopes a dependency within its `SingletonComponent`; Kotlin `object` is a language-level singleton declaration.

### Q7. Why can Singleton make unit testing difficult?

Tests may share mutable state, depend on execution order, or find it difficult to replace a globally accessed dependency with a fake.

## 12. Common Mistakes

- Assuming Singleton automatically makes all properties thread-safe.
- Using Singleton for every service or manager.
- Storing an Activity context in a long-lived singleton.
- Using global mutable state unnecessarily.
- Confusing Singleton with dependency injection.
- Ignoring lifecycle and testability.

## 13. Interview Summary

- **Category:** Creational.
- **Intent:** Control instance creation and provide a shared access point.
- **Kotlin implementation:** `object`.
- **Android examples:** Shared configuration, caches and Hilt-scoped services.
- **Thread safety:** Initialization is thread-safe; mutable operations require appropriate protection.
- **Best practice:** Use Singleton only when shared instance semantics are justified, and prefer dependency injection for managing application dependencies.

### One-Line Interview Answer

"Singleton is a creational design pattern that provides a single shared instance within a defined scope. In Kotlin, I generally use an `object` declaration for simple cases and dependency injection with an appropriate scope for application services, while paying attention to mutable state, concurrency, lifecycle and testability."





## 4.3 Singleton Pattern in Java

In Java, Singleton is commonly implemented using a private constructor and a static instance.

### Approach 1: Eager Initialization

The instance is created when the class is initialized.

```java
public class AppConfig {

    private static final AppConfig INSTANCE = new AppConfig();

    private AppConfig() {
        // Prevent external object creation
    }

    public static AppConfig getInstance() {
        return INSTANCE;
    }

    public void printConfig() {
        System.out.println("App configuration");
    }
}
```

**Usage:**

```java
AppConfig config = AppConfig.getInstance();
config.printConfig();
```

**Advantages**
- Simple implementation.
- Thread-safe instance initialization through Java class initialization.
- No synchronization required in `getInstance()`.

**Disadvantage**
- The instance is created when the class is initialized, even if it is never used.

### Approach 2: Lazy Initialization (Not Thread-Safe)

The instance is created only when requested for the first time.

```java
public class AppConfig {

    private static AppConfig instance;

    private AppConfig() {
    }

    public static AppConfig getInstance() {
        if (instance == null) {
            instance = new AppConfig();
        }
        return instance;
    }
}
```

**Problem:** If multiple threads call `getInstance()` simultaneously, they may create multiple instances.

Use this implementation only when concurrent access is not possible or as an example of what not to use in multithreaded code.

### Approach 3: Synchronized Lazy Initialization

```java
public class AppConfig {

    private static AppConfig instance;

    private AppConfig() {
    }

    public static synchronized AppConfig getInstance() {
        if (instance == null) {
            instance = new AppConfig();
        }
        return instance;
    }
}
```

**Advantages**
- Lazy initialization.
- Thread-safe instance creation.

**Disadvantage**
- Synchronization is performed on every call to `getInstance()`, which may add overhead.

### Approach 4: Double-Checked Locking

This approach reduces synchronization overhead after the instance has been created.

```java
public class AppConfig {

    private static volatile AppConfig instance;

    private AppConfig() {
    }

    public static AppConfig getInstance() {
        if (instance == null) {
            synchronized (AppConfig.class) {
                if (instance == null) {
                    instance = new AppConfig();
                }
            }
        }
        return instance;
    }
}
```

**Why is `volatile` required?**

- It ensures visibility of the published instance across threads.
- It prevents problematic reordering around publication of the instance.
- Without `volatile`, double-checked locking is not correct under the Java Memory Model.

**Advantages**
- Lazy initialization.
- Thread-safe instance creation.
- Avoids acquiring the lock on every call after initialization.

**Disadvantages**
- More complex than other implementations.
- Easier to implement incorrectly.

### Approach 5: Initialization-on-Demand Holder Idiom

This is a clean option for lazy, thread-safe initialization in Java.

```java
public class AppConfig {

    private AppConfig() {
    }

    private static class Holder {
        private static final AppConfig INSTANCE = new AppConfig();
    }

    public static AppConfig getInstance() {
        return Holder.INSTANCE;
    }
}
```

**How it works:**

- Loading `AppConfig` does not initialize `Holder`.
- The nested class is initialized when `getInstance()` first accesses it.
- Java class initialization provides thread safety.
- No explicit synchronization is required.

### Approach 6: Enum Singleton

```java
public enum AppConfig {
    INSTANCE;

    public void printConfig() {
        System.out.println("App configuration");
    }
}
```

**Usage:**

```java
AppConfig.INSTANCE.printConfig();
```

**Advantages**
- Thread-safe instance initialization.
- Strong built-in protection against ordinary serialization and reflection attacks that can affect conventional Singleton implementations.
- Simple implementation.

**Limitation**
- An enum is not suitable for every class-based design, especially when inheritance from another class is required.

## 4.4 Java Singleton Comparison

| Approach | Lazy? | Thread-safe initialization? |
|---|---|---|
| Eager initialization | No | Yes |
| Basic lazy initialization | Yes | No |
| Synchronized method | Yes | Yes |
| Double-checked locking with `volatile` | Yes | Yes |
| Holder idiom | Yes | Yes |
| Enum Singleton | On enum initialization | Yes |

## 4.5 Interview Takeaway: Java vs Kotlin

- **Java:** The holder idiom is a strong default for lazy, thread-safe initialization; eager initialization is simpler when lazy creation is unnecessary.
- **Kotlin:** Prefer `object` for a straightforward singleton.
- **Android:** Consider Hilt or another dependency injection mechanism when you need a shared application-scoped dependency.
- **Important:** All these approaches control instance creation. They do not automatically make mutable data or methods thread-safe.
