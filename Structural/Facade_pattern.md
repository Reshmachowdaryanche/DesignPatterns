# Facade Design Pattern — Java & Kotlin

An interview-focused guide to the Facade Pattern, using a real-world home theater example with multiple subsystems, Java and Kotlin implementations, class relationships, comparisons, and common interview questions.

## Table of Contents

1. What Is the Facade Pattern?
2. Why Do We Need It?
3. Java Implementation
4. Kotlin Implementation
5. How Facade Works
6. When to Use Facade
7. Facade vs. Adapter vs. Decorator vs. Proxy
8. Common Interview Questions
9. Quick Revision

# 1. What Is the Facade Pattern?

The Facade Pattern is a structural design pattern that provides a simplified interface to a complex subsystem.

Instead of requiring clients to interact with multiple classes and understand their internal coordination, a facade provides a single entry point that handles the required operations.

### Real-world example: Home theater system

Imagine you want to watch a movie at home.

Your home theater system contains several devices:

- A projector.
- A sound system.
- A media player.
- A screen.
- A lighting system.

home acoustique paris intégration audiovisuel haut de gamme | Home Acoustique Paris

7.2-channel Home Theater Av Receiver Bundled With Active Subwoofer at 68000.00 INR in Vijayawada | Prakash Electronics

Installation home-cinéma : guide complet pour un agencement optimal

Without a facade, you need to control each device individually:

1. Turn on the projector.
2. Configure the projector input.
3. Turn on the sound system.
4. Start the media player.
5. Lower the screen.
6. Dim the lights.

Instead, you can create a `HomeTheaterFacade` that provides a simple method called `watchMovie()`.

Java example:

java
```
HomeTheaterFacade homeTheater =
    new HomeTheaterFacade(
        projector,
        soundSystem,
        mediaPlayer,
        screen,
        lights
    );

homeTheater.watchMovie("Inception");
```

The facade coordinates all the underlying devices.

The client does not need to know the correct sequence of operations.

Memory trick: Facade = Hide subsystem complexity behind a simple interface.

# 2. Why Do We Need It?

Suppose you are building a movie-watching application without the Facade Pattern.

The client might need to execute several operations directly:

java
```
projector.on();
projector.setInput("HDMI");

soundSystem.on();
soundSystem.setVolume(15);

screen.down();

lights.dim(20);

mediaPlayer.on();
mediaPlayer.play("Inception");
```

This works, but every client that wants to watch a movie may need to know the same sequence.

As the system grows, several problems can arise:

- Clients become tightly coupled to subsystem classes.
- The same initialization logic is repeated.
- The correct order of operations becomes the client's responsibility.
- Changes to the subsystem may require changes in multiple clients.

## With Facade

Create a single entry point:

java
```
homeTheater.watchMovie("Inception");
```

The facade internally coordinates the subsystem.

text
```
                 Client
                   |
                   v
           HomeTheaterFacade
                   |
        +----------+----------+
        |          |          |
        v          v          v
    Projector  SoundSystem  MediaPlayer
        |          |          |
        +----------+----------+
                   |
                Screen
                   |
                 Lights
```

The facade does not replace the underlying devices. It simplifies how the client uses them.

Important: Facade is especially useful when a subsystem has multiple classes that must work together to complete a common operation.

# 3. Java Implementation

Let's implement a home theater system in which the facade coordinates the projector, sound system, screen, lights, and media player.

## Step 1: Create the Subsystem Classes

Subsystem classes contain the actual functionality. They do not need to know about the facade.

### Projector

java
```
class Projector {

    public void on() {
        System.out.println("Projector is ON");
    }

    public void setInput(String input) {
        System.out.println("Projector input: " + input);
    }

    public void off() {
        System.out.println("Projector is OFF");
    }
}
```

### Sound System

java
```
class SoundSystem {

    public void on() {
        System.out.println("Sound system is ON");
    }

    public void setVolume(int volume) {
        System.out.println("Volume set to " + volume);
    }

    public void off() {
        System.out.println("Sound system is OFF");
    }
}
```

