
# Bridge Design Pattern — Java & Kotlin

An interview-focused guide to the Bridge Pattern, including practical examples, Java and Kotlin implementations, abstraction vs. implementation, comparisons, and common interview questions.

## Table of Contents

1. What Is the Bridge Pattern?
2. Why Do We Need It?
3. Java Implementation
4. Kotlin Implementation
5. How Bridge Works
6. When to Use Bridge
7. Bridge vs. Adapter vs. Strategy
8. Common Interview Questions
9. Quick Revision

# 1. What Is the Bridge Pattern?

The Bridge Pattern is a structural design pattern that separates an abstraction from its implementation so that both can vary independently.

In simple terms, it helps you avoid creating too many classes when a system has multiple independent dimensions of variation.

### Real-world example: Remote control and devices

Imagine you have different devices, such as a TV and a radio, and different types of remote controls.

Without the Bridge Pattern, you might create separate classes for every combination:

- TVRemote
- RadioRemote
- AdvancedTVRemote
- AdvancedRadioRemote

As more devices and remote types are introduced, the number of classes can grow rapidly.

With the Bridge Pattern, the remote control holds a reference to a device. The remote and the device can evolve independently.

Xiaomi wchodzi do Polski z telewizorami. Przetestowaliśmy pierwszy model. Znamy ceny \[TEST\] | Technologie na Next Gazeta

7 Best Portable Radios of 2025- Portable AM/FM Radio Reviews

Conceptually:

java
```
Remote remote = new BasicRemote(new TV());
remote.powerOn();

Remote radioRemote = new BasicRemote(new Radio());
radioRemote.powerOn();
```

The same `BasicRemote` works with different devices through a common interface.

Memory trick: Bridge = Separate abstraction from implementation so both can change independently.

# 2. Why Do We Need It?

Suppose you are building a notification system that supports different notification types and delivery channels.

Notification types:

- Urgent notification
- Normal notification

Delivery channels:

- Email
- SMS

## Without Bridge

You might create a separate class for every combination:

text
```
UrgentEmailNotification
UrgentSMSNotification
NormalEmailNotification
NormalSMSNotification
```

If you add push notifications, you need more classes. If you add additional notification types, the number grows further.

For $m$ notification types and $n$ delivery channels, you may end up maintaining approximately $m \times n$ combination classes.

## With Bridge

Separate the two dimensions:

- Abstraction: Notification type.
- Implementation: Delivery mechanism.

Then connect them through composition.

text
```
Notification
    |
    +---- UrgentNotification
    |
    +---- NormalNotification

MessageSender
    |
    +---- EmailSender
    |
    +---- SMSSender
```

A notification contains a reference to a message sender, so either side can be extended independently.

Important: Bridge is not simply about reducing the number of classes. Its main purpose is to separate two dimensions that need to evolve independently.

# 3. Java Implementation

Let's implement the Bridge Pattern using notifications.

## Step 1: Define the Implementation Interface

This interface defines how messages are delivered.

java
```
interface MessageSender {
    void sendMessage(String message);
}
```

This is the implementation hierarchy's common interface.

## Step 2: Create Concrete Implementations

### Email sender

java
```
class EmailSender implements MessageSender {

    @Override
    public void sendMessage(String message) {
        System.out.println("Email: " + message);
    }
}
```

### SMS sender

java
```
class SMSSender implements MessageSender {

    @Override
    public void sendMessage(String message) {
        System.out.println("SMS: " + message);
    }
}
```

Both classes implement the same interface but deliver messages differently.

## Step 3: Create the Abstraction

The `Notification` class holds a reference to the implementation.

java
```
abstract class Notification {

    protected final MessageSender sender;

    public Notification(MessageSender sender) {
        this.sender = sender;
    }

    public abstract void notifyUser(String message);
}
```

The important line is:

java
```
protected final MessageSender sender;
```

The notification does not create an email sender or SMS sender itself. It receives an implementation through its constructor.

This relationship is the bridge.

## Step 4: Create Refined Abstractions

### Urgent notification

java
```
class UrgentNotification extends Notification {

    public UrgentNotification(MessageSender sender) {
        super(sender);
    }

    @Override
    public void notifyUser(String message) {
        sender.sendMessage("[URGENT] " + message);
    }
}
```

### Normal notification

java
```
class NormalNotification extends Notification {

    public NormalNotification(MessageSender sender) {
        super(sender);
    }

    @Override
    public void notifyUser(String message) {
        sender.sendMessage(message);
    }
}
```

Each notification type controls its own behavior while delegating delivery to the selected sender.

