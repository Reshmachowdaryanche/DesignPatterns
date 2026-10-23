# Proxy Design Pattern — Java & Kotlin

An interview-focused guide to the Proxy Pattern, using a real-world image loading example with lazy initialization, Java and Kotlin implementations, class relationships, comparisons, and common interview questions.

## Table of Contents

1. What Is the Proxy Pattern?
2. Why Do We Need It?
3. Java Implementation
4. Kotlin Implementation
5. How Proxy Works
6. Types of Proxy
7. When to Use Proxy
8. Proxy vs. Decorator vs. Adapter vs. Facade
9. Common Interview Questions
10. Quick Revision

# 1. What Is the Proxy Pattern?

The Proxy Pattern is a structural design pattern that provides a substitute or placeholder for another object to control access to it.

The proxy implements the same interface as the real object and forwards requests to that object when appropriate.

### Real-world example: Image loading

Imagine you are building a photo gallery application that displays hundreds of high-resolution images.

Loading every image immediately can consume significant memory and slow down the application.

Instead, you can use a proxy object that delays loading an image until the user actually requests it.

Speicher voll durch Fotos & Videos - so räumst du effizient auf – Handy24

Waar doeners blijven dromen - Woningaanbod ENTREE - De Glasfabriek | De Glasfabriek

android - When using uri to load a large local image, it is very slow. How can I optimize it? - Stack Overflow

For example, the application initially creates an `ImageProxy` instead of loading the actual image.

When the user opens an image, the proxy loads the real image and displays it.

Java example:

java
```
Image image = new ImageProxy("mountain.jpg");

// The real image has not been loaded yet.

image.display();
```

The proxy controls when the expensive image-loading operation occurs.

Memory trick: Proxy = A substitute object that controls access to the real object.

# 2. Why Do We Need It?

Suppose your application loads an image immediately whenever an image object is created.

java
```
class RealImage {

    private final String filename;

    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }

    private void loadFromDisk() {
        System.out.println(
            "Loading image: " + filename
        );
    }

    public void display() {
        System.out.println(
            "Displaying image: " + filename
        );
    }
}
```

Now imagine creating 100 image objects:

java
```
RealImage image1 = new RealImage("photo1.jpg");
RealImage image2 = new RealImage("photo2.jpg");
RealImage image3 = new RealImage("photo3.jpg");
```

Each constructor loads an image immediately, even if the user never views it.

This can lead to:

- Unnecessary disk or network operations.
- Increased memory consumption.
- Slower application startup.
- Wasted resources for objects that are never used.

## With Proxy

Introduce a common interface implemented by both the proxy and the real object.

text
```
             Client
                |
                v
             Image
           (Interface)
                ^
                |
        +-------+-------+
        |               |
     RealImage       ImageProxy
        ^               |
        |               |
        +<--------------+
          Loads and
          delegates to
```

The proxy can decide when to create the real object, whether to allow an operation, or how to handle the request.

For lazy loading, the flow is:

1. The client creates an `ImageProxy`.
2. The proxy stores the image filename.
3. The client calls `display()`.
4. The proxy creates the `RealImage` if it has not already been created.
5. The real image loads and displays the image.

Important: A proxy controls access to an object; it does not necessarily add new behavior to the object itself.

# 3. Java Implementation

Let's build a photo gallery using the Proxy Pattern.

## Step 1: Define the Subject Interface

The Subject defines the common operations that the real object and proxy must support.

java
```
interface Image {
    void display();
}
```

The client will use this interface without needing to know whether it is interacting with the real image or a proxy.

## Step 2: Create the Real Subject

The Real Subject performs the actual work.

java
```
class RealImage implements Image {

    private final String filename;

    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }

    private void loadFromDisk() {
        System.out.println(
            "Loading image: " + filename
        );
    }

    @Override
    public void display() {
        System.out.println(
            "Displaying image: " + filename
        );
    }
}
```

### What does `RealImage` do?