### Media Player

java
```
class MediaPlayer {

    public void on() {
        System.out.println("Media player is ON");
    }

    public void play(String movie) {
        System.out.println("Playing movie: " + movie);
    }

    public void stop() {
        System.out.println("Stopping movie");
    }

    public void off() {
        System.out.println("Media player is OFF");
    }
}
```

### Screen

java
```
class Screen {

    public void down() {
        System.out.println("Screen is DOWN");
    }

    public void up() {
        System.out.println("Screen is UP");
    }
}
```

### Lights

java
```
class Lights {

    public void dim(int level) {
        System.out.println("Lights dimmed to " + level + "%");
    }

    public void on() {
        System.out.println("Lights are ON");
    }
}
```

Each class controls one part of the system.

Without a facade, the client must coordinate these classes directly.

## Step 2: Create the Facade

The facade holds references to the subsystem objects and provides high-level operations.

java
```
class HomeTheaterFacade {

    private final Projector projector;
    private final SoundSystem soundSystem;
    private final MediaPlayer mediaPlayer;
    private final Screen screen;
    private final Lights lights;

    public HomeTheaterFacade(
            Projector projector,
            SoundSystem soundSystem,
            MediaPlayer mediaPlayer,
            Screen screen,
            Lights lights) {

        this.projector = projector;
        this.soundSystem = soundSystem;
        this.mediaPlayer = mediaPlayer;
        this.screen = screen;
        this.lights = lights;
    }

    public void watchMovie(String movie) {

        System.out.println("Preparing home theater...");

        lights.dim(20);

        screen.down();

        projector.on();
        projector.setInput("HDMI");

        soundSystem.on();
        soundSystem.setVolume(15);

        mediaPlayer.on();
        mediaPlayer.play(movie);
    }

    public void endMovie() {

        System.out.println("Shutting down home theater...");

        mediaPlayer.stop();
        mediaPlayer.off();

        soundSystem.off();
        projector.off();

        screen.up();
        lights.on();
    }
}
```

### How does the facade work?

The most important method is:

java
```
public void watchMovie(String movie)
```

It coordinates all the necessary subsystem operations.

The client no longer needs to remember the setup sequence.

Likewise, `endMovie()` handles the shutdown sequence.

Notice that the facade delegates actual work to the subsystem classes rather than reimplementing their functionality.

## Step 3: Use the Facade

java
```
public class Main {

    public static void main(String[] args) {

        Projector projector = new Projector();
        SoundSystem soundSystem = new SoundSystem();
        MediaPlayer mediaPlayer = new MediaPlayer();
        Screen screen = new Screen();
        Lights lights = new Lights();

        HomeTheaterFacade homeTheater =
            new HomeTheaterFacade(
                projector,
                soundSystem,
                mediaPlayer,
                screen,
                lights
            );

        homeTheater.watchMovie("Inception");

        System.out.println();

        homeTheater.endMovie();
    }
}
```

Output:

text
```
Preparing home theater...
Lights dimmed to 20%
Screen is DOWN
Projector is ON
Projector input: HDMI
Sound system is ON
Volume set to 15
Media player is ON
Playing movie: Inception

Shutting down home theater...
Stopping movie
Media player is OFF
Sound system is OFF
Projector is OFF
Screen is UP
Lights are ON
```

### What happens?

- The client creates the subsystem objects.
- The client passes them to `HomeTheaterFacade`.
- `watchMovie()` coordinates the setup sequence.
- `endMovie()` coordinates the shutdown sequence.
- Each subsystem remains responsible for its own operations.

The client does not need to coordinate every device manually.

# 4. Kotlin Implementation

Kotlin supports the Facade Pattern using classes, constructor parameters, and method delegation.

## Complete Implementation

