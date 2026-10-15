# Factory Design Pattern — Java & Kotlin

## 1. Definition

The Factory Design Pattern is a creational design approach that encapsulates object-creation logic in a separate factory class or function.

Instead of creating objects directly in client code, the client requests an object from the factory.

**Main goal:** Separate object creation from object usage.

### Key ideas
- Client requests an object from the factory.
- Factory decides which concrete object to create.
- Factory returns the object through a common interface.
- Client uses the object without depending directly on its concrete implementation.

> Note: Simple Factory is a common design idiom, but it is not one of the original 23 Gang of Four (GoF) design patterns.

## 2. Problem Without a Factory

Suppose an application supports Email and SMS notifications.

```java
EmailNotification notification = new EmailNotification();
notification.send("Hello");
```

### Problems
- Client code directly depends on the concrete class.
- Object-creation logic may be duplicated across the application.
- Changing creation logic may require modifying multiple classes.
- Adding new implementations can make client code more complicated.

## 3. Solution Using a Factory

Move object creation into a dedicated factory.

```java
Notification notification =
    NotificationFactory.create("email");

notification.send("Hello");
```

The client asks the factory for a notification rather than directly instantiating `EmailNotification`.

### Class structure

```text
             Client
                |
                v
      NotificationFactory
                |
        Creates an object
                |
        +-------+-------+
        |               |
        v               v
 EmailNotification  SmsNotification
        |               |
        +-------+-------+
                |
                v
           Notification
             (Interface)
```

Both concrete classes implement the same `Notification` interface.

## 4. Java Implementation

### Step 1: Create the interface

```java
interface Notification {
    void send(String message);
}
```

The interface defines a common contract for all notification implementations.

### Step 2: Create concrete implementations

```java
class EmailNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

class SmsNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}
```

Each class provides its own implementation of `send()`.

### Step 3: Create the factory

```java
class NotificationFactory {
    public static Notification create(String type) {
        if (type.equalsIgnoreCase("email")) {
            return new EmailNotification();
        }

        if (type.equalsIgnoreCase("sms")) {
            return new SmsNotification();
        }

        throw new IllegalArgumentException(
            "Unknown notification type: " + type
        );
    }
}
```

The factory centralizes object creation and returns the common interface.

### Step 4: Use the factory

```java
public class Main {
    public static void main(String[] args) {
        Notification notification =
            NotificationFactory.create("email");

        notification.send("Hello!");
    }
}
```

**Output**
```text
Email: Hello!
```

### Execution flow
1. Client calls `NotificationFactory.create("email")`.
2. Factory checks the requested type.
3. Factory creates an `EmailNotification`.
4. Factory returns it as a `Notification`.
5. Client calls `send()` on the returned object.

## 5. Kotlin Implementation

### Step 1: Create the interface

```kotlin
interface Notification {
    fun send(message: String)
}
```

### Step 2: Create concrete implementations

```kotlin
class EmailNotification : Notification {
    override fun send(message: String) {
        println("Email: $message")
    }
}

class SmsNotification : Notification {
    override fun send(message: String) {
        println("SMS: $message")
    }
}
```

### Step 3: Create the factory

```kotlin
object NotificationFactory {
    fun create(type: String): Notification {
        return when (type.lowercase()) {
            "email" -> EmailNotification()
            "sms" -> SmsNotification()
            else -> throw IllegalArgumentException(
                "Unknown notification type: $type"
            )
        }
    }
}
```

### Step 4: Use the factory

```kotlin
fun main() {
    val notification =
        NotificationFactory.create("email")

    notification.send("Hello!")
}
```

**Output**
```text
Email: Hello!
```

### Important Kotlin concepts
- `object` declares a singleton factory.
- `when` handles the selection logic.
- `: Notification` declares an implemented interface or return type, depending on its position.
- `override` indicates that a member implements or overrides an inherited declaration.
- Kotlin classes are final by default unless marked `open` or `abstract`.

## 6. Why Return an Interface Instead of a Concrete Class?

Compare these declarations:

```java
EmailNotification notification =
    new EmailNotification();

Notification notification =
    NotificationFactory.create("email");
```

The first declaration depends on the concrete `EmailNotification` class. The second depends on the `Notification` interface.

