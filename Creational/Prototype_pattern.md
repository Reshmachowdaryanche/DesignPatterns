

# Prototype Design Pattern — Java & Kotlin

A practical guide to the Prototype Pattern, with real-world examples, Java and Kotlin implementations, object-cloning diagrams, shallow vs. deep copy, comparisons, and interview questions.

## Table of Contents

1. What Is the Prototype Pattern?
2. The Problem Without Prototype
3. Prototype Pattern in Java
4. Prototype Pattern in Kotlin
5. Shallow Copy vs. Deep Copy
6. How Prototype Works
7. Advantages and Limitations
8. When to Use Prototype
9. Prototype vs. Factory and Builder
10. Common Interview Questions
11. Quick Revision

---

# 1\. What Is the Prototype Pattern?

The **Prototype Pattern** is a creational design pattern that creates new objects by copying an existing object, called the _prototype_, rather than constructing every object from scratch.

It is useful when:

- Object creation is expensive or complicated.
- Many objects share similar initial configuration.
- You need copies of an existing object with small changes.
- The exact concrete class should not need to be known by the client.

### Real-world example: Game character creation

Imagine a game containing different enemy types:

- Warrior
- Archer
- Wizard

Each enemy has properties such as health, attack power, weapons, and movement speed.

Creating and configuring every enemy from scratch can involve many setup steps. Instead, the game can maintain configured prototypes and clone them when new enemies are needed.

For example:

```
Enemy enemy = warriorPrototype.copy();
enemy.setHealth(150);
```

The game copies the existing warrior configuration and customizes the new instance.

**Memory trick:** Prototype = Copy an existing object to create a new one.

---

# 2\. The Problem Without Prototype

Suppose a game creates hundreds of enemies with similar configurations.

Without Prototype, every enemy might require repeated construction and setup:

```
Enemy enemy = new Enemy();
enemy.setType("Warrior");
enemy.setHealth(100);
enemy.setAttackPower(20);
enemy.setWeapon("Sword");
enemy.initializeAnimations();
```

If this configuration is repeated frequently, it can lead to:

- Duplicated initialization logic.
- Repeated expensive setup.
- Difficulty maintaining consistent default configurations.
- More knowledge of concrete classes in client code.

A Prototype-based design creates a configured template once and copies it whenever required.

```
Enemy enemy = warriorPrototype.copy();
```

The copy can then be customized without changing the original prototype.

**Important:** Cloning is not automatically faster than constructing a new object. The benefit depends on the cost of initialization, the copy implementation, and the object's internal structure.

---

# 3\. Prototype Pattern in Java

We'll create an `Enemy` prototype that can be copied and customized.

## Step 1: Define the Prototype Interface

```
interface Prototype<T> {
    T copy();
}
```

The interface defines the operation clients use to create a copy without depending on the concrete implementation.

## Step 2: Implement the Enemy Class

```
class Enemy implements Prototype<Enemy> {

    private String type;
    private int health;
    private int attackPower;
    private String weapon;

    public Enemy(
        String type,
        int health,
        int attackPower,
        String weapon
    ) {
        this.type = type;
        this.health = health;
        this.attackPower = attackPower;
        this.weapon = weapon;
    }

    @Override
    public Enemy copy() {
        return new Enemy(
            type,
            health,
            attackPower,
            weapon
        );
    }

    public void setHealth(int health) {
        this.health = health;
    }

    public void display() {
        System.out.println(
            "Type: " + type
            + ", Health: " + health
            + ", Attack: " + attackPower
            + ", Weapon: " + weapon
        );
    }
}
```

### Understanding `copy()`

```
@Override
public Enemy copy() {
    return new Enemy(type, health, attackPower, weapon);
}
```

This method creates a new `Enemy` instance and copies the current field values.

The new enemy is a separate object. Changing its health does not change the original enemy's health.

This example uses a **manual copy constructor approach** rather than Java's built-in `Object.clone()` mechanism.

