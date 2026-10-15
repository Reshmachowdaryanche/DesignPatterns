# Builder Design Pattern — Java & Kotlin

## Table of Contents

1. What Is the Builder Pattern?
2. The Problem Without Builder
3. Builder Pattern in Java
4. Builder Pattern in Kotlin
5. How Builder Works
6. Advantages and Limitations
7. When to Use Builder
8. Builder vs Factory Patterns
9. Common Interview Questions
10. Quick Revision

---

# 1\. What Is the Builder Pattern?

The **Builder Pattern** is a creational design pattern that constructs a complex object step by step.

It is especially useful when an object has many optional parameters, different configuration options, or a construction process that should be separated from the final object.

### Real-world example: Food ordering application

Imagine creating a burger with the following options:

- Required: burger size
- Optional: cheese
- Optional: extra patty
- Optional: sauces
- Optional: vegetables

Without a Builder, the constructor might look like this:

```
Burger burger = new Burger(
    "Large", true, true, "BBQ", false
);
```

What do the boolean values mean? Which parameter represents cheese? Which one represents an extra patty?

The Builder Pattern makes object construction more readable:

```
Burger burger = new Burger.Builder("Large")
    .cheese(true)
    .extraPatty(true)
    .sauce("BBQ")
    .build();
```

The client specifies only the required and desired options, and `build()` creates the final object.

**Memory trick:** Builder = Configure step by step, then build.

---

# 2\. The Problem Without Builder

Consider a `User` object with several fields:

```
class User {
    String name;
    String email;
    int age;
    String address;
    String phone;
}
```

A constructor containing every field might look like this:

```
User user = new User(
    "Alex",
    "alex@example.com",
    25,
    "Hyderabad",
    "9876543210"
);
```

This approach becomes problematic as the number of parameters increases.

### Problem 1: Too many constructor parameters

A class with ten optional fields can require a constructor with ten parameters.

### Problem 2: Unclear arguments

Multiple parameters of the same type can easily be confused.

```
new User("Alex", "alex@example.com", "Hyderabad", "9876543210");
```

The compiler cannot tell whether the address and phone number have been passed in the wrong order if both are strings.

### Problem 3: Many overloaded constructors

One solution is to create multiple constructors for different combinations of fields. However, the number of constructors can grow rapidly.

### Problem 4: Difficult-to-read object creation

When creating an object, the client must remember the order and meaning of its arguments.

The Builder Pattern addresses these issues by giving each configuration option a descriptive method.

---

# 3\. Builder Pattern in Java

We will build a `User` object with one required field and several optional fields.

## Step 1: Create the User class

```
public class User {

    private final String name;
    private final String email;
    private final int age;
    private final String address;

    private User(Builder builder) {
        this.name = builder.name;
        this.email = builder.email;
        this.age = builder.age;
        this.address = builder.address;
    }

    public void display() {
        System.out.println("Name: " + name);
        System.out.println("Email: " + email);
        System.out.println("Age: " + age);
        System.out.println("Address: " + address);
    }

    public static class Builder {

        private final String name;

        private String email = "";
        private int age = 0;
        private String address = "";

        public Builder(String name) {
            this.name = name;
        }

        public Builder email(String email) {
            this.email = email;
            return this;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public Builder address(String address) {
            this.address = address;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }
}
```

## Step 2: Create the object

```
public class Main {

    public static void main(String[] args) {

        User user = new User.Builder("Alex")
            .email("alex@example.com")
            .age(25)
            .address("Hyderabad")
            .build();

        user.display();
    }
}
```

Output:

```
Name: Alex
Email: alex@example.com
Age: 25
Address: Hyderabad
```

## Step 3: Understand the important parts

### 1\. Required parameters

```
public Builder(String name) {
    this.name = name;
}
```

The builder requires a name before configuration can begin.

This is a design choice: the builder can enforce required information through its constructor.

### 2\. Optional parameters

```
private String email = "";
private int age = 0;
private String address = "";
```

These fields have default values, so callers can omit them.

In production code, choose defaults that make sense for the domain. An empty string or zero is not always an appropriate default.

