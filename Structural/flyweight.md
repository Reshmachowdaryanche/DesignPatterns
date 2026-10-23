# Flyweight Design Pattern — Java & Kotlin

An interview-focused guide to the Flyweight Pattern, using a video game forest with thousands of trees as a real-world example, including Java and Kotlin implementations, class relationships, memory optimization, comparisons, and common interview questions.

## Table of Contents

1. What Is the Flyweight Pattern?
2. Why Do We Need It?
3. Java Implementation
4. Kotlin Implementation
5. How Flyweight Works
6. When to Use Flyweight
7. Flyweight vs. Singleton vs. Prototype vs. Object Pool
8. Common Interview Questions
9. Quick Revision

# 1. What Is the Flyweight Pattern?

The Flyweight Pattern is a structural design pattern that reduces memory consumption by sharing common data among many objects instead of storing duplicate copies.

It is especially useful when an application creates thousands or millions of similar objects.

### Real-world example: Video game forest

Imagine you are developing an open-world video game containing 100,000 trees.

Each tree has properties such as:

- Tree type, such as oak, pine, or palm.
- Texture and 3D model.
- Position in the game world.
- Rotation and scale.
- Health or growth state, if applicable.

Way of the Hunter 2

Stylized Pine Environment - Unreal Engine Asset

Stylized Tree Package – Topologic Team by Topologic Team

Without the Flyweight Pattern, every tree might hold its own copy of the tree model, texture, and other common data.

But 50,000 pine trees do not need 50,000 copies of the same pine tree model and texture.

Instead, share the common data and keep the unique data separate.

- Shared data: Tree model, texture, species.
- Unique data: Position, rotation, scale.

Java example:

java
```
TreeType pine = TreeFactory.getTreeType(
    "Pine",
    "pine.png",
    "Green"
);

pine.draw(10, 20, 1.0);
pine.draw(50, 80, 1.2);
pine.draw(100, 150, 0.8);
```

All three trees use the same shared `TreeType` object, but each tree is drawn at a different position and scale.

Memory trick: Flyweight = Share the tree's common data, not its individual location.

# 2. Why Do We Need It?

Suppose you create a separate object containing all the properties for every tree.

java
```
class Tree {

    String type;
    String texture;
    String color;

    int x;
    int y;

    double scale;
}
```

If your game contains 100,000 trees, it may create 100,000 objects that repeatedly store the same species, texture, and color information.

This can lead to:

- Unnecessary memory consumption.
- Duplicate storage of large resources.
- More objects to allocate and manage.
- Reduced efficiency in memory-intensive environments.

## With Flyweight

Separate tree data into two categories.

Intrinsic state — common data stored in the shared object:

- Tree type.
- Texture.
- Base color.
- Shared model or resource reference.

Extrinsic state — individual data supplied by each tree instance:

- X and Y coordinates.
- Rotation.
- Scale.

text
```
                 Game World
                     |
                     v
               TreeFactory
                     |
            +--------+--------+
            |                 |
            v                 v
        Pine Tree         Oak Tree
       Shared Type      Shared Type
            |                 |
       +----+----+        +----+----+
       |    |    |        |    |    |
      T1   T2   T3       T4   T5   T6
      x,y  x,y  x,y      x,y  x,y  x,y
```

Instead of storing common data for every tree, the game reuses one `TreeType` object per distinct configuration.

Important: Flyweight is useful when many objects share expensive or repeated state and their unique state can be kept separate.

# 3. Java Implementation

Let's implement a forest containing thousands of trees while sharing tree types.

## Step 1: Create the Flyweight

The Flyweight interface defines the operation that uses shared and unique state.

java
```
interface TreeType {
    void draw(int x, int y, double scale);
}
```

The `draw()` method receives the tree's location and scale externally.

These properties vary between individual trees, so they do not need to be stored inside the shared type object.

## Step 2: Create the Concrete Flyweight