## Step 3: Create and Copy a Prototype

```
public class Main {

    public static void main(String[] args) {

        Enemy warriorPrototype = new Enemy(
            "Warrior", 100, 20, "Sword"
        );

        Enemy enemy1 = warriorPrototype.copy();
        Enemy enemy2 = warriorPrototype.copy();

        enemy1.setHealth(150);

        warriorPrototype.display();
        enemy1.display();
        enemy2.display();
    }
}
```

Output:

```
Type: Warrior, Health: 100, Attack: 20, Weapon: Sword
Type: Warrior, Health: 150, Attack: 20, Weapon: Sword
Type: Warrior, Health: 100, Attack: 20, Weapon: Sword
```

### What does this demonstrate?

- `warriorPrototype` stores the original configuration.
- `enemy1` and `enemy2` are independent copies.
- Modifying `enemy1` does not affect the prototype or `enemy2`.

The client does not need to repeat the configuration or know how to initialize every field.

---

# 4\. Prototype Pattern in Kotlin

Kotlin does not require a special cloning mechanism. You can define an explicit copy operation or use a data class when its generated copying behavior is sufficient.

## Kotlin implementation using a regular class

```
interface Prototype<T> {
    fun copy(): T
}

class Enemy(
    private val type: String,
    private var health: Int,
    private val attackPower: Int,
    private val weapon: String
) : Prototype<Enemy> {

    override fun copy(): Enemy =
        Enemy(type, health, attackPower, weapon)

    fun setHealth(health: Int) {
        this.health = health
    }

    fun display() {
        println(
            "Type: $type, " +
            "Health: $health, " +
            "Attack: $attackPower, " +
            "Weapon: $weapon"
        )
    }
}

fun main() {
    val warriorPrototype = Enemy(
        "Warrior", 100, 20, "Sword"
    )

    val enemy1 = warriorPrototype.copy()
    val enemy2 = warriorPrototype.copy()

    enemy1.setHealth(150)

    warriorPrototype.display()
    enemy1.display()
    enemy2.display()
}
```

The behavior is the same as the Java example: the `copy()` method explicitly creates a new instance with the prototype's current values.

## Kotlin alternative: Data class

For simple data objects, Kotlin's `data class` provides a generated `copy()` method.

```
data class Enemy(
    val type: String,
    val health: Int,
    val attackPower: Int,
    val weapon: String
)

fun main() {
    val warriorPrototype = Enemy(
        type = "Warrior",
        health = 100,
        attackPower = 20,
        weapon = "Sword"
    )

    val enemy1 = warriorPrototype.copy(health = 150)
    val enemy2 = warriorPrototype.copy()

    println(warriorPrototype)
    println(enemy1)
    println(enemy2)
}
```

Here:

```
val enemy1 = warriorPrototype.copy(health = 150)
```

creates a new `Enemy` with the same values as the prototype, except for `health`.

Kotlin's generated `copy()` method is convenient, but it performs a **shallow copy**. Nested mutable objects are not automatically duplicated.

**Note:** A Kotlin data class is not automatically the Prototype design pattern in every use case. Its `copy()` method simply provides a convenient copying mechanism that can implement the pattern when used for prototype-based creation.

---

# 5\. Shallow Copy vs. Deep Copy

This is one of the most important Prototype interview topics.

Suppose an object contains another mutable object.

```
class Weapon {
    String name;

    Weapon(String name) {
        this.name = name;
    }
}
```

An enemy contains a reference to a `Weapon`:

```
class Enemy {
    String type;
    Weapon weapon;
}
```

When copying an enemy, there are two possible approaches.

## 5.1 Shallow Copy

A shallow copy creates a new outer object but copies references to nested objects.

```
Enemy copy = new Enemy();
copy.type = original.type;
copy.weapon = original.weapon;
```

Object relationship:

```
Original Enemy ──┐
                 ├──> Same Weapon object
Copied Enemy ────┘
```

Both enemies refer to the same `Weapon` instance.

Therefore:

```
copy.weapon.name = "Bow";
```

also changes the weapon name observed through `original.weapon`.

### When is shallow copy acceptable?

- Nested objects are immutable.
- Shared references are intentional.
- The object contains only primitives, immutable values, or references that are safe to share.

## 5.2 Deep Copy

A deep copy duplicates the outer object and the relevant mutable nested objects.

```
Enemy copy = new Enemy();
copy.type = original.type;
copy.weapon = new Weapon(original.weapon.name);
```

Object relationship:

```
Original Enemy ──> Original Weapon

Copied Enemy ────> Copied Weapon
```

Now each enemy has its own weapon object.

Changes to one weapon do not affect the other.

### Important considerations

Deep copying becomes more complicated when objects contain:

- Nested collections.
- Shared references that must remain shared within the copy.
- Circular object references.
- Resources such as files, sockets, or database connections.

A correct deep-copy implementation must decide which objects should be duplicated and which should be shared or recreated.

### Java's `Object.clone()`

Java provides a protected `Object.clone()` method, traditionally used with `Cloneable`.

However, `Object.clone()` performs a shallow field copy by default. Implementing `Cloneable` does not automatically provide a deep copy, and the API has design limitations.

For many applications, an explicit copy constructor or `copy()` method is clearer and easier to maintain.

### Kotlin's `data class.copy()`

Kotlin's generated `copy()` method is also shallow.

```
data class Weapon(var name: String)

data class Enemy(
    val type: String,
    val weapon: Weapon
)

val original = Enemy("Warrior", Weapon("Sword"))
val copy = original.copy()

copy.weapon.name = "Bow"

println(original.weapon.name) // Bow
```

Both enemies reference the same mutable `Weapon`, so changing the copy's weapon also affects the original.

To create a separate weapon:

```
val deepCopy = original.copy(
    weapon = original.weapon.copy()
)
```

This works because `Weapon` is also a data class and its own `copy()` creates a new instance.

**Memory trick:**

- Shallow copy = New outer object, nested references may be shared.
- Deep copy = Duplicate the relevant nested mutable objects too.

---

# 6\. How Prototype Works

The general flow is:

```
Client
  |
  v
Choose an existing prototype
  |
  v
Call copy()
  |
  v
Create a new object
  |
  v
Customize the copy if needed
```

For example:

```
Enemy enemy = warriorPrototype.copy();
enemy.setHealth(150);
```

Execution flow:

1. The application prepares a configured prototype.
2. The client requests a copy.
3. The prototype creates a new object using its copying logic.
4. The client optionally modifies the copy.
5. The original prototype remains unchanged.

## Is a Prototype Registry required?

No. A registry is optional.

A registry can store named prototypes and return copies when requested.

For example, a game might register:

- `"warrior"` → warrior prototype
- `"archer"` → archer prototype
- `"wizard"` → wizard prototype

The client asks the registry for a named prototype, and the registry returns a copy.

This is useful when the application has many predefined configurations, but it adds another component and is unnecessary for simple cases.

---

# 7\. Advantages and Limitations

## Advantages

- **Avoids repeated configuration:** Copy a configured object instead of setting every field again.
- **Reduces dependence on concrete classes:** Clients can request copies through a common prototype interface.
- **Supports runtime configuration:** Prototypes can be configured dynamically.
- **Useful for repeated objects:** Common templates can be copied and customized.
- **Can simplify expensive initialization:** Copying may be cheaper when construction involves costly setup.

## Limitations

- **Copying can be complicated:** Nested mutable objects require careful handling.
- **Shallow copies can cause bugs:** Multiple objects may unintentionally share mutable state.
- **Resource handling is difficult:** Files, connections, and other external resources should not necessarily be duplicated by copying.
- **Copying may not be faster:** The performance benefit depends on the object and its construction process.
- **Maintaining copy logic:** New fields must be included correctly in manual copy implementations.