### 3\. Fluent methods

```
public Builder age(int age) {
    this.age = age;
    return this;
}
```

The method updates a builder field and returns the same builder instance.

Returning `this` enables method chaining:

```
builder.email("alex@example.com")
       .age(25)
       .address("Hyderabad");
```

### 4\. The `build()` method

```
public User build() {
    return new User(this);
}
```

The builder passes its collected configuration to the `User` constructor.

This is the point at which the final object is created.

### 5\. A private constructor

```
private User(Builder builder) {
    this.name = builder.name;
    this.email = builder.email;
    this.age = builder.age;
    this.address = builder.address;
}
```

The private constructor prevents ordinary external code from bypassing the builder and directly calling this constructor.

### 6\. Why use `final` fields?

The fields are assigned once during construction and cannot subsequently be reassigned.

This makes the `User` object immutable with respect to these field references and values. Immutability is particularly useful when an object represents configuration or a value that should not change after creation.

---

# 4\. Builder Pattern in Kotlin

Kotlin provides named arguments and default parameter values, so simple objects often do not need a separate Builder class.

However, the Builder Pattern can still be useful when you need staged configuration, validation, interoperability with Java, or a fluent configuration API.

## Kotlin implementation

```
class User private constructor(
    val name: String,
    val email: String,
    val age: Int,
    val address: String
) {

    fun display() {
        println("Name: $name")
        println("Email: $email")
        println("Age: $age")
        println("Address: $address")
    }

    class Builder(private val name: String) {

        private var email: String = ""
        private var age: Int = 0
        private var address: String = ""

        fun email(email: String) = apply {
            this.email = email
        }

        fun age(age: Int) = apply {
            this.age = age
        }

        fun address(address: String) = apply {
            this.address = address
        }

        fun build(): User {
            require(name.isNotBlank()) {
                "Name must not be blank"
            }

            require(age >= 0) {
                "Age cannot be negative"
            }

            return User(name, email, age, address)
        }
    }
}

fun main() {
    val user = User.Builder("Alex")
        .email("alex@example.com")
        .age(25)
        .address("Hyderabad")
        .build()

    user.display()
}
```

## Understanding Kotlin's `apply`

Consider:

```
fun age(age: Int) = apply {
    this.age = age
}
```

`apply` executes the given block on the receiver object and returns that same object.

Therefore, the builder methods can be chained:

```
User.Builder("Alex")
    .email("alex@example.com")
    .age(25)
    .address("Hyderabad")
    .build()
```

The final `build()` method validates the configuration and constructs the `User`.

## A simpler Kotlin alternative

For many Kotlin data objects, a constructor with named arguments and default values is enough:

```
data class User(
    val name: String,
    val email: String = "",
    val age: Int = 0,
    val address: String = ""
)

fun main() {
    val user = User(
        name = "Alex",
        age = 25,
        address = "Hyderabad"
    )
}
```

This is often the more idiomatic Kotlin solution when construction is straightforward.

**Important:** Named arguments and default parameters provide similar convenience, but they are not themselves an implementation of the Builder design pattern.

---

# 5\. How Builder Works

The Builder Pattern generally follows this sequence:

```
Client
  |
  v
Create Builder
  |
  v
Set required information
  |
  v
Configure optional fields
  |
  v
Call build()
  |
  v
Construct final object
```

For example:

```
User user = new User.Builder("Alex")
    .email("alex@example.com")
    .age(25)
    .build();
```

Execution flow:

1. `new User.Builder("Alex")` creates the builder.
2. `email()` stores the email in the builder.
3. `age()` stores the age in the builder.
4. `build()` passes the accumulated configuration to the `User` constructor.
5. The constructor creates the final `User` instance.

The builder is responsible for collecting configuration; the final object represents the completed result.

## Is a separate Director class required?

No. The classic GoF Builder pattern describes a Director that can coordinate construction steps, but many modern implementations omit it.

A Director is useful when a predefined construction procedure must be reused, such as creating different standard configurations of the same object.

For simpler cases, the client can call builder methods directly.

---

# 6\. Advantages and Limitations

