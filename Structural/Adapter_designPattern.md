# Adapter Design Pattern — Java & Kotlin

An interview-focused guide to the Adapter Pattern, including practical examples, Java and Kotlin implementations, object adapters vs. class adapters, comparisons, and common interview questions.

## Table of Contents

1. What Is the Adapter Pattern?
2. Why Do We Need It?
3. Java Implementation
4. Kotlin Implementation
5. Object Adapter vs. Class Adapter
6. How Adapter Works
7. When to Use Adapter
8. Adapter vs. Decorator vs. Facade
9. Common Interview Questions
10. Quick Revision

# 1. What Is the Adapter Pattern?

The Adapter Pattern is a structural design pattern that allows two incompatible interfaces to work together by converting the interface of an existing class into the interface expected by the client.

### Real-world example: Power adapter

Imagine you are traveling to another country. Your laptop charger has a plug that does not fit the available wall socket.

Instead of changing the charger or the socket, you use a power adapter that allows the two incompatible interfaces to work together.



The same idea applies in software.

Suppose your application expects a `PaymentProcessor`, but an external payment library provides a `ThirdPartyPaymentGateway` with a different method.

An adapter translates between them:

java
```
PaymentProcessor processor =
    new PaymentGatewayAdapter(new ThirdPartyPaymentGateway());

processor.pay(1000);
```

The application uses the interface it already understands, while the adapter handles communication with the incompatible library.

Memory trick: Adapter = Make an incompatible interface compatible.

# 2. Why Do We Need It?

Suppose your application expects this interface:

java
```
interface PaymentProcessor {
    void pay(double amount);
}
```

Your application calls it like this:

java
```
PaymentProcessor processor = getPaymentProcessor();
processor.pay(1000);
```

Now imagine you integrate a third-party payment library:

java
```
class ThirdPartyPaymentGateway {

    public void makePayment(double amountInRupees) {
        System.out.println(
            "Payment processed: ₹" + amountInRupees
        );
    }
}
```

The problem is that the two interfaces do not match:

- Your application expects `pay()`.
- The third-party library provides `makePayment()`.

You cannot directly use the third-party class where a `PaymentProcessor` is required.

## Without Adapter

You might change the existing application to use the third-party API directly:

java
```
ThirdPartyPaymentGateway gateway =
    new ThirdPartyPaymentGateway();

gateway.makePayment(1000);
```

This can introduce tight coupling between your application and the external library.

If the external API changes, your application code may also need to change.

## With Adapter

Create an adapter that implements your application's expected interface:

java
```
class PaymentGatewayAdapter implements PaymentProcessor {

    private final ThirdPartyPaymentGateway gateway;

    public PaymentGatewayAdapter(
        ThirdPartyPaymentGateway gateway
    ) {
        this.gateway = gateway;
    }

    @Override
    public void pay(double amount) {
        gateway.makePayment(amount);
    }
}
```

Now the application can continue using `PaymentProcessor` without knowing the details of the third-party API.

Important: Adapter does not necessarily modify the incompatible class. It wraps or extends it and translates the expected operations into the operations the existing class supports.

# 3. Java Implementation

We will build a complete example using a payment processor.

## Step 1: Define the Target Interface

The Target is the interface expected by the client.

java
```
interface PaymentProcessor {
    void pay(double amount);
}
```

The application depends on this interface rather than on a specific payment provider.

## Step 2: Create the Adaptee

The Adaptee is the existing class whose interface is incompatible with the target interface.

java
```
class ThirdPartyPaymentGateway {

    public void makePayment(double amountInRupees) {
        System.out.println(
            "Payment processed: ₹" + amountInRupees
        );
    }
}
```

Assume this class comes from an external library, so we cannot or do not want to modify it.

Notice that its method is named `makePayment()`, not `pay()`.

## Step 3: Create the Adapter

The Adapter implements the target interface and delegates the work to the adaptee.

java
```
class PaymentGatewayAdapter implements PaymentProcessor {

    private final ThirdPartyPaymentGateway gateway;

    public PaymentGatewayAdapter(
        ThirdPartyPaymentGateway gateway
    ) {
        this.gateway = gateway;
    }

    @Override
    public void pay(double amount) {
        gateway.makePayment(amount);
    }
}
```

