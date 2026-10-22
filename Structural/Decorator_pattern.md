# Decorator Design Pattern — Java & Kotlin

An interview-focused guide to the Decorator Pattern, using a real-world coffee shop example with add-ons, Java and Kotlin implementations, class relationships, comparisons, and common interview questions.

## Table of Contents

1. What Is the Decorator Pattern?
2. Why Do We Need It?
3. Java Implementation
4. Kotlin Implementation
5. How Decorator Works
6. When to Use Decorator
7. Decorator vs. Composite vs. Proxy vs. Adapter
8. Common Interview Questions
9. Quick Revision

# 1. What Is the Decorator Pattern?

The Decorator Pattern is a structural design pattern that lets you add new behavior or responsibilities to an individual object dynamically, without modifying its original class.

It achieves this by wrapping an object inside another object that implements the same interface.

### Real-world example: Coffee shop

Imagine you are building a coffee ordering application.

A customer can order:

- Plain coffee.
- Coffee with milk.
- Coffee with sugar.
- Coffee with milk and sugar.
- Coffee with milk, sugar, and extra espresso.

You could create a separate class for every combination, but the number of classes would grow rapidly.

Instead, use the Decorator Pattern to wrap a basic coffee object with additional ingredients.

Menemani Obrolan dengan Jenis-Jenis Minuman Kopi

Regular latte Nutrition

Einstein Bros. Cornell Rd & 188th | Bagels, Coffee, Breakfast

For example:

- `SimpleCoffee` provides the base coffee.
- `MilkDecorator` adds milk.
- `SugarDecorator` adds sugar.
- Each decorator wraps another coffee object.

The customer can combine decorators in different ways without creating a new class for every possible combination.

Java example:

java
```
Coffee coffee = new SimpleCoffee();

coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);

System.out.println(coffee.getDescription());
System.out.println(coffee.getCost());
```

Possible output:

text
```
Simple Coffee, Milk, Sugar
7.0
```

The cost depends on the prices defined by the implementation.

Memory trick: Decorator = Wrap an object to add behavior without changing its original class.

# 2. Why Do We Need It?

Suppose you are building the coffee ordering application without the Decorator Pattern.

You might create separate classes for every combination:

java
```
class SimpleCoffee { }
class MilkCoffee { }
class SugarCoffee { }
class MilkSugarCoffee { }
class MilkSugarVanillaCoffee { }
```

As new ingredients are introduced, the number of possible combinations grows quickly.

With $n$ optional add-ons, there can be up to $2^n$ combinations of add-ons, assuming each can independently be included or excluded.

This leads to several problems:

- Too many subclasses.
- Repeated code for calculating prices and descriptions.
- Difficulty introducing new add-ons.
- Poor flexibility when customers combine ingredients in different orders.

## With Decorator

Define a common interface:

java
```
interface Coffee {
    String getDescription();
    double getCost();
}
```

Both the basic coffee and its decorators implement this interface.

text
```
                 Coffee
                   |
          +--------+---------+
          |                  |
      SimpleCoffee      CoffeeDecorator
                              |
                    +---------+---------+
                    |         |         |
               MilkDecorator SugarDecorator
                              |
                       Other Decorators
```

Each decorator stores a reference to another `Coffee` object and adds its own behavior.

For example:

java
```
Coffee coffee = new SugarDecorator(
                    new MilkDecorator(
                        new SimpleCoffee()
                    )
                );
```

The outer decorator adds sugar, the inner decorator adds milk, and the original coffee supplies the base behavior.

Important: Decorator is especially useful when you need flexible combinations of optional behavior without creating a subclass for every combination.

# 3. Java Implementation

Let's implement a coffee ordering system where customers can add milk and sugar to a basic coffee.

## Step 1: Define the Component Interface

The Component defines the common operations supported by both the original object and its decorators.

java
```
interface Coffee {
    String getDescription();
    double getCost();
}
```

Both `SimpleCoffee` and every decorator must implement these methods.

## Step 2: Create the Concrete Component

The Concrete Component is the original object that provides the basic functionality.

java
```
class SimpleCoffee implements Coffee {

    @Override
    public String getDescription() {
        return "Simple Coffee";
    }

    @Override
    public double getCost() {
        return 5.0;
    }
}
```

Here:

- The description is `Simple Coffee`.
- The base cost is `5.0`.

The base object does not know anything about milk, sugar, or other add-ons.

## Step 3: Create the Abstract Decorator

The Decorator holds a reference to another object that implements the same interface.

java
```
abstract class CoffeeDecorator implements Coffee {

    protected final Coffee coffee;

    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
}
```

### Why does the decorator hold a `Coffee` reference?

java
```
protected final Coffee coffee;
```

