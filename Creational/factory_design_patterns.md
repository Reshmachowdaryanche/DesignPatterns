

# Factory Design Patterns — Java & Kotlin

A practical guide to Simple Factory, Factory Method, and Abstract Factory, with examples, diagrams, comparisons, and interview notes.

## Table of Contents

1. Factory Patterns: The Core Idea
2. Simple Factory
3. Factory Method
4. Abstract Factory
5. Comparison of All Three
6. Common Interview Questions
7. Quick Revision

---

# 1\. Factory Patterns: The Core Idea

Factory patterns separate **object creation** from **object usage**.

Imagine a payment application supporting Razorpay and Stripe.

Without a factory:

```
Payment payment = new RazorpayPayment();
payment.pay(1000);
```

The client directly chooses the concrete implementation.

With a factory:

```
Payment payment = PaymentFactory.create("razorpay");
payment.pay(1000);
```

The factory handles the creation decision, and the client uses the returned object through the `Payment` interface.

Three related approaches are:

- **Simple Factory:** One factory selects a concrete object.
- **Factory Method:** Concrete creator subclasses decide which object to create.
- **Abstract Factory:** A factory creates a family of related objects.

All three are creational patterns or idioms, but they solve different object-creation problems.

# 2\. Simple Factory Pattern

## Problem

A payment SDK supports multiple providers. We want the client to select a provider without directly constructing its implementation.

## Java implementation

```
interface Payment {
    void pay(double amount);
}

class RazorpayPayment implements Payment {
    public void pay(double amount) {
        System.out.println("Razorpay: " + amount);
    }
}

class StripePayment implements Payment {
    public void pay(double amount) {
        System.out.println("Stripe: " + amount);
    }
}

class PaymentFactory {
    public static Payment create(String type) {
        if (type.equalsIgnoreCase("razorpay")) {
            return new RazorpayPayment();
        }

        if (type.equalsIgnoreCase("stripe")) {
            return new StripePayment();
        }

        throw new IllegalArgumentException(
            "Unknown provider: " + type
        );
    }
}

class Main {
    public static void main(String[] args) {
        Payment payment = PaymentFactory.create("stripe");
        payment.pay(1000);
    }
}
```

## Kotlin implementation

```
interface Payment {
    fun pay(amount: Double)
}

class RazorpayPayment : Payment {
    override fun pay(amount: Double) {
        println("Razorpay: $amount")
    }
}

class StripePayment : Payment {
    override fun pay(amount: Double) {
        println("Stripe: $amount")
    }
}

object PaymentFactory {
    fun create(type: String): Payment =
        when (type.lowercase()) {
            "razorpay" -> RazorpayPayment()
            "stripe" -> StripePayment()
            else -> throw IllegalArgumentException(
                "Unknown provider: $type"
            )
        }
}

fun main() {
    val payment = PaymentFactory.create("stripe")
    payment.pay(1000.0)
}
```

## How it works

1. The client passes a provider identifier.
2. The factory selects and constructs the implementation.
3. The factory returns a `Payment` reference.
4. The client calls `pay()` without knowing the concrete class.

## Advantages and limitations

- Simple to implement and understand.
- Centralizes object-creation logic.
- Adding a provider usually requires modifying the factory's selection logic.
- Suitable for a small, stable set of implementation choices.

**Memory trick:** One factory, one selection method.

**Classification:** Simple Factory is a common programming idiom, not one of the original 23 GoF patterns.

---

# 3\. Factory Method Pattern

## Problem

Suppose different payment providers need different implementations, but we want to reuse a common payment-processing workflow.

Instead of one factory selecting a provider, each concrete factory decides which payment object to create.

## Java implementation

```
interface Payment {
    void pay(double amount);
}

class RazorpayPayment implements Payment {
    public void pay(double amount) {
        System.out.println("Razorpay: " + amount);
    }
}

class StripePayment implements Payment {
    public void pay(double amount) {
        System.out.println("Stripe: " + amount);
    }
}

abstract class PaymentFactory {

    protected abstract Payment createPayment();

    public void processPayment(double amount) {
        Payment payment = createPayment();
        payment.pay(amount);
    }
}

class RazorpayFactory extends PaymentFactory {
    @Override
    protected Payment createPayment() {
        return new RazorpayPayment();
    }
}

class StripeFactory extends PaymentFactory {
    @Override
    protected Payment createPayment() {
        return new StripePayment();
    }
}

class Main {
    public static void main(String[] args) {
        PaymentFactory factory = new StripeFactory();
        factory.processPayment(1000);
    }
}
```

## Kotlin implementation

