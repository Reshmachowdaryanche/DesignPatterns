

# Prototype Design Pattern — Java & Kotlin

An interview-focused guide to the Prototype Pattern, including practical examples, copying strategies, comparisons, and common interview questions.

## Table of Contents

1. What Is the Prototype Pattern?
2. Why Do We Need It?
3. Java Implementation
4. Kotlin Implementation
5. Shallow Copy vs. Deep Copy
6. How Prototype Works
7. When to Use Prototype
8. Prototype vs. Factory vs. Builder
9. Common Interview Questions
10. Quick Revision

---

# 1\. What Is the Prototype Pattern?

The **Prototype Pattern** is a creational design pattern that creates new objects by copying existing, configured objects instead of constructing each object from scratch.

### Real-world example: Game characters

Imagine a game that needs hundreds of warrior enemies. Each warrior has the same initial health, attack power, weapon, and configuration.

Instead of repeating the entire initialization process, create a configured warrior prototype and copy it whenever a new warrior is needed.

```
Enemy warrior = warriorPrototype.copy();
```

The copied enemy can then be customized independently, provided its copy implementation handles mutable state correctly.

**Memory trick:** Prototype = Copy an existing object to create a new one.

---

# 2\. Why Do We Need It?

Suppose creating an enemy requires several initialization steps:

```
Enemy enemy = new Enemy();
enemy.setHealth(100);
enemy.setAttackPower(20);
enemy.setWeapon(new Weapon("Sword"));
enemy.initializeAnimations();
enemy.loadConfiguration();
```

Repeating this setup for hundreds of enemies creates duplicated initialization logic.

Instead, configure a prototype once:

```
Enemy warriorPrototype = createConfiguredWarrior();
```

Then create new enemies by copying it:

```
Enemy enemy1 = warriorPrototype.copy();
Enemy enemy2 = warriorPrototype.copy();
```

This approach is useful when objects share a common configuration or initialization is expensive.

**Important:** Prototype does not automatically make object creation faster. Copying must be appropriate for the object's structure and lifecycle.

---

# 3\. Java Implementation

We will create an `Enemy` prototype with a `copy()` method.

## Step 1: Define the Prototype Interface

```
interface Prototype<T> {
    T copy();
}
```

The interface provides a common operation for creating copies.

## Step 2: Implement the Concrete Prototype

```
class Enemy implements Prototype<Enemy> {

    private final String type;
    private int health;
    private final int attackPower;
    private final Weapon weapon;

    public Enemy(
        String type,
        int health,
        int attackPower,
        Weapon weapon
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
            new Weapon(weapon.getName())
        );
    }

    public void setHealth(int health) {
        this.health = health;
    }

    public void display() {
        System.out.println(
            type + " | Health: " + health
            + " | Attack: " + attackPower
            + " | Weapon: " + weapon.getName()
        );
    }
}

class Weapon {

    private final String name;

    public Weapon(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

The `copy()` method creates a new `Enemy` and a new `Weapon`, so the copied enemy does not share that mutable object with the original.

## Step 3: Use the Prototype

```
public class Main {