The Concrete Flyweight stores intrinsic state that can be reused.

java
```
class ConcreteTreeType implements TreeType {

    private final String name;
    private final String texture;
    private final String color;

    public ConcreteTreeType(
            String name,
            String texture,
            String color) {

        this.name = name;
        this.texture = texture;
        this.color = color;
    }

    @Override
    public void draw(int x, int y, double scale) {

        System.out.println(
            "Drawing " + name
            + " tree at (" + x + ", " + y + ")"
            + ", scale: " + scale
            + ", texture: " + texture
            + ", color: " + color
        );
    }
}
```

### Why is this a Flyweight?

These fields represent shared data:

java
```
private final String name;
private final String texture;
private final String color;
```

The same pine tree type can be reused at many positions.

The object's shared properties remain unchanged, while each call to `draw()` receives different coordinates and scale.

## Step 3: Create the Flyweight Factory

The factory stores and reuses tree types.

java
```
import java.util.HashMap;
import java.util.Map;

class TreeFactory {

    private static final Map<String, TreeType> treeTypes =
        new HashMap<>();

    public static TreeType getTreeType(
            String name,
            String texture,
            String color) {

        String key = name + "|" + texture + "|" + color;

        if (!treeTypes.containsKey(key)) {
            treeTypes.put(
                key,
                new ConcreteTreeType(
                    name, texture, color
                )
            );
        }

        return treeTypes.get(key);
    }

    public static int getTreeTypeCount() {
        return treeTypes.size();
    }
}
```

### How does the factory work?

When the game needs a tree type:

1. It constructs a key from the tree's shared properties.
2. It checks whether that type is already cached.
3. If it exists, the factory returns the existing object.
4. Otherwise, the factory creates and caches a new type.

For example:

java
```
TreeType pine1 =
    TreeFactory.getTreeType(
        "Pine", "pine.png", "Green"
    );

TreeType pine2 =
    TreeFactory.getTreeType(
        "Pine", "pine.png", "Green"
    );

System.out.println(pine1 == pine2);
```

Output:

text
```
true
```

Both references point to the same shared tree type.

Production note: This simple factory is intended for a single-threaded demonstration. A multithreaded game should use a thread-safe cache and a robust key type.

## Step 4: Create Individual Trees

Each tree instance stores its own extrinsic state and references a shared tree type.

java
```
class Tree {

    private final int x;
    private final int y;
    private final double scale;
    private final TreeType treeType;

    public Tree(
            int x,
            int y,
            double scale,
            TreeType treeType) {

        this.x = x;
        this.y = y;
        this.scale = scale;
        this.treeType = treeType;
    }

    public void draw() {
        treeType.draw(x, y, scale);
    }
}
```

Notice that `Tree` does not store its own copy of the texture or color.

It holds a reference to the shared `TreeType` instead.

## Step 5: Use the Flyweight

java
```
import java.util.ArrayList;
import java.util.List;

public class Main {

    public static void main(String[] args) {

        List<Tree> forest = new ArrayList<>();

        TreeType pine = TreeFactory.getTreeType(
            "Pine", "pine.png", "Green"
        );

        TreeType oak = TreeFactory.getTreeType(
            "Oak", "oak.png", "Dark Green"
        );

        forest.add(new Tree(10, 20, 1.0, pine));
        forest.add(new Tree(50, 80, 1.2, pine));
        forest.add(new Tree(100, 150, 0.8, pine));

        forest.add(new Tree(200, 300, 1.5, oak));

        for (Tree tree : forest) {
            tree.draw();
        }

        System.out.println(
            "Unique tree types: "
            + TreeFactory.getTreeTypeCount()
        );
    }
}
```

Output:

text
```
Drawing Pine tree at (10, 20), scale: 1.0, texture: pine.png, color: Green
Drawing Pine tree at (50, 80), scale: 1.2, texture: pine.png, color: Green
Drawing Pine tree at (100, 150), scale: 0.8, texture: pine.png, color: Green
Drawing Oak tree at (200, 300), scale: 1.5, texture: oak.png, color: Dark Green
Unique tree types: 2
```