### What happens inside the adapter?

java
```
@Override
public void pay(double amount) {
    gateway.makePayment(amount);
}
```

- The client calls `pay()`.
- The adapter receives the request.
- The adapter translates it into a call to `makePayment()`.
- The third-party gateway processes the payment.

The adapter may also translate data formats, units, parameter structures, or error handling when necessary.

## Step 4: Use the Adapter

java
```
public class Main {

    public static void main(String[] args) {

        ThirdPartyPaymentGateway gateway =
            new ThirdPartyPaymentGateway();

        PaymentProcessor processor =
            new PaymentGatewayAdapter(gateway);

        processor.pay(1000);
    }
}
```

Output:

text
```
Payment processed: ₹1000.0
```

### How the objects interact

Client

Calls `pay(1000)`

PaymentProcessor

Target interface

PaymentGatewayAdapter

Translates `pay()` into `makePayment()`

ThirdPartyPaymentGateway

Adaptee — performs the operation

The client knows only about `PaymentProcessor`. It does not need to know how the third-party gateway works.

# 4. Kotlin Implementation

Kotlin supports the Adapter Pattern naturally through interfaces, delegation, and composition.

We will implement the same payment example.

## Implementation

kotlin
```
// Target interface
interface PaymentProcessor {
    fun pay(amount: Double)
}

// Adaptee
class ThirdPartyPaymentGateway {

    fun makePayment(amountInRupees: Double) {
        println("Payment processed: ₹$amountInRupees")
    }
}

// Adapter
class PaymentGatewayAdapter(
    private val gateway: ThirdPartyPaymentGateway
) : PaymentProcessor {

    override fun pay(amount: Double) {
        gateway.makePayment(amount)
    }
}

// Client
fun main() {

    val gateway = ThirdPartyPaymentGateway()

    val processor: PaymentProcessor =
        PaymentGatewayAdapter(gateway)

    processor.pay(1000.0)
}
```

Output:

text
```
Payment processed: ₹1000.0
```

## Why does this work?

kotlin
```
class PaymentGatewayAdapter(
    private val gateway: ThirdPartyPaymentGateway
) : PaymentProcessor
```

- `PaymentProcessor` is the target interface.
- `ThirdPartyPaymentGateway` is the adaptee.
- `PaymentGatewayAdapter` connects the two.
- `gateway` holds a reference to the existing implementation.
- `pay()` delegates to `makePayment()`.

### Kotlin alternative: Using interface delegation

Kotlin's `by` keyword can delegate interface implementation to another object. However, delegation alone does not solve incompatible method signatures.

For example, if an existing object already implements the required interface, Kotlin can delegate its implementation:

kotlin
```
interface Logger {
    fun log(message: String)
}

class ConsoleLogger : Logger {
    override fun log(message: String) {
        println(message)
    }
}

class LoggingService(
    logger: Logger
) : Logger by logger
```

This is language-level delegation. It is not necessarily an Adapter Pattern because no incompatible interface is being translated.

For a genuine adapter, you still need translation logic such as:

kotlin
```
override fun pay(amount: Double) {
    gateway.makePayment(amount)
}
```

Interview note: Kotlin's delegation feature can simplify implementations, but using `by` does not automatically make a class an adapter.

# 5. Object Adapter vs. Class Adapter

There are two commonly discussed ways to implement the Adapter Pattern.

## A. Object Adapter

An object adapter uses composition: the adapter contains a reference to the adaptee and delegates requests to it.

java
```
class PaymentGatewayAdapter implements PaymentProcessor {

    private final ThirdPartyPaymentGateway gateway;

    public PaymentGatewayAdapter(
        ThirdPartyPaymentGateway gateway
    ) {
        this.gateway = gateway;
    }

    @Override
    public void pay(double amount) {
        gateway.makePayment(amount);
    }
}
```

Relationship:

text
```
PaymentGatewayAdapter
          |
          | has-a
          v
ThirdPartyPaymentGateway
```

Advantages:

- Works with existing objects.
- Does not require subclassing the adaptee.
- Can adapt different implementations through constructor injection.
- Fits naturally with dependency injection.

This is generally the preferred approach in modern Java and Kotlin applications.

## B. Class Adapter

