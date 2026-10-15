
# Factory Method Design Pattern — Java & Kotlin

## 1\. Definition

The Factory Method Design Pattern is a **creational design pattern** in which a parent class defines a method for creating objects, and child classes decide which concrete object to create.

**Main goal:** Separate object creation from the business logic that uses the object.

### Simple Example

Imagine a notification application:

- `EmailCreator` creates an `EmailNotification`.
- `SmsCreator` creates an `SmsNotification`.
- Both notifications implement the same `Notification` interface.

The parent defines how the object will be used, but the child decides which object to create.

**Remember:** The parent defines the creation method; the child decides what object to create.

## 2\. Problem Without Factory Method

Suppose we want to send Email and SMS notifications.

```
Notification notification = new EmailNotification();
notification.send("Hello");
```

If we want to send an SMS instead, we must change the object creation code.

### Problems

- Code directly depends on concrete classes.
- Object creation is mixed with business logic.
- Adding new notification types can require modifying existing code.
- Maintaining multiple implementations becomes harder.

## 3\. Solution Using Factory Method

Instead of creating notification objects directly, we delegate object creation to child classes.

```
NotificationCreator creator = new EmailCreator();
creator.sendMessage("Hello");
```

### Class Structure

```
                 Client
                   |
                   v
          NotificationCreator
             (Parent Class)
                   |
                   v
          createNotification()
            (Factory Method)
                   |
          +--------+--------+
          |                 |
          v                 v
      EmailCreator       SmsCreator
          |                 |
          v                 v
   EmailNotification   SmsNotification
          |                 |
          +--------+--------+
                   |
                   v
              Notification
               (Interface)
```

### Four Main Components

1. **Product:** Common interface (`Notification`).
2. **Concrete Product:** Actual implementation (`EmailNotification`, `SmsNotification`).
3. **Creator:** Parent class that declares the factory method (`NotificationCreator`).
4. **Concrete Creator:** Child class that decides which object to create (`EmailCreator`, `SmsCreator`).

## 4\. Java Implementation

### Step 1: Create the interface

```
interface Notification {
    void send(String message);
}
```

Every notification must implement the `send()` method.

### Step 2: Create concrete classes

```
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

Each class implements the same interface but behaves differently.

### Step 3: Create the parent class

```
abstract class NotificationCreator {

    // Factory Method
    protected abstract Notification createNotification();

    // Common business logic
    public void sendMessage(String message) {
        Notification notification = createNotification();
        notification.send(message);
    }
}
```

**Understand these two methods:**

- `createNotification()` is the Factory Method. Child classes decide what object it returns.
- `sendMessage()` contains the common workflow for sending a message.

The parent knows how to send a message but does not know which concrete notification will be created.

### Step 4: Create child classes

```
class EmailCreator extends NotificationCreator {
    @Override
    protected Notification createNotification() {
        return new EmailNotification();
    }
}

class SmsCreator extends NotificationCreator {
    @Override
    protected Notification createNotification() {
        return new SmsNotification();
    }
}
```

- `EmailCreator` creates an `EmailNotification`.
- `SmsCreator` creates an `SmsNotification`.

### Step 5: Use the pattern

```
public class Main {
    public static void main(String[] args) {

        NotificationCreator creator = new EmailCreator();
        creator.sendMessage("Hello");

        creator = new SmsCreator();
        creator.sendMessage("Welcome");
    }
}
```

**Output**

```
Email: Hello
SMS: Welcome
```

### Execution Flow

1. Client creates an `EmailCreator`.
2. Client calls `sendMessage("Hello")`.
3. The parent calls `createNotification()`.
4. The overridden method in `EmailCreator` creates an `EmailNotification`.
5. The parent calls `send()` on the returned object.
6. The same process works for SMS using `SmsCreator`.

## 5\. Kotlin Implementation

### Step 1: Create the interface

```
interface Notification {
    fun send(message: String)
}
```

### Step 2: Create concrete classes

```
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

### Step 3: Create the parent class

```
abstract class NotificationCreator {

    // Factory Method
    protected abstract fun createNotification(): Notification

    // Common business logic
    fun sendMessage(message: String) {
        val notification = createNotification()
        notification.send(message)
    }
}
```

### Step 4: Create child classes

```
class EmailCreator : NotificationCreator() {
    override fun createNotification(): Notification {
        return EmailNotification()
    }
}

class SmsCreator : NotificationCreator() {
    override fun createNotification(): Notification {
        return SmsNotification()
    }
}
```

### Step 5: Use the pattern

```
fun main() {
    var creator: NotificationCreator = EmailCreator()
    creator.sendMessage("Hello")

    creator = SmsCreator()
    creator.sendMessage("Welcome")
}
```

**Output**

```
Email: Hello
SMS: Welcome
```

### Important Kotlin Concepts

- `abstract class`: A parent class that cannot be instantiated directly.
- `abstract fun`: A method that must be implemented by concrete subclasses.
- `override`: Provides an implementation of an inherited method.
- `protected`: Restricts access to the class hierarchy and permitted Kotlin access contexts.
- `: NotificationCreator()`: Inherits from the parent class and invokes its constructor.

## 6\. Why Do We Use an Interface?

Consider:

```
EmailNotification notification = new EmailNotification();
```

This directly depends on the `EmailNotification` class.

Compare it with:

```
Notification notification = createNotification();
```

This uses the common `Notification` interface.