- Loads the image from disk.
- Performs the actual display operation.
- Represents the expensive object that we want to access through a proxy.

Notice that loading happens in the constructor.

Therefore, creating a `RealImage` is potentially expensive.

## Step 3: Create the Proxy

The Proxy implements the same interface as the Real Subject.

java
```
class ImageProxy implements Image {

    private final String filename;
    private RealImage realImage;

    public ImageProxy(String filename) {
        this.filename = filename;
    }

    @Override
    public void display() {

        if (realImage == null) {
            realImage = new RealImage(filename);
        }

        realImage.display();
    }
}
```

### How does the proxy work?

The key part is:

java
```
if (realImage == null) {
    realImage = new RealImage(filename);
}
```

When `display()` is called for the first time, the proxy creates the real image.

On subsequent calls, it reuses the same `RealImage` object.

The proxy therefore provides lazy initialization.

## Step 4: Use the Proxy

java
```
public class Main {

    public static void main(String[] args) {

        Image image =
            new ImageProxy("mountain.jpg");

        System.out.println("Image proxy created");

        image.display();

        System.out.println("Displaying again");

        image.display();
    }
}
```

Output:

text
```
Image proxy created
Loading image: mountain.jpg
Displaying image: mountain.jpg
Displaying again
Displaying image: mountain.jpg
```

### What happens?

- Creating `ImageProxy` does not load the image.
- The first `display()` call creates `RealImage`.
- The image is loaded and displayed.
- The second `display()` call reuses the already-loaded image.
- The client interacts only with the `Image` interface.

This avoids loading images that are never displayed.

Interview note: This example demonstrates a Virtual Proxy, which delays the creation of an expensive object until it is needed.

# 4. Kotlin Implementation

Kotlin supports the Proxy Pattern using interfaces, classes, and nullable properties.

## Complete Implementation

kotlin
```
// Subject interface
interface Image {
    fun display()
}

// Real Subject
class RealImage(
    private val filename: String
) : Image {

    init {
        loadFromDisk()
    }

    private fun loadFromDisk() {
        println("Loading image: $filename")
    }

    override fun display() {
        println("Displaying image: $filename")
    }
}

// Proxy
class ImageProxy(
    private val filename: String
) : Image {

    private var realImage: RealImage? = null

    override fun display() {

        if (realImage == null) {
            realImage = RealImage(filename)
        }

        realImage!!.display()
    }
}

fun main() {

    val image: Image =
        ImageProxy("mountain.jpg")

    println("Image proxy created")

    image.display()

    println("Displaying again")

    image.display()
}
```

Output:

text
```
Image proxy created
Loading image: mountain.jpg
Displaying image: mountain.jpg
Displaying again
Displaying image: mountain.jpg
```

## Why is this Proxy?

Consider:

kotlin
```
private var realImage: RealImage? = null
```

The real image is initially absent.

When `display()` is called, the proxy initializes it only if necessary.

Kotlin also allows the same client-facing interface to be used for both objects:

kotlin
```
val image: Image = ImageProxy("mountain.jpg")
```

The client does not need to know that the underlying image is loaded lazily.

### Production consideration

The example above is suitable for learning but is not thread-safe. If multiple threads call `display()` simultaneously, they could create multiple `RealImage` instances. A production implementation should use appropriate synchronization or a thread-safe lazy initialization mechanism.

# 5. How Proxy Works

The Proxy Pattern has three main roles.

| Role | Responsibility | Example |
| --- | --- | --- |
| Subject | Defines the common interface | `Image` |
| Real Subject | Performs the actual operation | `RealImage` |
| Proxy | Controls access to the real subject | `ImageProxy` |

## Class diagram

text
```
                  Client
                    |
                    v
                  Image
                (Interface)
                    ^
                    |
           +--------+--------+
           |                 |
        RealImage         ImageProxy
           |                 |
           |                 | references
           |                 v
           |              RealImage
           |
       Loads image
       Displays image
```

