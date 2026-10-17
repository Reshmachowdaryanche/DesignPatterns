# Composite Design Pattern — Java & Kotlin

An interview-focused guide to the Composite Pattern, using a real-world file system example with files and folders, Java and Kotlin implementations, class relationships, comparisons, and common interview questions.

## Table of Contents

1. What Is the Composite Pattern?
2. Why Do We Need It?
3. Java Implementation
4. Kotlin Implementation
5. How Composite Works
6. When to Use Composite
7. Composite vs. Decorator vs. Bridge
8. Common Interview Questions
9. Quick Revision

# 1. What Is the Composite Pattern?

The Composite Pattern is a structural design pattern that lets you treat individual objects and groups of objects uniformly through a common interface.

### Real-world example: Files and folders

Think about a computer's file system:

- A file contains data.
- A folder contains files and other folders.
- A folder can contain another folder, creating a tree structure.

Even though files and folders behave differently, both can be treated as file-system items.

Zipping Large Size S3 Folders and Files Using Node.js Lambda And EFS

PCMan File Manager

Unix Commands and Files

For example, you might want to display the names of all items in a folder without caring whether each item is a file or another folder.

java
```
FileSystemItem item = new Folder("Documents");
item.showDetails();
```

The folder can perform the operation on its children recursively.

Memory trick: Composite = Treat an individual object and a group of objects the same way.

# 2. Why Do We Need It?

Suppose you are building a file management application.

Without the Composite Pattern, you might write separate logic for files and folders:

java
```
if (item instanceof File) {
    ((File) item).showDetails();
} else if (item instanceof Folder) {
    ((Folder) item).showDetails();
}
```

As the application grows, you may need similar checks whenever you display, calculate, or process files and folders.

This makes the client code more complicated.

## With Composite

Define a common interface:

java
```
interface FileSystemItem {
    void showDetails();
}
```

Both files and folders implement it.

text
```
FileSystemItem
      |
      +---- File
      |
      +---- Folder
                 |
                 +---- File
                 +---- Folder
                           |
                           +---- File
```

Now the client can call the same method regardless of whether the object represents a file or a folder.

Important: Composite is especially useful when objects naturally form a tree, and operations should apply uniformly to individual elements and groups.

# 3. Java Implementation

Let's build a file system where folders can contain files and other folders.

## Step 1: Define the Component Interface

The Component defines the common operations available to both individual objects and composites.

java
```
interface FileSystemItem {
    void showDetails();
}
```

This is the interface that the client will use.

## Step 2: Create the Leaf

A Leaf is an individual object that does not contain other components.

java
```
class File implements FileSystemItem {

    private final String name;

    public File(String name) {
        this.name = name;
    }

    @Override
    public void showDetails() {
        System.out.println("File: " + name);
    }
}
```

The `File` class implements the common interface but does not contain children.

## Step 3: Create the Composite

A Composite contains other components. Those components may be leaves or other composites.

java
```
import java.util.ArrayList;
import java.util.List;

class Folder implements FileSystemItem {

    private final String name;
    private final List<FileSystemItem> children =
        new ArrayList<>();

    public Folder(String name) {
        this.name = name;
    }

    public void add(FileSystemItem item) {
        children.add(item);
    }

    public void remove(FileSystemItem item) {
        children.remove(item);
    }

    @Override
    public void showDetails() {
        System.out.println("Folder: " + name);

        for (FileSystemItem item : children) {
            item.showDetails();
        }
    }
}
```

### How does the folder work?

The key part is:

java
```
for (FileSystemItem item : children) {
    item.showDetails();
}
```

The folder does not need to know whether each child is a file or another folder.

It simply calls `showDetails()` on each child.

- If the child is a file, the file displays its name.
- If the child is a folder, that folder displays its name and processes its own children.

This creates recursive tree traversal.

## Step 4: Use the Composite

java
```
public class Main {

    public static void main(String[] args) {

        File file1 = new File("resume.pdf");
        File file2 = new File("photo.png");
        File file3 = new File("notes.txt");

        Folder documents = new Folder("Documents");
        Folder pictures = new Folder("Pictures");
        Folder root = new Folder("Root");

        documents.add(file1);
        documents.add(file3);

        pictures.add(file2);

        root.add(documents);
        root.add(pictures);

        root.showDetails();
    }
}
```

Output:

text
```
Folder: Root
Folder: Documents
File: resume.pdf
File: notes.txt
Folder: Pictures
File: photo.png
```

### What happens?

- `File` is a leaf.
- `Folder` is a composite.
- Both implement `FileSystemItem`.
- A folder stores children through the common interface.
- Calling `showDetails()` on the root recursively processes the entire tree.

The client does not need to traverse every level manually.

# 4. Kotlin Implementation

Kotlin supports the Composite Pattern using interfaces, collections, and recursive method calls.

## Implementation

kotlin
```
// Component
interface FileSystemItem {
    fun showDetails()
}

// Leaf
class File(
    private val name: String
) : FileSystemItem {

    override fun showDetails() {
        println("File: $name")
    }
}

// Composite
class Folder(
    private val name: String
) : FileSystemItem {

    private val children =
        mutableListOf<FileSystemItem>()

    fun add(item: FileSystemItem) {
        children.add(item)
    }

    fun remove(item: FileSystemItem) {
        children.remove(item)
    }

    override fun showDetails() {
        println("Folder: $name")

        children.forEach { item ->
            item.showDetails()
        }
    }
}

fun main() {

    val file1 = File("resume.pdf")
    val file2 = File("photo.png")
    val file3 = File("notes.txt")

    val documents = Folder("Documents")
    val pictures = Folder("Pictures")
    val root = Folder("Root")

    documents.add(file1)
    documents.add(file3)

    pictures.add(file2)

    root.add(documents)
    root.add(pictures)

    root.showDetails()
}
```