## Step 5: Use the Bridge

java
```
public class Main {

    public static void main(String[] args) {

        MessageSender email = new EmailSender();
        MessageSender sms = new SMSSender();

        Notification urgentEmail =
            new UrgentNotification(email);

        Notification urgentSMS =
            new UrgentNotification(sms);

        Notification normalEmail =
            new NormalNotification(email);

        urgentEmail.notifyUser("Server is down!");
        urgentSMS.notifyUser("Server is down!");
        normalEmail.notifyUser("Daily report is ready.");
    }
}
```

Output:

text
```
Email: [URGENT] Server is down!
SMS: [URGENT] Server is down!
Email: Daily report is ready.
```

### What happens?

- `Notification` defines the abstraction.
- `UrgentNotification` and `NormalNotification` refine the abstraction.
- `MessageSender` defines the implementation interface.
- `EmailSender` and `SMSSender` provide concrete implementations.
- Constructor injection connects the two hierarchies.

The same urgent notification can use email or SMS without creating a new notification class for each combination.

# 4. Kotlin Implementation

Kotlin makes the Bridge Pattern concise through interfaces, abstract classes, and constructor injection.

## Implementation

kotlin
```
// Implementation interface
interface MessageSender {
    fun sendMessage(message: String)
}

// Concrete implementations
class EmailSender : MessageSender {
    override fun sendMessage(message: String) {
        println("Email: $message")
    }
}

class SMSSender : MessageSender {
    override fun sendMessage(message: String) {
        println("SMS: $message")
    }
}

// Abstraction
abstract class Notification(
    protected val sender: MessageSender
) {
    abstract fun notifyUser(message: String)
}

// Refined abstraction
class UrgentNotification(
    sender: MessageSender
) : Notification(sender) {

    override fun notifyUser(message: String) {
        sender.sendMessage("[URGENT] $message")
    }
}

class NormalNotification(
    sender: MessageSender
) : Notification(sender) {

    override fun notifyUser(message: String) {
        sender.sendMessage(message)
    }
}

// Client
fun main() {

    val email = EmailSender()
    val sms = SMSSender()

    val urgentEmail = UrgentNotification(email)
    val urgentSMS = UrgentNotification(sms)
    val normalEmail = NormalNotification(email)

    urgentEmail.notifyUser("Server is down!")
    urgentSMS.notifyUser("Server is down!")
    normalEmail.notifyUser("Daily report is ready.")
}
```

Output:

text
```
Email: [URGENT] Server is down!
SMS: [URGENT] Server is down!
Email: Daily report is ready.
```

## Why is this a Bridge?

kotlin
```
abstract class Notification(
    protected val sender: MessageSender
)
```

`Notification` depends on the `MessageSender` interface instead of a concrete implementation.

You can introduce `PushSender` without modifying the existing notification classes. Likewise, you can introduce `ReminderNotification` without changing the sender classes.

Interview note: Composition is central to the Bridge Pattern, but composition alone does not automatically make a design a Bridge. The key is that the abstraction and implementation are separate, independently extensible hierarchies.

# 5. How Bridge Works

The Bridge Pattern has four main roles.

| Role | Responsibility | Example |
| --- | --- | --- |
| Abstraction | Defines the high-level interface | `Notification` |
| Refined Abstraction | Extends high-level behavior | `UrgentNotification` |
| Implementor | Defines the implementation interface | `MessageSender` |
| Concrete Implementor | Provides actual implementation | `EmailSender`, `SMSSender` |

## Class relationship

text
```
          Abstraction
          Notification
               |
               | has-a
               v
          MessageSender
           /         \
          /           \
    EmailSender     SMSSender

Notification
     /       \
    /         \
Urgent      Normal
Notification Notification
```

There are two independent hierarchies:

1. The notification hierarchy determines what kind of notification is sent.
2. The sender hierarchy determines how the notification is delivered.

The bridge is the reference from `Notification` to `MessageSender`.

## How the client uses it

java
```
Notification notification =
    new UrgentNotification(new EmailSender());

notification.notifyUser("Server is down!");
```

The client chooses the abstraction and its implementation separately.

This is what allows both hierarchies to evolve independently.

# 6. When to Use Bridge

Use Bridge when:

- A class has multiple independent dimensions of variation.
- You want to avoid a large number of combination subclasses.
- Abstraction and implementation need to evolve independently.
- You want to switch implementations without changing the abstraction.
- You need to isolate platform-specific or vendor-specific implementations behind a stable interface.

### Common real-world examples