The proxy and real subject implement the same interface. The proxy also holds a reference to the real subject and delegates requests to it.

## Execution flow

mermaid
```
flowchart TD
    A["Client creates ImageProxy"] --> B["Client calls display()"]
    B --> C{"Real image exists?"}
    C -->|No| D["Create RealImage"]
    D --> E["Load image from disk"]
    E --> F["Delegate display()"]
    C -->|Yes| F
    F --> G["Display image"]
```

### Why does the client use an interface?

The client can work with either implementation:

java
```
Image image = new RealImage("photo.jpg");
```

Or:

java
```
Image image = new ImageProxy("photo.jpg");
```

Both support `display()`.

This allows the implementation to change from a real object to a proxy without requiring changes to code that depends only on the `Image` interface.

# 6. Types of Proxy

There are several common types of proxies. They share the same general idea but control access for different reasons.

Divi Torque Pro | Divi Extensions

### 1. Virtual Proxy

Delays creating an expensive object until it is needed.

Example: Loading a high-resolution image only when the user opens it.

How to Ensure Donor Data Privacy in Sponsorship Programs

### 2. Protection Proxy

Controls access based on permissions or security rules.

Example: Allowing only administrators to delete a record.

General 4 — Far North Digital

### 3. Remote Proxy

Represents an object that exists in another process or on another machine.

Example: A client-side object that forwards calls to a remote service.

Application Insights Dashboard instellen en aanpassen - Azure Monitor | Microsoft Learn

### 4. Logging or Monitoring Proxy

Records requests or measures execution time before or after delegating.

Example: Logging service calls for debugging and monitoring.

Bot Verification

### 5. Caching Proxy

Stores results and reuses them for repeated requests.

Example: Returning a cached API response instead of calling the remote service again.

These are common conceptual categories. A proxy may combine more than one responsibility, although keeping access control, caching, and logging separate can make a system easier to maintain.

# 7. When to Use Proxy

Use the Proxy Pattern when:

- Creating or accessing an object is expensive.
- You need lazy initialization.
- Access to an object must be controlled.
- An object exists remotely.
- You want caching, logging, or monitoring around an existing object.
- You need to hide the details of how the real object is accessed.

### Common real-world examples

- Image galleries: Load large images only when required.
- Security systems: Restrict operations based on user permissions.
- Network clients: Represent remote services.
- API clients: Cache selected responses.
- Database applications: Control access to database-related objects.
- Logging systems: Record calls without modifying the real service.
- Virtual file systems: Represent resources that are fetched or loaded only when accessed.

### When should you avoid it?

Avoid Proxy when:

- The real object is inexpensive and easy to use.
- The proxy introduces unnecessary complexity.
- Delegation or indirection makes debugging harder.
- Caching or lazy initialization adds no measurable benefit.
- The proxy's behavior changes the expected contract of the underlying object.

Main benefit: Controls access to an object without requiring the client to interact directly with the real implementation.

# 8. Proxy vs. Decorator vs. Adapter vs. Facade

These patterns are all structural design patterns, but they solve different problems.

| Feature | Proxy | Decorator | Adapter | Facade |
| --- | --- | --- | --- | --- |
| Main purpose | Control access to an object | Add responsibilities or behavior | Convert an incompatible interface | Simplify a complex subsystem |
| Key relationship | Holds and delegates to real object | Wraps a component | Translates one interface to another | Coordinates subsystem classes |
| Interface | Usually same as real subject | Usually same as wrapped component | Exposes the interface expected by the client | Provides a simplified interface |
| Example | Lazy image loading | Add compression to a stream | Adapt an old payment API | Simplify an order-processing workflow |
| Main focus | Access control | Additional behavior | Compatibility | Simplification |

### Easy way to remember

- Proxy: Controls access to an object.
- Decorator: Adds behavior to an object.
- Adapter: Makes incompatible interfaces work together.
- Facade: Simplifies access to a complex subsystem.

## Proxy vs. Decorator

Both patterns commonly wrap another object and implement the same interface.