Output:

text
```
Folder: Root
Folder: Documents
File: resume.pdf
File: notes.txt
Folder: Pictures
File: photo.png
```

## Why is this Composite?

kotlin
```
private val children =
    mutableListOf<FileSystemItem>()
```

The folder stores `FileSystemItem` objects rather than maintaining separate lists for files and folders.

That means the same collection can contain:

kotlin
```
Folder("Documents")
File("resume.pdf")
```

The common interface allows the folder to process both uniformly.

Interview note: The Composite Pattern does not require the component interface to declare every possible operation. Operations such as `add()` and `remove()` may be declared only on the composite, as shown here.

# 5. How Composite Works

The Composite Pattern has three main roles.

| Role | Responsibility | Example |
| --- | --- | --- |
| Component | Defines the common interface | `FileSystemItem` |
| Leaf | Represents an individual object | `File` |
| Composite | Stores and manages child components | `Folder` |

## Class diagram

text
```
             FileSystemItem
                    |
          +---------+---------+
          |                   |
         File               Folder
       (Leaf)             (Composite)
                               |
                               | contains
                               v
                      List<FileSystemItem>
                               |
                         +-----+-----+
                         |           |
                        File       Folder
```

## Execution flow

Client calls `root.showDetails()`

Root folder displays its name

Iterate through child components

File

Displays its name

Nested folder

Processes its children

The important idea is recursive composition: a composite can contain leaves, other composites, or both.

# 6. When to Use Composite

Use Composite when:

- Your objects naturally form a tree structure.
- You need to treat individual objects and groups uniformly.
- Operations should be applied recursively to nested groups.
- You want clients to avoid writing separate logic for leaves and composites.

### Common real-world examples

- File systems: Files and folders.
- Organization charts: Employees and departments.
- UI components: Individual controls and nested layouts.
- Menus: Menu items and submenus.
- Graphics: Individual shapes and groups of shapes.
- Product categories: Individual products and nested categories.

### When should you avoid it?

Avoid Composite when your objects do not naturally form a hierarchy or when leaf and composite operations are fundamentally different.

Also, be careful with cyclic references. A folder containing itself, directly or indirectly, can cause infinite recursion.

Main benefit: The client can process a whole hierarchy through one common interface.

# 7. Composite vs. Decorator vs. Bridge

These are all structural design patterns, but they solve different problems.

| Feature | Composite | Decorator | Bridge |
| --- | --- | --- | --- |
| Main purpose | Treat individual objects and groups uniformly | Add behavior to an object | Separate abstraction from implementation |
| Structure | Tree of components | Wrapper around a component | Two independent hierarchies |
| Key relationship | Contains children | Wraps a component | Abstraction holds an implementor |
| Example | Folder containing files and folders | Add compression to a data stream | Remote control and device |
| Main focus | Part-whole hierarchy | Additional responsibilities | Independent variation |

### Easy way to remember

- Composite: One object or a group of objects.
- Decorator: An object with additional behavior.
- Bridge: Two dimensions that can vary independently.

# 8. Common Interview Questions

### Q1. What is the Composite Pattern?

A structural design pattern that allows individual objects and groups of objects to be treated uniformly through a common interface.

### Q2. What are the three main components?

- Component: Common interface.
- Leaf: Individual object without children.
- Composite: Object that contains child components.

### Q3. Why does Composite use recursion?

A composite can contain other composites. Calling an operation on the root and delegating it to children naturally processes the nested tree.

### Q4. Is Composite always implemented using a tree?

It is typically used for tree-like part-whole hierarchies. The design should avoid cycles if operations rely on recursive traversal.

### Q5. What is the difference between Leaf and Composite?

A leaf represents an individual object and does not contain children. A composite contains and manages child components.

### Q6. Why do we store children using the component interface?

It allows a composite to store leaves and other composites in the same collection without needing to know their concrete classes.

### Q7. Should `add()` and `remove()` be part of the component interface?

Not necessarily.

- In a transparent design, these operations are declared on the component interface, so clients can use them uniformly. However, leaf classes may need to reject unsupported operations.
- In a safe design, these operations exist only on composites, as in our example. This avoids exposing meaningless operations on leaves but requires clients to know when they have a composite.

Both approaches are valid, depending on the API design.

### Q8. What are the advantages of Composite?

- Simplifies client code.
- Supports recursive structures.
- Makes it easier to add new component types.
- Treats leaves and composites uniformly.

### Q9. What are the disadvantages of Composite?

- The common interface can be difficult to design.
- Type-specific operations may require additional checks.
- Recursive operations can be expensive for large trees.
- Cyclic references can cause infinite recursion.

### Q10. Does Composite follow SOLID principles?

It can support the Open/Closed Principle by allowing new component types to be added without changing existing traversal code. However, the design must still respect each component's responsibilities and contracts.

### Q11. Can Composite calculate the total size of a folder?

Yes. You can define a `getSize()` operation in the component interface. A file returns its own size, while a folder returns the sum of its children's sizes.

### Q12. Is Composite the same as inheritance?

No. Inheritance is used to implement the common component interface, but Composite specifically organizes objects through a part-whole hierarchy using composition.

# 9. Quick Revision

- Category: Structural design pattern.
- Purpose: Treat individual objects and groups uniformly.
- Component: Common interface.
- Leaf: Individual object without children.
- Composite: Contains child components.
- Java: Commonly implemented with interfaces and collections.
- Kotlin: Commonly implemented with interfaces and mutable lists.
- Main benefit: Simplifies operations on hierarchical structures.
- Main limitation: Recursive processing and component-interface design can become complicated.

## One-Line Memory Trick

Composite = Build a tree of objects and treat each individual object or group through the same interface.