kotlin
```
// Subsystem
class Projector {

    fun on() {
        println("Projector is ON")
    }

    fun setInput(input: String) {
        println("Projector input: $input")
    }

    fun off() {
        println("Projector is OFF")
    }
}

class SoundSystem {

    fun on() {
        println("Sound system is ON")
    }

    fun setVolume(volume: Int) {
        println("Volume set to $volume")
    }

    fun off() {
        println("Sound system is OFF")
    }
}

class MediaPlayer {

    fun on() {
        println("Media player is ON")
    }

    fun play(movie: String) {
        println("Playing movie: $movie")
    }

    fun stop() {
        println("Stopping movie")
    }

    fun off() {
        println("Media player is OFF")
    }
}

class Screen {

    fun down() {
        println("Screen is DOWN")
    }

    fun up() {
        println("Screen is UP")
    }
}

class Lights {

    fun dim(level: Int) {
        println("Lights dimmed to $level%")
    }

    fun on() {
        println("Lights are ON")
    }
}

// Facade
class HomeTheaterFacade(
    private val projector: Projector,
    private val soundSystem: SoundSystem,
    private val mediaPlayer: MediaPlayer,
    private val screen: Screen,
    private val lights: Lights
) {

    fun watchMovie(movie: String) {

        println("Preparing home theater...")

        lights.dim(20)

        screen.down()

        projector.on()
        projector.setInput("HDMI")

        soundSystem.on()
        soundSystem.setVolume(15)

        mediaPlayer.on()
        mediaPlayer.play(movie)
    }

    fun endMovie() {

        println("Shutting down home theater...")

        mediaPlayer.stop()
        mediaPlayer.off()

        soundSystem.off()
        projector.off()

        screen.up()
        lights.on()
    }
}

fun main() {

    val homeTheater = HomeTheaterFacade(
        projector = Projector(),
        soundSystem = SoundSystem(),
        mediaPlayer = MediaPlayer(),
        screen = Screen(),
        lights = Lights()
    )

    homeTheater.watchMovie("Inception")

    println()

    homeTheater.endMovie()
}
```

Output:

text
```
Preparing home theater...
Lights dimmed to 20%
Screen is DOWN
Projector is ON
Projector input: HDMI
Sound system is ON
Volume set to 15
Media player is ON
Playing movie: Inception

Shutting down home theater...
Stopping movie
Media player is OFF
Sound system is OFF
Projector is OFF
Screen is UP
Lights are ON
```

## Why is this Facade?

Consider the Kotlin constructor:

kotlin
```
class HomeTheaterFacade(
    private val projector: Projector,
    private val soundSystem: SoundSystem,
    private val mediaPlayer: MediaPlayer,
    private val screen: Screen,
    private val lights: Lights
)
```

The facade receives the subsystem objects it needs to coordinate.

Then the client calls:

kotlin
```
homeTheater.watchMovie("Inception")
```

The facade takes responsibility for coordinating the subsystem calls.

The same approach works in Java and Kotlin: the facade simplifies usage through delegation and orchestration, not by replacing the subsystem implementations.

Interview note: A facade does not necessarily make the subsystem inaccessible. Clients can still use subsystem classes directly when they need functionality that the facade does not expose.

# 5. How Facade Works

The Facade Pattern has two main conceptual roles.

| Role | Responsibility | Example |
| --- | --- | --- |
| Facade | Provides a simplified, high-level interface | `HomeTheaterFacade` |
| Subsystem | Performs the underlying operations | `Projector`, `SoundSystem`, `MediaPlayer`, `Screen`, `Lights` |

The client uses the facade instead of coordinating all the subsystem operations itself.

## Class diagram

text
```
                 Client
                   |
                   v
          HomeTheaterFacade
           /    |    |    \
          v     v    v     v
     Projector Sound Media Screen
               System Player
                   |
                  Lights
```

The facade has references to the subsystem objects and coordinates them to implement higher-level operations.

## Execution flow