```
interface Payment {
    fun pay(amount: Double)
}

class RazorpayPayment : Payment {
    override fun pay(amount: Double) {
        println("Razorpay: $amount")
    }
}

class StripePayment : Payment {
    override fun pay(amount: Double) {
        println("Stripe: $amount")
    }
}

abstract class PaymentFactory {
    protected abstract fun createPayment(): Payment

    fun processPayment(amount: Double) {
        val payment = createPayment()
        payment.pay(amount)
    }
}

class RazorpayFactory : PaymentFactory() {
    override fun createPayment(): Payment =
        RazorpayPayment()
}

class StripeFactory : PaymentFactory() {
    override fun createPayment(): Payment =
        StripePayment()
}

fun main() {
    val factory: PaymentFactory = StripeFactory()
    factory.processPayment(1000.0)
}
```

## How it works

1. The client chooses a concrete factory.
2. The client calls `processPayment()`.
3. The shared workflow invokes `createPayment()`.
4. Runtime polymorphism selects the concrete factory's overridden method.
5. The returned payment object performs the payment.

The key difference is that **the subclass decides what to create, while the parent class can define what to do with the created object**.

## Advantages and limitations

- Supports adding new concrete creators without modifying existing creators.
- Allows shared workflows to be implemented once.
- Introduces more classes and inheritance.
- The application still needs a way to select the concrete factory.

**Memory trick:** Subclasses decide object creation through overriding.

**Classification:** Factory Method is one of the original 23 GoF design patterns.

---

# 4\. Abstract Factory Pattern

## 4.1 What problem does it solve?

Simple Factory and Factory Method typically focus on creating one product at a time.

But what if an application needs a **family of related objects that must work together**?

Imagine a cross-platform UI application supporting Android-style and iOS-style components.

Each platform needs a matching set of components:

- A button.
- A checkbox.

We want Android screens to use Android components and iOS screens to use iOS components.

Creating each component independently could accidentally mix styles. Abstract Factory lets us select a family of compatible products through one factory.

## 4.2 Class structure

```
                 UIFactory
              /             \
             v               v
    AndroidUIFactory     IOSUIFactory
       /       \           /       \
      v         v         v         v
 Android     Android     IOS       IOS
 Button      Checkbox   Button    Checkbox
```

The abstract factory declares methods for creating multiple product types. Each concrete factory creates a matching family.

## 4.3 Java implementation

### Step 1: Define product interfaces

```
interface Button {
    void render();
}

interface Checkbox {
    void render();
}
```

These interfaces represent two different product types.

### Step 2: Create concrete products

```
class AndroidButton implements Button {
    public void render() {
        System.out.println("Android button");
    }
}

class AndroidCheckbox implements Checkbox {
    public void render() {
        System.out.println("Android checkbox");
    }
}

class IOSButton implements Button {
    public void render() {
        System.out.println("iOS button");
    }
}

class IOSCheckbox implements Checkbox {
    public void render() {
        System.out.println("iOS checkbox");
    }
}
```

### Step 3: Define the abstract factory

```
interface UIFactory {
    Button createButton();
    Checkbox createCheckbox();
}
```

Notice that the factory creates two different product types.

### Step 4: Implement concrete factories

```
class AndroidUIFactory implements UIFactory {

    public Button createButton() {
        return new AndroidButton();
    }

    public Checkbox createCheckbox() {
        return new AndroidCheckbox();
    }
}

class IOSUIFactory implements UIFactory {

    public Button createButton() {
        return new IOSButton();
    }

    public Checkbox createCheckbox() {
        return new IOSCheckbox();
    }
}
```

Each factory creates a consistent family of products.

### Step 5: Use the factory

```
class Application {

    private final Button button;
    private final Checkbox checkbox;

    Application(UIFactory factory) {
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }

    void renderUI() {
        button.render();
        checkbox.render();
    }
}

class Main {
    public static void main(String[] args) {
        UIFactory factory = new AndroidUIFactory();

        Application app = new Application(factory);
        app.renderUI();
    }
}
```

Output:

```
Android button
Android checkbox
```

To switch the application to the iOS product family, change the factory:

```
UIFactory factory = new IOSUIFactory();
```

The application code can remain unchanged.

## 4.4 Kotlin implementation