This allows the decorator to wrap either:

- A `SimpleCoffee`.
- Another decorator.
- Any other implementation of `Coffee`.

The decorator does not depend on a specific concrete class.

This is the key relationship that makes the pattern flexible.

## Step 4: Create Concrete Decorators

Each concrete decorator adds a specific responsibility to the wrapped object.

### Milk Decorator

java
```
class MilkDecorator extends CoffeeDecorator {

    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }

    @Override
    public double getCost() {
        return coffee.getCost() + 1.0;
    }
}
```

The milk decorator:

- Delegates the existing description to the wrapped coffee.
- Appends `Milk`.
- Adds `1.0` to the cost.

### Sugar Decorator

java
```
class SugarDecorator extends CoffeeDecorator {

    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Sugar";
    }

    @Override
    public double getCost() {
        return coffee.getCost() + 0.5;
    }
}
```

The sugar decorator follows the same structure but adds sugar and increases the cost by `0.5`.

Notice that neither decorator modifies `SimpleCoffee`. Both extend its behavior by wrapping it.

## Step 5: Use the Decorators

java
```
public class Main {

    public static void main(String[] args) {

        Coffee coffee = new SimpleCoffee();

        coffee = new MilkDecorator(coffee);
        coffee = new SugarDecorator(coffee);

        System.out.println(coffee.getDescription());
        System.out.println(coffee.getCost());
    }
}
```

Output:

text
```
Simple Coffee, Milk, Sugar
6.5
```

### What happens step by step?

Base

`SimpleCoffee`

Description: Simple Coffee

Cost: 5.0

Wrapper 1

`MilkDecorator`

Adds milk to the description

Cost: 5.0 + 1.0 = 6.0

Wrapper 2

`SugarDecorator`

Adds sugar to the existing result

Cost: 6.0 + 0.5 = 6.5

The final object is a `SugarDecorator`, which wraps a `MilkDecorator`, which wraps a `SimpleCoffee`.

The final call to `getCost()` delegates inward and adds each decorator's price.

### What if we change the order?

java
```
Coffee coffee = new MilkDecorator(
    new SugarDecorator(
        new SimpleCoffee()
    )
);
```

The cost is still `6.5` because these two add-ons simply add fixed amounts. However, the description becomes:

text
```
Simple Coffee, Sugar, Milk
```

Decorator order can affect behavior and output. The fact that order does not affect this example's total price is specific to the chosen add-ons.

# 4. Kotlin Implementation

Kotlin supports the Decorator Pattern using interfaces, classes, constructor delegation, and overridden methods.

## Complete Implementation

kotlin
```
// Component
interface Coffee {
    fun getDescription(): String
    fun getCost(): Double
}

// Concrete Component
class SimpleCoffee : Coffee {

    override fun getDescription(): String {
        return "Simple Coffee"
    }

    override fun getCost(): Double {
        return 5.0
    }
}

// Abstract Decorator
abstract class CoffeeDecorator(
    protected val coffee: Coffee
) : Coffee

// Concrete Decorator
class MilkDecorator(
    coffee: Coffee
) : CoffeeDecorator(coffee) {

    override fun getDescription(): String {
        return "${coffee.getDescription()}, Milk"
    }

    override fun getCost(): Double {
        return coffee.getCost() + 1.0
    }
}

// Concrete Decorator
class SugarDecorator(
    coffee: Coffee
) : CoffeeDecorator(coffee) {

    override fun getDescription(): String {
        return "${coffee.getDescription()}, Sugar"
    }

    override fun getCost(): Double {
        return coffee.getCost() + 0.5
    }
}

fun main() {

    var coffee: Coffee = SimpleCoffee()

    coffee = MilkDecorator(coffee)
    coffee = SugarDecorator(coffee)

    println(coffee.getDescription())
    println(coffee.getCost())
}
```

Output:

text
```
Simple Coffee, Milk, Sugar
6.5
```

## Why is this Decorator?

Consider the constructor:

kotlin
```
class MilkDecorator(
    coffee: Coffee
) : CoffeeDecorator(coffee)
```

The milk decorator receives any `Coffee` implementation, not just `SimpleCoffee`.

That means it can wrap another decorator as well:

kotlin
```
val coffee: Coffee =
    SugarDecorator(
        MilkDecorator(
            SimpleCoffee()
        )
    )
```

The same interface is used at every level of the wrapping chain.

### Kotlin-specific note

Kotlin's `by` delegation can forward interface implementations to another object, but it does not automatically add the extra behavior required by a decorator. When you need to change the description, add a price, or alter execution, explicit overrides are usually clearer.

# 5. How Decorator Works

The Decorator Pattern has four main roles.