### Benefits

- **Loose coupling:** Code depends on an interface rather than a concrete implementation.
- **Polymorphism:** Different notification types can be used through the same interface.
- **Maintainability:** The common workflow does not need to know the concrete class.
- **Extensibility:** New notification implementations can be introduced.
- **Testability:** Alternative implementations can be used in suitable tests.

## 7\. Factory Method vs Simple Factory

| Simple Factory | Factory Method |
| --- | --- |
| Usually uses a factory class or function. | Uses a creator class with a factory method. |
| Often uses `if/else` or `switch`. | Uses method overriding and polymorphism. |
| One factory selects among concrete objects. | Concrete creators decide which objects to create. |
| Adding a type often requires changing selection logic. | A new variant can often be added with a new creator and product. |
| Not an original GoF pattern. | One of the original 23 GoF patterns. |

**Interview tip:** Simple Factory selects the object centrally. Factory Method delegates the creation decision to subclasses.

## 8\. Advantages

- Separates object creation from business logic.
- Reduces direct dependencies on concrete products.
- Encourages programming to interfaces.
- Supports polymorphism.
- Makes it easier to introduce new product variants.
- Keeps common workflow logic in one place.

## 9\. Disadvantages

- Introduces additional classes.
- Can make a simple application unnecessarily complex.
- Adding a new variant may require a new product and creator.
- Requires understanding inheritance and polymorphism.

**Rule of thumb:** Use Factory Method when subclasses should decide which object to create and the parent class contains a common workflow that uses that object.

## 10\. Real-World Use Cases

- **Notifications:** Email, SMS, and push notifications.
- **File processing:** JSON, XML, and CSV parsers.
- **Document generation:** PDF and CSV document generators.
- **Payment processing:** Different payment provider implementations.
- **Logging:** File-based and remote logging implementations.
- **Frameworks:** A framework defines a workflow while application-specific subclasses provide the objects used by that workflow.

## 11\. Interview Questions and Answers

### Q1. What is the Factory Method Design Pattern?

It is a creational design pattern that defines a method for creating objects and allows subclasses to decide which concrete object to instantiate.

### Q2. Why do we use Factory Method?

To separate object creation from business logic, reduce direct dependencies on concrete classes, and allow subclasses to choose implementations.

### Q3. What are the four main components?

Product, Concrete Product, Creator, and Concrete Creator.

### Q4. Is Factory Method a GoF pattern?

Yes. It is one of the original 23 Gang of Four design patterns.

### Q5. Does Factory Method eliminate the `new` keyword?

No. Concrete creators often use `new` internally. The pattern delegates the decision about which object to create.

### Q6. Why does the factory method return an interface?

So the parent class can work with different implementations through a common contract.

### Q7. How does Factory Method use polymorphism?

Concrete creator classes override the factory method. The implementation executed depends on the actual creator object at runtime.

### Q8. What is the difference between Factory Method and Simple Factory?

Simple Factory usually uses conditional logic in one factory. Factory Method uses an overridable method that concrete creators implement.

### Q9. Does Factory Method follow the Open/Closed Principle?

It can support the Open/Closed Principle because new variants can often be added through new subclasses without modifying the existing workflow. However, client-side selection or configuration may still need changes.

### Q10. Is Factory Method the same as Dependency Injection?

No. Factory Method delegates object creation to a factory method. Dependency Injection supplies dependencies from an external source. They can be used together.

### Q11. What is the main disadvantage of Factory Method?

It introduces extra classes and can make simple object creation unnecessarily complicated.

### Q12. What is the difference between Factory Method and Abstract Factory?

Factory Method delegates creation of a product to an overridable method. Abstract Factory provides an interface for creating families of related products.

### Q13. Can Factory Method be implemented in Kotlin?

Yes. Kotlin supports abstract classes, abstract functions, inheritance, and overriding, which are sufficient for the classic implementation.

### Q14. When should we avoid Factory Method?

Avoid it when object creation is simple, implementations are unlikely to change, and introducing a creator hierarchy provides little benefit.

## 12\. Quick Revision

- **Category:** Creational design pattern.
- **GoF status:** One of the original 23 GoF patterns.
- **Purpose:** Let subclasses decide which object to create.
- **Product:** Common interface.
- **Concrete Product:** Actual implementation.
- **Creator:** Parent class containing the factory method.
- **Concrete Creator:** Child class that creates a specific product.
- **Key mechanism:** Method overriding and polymorphism.
- **Main benefit:** Separates object creation decisions from the common workflow.
- **Main disadvantage:** Extra classes and complexity.

### One-Line Interview Answer

"The Factory Method Design Pattern defines a method for creating objects and lets subclasses decide which concrete implementation to instantiate, separating object-creation decisions from the common business workflow."

## 13\. Easy Way to Remember

Remember this sentence:

**The parent defines the creation method; the child decides what object to create.**

For example:

- `NotificationCreator` defines `createNotification()`.
- `EmailCreator` returns `EmailNotification`.
- `SmsCreator` returns `SmsNotification`.

That is the core of Factory Method.

## 14\. Related Patterns to Study Next

- **Simple Factory:** Centralizes object creation using selection logic.
- **Factory Method:** Delegates object creation to subclasses.
- **Abstract Factory:** Creates families of related objects.
- **Builder:** Constructs complex objects step by step.
- **Dependency Injection:** Supplies dependencies from an external source.