### What happens?

- Four individual tree objects are created.
- Three trees share the same pine type.
- One tree uses the oak type.
- The factory creates only two unique tree type objects.
- Each tree retains its own location and scale.

This example creates four tree instances but only two shared tree types.

In a real game, thousands of tree instances could share a much smaller collection of tree types.

# 4. Kotlin Implementation

Kotlin supports Flyweight using interfaces, immutable properties, a factory object, and data structures for storing unique instances.

## Complete Implementation

kotlin
```
// Flyweight interface
interface TreeType {
    fun draw(
        x: Int,
        y: Int,
        scale: Double
    )
}

// Concrete Flyweight
class ConcreteTreeType(
    private val name: String,
    private val texture: String,
    private val color: String
) : TreeType {

    override fun draw(
        x: Int,
        y: Int,
        scale: Double
    ) {
        println(
            "Drawing $name tree at ($x, $y), " +
            "scale: $scale, " +
            "texture: $texture, " +
            "color: $color"
        )
    }
}

// Flyweight Factory
object TreeFactory {

    private val treeTypes =
        mutableMapOf<String, TreeType>()

    fun getTreeType(
        name: String,
        texture: String,
        color: String
    ): TreeType {

        val key = "$name|$texture|$color"

        return treeTypes.getOrPut(key) {
            ConcreteTreeType(
                name,
                texture,
                color
            )
        }
    }

    fun getTreeTypeCount(): Int =
        treeTypes.size
}

// Individual tree with extrinsic state
class Tree(
    private val x: Int,
    private val y: Int,
    private val scale: Double,
    private val treeType: TreeType
) {

    fun draw() {
        treeType.draw(x, y, scale)
    }
}

fun main() {

    val forest = mutableListOf<Tree>()

    val pine = TreeFactory.getTreeType(
        "Pine", "pine.png", "Green"
    )

    val oak = TreeFactory.getTreeType(
        "Oak", "oak.png", "Dark Green"
    )

    forest.add(Tree(10, 20, 1.0, pine))
    forest.add(Tree(50, 80, 1.2, pine))
    forest.add(Tree(100, 150, 0.8, pine))

    forest.add(Tree(200, 300, 1.5, oak))

    forest.forEach { tree ->
        tree.draw()
    }

    println(
        "Unique tree types: " +
        TreeFactory.getTreeTypeCount()
    )
}
```

Output:

text
```
Drawing Pine tree at (10, 20), scale: 1.0, texture: pine.png, color: Green
Drawing Pine tree at (50, 80), scale: 1.2, texture: pine.png, color: Green
Drawing Pine tree at (100, 150), scale: 0.8, texture: pine.png, color: Green
Drawing Oak tree at (200, 300), scale: 1.5, texture: oak.png, color: Dark Green
Unique tree types: 2
```

## Why is this Flyweight?

Consider:

kotlin
```
object TreeFactory
```

Kotlin's `object` declaration creates a singleton factory instance. The factory manages multiple shared tree types.

The singleton is merely how the factory is implemented here. The Flyweight Pattern's central purpose is sharing common state.

Also notice:

kotlin
```
return treeTypes.getOrPut(key) {
    ConcreteTreeType(name, texture, color)
}
```

The factory reuses an existing tree type or creates one if the requested configuration is new.

Interview note: The individual `Tree` objects still exist. Flyweight reduces duplicated shared data; it does not eliminate the need to represent each tree's position and other unique properties.

# 5. How Flyweight Works

The Flyweight Pattern has three main roles.

| Role | Responsibility | Example |
| --- | --- | --- |
| Flyweight | Defines the shared object's interface | `TreeType` |
| Concrete Flyweight | Stores common, reusable state | `ConcreteTreeType` |
| Flyweight Factory | Creates and reuses flyweights | `TreeFactory` |