| Role | Responsibility | Example |
| --- | --- | --- |
| Component | Defines the common interface | `Coffee` |
| Concrete Component | Provides the original behavior | `SimpleCoffee` |
| Decorator | Holds and delegates to a component | `CoffeeDecorator` |
| Concrete Decorator | Adds a specific responsibility | `MilkDecorator`, `SugarDecorator` |

## Class diagram

text
```
                  <<interface>>
                      Coffee
                         ^
                         |
              +----------+-----------+
              |                      |
        SimpleCoffee          CoffeeDecorator
        (Component)            (Abstract Wrapper)
                                      |
                                      | has-a
                                      v
                                    Coffee
                                      ^
                                      |
                             +--------+--------+
                             |                 |
                       MilkDecorator     SugarDecorator
```

The important relationship is:

java
```
protected final Coffee coffee;
```

The decorator contains a reference to the common interface and forwards calls to it.

## Execution flow

Suppose the client executes:

java
```
Coffee coffee =
    new SugarDecorator(
        new MilkDecorator(
            new SimpleCoffee()
        )
    );

System.out.println(coffee.getCost());
```

The execution flow is:

mermaid
```
flowchart TD
    A["Client calls getCost()"] --> B["SugarDecorator"]
    B --> C["MilkDecorator"]
    C --> D["SimpleCoffee: returns 5.0"]
    D --> E["MilkDecorator adds 1.0"]
    E --> F["SugarDecorator adds 0.5"]
    F --> G["Final cost: 6.5"]
```

The base object supplies the original result. Each decorator adds its own responsibility as the call returns through the wrapping chain.

### Key characteristics

- Wrapping: Each decorator contains another component.
- Delegation: Existing behavior is forwarded to the wrapped object.
- Extension: A decorator adds or changes behavior.
- Composition: Multiple decorators can be combined at runtime.
- Same interface: The client can continue using the `Coffee` interface.

# 6. When to Use Decorator

Use Decorator when:

- You need to add optional behavior to objects dynamically.
- You want to combine multiple independent features.
- Subclassing would create too many classes for different feature combinations.
- You cannot or do not want to modify the original class.
- You want to follow the Open/Closed Principle by extending behavior without changing existing implementations.

### Common real-world examples

- Java I/O: Wrapping streams to add buffering, data conversion, or other processing.
- Coffee ordering: Adding milk, sugar, syrup, and extra espresso.
- Logging: Wrapping a service with additional logging.
- Caching: Adding a caching layer around an existing service.
- Compression and encryption: Wrapping a data-processing stream with additional transformations.
- UI components: Adding borders, scrollbars, or visual effects to existing components.
- Notifications: Wrapping a notification sender with email, SMS, or logging behavior.

For example, Java's I/O library uses decorators:

java
```
InputStream input =
    new BufferedInputStream(
        new FileInputStream("data.txt")
    );
```

Here, `BufferedInputStream` wraps another `InputStream` and adds buffering while retaining the common interface.

### When should you avoid it?

Avoid Decorator when:

- The added behavior is mandatory and simple enough to implement directly.
- The wrapping chain becomes too deep to understand or debug.
- You need to modify an object's behavior in ways that cannot be expressed cleanly through the shared interface.
- The added responsibilities have complex dependencies on one another.

Main benefit: Add responsibilities to individual objects flexibly, without creating a separate subclass for every combination.

# 7. Decorator vs. Composite vs. Proxy vs. Adapter

These are all structural design patterns, but they solve different problems.

| Feature | Decorator | Composite | Proxy | Adapter |
| --- | --- | --- | --- | --- |
| Main purpose | Add responsibilities | Treat individuals and groups uniformly | Control access to an object | Make incompatible interfaces work together |
| Structure | Wrapper around a component | Tree of components | Representative or stand-in for an object | Converts one interface to another |
| Key relationship | Wraps and delegates | Contains children | Controls access or delegates | Translates calls |
| Interface | Usually same as wrapped component | Common component interface | Usually same as the subject | Target interface differs from adaptee |
| Example | Add buffering to a stream | Folder containing files | Lazy-loading a remote resource | Integrate a legacy payment API |

### Easy way to remember

- Decorator: Add behavior.
- Composite: Organize objects into a hierarchy.
- Proxy: Control access.
- Adapter: Convert an interface.

## Decorator vs. Inheritance

Inheritance adds behavior through class relationships, while Decorator adds behavior by wrapping objects.

For example, inheritance might require classes such as:

java
```
class MilkCoffee extends SimpleCoffee { }
class SugarCoffee extends SimpleCoffee { }
class MilkSugarCoffee extends SimpleCoffee { }
```

With Decorator, you can combine behavior at runtime:

java
```
Coffee coffee =
    new SugarDecorator(
        new MilkDecorator(
            new SimpleCoffee()
        )
    );
```

Decorator favors composition over creating a subclass for every feature combination.

## Decorator vs. Proxy: an interview favorite

Both patterns wrap an object and often implement the same interface, but their intent differs.

- A Decorator adds responsibilities, such as buffering or extra behavior.
- A Proxy controls access, such as lazy initialization, access checks, or remote communication.

Their structures can look similar, so the intended purpose is the best way to distinguish them.

# 8. Common Interview Questions

### Q1. What is the Decorator Pattern?

A structural design pattern that adds responsibilities to an object dynamically by wrapping it in another object that implements the same interface.

### Q2. What problem does Decorator solve?

It avoids creating a large number of subclasses for every possible combination of optional behaviors.

### Q3. What are the four main components?

- Component: Common interface.
- Concrete Component: Original implementation.
- Decorator: Abstract wrapper that stores a component.
- Concrete Decorator: Adds a specific responsibility.

### Q4. Why does a decorator implement the same interface as the wrapped object?

It allows the decorator to be used wherever the original component is expected. This also enables decorators to wrap other decorators.

### Q5. Can we add multiple decorators to one object?

Yes. Decorators can be nested to combine multiple responsibilities.

java
```
Coffee coffee =
    new SugarDecorator(
        new MilkDecorator(
            new SimpleCoffee()
        )
    );
```

### Q6. Does the order of decorators matter?

It can.

In our coffee example, the order changes the description but not the total cost. In other applications, order can affect results. For example, compression followed by encryption may behave differently from encryption followed by compression.

### Q7. Is Decorator an example of composition or inheritance?

It primarily uses object composition to wrap an object. Concrete decorators may use inheritance to extend a shared decorator class, but composition is the central mechanism that adds behavior.

### Q8. How does Decorator follow the Open/Closed Principle?

You can introduce a new decorator without modifying the existing component or existing decorators, provided they follow the established interface and contracts.

For example, a `VanillaDecorator` can be added without changing `SimpleCoffee`, `MilkDecorator`, or `SugarDecorator`.

### Q9. What are the advantages of Decorator?

- Adds behavior dynamically.
- Encourages composition over subclassing.
- Supports flexible combinations of features.
- Keeps individual responsibilities in separate classes.
- Can extend existing classes without modifying their source code.

### Q10. What are the disadvantages of Decorator?

- Creates additional wrapper objects.
- Can make the object structure harder to understand.
- Debugging may involve traversing several layers.
- The order of decorators can affect behavior.
- Clients that depend on concrete implementation types may not work as expected with wrappers.

### Q11. Is Decorator always better than inheritance?

No. Decorator is useful when behaviors are optional, combinable, or need to be added at runtime. Inheritance may be simpler when the variation is fixed and naturally represents an "is-a" relationship.

### Q12. Is Java's `BufferedInputStream` an example of Decorator?

Yes. It wraps an `InputStream` and adds buffering while exposing the `InputStream` interface.

### Q13. Can a decorator change existing behavior instead of just adding behavior?

Yes. A decorator can augment, transform, restrict, or replace aspects of the wrapped object's behavior while preserving the expected interface contract.

For example, a caching decorator may return a cached result instead of invoking the wrapped service on every call.

### Q14. How does Decorator differ from Strategy?

- Decorator wraps an object to add responsibilities while retaining the component interface.
- Strategy encapsulates an interchangeable algorithm or behavior that a context uses.

Decorator focuses on layering responsibilities; Strategy focuses on selecting an implementation of an algorithm.

### Q15. Does Decorator follow SOLID principles?

It can support several SOLID principles:

- Single Responsibility Principle: Each decorator can focus on one additional responsibility.
- Open/Closed Principle: New decorators can extend functionality without modifying existing classes.
- Dependency Inversion Principle: Decorators can depend on the common component abstraction rather than concrete implementations.

As with any pattern, the actual design determines whether these principles are followed well.

# 9. Quick Revision

- Category: Structural design pattern.
- Purpose: Add responsibilities to an object dynamically.
- Component: Common interface.
- Concrete Component: Original object.
- Decorator: Wrapper that holds a component reference.
- Concrete Decorator: Adds or modifies behavior.
- Java: Commonly implemented with interfaces and wrapper classes.
- Kotlin: Commonly implemented with interfaces, constructors, and method overrides.
- Key relationship: Has-a relationship with the same component abstraction.
- Main benefit: Combines optional behaviors without creating subclasses for every combination.
- Main limitation: Too many nested wrappers can make the system difficult to debug.

## One-Line Memory Trick

Decorator = Wrap an object with another object to add behavior dynamically while keeping the same interface.