```
interface Button {
    fun render()
}

interface Checkbox {
    fun render()
}

class AndroidButton : Button {
    override fun render() {
        println("Android button")
    }
}

class AndroidCheckbox : Checkbox {
    override fun render() {
        println("Android checkbox")
    }
}

class IOSButton : Button {
    override fun render() {
        println("iOS button")
    }
}

class IOSCheckbox : Checkbox {
    override fun render() {
        println("iOS checkbox")
    }
}

interface UIFactory {
    fun createButton(): Button
    fun createCheckbox(): Checkbox
}

class AndroidUIFactory : UIFactory {
    override fun createButton(): Button =
        AndroidButton()

    override fun createCheckbox(): Checkbox =
        AndroidCheckbox()
}

class IOSUIFactory : UIFactory {
    override fun createButton(): Button =
        IOSButton()

    override fun createCheckbox(): Checkbox =
        IOSCheckbox()
}

class Application(factory: UIFactory) {
    private val button = factory.createButton()
    private val checkbox = factory.createCheckbox()

    fun renderUI() {
        button.render()
        checkbox.render()
    }
}

fun main() {
    val app = Application(AndroidUIFactory())
    app.renderUI()
}
```

## 4.5 How it works

1. The client selects a family factory, such as `AndroidUIFactory`.
2. The application asks that factory to create a button and a checkbox.
3. The factory returns Android-specific implementations through the product interfaces.
4. The application uses those objects without knowing their concrete classes.
5. Changing the factory selects a different product family.

## 4.6 Advantages and limitations

- Keeps related products consistent.
- Allows an entire product family to be switched through one factory.
- Reduces dependence on concrete product classes.
- Adding a new family is straightforward when the product interfaces remain unchanged.
- Adding a new product type, such as a dialog, requires updating the abstract factory and its concrete implementations.

**Memory trick:** One factory creates a family of related products.

**Classification:** Abstract Factory is one of the original 23 GoF design patterns.

---

# 5\. Comparison of All Three Patterns

| Feature | Simple Factory | Factory Method | Abstract Factory |
| --- | --- | --- | --- |
| Main purpose | Select one product | Delegate creation to subclasses | Create related product families |
| Creation decision | Centralized selection logic | Overridden factory method | Concrete family factory |
| Typical structure | One factory | Creator hierarchy | Factory interface and implementations |
| Products created | Usually one product per call | Usually one product per factory method | Multiple related product types |
| Main mechanism | Conditional or lookup | Inheritance and polymorphism | Composition and product interfaces |
| Adding a new implementation | Modify selection logic | Add a concrete creator and product | Add a concrete family factory and products |
| Adding a new product type | Often modify factory API | Depends on the design | Modify factory interface and all implementations |
| Complexity | Low | Medium | Medium to high |

## The easiest way to distinguish them

### Simple Factory

```
PaymentFactory.create("stripe");
```

**Question answered:** Which implementation should I create?

### Factory Method

```
new StripeFactory().processPayment(1000);
```

**Question answered:** Which implementation should this concrete creator instantiate?

### Abstract Factory

```
UIFactory factory = new AndroidUIFactory();

Button button = factory.createButton();
Checkbox checkbox = factory.createCheckbox();
```

**Question answered:** Which compatible family of products should I create?

---

# 6\. Common Interview Questions

### Q1. Why use factory patterns?

To separate object creation from object usage, centralize or delegate construction logic, and reduce unnecessary dependencies on concrete classes.

### Q2. Does a factory eliminate `new`?

No. Factories often use constructors internally. They hide or organize object-creation details.

### Q3. Is Simple Factory a GoF pattern?

No. It is a common idiom. Factory Method and Abstract Factory are among the original 23 GoF patterns.

### Q4. What is the main difference between Factory Method and Abstract Factory?

Factory Method focuses on a creation method whose implementation can vary by subclass.

Abstract Factory provides an interface for creating a family of related products.

### Q5. Which pattern is best?

There is no universally best pattern.

- Choose Simple Factory for straightforward selection among a small number of implementations.
- Choose Factory Method when creation behavior varies across subclasses and a shared workflow is useful.
- Choose Abstract Factory when several related products must be created as a compatible family.

### Q6. How do factories relate to SOLID?

They can support the Dependency Inversion Principle by allowing clients to work through abstractions. Factory Method and Abstract Factory can also support the Open/Closed Principle by allowing new implementations or families to be added.

However, the complete design must still be evaluated. A factory alone does not guarantee compliance with SOLID.

---

# 7\. Quick Revision

- **Simple Factory:** One factory selects a concrete object.
- **Factory Method:** Concrete creator subclasses override a method to choose the product.
- **Abstract Factory:** One factory creates a family of related products.
- **Common benefit:** Separate object creation from object usage.
- **Common technique:** Return interface types rather than concrete types when possible.
- **Common trade-off:** More flexibility usually requires more abstraction and code.

## One-line memory trick

**Simple Factory = Select one.**

**Factory Method = Override creation.**

**Abstract Factory = Create a family.**