mermaid
```
flowchart TD
    A["Client calls watchMovie()"] --> B["HomeTheaterFacade"]
    B --> C["Dim the lights"]
    C --> D["Lower the screen"]
    D --> E["Configure the projector"]
    E --> F["Start the sound system"]
    F --> G["Start the media player"]
    G --> H["Movie is playing"]
```

The client only needs to understand the facade's public API. The facade knows which subsystem operations to invoke and in what order.

### Important characteristics

- Simplification: Hides unnecessary implementation details from clients.
- Delegation: Calls subsystem methods to perform actual work.
- Loose coupling: Clients can depend on the facade instead of many subsystem classes.
- Orchestration: Coordinates multiple operations into a meaningful workflow.
- Optional access: Does not prevent clients from accessing the subsystem directly.

# 6. When to Use Facade

Use Facade when:

- A subsystem contains many classes and operations.
- Clients repeatedly perform the same sequence of operations.
- You want a simpler API for a complicated library or module.
- You want to reduce coupling between clients and implementation details.
- You need a high-level entry point for a workflow.

### Common real-world examples

- Home theater: Coordinate the projector, sound system, screen, and lights.
- E-commerce checkout: Coordinate inventory validation, payment, order creation, and notification.
- Banking applications: Provide a single operation for a workflow involving multiple internal services.
- Database access: Hide connection setup, query execution, and resource cleanup behind a simpler API.
- Cloud storage: Hide the sequence needed to upload, validate, and register a file.
- Application startup: Coordinate configuration loading, dependency initialization, and service startup.

### Example: E-commerce checkout

Without a facade, the client might do this:

java
```
inventoryService.checkStock(productId);
paymentService.processPayment(order);
orderService.createOrder(order);
notificationService.sendConfirmation(order);
```

A facade could provide a high-level operation:

java
```
checkoutFacade.checkout(order);
```

The facade coordinates the underlying services. In a real payment workflow, it must also handle failures, retries, and transactional consistency appropriately; a facade alone does not guarantee those properties.

### When should you avoid it?

Avoid Facade when:

- The subsystem is already simple and easy to use.
- The facade merely duplicates methods without simplifying anything.
- You are creating a facade with too many unrelated responsibilities.
- Clients genuinely need direct access to a broad range of subsystem functionality.

Main benefit: Simplifies client code and reduces the need for clients to understand subsystem coordination.

# 7. Facade vs. Adapter vs. Decorator vs. Proxy

These are all structural design patterns, but they solve different problems.

| Feature | Facade | Adapter | Decorator | Proxy |
| --- | --- | --- | --- | --- |
| Main purpose | Simplify a subsystem | Convert an interface | Add responsibilities | Control access |
| Structure | High-level interface over subsystem classes | Wrapper translating between interfaces | Wrapper adding behavior | Representative or stand-in |
| Key relationship | Coordinates subsystem objects | Delegates through a target interface | Wraps a component | Controls or delegates to a subject |
| Interface | Usually simpler than the subsystem APIs | Matches the client's expected interface | Usually matches the wrapped component | Usually matches the subject |
| Example | `HomeTheaterFacade` | Integrate a legacy API | Add buffering to a stream | Lazy-load a resource |

### Easy way to remember

- Facade: Simplify usage.
- Adapter: Make incompatible interfaces compatible.
- Decorator: Add behavior.
- Proxy: Control access.

## Facade vs. Adapter

Both can wrap or delegate to other objects, but their intentions differ.

- Facade provides a simpler interface to a subsystem.
- Adapter converts one interface into another expected by the client.

For example, `HomeTheaterFacade` coordinates several devices behind `watchMovie()`. An adapter would instead make an incompatible device conform to the interface expected by the home theater system.

## Facade vs. Decorator

- Facade reduces complexity by providing a high-level entry point.
- Decorator extends an individual object's behavior while retaining its interface.

A facade typically coordinates several objects; a decorator typically wraps one component to add responsibilities.

## Facade vs. Proxy

- Facade simplifies interactions with a subsystem.
- Proxy represents or controls access to another object, such as through lazy initialization or access control.