    public static void main(String[] args) {

        Enemy warriorPrototype = new Enemy(
            "Warrior",
            100,
            20,
            new Weapon("Sword")
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
Warrior | Health: 100 | Attack: 20 | Weapon: Sword
Warrior | Health: 150 | Attack: 20 | Weapon: Sword
Warrior | Health: 100 | Attack: 20 | Weapon: Sword
```

### What happens?

- The prototype stores the original configuration.
- `copy()` creates a new object with equivalent initial values.
- `enemy1` and `enemy2` are separate instances.
- Changing `enemy1`'s health does not change the other enemies.

The client does not need to repeat the construction logic.

---

# 4\. Kotlin Implementation

Kotlin data classes provide a convenient `copy()` method, which is useful for simple prototype-style creation.

## Implementation

```
data class Weapon(val name: String)

data class Enemy(
    val type: String,
    val health: Int,
    val attackPower: Int,
    val weapon: Weapon
)

fun main() {
    val warriorPrototype = Enemy(
        type = "Warrior",
        health = 100,
        attackPower = 20,
        weapon = Weapon("Sword")
    )

    val enemy1 = warriorPrototype.copy(health = 150)
    val enemy2 = warriorPrototype.copy()

    println(warriorPrototype)
    println(enemy1)
    println(enemy2)
}
```

### Why is `copy()` useful here?

```
val enemy1 = warriorPrototype.copy(health = 150)
```

This creates a new `Enemy` with the same values as the prototype, except that its health is set to `150`.

Kotlin's generated `copy()` method is shallow. In this example, `Weapon` is immutable, so sharing its reference is safe.

**Interview note:** A data class's `copy()` method can support prototype-style creation, but using a data class does not automatically mean the entire application implements the Prototype design pattern.

---

# 5\. Shallow Copy vs. Deep Copy

This is one of the most frequently asked Prototype-related interview questions.

## Shallow Copy

A shallow copy creates a new outer object but copies references to nested objects.

```
Original Enemy ──┐
                 ├──> Same Weapon object
Copied Enemy ────┘
```

If the weapon is mutable, changing it through one enemy can affect the other.

## Deep Copy

A deep copy creates a new outer object and duplicates the relevant nested mutable objects.

```
Original Enemy ───> Original Weapon

Copied Enemy ─────> Copied Weapon
```

Now, changes to one weapon do not affect the other.

### Kotlin example

```
data class Weapon(var name: String)

data class Enemy(
    val health: Int,
    val weapon: Weapon
)

val original = Enemy(100, Weapon("Sword"))

// Shallow copy
val shallowCopy = original.copy()

shallowCopy.weapon.name = "Bow"
println(original.weapon.name) // Bow

// Deep copy
val deepCopy = original.copy(
    weapon = original.weapon.copy()
)

deepCopy.weapon.name = "Axe"
println(original.weapon.name) // Bow
```

The shallow copy shares the same `Weapon`. The deep copy creates a separate `Weapon`.

### Important distinction

- **Shallow vs. deep copy** describes how an object's contents are copied.
- **Prototype** describes an object-creation approach that uses an existing object as a template.

Both shallow and deep copying can be used in a Prototype implementation, depending on the required semantics.

---

# 6\. How Prototype Works

```
Client
  |
  v
Select configured prototype
  |
  v
Call copy()
  |
  v
Create a new instance
  |
  v
Customize the copy
```

The pattern typically involves:

1. Creating or configuring a prototype.
2. Requesting a copy of that prototype.
3. Returning a new object with the intended initial state.
4. Optionally customizing the new object.

## Is a Prototype Registry required?

No. A registry is optional.

A registry stores prototypes so that clients can retrieve and copy them by name.

For example:

```
Enemy enemy = registry.get("warrior").copy();
```

This is useful when many predefined prototypes are selected dynamically. For simple applications, a registry may be unnecessary.

---

# 7\. When to Use Prototype

Use Prototype when:

- Object initialization is expensive or complicated.
- Many objects share a common initial configuration.
- You need copies of objects configured at runtime.
- Clients should create objects without knowing their concrete construction details.
- Existing templates can be customized to produce new instances.

Common examples include game characters, document templates, graphical objects, configuration templates, and test fixtures.

Avoid Prototype when a normal constructor or simple factory already provides a clear and efficient solution.

---

# 8\. Prototype vs. Factory vs. Builder

| Feature | Prototype | Factory | Builder |
| --- | --- | --- | --- |
| Main purpose | Copy an existing object | Select or create an implementation | Configure an object step by step |
| Creation mechanism | Copying | Factory method or selection logic | Configuration methods and `build()` |
| Starting point | Existing configured object | Creation request or concrete creator | Builder configuration |
| Typical use | Repeated objects based on templates | Selecting implementations | Objects with many optional parameters |
| Main concern | Copying semantics | Which object to create | How to construct the object |

### Easy way to remember

- **Factory:** Which object should be created?
- **Builder:** How should the object be assembled?
- **Prototype:** Can we create the new object by copying an existing one?

These patterns can be combined. For example, a factory may select a prototype and return a copy of it.

---

# 9\. Common Interview Questions

### Q1. What is the Prototype Pattern?

A creational design pattern that creates new objects by copying existing instances rather than constructing each object from scratch.

### Q2. Why do we need Prototype if shallow and deep copy already exist?

Shallow and deep copy describe copying behavior. Prototype organizes object creation around copying configured instances. You do not need the pattern just because copying is possible; use it when copying is a useful creation strategy.

### Q3. Does Prototype always improve performance?

No. It can reduce repeated initialization, but copying may be just as expensive or more expensive than construction. Performance depends on the object and its copy implementation.

### Q4. What is the difference between shallow copy and deep copy?

Shallow copy may share nested object references. Deep copy duplicates the relevant nested mutable objects as well.

### Q5. Does Java's `Object.clone()` automatically perform a deep copy?

No. `Object.clone()` performs a shallow field copy by default. Nested objects must be copied separately if deep-copy behavior is required.

### Q6. Does Kotlin's data class `copy()` perform a deep copy?

No. Kotlin's generated `copy()` is shallow. Nested mutable objects remain shared unless explicitly copied.

### Q7. Is a Prototype Registry mandatory?

No. It is an optional mechanism for storing and retrieving predefined prototypes.

### Q8. Can we use a copy constructor instead of Prototype?

Yes. A copy constructor or explicit copy method is often a simpler solution. The Prototype pattern becomes more useful when copying serves as a general, polymorphic object-creation mechanism.

### Q9. What are the disadvantages of Prototype?

Copying complex object graphs can be difficult, shallow copies may cause shared-state bugs, and copying resources such as database connections requires special handling.

---

# 10\. Quick Revision

- **Purpose:** Create objects by copying existing instances.
- **Core operation:** `copy()`.
- **Java:** Use an explicit copy method or copy constructor when appropriate.
- **Kotlin:** `data class.copy()` is convenient for shallow copies.
- **Shallow copy:** Nested references may be shared.
- **Deep copy:** Relevant nested mutable objects are duplicated.
- **Registry:** Optional collection of prototypes.
- **Main benefit:** Reuse existing configuration.
- **Main limitation:** Correct copying can be difficult for complex objects.

## One-Line Memory Trick

**Prototype = Copy an existing object, then customize the copy.**