The difference is their intent:

- Proxy controls access to the real object.
- Decorator adds responsibilities while preserving the component interface.

For example, an image proxy delays loading the image. An image decorator might add a border or watermark when displaying it.

## Proxy vs. Facade

A proxy represents and controls access to a particular object. A facade provides a simplified entry point to a collection of subsystem components.

For example, an `ImageProxy` controls access to one image object, while a `MediaLibraryFacade` might coordinate image loading, metadata retrieval, and thumbnail generation.

# 9. Common Interview Questions

### Q1. What is the Proxy Pattern?

A structural design pattern that provides a substitute for another object to control access to it.

### Q2. What are the main components of Proxy?

- Subject: Common interface.
- Real Subject: Performs the actual work.
- Proxy: Controls access and delegates operations.

### Q3. Why does Proxy implement the same interface as the Real Subject?

It allows the client to use the proxy and real object interchangeably through the same abstraction.

### Q4. What is lazy initialization?

Lazy initialization delays creating an object until it is first needed. A virtual proxy commonly uses this technique.

### Q5. What is the difference between Proxy and Decorator?

Proxy primarily controls access to an object. Decorator primarily adds responsibilities or behavior to an object.

Their structures can look similar, so the main distinction is their intent.

### Q6. What is a Protection Proxy?

A proxy that checks permissions or security conditions before allowing operations on the real object.

### Q7. What is a Remote Proxy?

A local representative of an object that exists in another process or on a remote machine. It handles communication with the remote object.

### Q8. What is a Caching Proxy?

A proxy that stores results of previous requests and reuses them when appropriate to avoid repeated work.

### Q9. What are the advantages of Proxy?

- Supports lazy initialization.
- Provides access control.
- Can add caching and logging.
- Hides details of remote access.
- Allows clients to use a common interface.

### Q10. What are the disadvantages of Proxy?

- Adds another layer of indirection.
- Can increase complexity.
- May introduce latency.
- Requires careful cache invalidation when caching is involved.
- Can introduce concurrency issues if shared state is not managed safely.

### Q11. Is Proxy a creational or structural pattern?

Proxy is a structural design pattern. Although it may control object creation, its main purpose is to control access to an object.

### Q12. Does Proxy always create the real object lazily?

No. Lazy creation is common in virtual proxies, but other proxies may create or access the real object immediately. Protection proxies and remote proxies, for example, may focus on security or communication instead.

### Q13. Can Proxy be used with Spring Framework?

Yes. Spring uses proxies for features such as declarative transaction management, method interception, and certain forms of aspect-oriented programming. Depending on configuration, these proxies can be implemented using JDK dynamic proxies or class-based proxies.

### Q14. What is the difference between a static proxy and a dynamic proxy?

- Static proxy: A developer writes a specific proxy class.
- Dynamic proxy: A proxy is generated at runtime, often based on an interface or class-based proxy mechanism.

The Java example in this guide uses a static proxy because `ImageProxy` is explicitly written.

### Q15. Does a Proxy Pattern improve performance?

Sometimes. Lazy loading and caching can improve efficiency, but delegation adds overhead. The outcome depends on the purpose of the proxy and how frequently it is used.

# 10. Quick Revision

- Category: Structural design pattern.
- Purpose: Control access to another object.
- Subject: Common interface.
- Real Subject: Performs the actual work.
- Proxy: Intercepts requests and delegates them when appropriate.
- Java: Commonly implemented using an interface and a wrapper class.
- Kotlin: Commonly implemented using an interface and a delegating class.
- Virtual Proxy: Delays expensive object creation.
- Protection Proxy: Controls access using permissions.
- Remote Proxy: Represents a remote object.
- Caching Proxy: Reuses stored results.
- Main benefit: Controls access without requiring changes to client code.
- Main limitation: Adds indirection and implementation complexity.

## One-Line Memory Trick

Proxy = Use a substitute object to control when, how, or whether the real object is accessed.