A class adapter uses inheritance to adapt an existing class to the required interface.

In Java, it can be implemented when the adaptee is a class that can be extended:

java
```
class PaymentGatewayClassAdapter
        extends ThirdPartyPaymentGateway
        implements PaymentProcessor {

    @Override
    public void pay(double amount) {
        makePayment(amount);
    }
}
```

Usage:

java
```
PaymentProcessor processor =
    new PaymentGatewayClassAdapter();

processor.pay(1000);
```

Relationship:

text
```
PaymentGatewayClassAdapter
       /             \
   extends          implements
      /                 \
ThirdPartyGateway   PaymentProcessor
```

Advantages:

- Can directly access inherited methods.
- May avoid storing a separate adaptee reference.

Limitations:

- Java classes can extend only one class.
- The adapter is more tightly coupled to the adaptee's implementation.
- It cannot easily adapt an already-created adaptee instance.

### Comparison

| Feature | Object Adapter | Class Adapter |
| --- | --- | --- |
| Mechanism | Composition | Inheritance |
| Relationship | Has-a | Is-a |
| Reuses an existing instance | Yes | Generally no |
| Java multiple inheritance of classes required | No | No, but the adaptee must be extendable |
| Flexibility | Higher | Lower |
| Typical preference | Modern applications | Specific inheritance-based cases |

Interview tip: If asked which is preferred, say that the object adapter is generally more flexible because it uses composition instead of inheriting from the adaptee.

# 6. How Adapter Works

The Adapter Pattern typically involves four roles.

| Role | Responsibility | Example |
| --- | --- | --- |
| Client | Uses the expected interface | Application |
| Target | Defines the interface the client expects | `PaymentProcessor` |
| Adapter | Translates calls between interfaces | `PaymentGatewayAdapter` |
| Adaptee | Provides the existing incompatible behavior | `ThirdPartyPaymentGateway` |

The interaction follows this sequence:

1. Client requests an operation

2. Adapter receives the target-interface call

3. Adapter translates the request

4. Adaptee performs the operation

5. Result is returned through the adapter, if applicable

## Does Adapter always change method names?

No. Method-name translation is only one example.

An adapter can also:

- Convert Celsius to Fahrenheit.
- Convert XML data to JSON.
- Map one set of method parameters to another.
- Convert a legacy API into a modern interface.
- Translate third-party exceptions into application-specific exceptions.
- Adapt a synchronous interface to an asynchronous one, when the behavior and lifecycle can be handled correctly.

The key requirement is that the adapter makes an existing implementation usable through the interface the client expects.

# 7. When to Use Adapter

Use the Adapter Pattern when:

- You need to integrate a third-party library with an incompatible interface.
- You need to reuse legacy code without modifying it.
- You want your application to depend on its own stable interface rather than a vendor-specific API.
- You need to translate between different data formats or units.
- You want to replace an external implementation with another implementation while minimizing changes to client code.

### Common real-world examples

- Payment integration: Adapt different payment providers to one internal payment interface.
- Logging: Adapt an external logging library to your application's logging interface.
- Legacy systems: Wrap an older service so newer code can use it.
- Data formats: Convert a legacy XML-based API into an interface that exposes application-friendly objects.
- Hardware integration: Adapt different device APIs to a common interface.

### When should you avoid it?

Avoid adding an adapter when the existing interface already meets the client's needs and a wrapper provides no meaningful benefit.

Also, do not use an adapter to hide major behavioral incompatibilities that cannot be safely translated. Differences in transaction semantics, error handling, threading, or resource ownership may require more than simple method mapping.

Main benefit: Adapter allows incompatible components to collaborate without forcing the client to depend directly on the adaptee.

# 8. Adapter vs. Decorator vs. Facade

These three structural design patterns can look similar because they often wrap existing objects. Their intentions are different.

| Feature | Adapter | Decorator | Facade |
| --- | --- | --- | --- |
| Main purpose | Make incompatible interfaces work together | Add or modify responsibilities | Simplify a complex subsystem |
| Primary focus | Interface compatibility | Additional behavior | Easier access |
| Interface exposed | Client's expected interface | Usually the same interface | A simplified interface |
| Typical example | Wrap a legacy payment API | Add logging or caching around a service | Simplify an order-processing subsystem |
| Changes functionality? | Translates operations as needed | Adds behavior | Coordinates existing functionality |