The individual `Tree` class is the client-side context that stores unique state and uses a shared flyweight.

## Class diagram

text
```
                   Client / Forest
                         |
                         v
                     TreeFactory
                         |
                    Cache / Map
                         |
                         v
                      TreeType
                    (Interface)
                         ^
                         |
                 ConcreteTreeType
                 - Name
                 - Texture
                 - Color
                         ^
                         |
                   Shared by many
                         |
               +---------+---------+
               |         |         |
             Tree 1    Tree 2    Tree 3
               |         |         |
            (x, y)    (x, y)    (x, y)
             scale     scale     scale
```

## Execution flow

mermaid
```
flowchart TD
    A["Create a tree"] --> B["Request tree type"]
    B --> C["TreeFactory"]
    C --> D{"Type already exists?"}
    D -->|Yes| E["Reuse shared type"]
    D -->|No| F["Create and cache type"]
    F --> E
    E --> G["Store unique position and scale"]
    G --> H["Draw tree using shared type"]
```

### Intrinsic vs. extrinsic state

| Intrinsic state | Extrinsic state |
| --- | --- |
| Shared between trees of the same type | Unique to each tree instance |
| Stored in `ConcreteTreeType` | Stored in `Tree` |
| Tree species | X and Y coordinates |
| Texture | Scale |
| Base color | Rotation, if added |

### Memory optimization

Imagine a game with 100,000 trees and only five unique tree types.

Without sharing, each tree could hold its own references and copies of the common configuration data.

With Flyweight, the game can reuse five `TreeType` objects and maintain 100,000 lightweight tree instances containing their unique state.

The exact memory savings depend on object layout, resource sizes, and whether textures and meshes were already shared by the game engine.

# 6. When to Use Flyweight

Use Flyweight when:

- Your application creates a very large number of similar objects.
- Many objects share identical state.
- Shared state consumes significant memory.
- Unique state can be separated from common state.
- Reusing shared objects is safe and practical.

### Common real-world examples

- Video games: Share tree models, textures, and other resources.
- Text editors: Share font and character formatting.
- Maps: Reuse marker icons and visual styles.
- Graphics engines: Share meshes and material definitions.
- Document rendering: Reuse style information among many elements.
- Large simulations: Share immutable configuration data across many entities.

### When should you avoid it?

Avoid Flyweight when:

- The number of objects is small.
- Objects share very little state.
- Separating shared and unique data makes the design unnecessarily complicated.
- Cache management costs more than the memory it saves.
- Shared mutable state introduces difficult concurrency problems.

Main benefit: Reduce memory consumption by sharing common data among many objects.

# 7. Flyweight vs. Singleton vs. Prototype vs. Object Pool

These patterns can all affect object creation and reuse, but they solve different problems.

| Feature | Flyweight | Singleton | Prototype | Object Pool |
| --- | --- | --- | --- | --- |
| Main purpose | Share common state | Ensure one instance | Create objects by cloning | Reuse available instances |
| Object count | One per distinct shared state or key | One per singleton class | New copies as needed | Managed collection of instances |
| Main mechanism | Shared objects and factory/cache | Controlled single instance | Cloning | Borrow and return |
| Primary benefit | Memory efficiency | Single access point to one instance | Avoid expensive initialization | Reduce repeated allocation or setup |
| Example | Shared tree types | Application configuration | Clone a document template | Reuse database connections |

### Easy way to remember

- Flyweight: Share common data.
- Singleton: One instance of a class.
- Prototype: Clone an existing object.
- Object Pool: Reuse available objects.

## Flyweight vs. Singleton

A Flyweight factory can use a singleton instance, but the patterns have different goals.

A singleton ensures one instance of a particular class. A Flyweight system can contain many shared instances, often one for each distinct configuration.

## Flyweight vs. Object Pool