---

# 8\. When to Use Prototype

Use Prototype when:

- Creating an object from scratch involves expensive setup.
- Many objects share a common initial configuration.
- You need to create copies of runtime-configured objects.
- The exact concrete type should be hidden from the client.
- You want to create objects from templates that can be customized.

### Common examples

- Game enemies and character templates.
- Preconfigured document templates.
- Graphical objects with complex styles.
- Configuration templates.
- Test fixtures with a common starting state.

Avoid Prototype when the object is inexpensive to construct and a simple constructor or factory is clearer.

---

# 9\. Prototype vs. Factory and Builder

| Feature | Prototype | Simple Factory | Factory Method | Builder |
| --- | --- | --- | --- | --- |
| Main purpose | Copy an existing object | Select a concrete object | Delegate creation to subclasses | Assemble an object step by step |
| Creation mechanism | Copying | Conditional or lookup logic | Overridden creation method | Configuration methods and `build()` |
| Starting point | Existing prototype | Type or creation request | Concrete creator | Builder configuration |
| Typical use | Repeated configured objects | Small set of implementation choices | Extensible creation behavior | Objects with many options |
| Main concern | Copy semantics | Which object to create | Which object a subclass creates | How to configure the object |

### Key distinction

- **Factory:** Creates an object by selecting or delegating to an implementation.
- **Builder:** Creates an object by collecting and applying configuration.
- **Prototype:** Creates an object by copying an existing instance.

These patterns can be combined. For example, a factory may retrieve a configured prototype and return a copy of it.

---

# 10\. Common Interview Questions

### Q1. What is the Prototype Pattern?

Prototype is a creational design pattern that creates new objects by copying existing instances instead of constructing each object from scratch.

### Q2. Why use Prototype instead of Factory?

A factory typically constructs an object based on a selection or creation rule. Prototype is useful when an existing, configured object is a convenient starting point for creating another object.

A factory can also use prototypes internally, so the approaches are not mutually exclusive.

### Q3. What is the difference between shallow copy and deep copy?

A shallow copy creates a new outer object but can share references to nested objects. A deep copy duplicates the relevant nested mutable objects as well.

### Q4. Is `Cloneable` enough to create a deep copy in Java?

No. Java's `Object.clone()` copies fields shallowly by default. Nested mutable objects must be duplicated separately if deep-copy semantics are required.

### Q5. Does Kotlin's `data class.copy()` perform a deep copy?

No. It performs a shallow copy. Nested mutable objects remain shared unless they are explicitly copied.

### Q6. Can Prototype improve performance?

Sometimes. It can help when copying a configured object is cheaper than repeating expensive initialization. The actual benefit depends on the object and should be measured when performance matters.

### Q7. Is a Prototype Registry mandatory?

No. A registry is optional and is useful when many named prototypes must be managed and retrieved dynamically.

### Q8. Is Prototype always the best way to duplicate objects?

No. Use the simplest safe approach for the object. Constructors, copy constructors, explicit `copy()` methods, and data-class copying may all be appropriate depending on the requirements.

---

# 11\. Quick Revision

- **Purpose:** Create new objects by copying existing objects.
- **Core concept:** Prototype → `copy()` → New object.
- **Java:** Explicit copy methods or copy constructors are often clear alternatives to `Object.clone()`.
- **Kotlin:** `data class.copy()` provides convenient shallow copying.
- **Shallow copy:** Nested object references may be shared.
- **Deep copy:** Relevant nested mutable objects are duplicated.
- **Registry:** Optional collection of named prototypes.
- **Main benefit:** Reuse an existing object's configuration.
- **Main trade-off:** Correct copy behavior can be difficult for complex object graphs.

## One-Line Memory Trick

**Prototype = Copy an existing object, then customize the copy.**