- Notifications: Notification type × delivery channel.
- Graphics: Shape × rendering engine.
- Remote controls: Remote type × electronic device.
- Database access: High-level repository abstraction × database implementation.
- Cross-platform applications: Application features × operating-system-specific services.

### When should you avoid it?

Avoid Bridge when the system is simple and there is only one implementation that is unlikely to vary.

Introducing two hierarchies and additional abstractions can make a small application unnecessarily complicated.

Main benefit: Bridge allows abstraction and implementation to vary independently without creating a subclass for every combination.

# 7. Bridge vs. Adapter vs. Strategy

These patterns can look similar because they often use interfaces and composition. Their intentions are different.

| Feature | Bridge | Adapter | Strategy |
| --- | --- | --- | --- |
| Main purpose | Separate abstraction and implementation | Make incompatible interfaces work together | Make an algorithm interchangeable |
| Primary concern | Independent dimensions of variation | Interface compatibility | Algorithm selection |
| Typical structure | Two related hierarchies | Target interface plus adapter | Context plus strategy interface |
| When selected | Usually designed into the architecture | Often introduced for existing code | Often selected to change behavior |
| Example | Notification type × sender | Legacy sender × modern interface | Different pricing algorithms |

### Easy way to remember

- Bridge: Separate two dimensions so both can evolve independently.
- Adapter: Make an existing interface compatible with the expected interface.
- Strategy: Make different algorithms interchangeable.

### Can Bridge and Strategy look similar?

Yes. Both may use composition and interfaces.

The difference is their intent:

- Bridge separates an abstraction from its implementation across two dimensions.
- Strategy encapsulates interchangeable algorithms or behaviors.

A Bridge implementation might use a Strategy-like interface internally, but that does not make their design goals identical.

# 8. Common Interview Questions

### Q1. What is the Bridge Pattern?

A structural design pattern that separates an abstraction from its implementation so that both can evolve independently.

### Q2. Why do we need Bridge?

It avoids tightly coupling two independent dimensions of a design and reduces the need for classes representing every possible combination.

### Q3. What are the four components of Bridge?

- Abstraction
- Refined Abstraction
- Implementor
- Concrete Implementor

### Q4. What is the difference between abstraction and implementation?

Abstraction represents the high-level functionality the client uses. Implementation provides the lower-level behavior used to carry out that functionality.

In the example, notification types are abstractions, while email and SMS delivery are implementations.

### Q5. Is Bridge based on inheritance or composition?

Bridge uses both where appropriate. Inheritance is used to extend the abstraction hierarchy, while composition connects an abstraction to an implementation.

### Q6. How does Bridge help follow SOLID principles?

It can support the Open/Closed Principle by allowing new notification types and sender implementations to be added independently. It can also support Dependency Inversion by making the abstraction depend on an implementation interface rather than concrete senders.

The actual design still needs to respect the relevant principles.

### Q7. What is the difference between Bridge and Adapter?

Bridge separates two dimensions of a design, often planned from the beginning. Adapter reconciles interfaces that are already incompatible, often to reuse existing or third-party code.

### Q8. What is the difference between Bridge and Strategy?

Bridge separates an abstraction from its implementation. Strategy makes alternative algorithms interchangeable. They may have similar structures but solve different design problems.

### Q9. Does Bridge reduce the total number of classes?

Not necessarily. Bridge can introduce additional interfaces and classes. Its main advantage is avoiding tightly coupled combinations and allowing independent extension.

### Q10. Can Bridge be implemented in Kotlin?

Yes. Kotlin interfaces, abstract classes, and constructor injection support Bridge naturally. The pattern does not require any special language feature.

### Q11. Is Bridge useful when there is only one implementation?

It can be, if additional implementations are expected or independent evolution is important. Otherwise, it may introduce unnecessary complexity.

### Q12. What is the biggest advantage of Bridge?

The abstraction and implementation can change independently without requiring changes across every combination of their subclasses.

# 9. Quick Revision

- Category: Structural design pattern.
- Purpose: Separate abstraction from implementation.
- Abstraction: High-level functionality exposed to the client.
- Refined Abstraction: Specialized abstraction.
- Implementor: Interface for lower-level behavior.
- Concrete Implementor: Actual implementation of that behavior.
- Java: Use interfaces, abstract classes, and composition.
- Kotlin: Use interfaces, abstract classes, and constructor injection.
- Main benefit: Independent evolution of two dimensions.
- Main limitation: Can introduce extra abstractions and complexity.

## One-Line Memory Trick

Bridge = Connect two independent hierarchies through composition so each can change without changing the other.