## Adapter example

java
```
processor.pay(1000);
```

The adapter translates `pay()` into the method understood by the third-party gateway.

## Decorator example

java
```
PaymentProcessor processor =
    new LoggingPaymentProcessor(realProcessor);
```

The decorator adds logging while preserving the `PaymentProcessor` interface.

## Facade example

java
```
orderService.placeOrder(order);
```

The facade may coordinate inventory, payment, and shipping services behind one simpler operation.

### Easy way to remember

- Adapter: Change the interface.
- Decorator: Add behavior.
- Facade: Simplify usage.

Important: These patterns can be combined. For example, a decorator could add logging around an adapter.

# 9. Common Interview Questions

### Q1. What is the Adapter Pattern?

The Adapter Pattern is a structural design pattern that converts an existing class's interface into an interface expected by the client, allowing incompatible components to work together.

### Q2. Why do we need the Adapter Pattern?

It enables reuse of existing or third-party code without requiring changes to the client or the incompatible class. It also reduces direct coupling to external interfaces.

### Q3. What are the four main components of the Adapter Pattern?

- Client: Uses the target interface.
- Target: The interface expected by the client.
- Adapter: Translates requests between interfaces.
- Adaptee: The existing class providing the incompatible interface.

### Q4. What is the difference between an adapter and a wrapper?

A wrapper is a general term for an object that contains or surrounds another object. An adapter is a wrapper whose specific purpose is to reconcile incompatible interfaces.

A decorator is also a kind of wrapper, but its primary purpose is to add behavior rather than change interface compatibility.

### Q5. What is the difference between object adapter and class adapter?

An object adapter uses composition to hold an adaptee instance. A class adapter uses inheritance to extend the adaptee while implementing the target interface.

Object adapters are usually more flexible because they can work with different instances without inheriting from the adaptee.

### Q6. Does Adapter follow SOLID principles?

It can support several SOLID principles:

- Single Responsibility Principle: The adapter handles interface translation.
- Open/Closed Principle: New adapters can support additional integrations without modifying existing client code.
- Dependency Inversion Principle: Clients can depend on a stable target interface instead of a concrete third-party implementation.

Whether the overall design follows SOLID depends on the implementation, not merely on using the pattern.

### Q7. Is Adapter a creational, structural, or behavioral pattern?

Adapter is a structural design pattern because it focuses on how existing classes or objects are connected.

### Q8. Can Adapter work with third-party libraries?

Yes. This is one of its most common uses. An adapter can wrap a third-party library and expose an interface defined by your application.

### Q9. What is the difference between Adapter and Facade?

Adapter makes an existing interface compatible with the interface the client expects. Facade provides a simpler interface to a complex subsystem.

Adapter focuses on compatibility; Facade focuses on simplicity.

### Q10. What is the difference between Adapter and Decorator?

Adapter changes the interface presented to the client so incompatible components can work together. Decorator generally preserves the component's interface while adding responsibilities, such as caching, logging, or validation.

### Q11. Can an adapter convert data as well as method calls?

Yes. An adapter may transform parameter types, units, data formats, or return values in addition to delegating method calls.

### Q12. What are the disadvantages of Adapter?

- Adds additional classes and indirection.
- Can make debugging more complicated.
- May hide important differences in behavior if designed poorly.
- Can become difficult to maintain when the external API changes frequently.
- Does not automatically solve incompatible business semantics or lifecycle requirements.

# 10. Quick Revision

- Category: Structural design pattern.
- Purpose: Make incompatible interfaces work together.
- Target: Interface expected by the client.
- Adaptee: Existing class with an incompatible interface.
- Adapter: Connects the target interface to the adaptee.
- Object adapter: Uses composition.
- Class adapter: Uses inheritance.
- Java: Commonly implemented with `implements` and constructor injection.
- Kotlin: Commonly implemented with interfaces, composition, and delegation.
- Main benefit: Reuse existing code without changing its interface or coupling the client directly to it.
- Main limitation: Adds indirection and cannot automatically reconcile fundamentally incompatible behavior.

## One-Line Memory Trick

Adapter = Convert the interface the client expects into the interface an existing object provides.