Flyweight objects are shared because their common state can be reused simultaneously.

An object pool typically lends an object to one consumer at a time and reclaims it for later use. Pools are often used for resources such as database connections.

## Flyweight vs. Prototype

Prototype creates new objects by copying existing ones.

Flyweight reduces duplication by reusing shared objects rather than creating another object with the same intrinsic state.

# 8. Common Interview Questions

### Q1. What is the Flyweight Pattern?

A structural design pattern that reduces memory consumption by sharing common state among multiple objects.

### Q2. What problem does Flyweight solve?

It reduces duplicated data and the memory overhead associated with creating large numbers of similar objects.

### Q3. What are intrinsic and extrinsic states?

- Intrinsic state: Common state stored inside the shared object.
- Extrinsic state: Context-specific state supplied externally when the object is used.

In our forest example, the texture is intrinsic, while the tree's position is extrinsic.

### Q4. Why do we need a Flyweight Factory?

The factory manages reusable objects. It checks whether a matching flyweight exists and returns it instead of creating a duplicate.

### Q5. Is Flyweight the same as Singleton?

No. Singleton restricts a class to one instance, while Flyweight shares objects that have common intrinsic state. A Flyweight system can contain many flyweight instances.

### Q6. Is Flyweight a creational or structural pattern?

It is classified as a structural design pattern because it focuses on sharing and organizing objects efficiently.

### Q7. Can Flyweight objects be mutable?

They can technically be mutable, but shared mutable state is risky because a change may affect every client using that object.

Flyweights are generally designed with immutable intrinsic state, while extrinsic state is supplied separately.

### Q8. Does Flyweight reduce the total number of objects to one?

No. It reduces duplicate instances of shared state. There may still be many flyweights and many individual tree instances.

### Q9. What are the advantages of Flyweight?

- Reduces memory consumption.
- Avoids duplicate storage of common data.
- Encourages reuse of immutable objects.
- Can improve efficiency in memory-intensive applications.

### Q10. What are the disadvantages of Flyweight?

- Adds complexity by separating intrinsic and extrinsic state.
- Requires cache or factory management.
- Can introduce thread-safety concerns.
- May increase lookup overhead.
- Is less useful when objects share very little data.

### Q11. How does Flyweight help in game development?

It allows thousands of entities to share common models, textures, and configuration data while retaining their own positions, rotations, and other unique state.

### Q12. Can Flyweight be used in multithreaded applications?

Yes. However, the factory or cache must be designed for concurrent access when multiple threads may create or retrieve flyweights simultaneously. Immutable flyweight objects are generally easier to share safely.

### Q13. Is Java's String pool an example of Flyweight?

String interning shares identical string instances, illustrating the core idea of reusing objects. It is commonly discussed as Flyweight-like behavior, although string pooling is a runtime feature rather than a direct implementation of the complete GoF pattern.

### Q14. Does Flyweight improve time complexity?

Not necessarily. Its primary purpose is memory optimization. Factory lookups and cache operations have their own costs, so performance depends on the implementation and workload.

### Q15. What is the difference between a Flyweight and a regular object?

A regular object may store both common and unique state. A Flyweight separates and shares the common state, requiring the client to supply unique state when using the shared object.

# 9. Quick Revision

- Category: Structural design pattern.
- Purpose: Reduce memory usage by sharing common state.
- Flyweight: Shared object containing intrinsic state.
- Factory: Creates, stores, and reuses flyweights.
- Intrinsic state: Common data stored in the flyweight.
- Extrinsic state: Unique data supplied from outside.
- Java: Commonly implemented with a map-based factory.
- Kotlin: Commonly implemented with a factory object and `getOrPut()`.
- Main benefit: Reduces duplicate state across large numbers of objects.
- Main limitation: Adds complexity when separating shared and unique state.

## One-Line Memory Trick

Flyweight = Share the common data among many objects and keep each object's unique data separate.