### Benefits of returning an interface
- **Loose coupling:** Client code does not need to depend on a particular implementation.
- **Polymorphism:** Different implementations can be used through the same contract.
- **Maintainability:** Implementation changes are less likely to affect client code.
- **Testability:** Mock or fake implementations can be passed to clients that depend on the interface.
- **Extensibility:** New implementations can be introduced without changing clients that already work through the interface.

**Important:** The factory's conditional logic still needs updating when a new type is added. A factory does not automatically guarantee adherence to the Open/Closed Principle.

## 7. Using the Interface in a Service

The service should depend on the abstraction rather than on a concrete notification class.

### Java

```java
class NotificationService {
    public void sendNotification(
            Notification notification,
            String message) {
        notification.send(message);
    }
}
```

Usage:

```java
NotificationService service = new NotificationService();

Notification notification =
    NotificationFactory.create("email");

service.sendNotification(notification, "Welcome!");
```

### Kotlin

```kotlin
class NotificationService {
    fun sendNotification(
        notification: Notification,
        message: String
    ) {
        notification.send(message)
    }
}
```

Usage:

```kotlin
val service = NotificationService()
val notification = NotificationFactory.create("email")

service.sendNotification(notification, "Welcome!")
```

The service can work with Email, SMS, or any other implementation of `Notification` without needing to know the concrete class.

## 8. Advantages

- Centralizes object-creation logic.
- Reduces direct coupling between clients and concrete classes.
- Makes object creation easier to manage.
- Encourages programming to interfaces.
- Can improve testability and maintainability.
- Provides one place to validate creation parameters and handle unsupported types.

## 9. Disadvantages

- Adds extra classes or abstractions.
- A conditional-based factory must be modified when new types are added.
- Can become large if it handles too many unrelated object types.
- May be unnecessary when object construction is simple and unlikely to change.

**Rule of thumb:** Use a factory when object creation involves selection logic, configuration, validation, or complexity that should not be scattered across client code.

## 10. Real-World Use Cases

- Selecting a payment processor based on payment type.
- Creating notification handlers based on user preferences.
- Creating parsers based on file format.
- Selecting an exporter such as PDF, CSV, or JSON.
- Creating a database-specific component based on configuration.

A factory can work alongside dependency injection, but they are not the same thing.

## 11. Interview Questions and Answers

### Q1. What is the Factory Design Pattern?
It is a creational design approach that encapsulates object creation and separates it from client code.

### Q2. Why do we use a factory?
To centralize object-creation logic, reduce coupling, and allow client code to use implementations through a common abstraction.

### Q3. Why does the factory return an interface?
Returning an interface lets the client use different implementations through the same contract without depending directly on their concrete classes.

### Q4. Is a factory always a singleton?
No. A factory can be a normal object, an instance, a static utility, or a singleton. The choice depends on the requirements.

### Q5. Is Simple Factory a GoF design pattern?
No. Simple Factory is a widely used idiom, but it is not one of the original 23 GoF patterns.

### Q6. Does Factory eliminate the `new` keyword?
No. The factory often uses `new` internally. It hides object creation from the client rather than eliminating instantiation.

### Q7. What happens if an unsupported type is passed?
The factory should handle it explicitly, for example by throwing an exception, returning an appropriate result type, or using a documented fallback.

### Q8. What is the relationship between Factory and polymorphism?
The factory returns an object through a common interface, and polymorphism allows the correct implementation to execute when an interface method is called.

### Q9. Is Factory the same as Dependency Injection?
No. A factory creates or selects objects. Dependency Injection supplies dependencies to a component from an external source. They can be used together.

### Q10. What is the main disadvantage of a conditional-based factory?
Adding a new implementation often requires modifying the factory's selection logic, which can become difficult to maintain as the number of variants grows.

## 12. Quick Revision

- **Category:** Creational design approach.
- **Purpose:** Encapsulate object creation.
- **Core components:** Common interface, concrete implementations, factory, and client.
- **Key principle:** Program to an interface, not an implementation.
- **Main benefit:** Separate creation logic from object usage.
- **Main limitation:** Conditional-based factories can require modification for new variants.
- **Kotlin feature:** `object` can be used for a singleton factory.
- **Java feature:** A static factory method is a common implementation.

### One-line interview answer

"The Factory Design Pattern encapsulates object-creation logic in a separate factory, allowing client code to request objects through a common interface without directly depending on their concrete implementations."

## 13. Scope of This Note

This document covers the basic conditional-based Factory, commonly called Simple Factory.

Keep these as separate topics for later study:
- Factory Method Pattern
- Abstract Factory Pattern