## Advantages

- **Readable construction:** Named methods explain what each argument means.
- **Optional parameters:** Clients can configure only the fields they need.
- **Validation:** The builder can validate the configuration before creating the final object.
- **Immutability:** The final object can expose read-only fields.
- **Flexible construction:** Different combinations of options can produce different valid objects.

## Limitations

- **More code:** A builder introduces an additional class and methods.
- **Extra complexity:** It may be unnecessary for objects with only one or two simple fields.
- **Validation still matters:** A builder does not automatically guarantee that the resulting object is valid.
- **Mutable builder state:** Reusing a builder can unintentionally carry settings from one object to another.

For production implementations, consider validating required fields, ranges, and cross-field constraints inside `build()` or the final constructor.

---

# 7\. When to Use Builder

Use Builder when:

- A constructor has many parameters.
- Several parameters are optional.
- Different configurations of the same object are required.
- Object creation involves validation or multiple setup steps.
- The final object should be immutable.

Avoid Builder when a simple constructor or Kotlin's named arguments and default parameters already make construction clear.

Typical examples include:

- HTTP request configuration.
- Complex application settings.
- Query construction.
- Notification configuration.
- Test data objects with many optional fields.

---

# 8\. Builder vs Factory Patterns

| Feature | Builder | Simple Factory | Factory Method | Abstract Factory |
| --- | --- | --- | --- | --- |
| Main purpose | Construct an object step by step | Select a concrete object | Delegate creation to subclasses | Create related product families |
| Main concern | How an object is configured | Which object to create | Which object a creator subclass creates | Which family of products to create |
| Typical API | `builder.option().build()` | `Factory.create(type)` | `factory.createProduct()` | `factory.createA()` and `factory.createB()` |
| Common use | Complex objects with many options | A small set of implementation choices | Varying creation behavior across subclasses | Related objects that must work together |

### Key distinction

A factory primarily decides **which object to create**.

A builder primarily controls **how to assemble and configure an object**.

They can also be combined. For example, a factory may select the appropriate request type, while a builder configures the selected request's headers, timeout, and body.

---

# 9\. Common Interview Questions

### Q1. What is the Builder Pattern?

Builder is a creational design pattern that separates the construction of an object from its representation or configuration, allowing it to be assembled step by step.

### Q2. Why use Builder instead of a constructor?

Builder improves readability and flexibility when an object has many optional parameters or configuration choices. It also provides a convenient place for validation.

### Q3. Why do builder methods return `this`?

Returning the same builder instance enables method chaining, so clients can configure multiple fields in one expression.

In Kotlin, `apply` is commonly used to achieve this behavior.

### Q4. Is Builder useful in Kotlin?

Yes, but Kotlin's named arguments and default parameter values often remove the need for a separate builder. Use one when it adds meaningful value, such as staged construction, validation, or a Java-friendly API.

### Q5. Does Builder guarantee immutability?

No. Immutability depends on the final class design. Use private fields, prevent unintended mutation, and ensure mutable objects are handled appropriately.

### Q6. What is the difference between Builder and Abstract Factory?

Builder constructs one object through a sequence of configuration steps. Abstract Factory creates families of related objects through a common factory interface.

### Q7. What is the role of `build()`?

`build()` creates and returns the final object using the configuration accumulated by the builder. It is also a natural place to validate the configuration.

### Q8. Is a Director always required?

No. A Director is useful when construction steps must be coordinated into reusable procedures. Many practical Builder implementations let the client configure the builder directly.

---

# 10\. Quick Revision

- **Purpose:** Construct complex objects step by step.
- **Problem solved:** Constructors with many parameters and unclear argument combinations.
- **Core API:** `Builder → configuration methods → build()`.
- **Java technique:** Fluent methods return `this`.
- **Kotlin technique:** `apply` supports fluent configuration.
- **Kotlin alternative:** Named arguments and default parameter values for simpler objects.
- **Main benefit:** Readable, flexible construction.
- **Main trade-off:** More code and abstraction.

## One-Line Memory Trick

**Builder = Configure step by step, then build the object.**