Their structures may differ, but their primary distinction is intent.

# 8. Common Interview Questions

### Q1. What is the Facade Pattern?

A structural design pattern that provides a simplified interface to a complex subsystem, hiding the details of coordinating its internal classes.

### Q2. What problem does Facade solve?

It reduces the complexity and coupling that arise when clients must directly coordinate many subsystem classes.

### Q3. What are the main components of Facade?

- Facade: Provides the high-level interface.
- Subsystem classes: Implement the underlying functionality.
- Client: Uses the facade to perform higher-level operations.

### Q4. Does Facade hide the subsystem completely?

No. A facade simplifies access but does not necessarily prevent direct access to subsystem classes. Clients can still use those classes when necessary.

### Q5. Is Facade the same as encapsulation?

No. Encapsulation bundles data and behavior and controls access to an object's internal state. Facade provides a simplified interface to a collection of subsystem functionality. A facade may support encapsulation at the architectural level, but the concepts are different.

### Q6. Does Facade reduce coupling?

Yes, between clients and subsystem implementation details. Clients can depend on the facade instead of coordinating multiple subsystem classes themselves.

However, the facade itself may still be coupled to the concrete subsystem implementations.

### Q7. Can a subsystem have multiple facades?

Yes. Different clients may need different high-level interfaces for the same subsystem.

For example, a home theater might expose one facade for watching movies and another for gaming.

### Q8. Does Facade follow the Single Responsibility Principle?

It can, if the facade coordinates a cohesive set of related operations. A facade that grows to manage unrelated workflows may become a large, difficult-to-maintain class.

### Q9. Is Facade the same as a service layer?

No. A service layer organizes application operations and business logic. A facade focuses on simplifying access to an underlying subsystem. A service layer can use a facade, and a facade may sometimes serve a similar architectural role, but their purposes are not identical.

### Q10. Can a facade contain business logic?

A facade can coordinate workflow steps and enforce high-level sequencing. However, core business rules are usually better kept in the appropriate domain or service classes instead of accumulating inside the facade.

### Q11. Does Facade violate the Open/Closed Principle?

Not inherently. Adding new subsystem functionality does not always require modifying the facade. However, if every new subsystem feature must be exposed through the facade, that facade may need frequent changes.

### Q12. What are the advantages of Facade?

- Simplifies complex interactions.
- Reduces client coupling to subsystem classes.
- Encourages a clean entry point for common workflows.
- Reduces duplicated orchestration logic.
- Makes subsystem usage easier to understand.

### Q13. What are the disadvantages of Facade?

- Can become a large class with too many responsibilities.
- May hide useful subsystem capabilities if its API is too restrictive.
- Can introduce another dependency that must be maintained.
- Does not automatically handle failures, transactions, or concurrency.

### Q14. Is Facade an example of composition?

Typically, yes. A facade holds references to subsystem objects and delegates work to them. However, composition alone does not make a class a facade; its purpose of simplifying subsystem usage is what defines the pattern.

### Q15. Can Facade be used with microservices?

Yes. A facade can provide a unified entry point that coordinates calls to multiple services. In distributed systems, additional considerations include network failures, latency, timeouts, retries, and partial failures. A facade by itself does not solve those problems.

# 9. Quick Revision

- Category: Structural design pattern.
- Purpose: Provide a simplified interface to a complex subsystem.
- Facade: Coordinates subsystem operations.
- Subsystem: Contains the actual functionality.
- Client: Calls the facade instead of orchestrating every subsystem operation.
- Java: Commonly implemented with a class that holds subsystem references.
- Kotlin: Commonly implemented with constructor-injected subsystem objects and delegation.
- Key relationship: The facade uses and coordinates subsystem objects.
- Main benefit: Reduces complexity and coupling for clients.
- Main limitation: A poorly designed facade can become a large class with too many responsibilities.

## One-Line Memory Trick

Facade = Provide one simple entry point to coordinate a complex subsystem.
