# Mediator Design Pattern — Java & Kotlin

An interview-focused guide to the Mediator Pattern, using a real-world chat room example with message handling, Java and Kotlin implementations, class relationships, comparisons, and common interview questions.

## Table of Contents

1. What Is the Mediator Pattern?
2. Why Do We Need It?
3. Java Implementation
4. Kotlin Implementation
5. How the Mediator Pattern Works
6. Types and Common Use Cases
7. When to Use the Mediator Pattern
8. Mediator vs. Other Patterns
9. Common Interview Questions
10. Quick Revision

# 1. What Is the Mediator Pattern?

The Mediator Pattern is a behavioral design pattern that centralizes communication between multiple objects through a mediator object.

In simple words, instead of objects communicating directly with each other, they communicate through a central object called the Mediator.

### Real-world example: Chat room

Imagine you are building a chat application with three users:

- Alice
- Bob
- Charlie

When Alice sends a message to Bob, she should not need to know how Bob receives it or how the application delivers messages to other users.

Instead, Alice sends the message to a central `ChatRoom` mediator. The chat room decides how to deliver the message.

GitHub - PubNubDevelopers/GettingStarted-Flutter-SDK-Tutorial: Getting Started application to accompany Flutter SDK tutorial

Add an emoji picker to the textarea in two steps | TinyMCE

The communication looks like this:

text

```
          Alice
             |
             v
        +----------+
        | ChatRoom |
        | Mediator |
        +----------+
          /      \
         v        v
       Bob      Charlie
```

For example:

java

```
alice.send("Hello everyone!");
```

Alice sends the message through the mediator. The `ChatRoom` handles delivery to the other participants.

Memory trick: Mediator Pattern = Objects communicate through a central coordinator instead of communicating directly with one another.

# 2. Why Do We Need It?

Suppose you build a chat application without the Mediator Pattern.

Each user directly references the other users.

java

```
class Alice {

    Bob bob;
    Charlie charlie;

    void sendToBob(String message) {
        bob.receive(message);
    }

    void sendToCharlie(String message) {
        charlie.receive(message);
    }
}
```

Bob and Charlie may also need references to Alice and each other.

As the number of users grows, the number of direct relationships increases.

This can lead to:

- Tight coupling between objects.
- Many references between classes.
- Difficult maintenance when communication rules change.
- Complicated testing.
- Changes in one object affecting several others.

## With the Mediator Pattern

Introduce a central object that coordinates communication.

text

```
Without Mediator:

Alice <------> Bob
  ^              ^
  |              |
  v              v
Charlie <----> David

With Mediator:

Alice ----\
Bob -------\
Charlie ----> ChatRoom
David -----/
```

The flow becomes:

1. Alice wants to send a message.
2. Alice calls the mediator.
3. The mediator determines which participants should receive the message.
4. The mediator delivers the message to those participants.
5. The participants do not need direct references to one another.

Important: The mediator does not necessarily perform all the business work itself. Its primary responsibility is coordinating communication between objects.

# 3. Java Implementation

Let's build a chat room using the Mediator Pattern.

## Step 1: Create the Mediator interface

The interface defines the communication contract.

java

```
interface ChatMediator {

    void sendMessage(String message, User sender);
}
```

### What does `ChatMediator` do?

- Defines how users communicate through the mediator.
- Hides the details of message delivery from users.
- Allows different mediator implementations to be created.

The users do not need to know how messages are distributed.

## Step 2: Create the User class

Each user stores a reference to the mediator instead of references to every other user.

java

```
abstract class User {

    protected ChatMediator mediator;
    protected String name;

    public User(ChatMediator mediator, String name) {
        this.mediator = mediator;
        this.name = name;
    }

    public abstract void receive(String message);

    public void send(String message) {
        System.out.println(name + " sends: " + message);
        mediator.sendMessage(message, this);
    }

    public String getName() {
        return name;
    }
}
```

### What does `User` do?

- Stores the user's name.
- Stores a reference to the mediator.
- Sends messages through the mediator.
- Defines how a user receives a message.

Notice this line:

java

```
mediator.sendMessage(message, this);
```

The user delegates message delivery to the mediator. It does not directly call Bob or Charlie.

## Step 3: Create the Concrete User

Now implement the user behavior.

java

```
class ChatUser extends User {

    public ChatUser(ChatMediator mediator, String name) {
        super(mediator, name);
    }

    @Override
    public void receive(String message) {
        System.out.println(name + " receives: " + message);
    }
}
```

Every `ChatUser` can send and receive messages through the mediator.

We can create Alice, Bob, and Charlie using the same class.

## Step 4: Create the Concrete Mediator

The `ChatRoom` coordinates communication between users.

java

```
import java.util.ArrayList;
import java.util.List;

class ChatRoom implements ChatMediator {

    private final List<User> users = new ArrayList<>();

    public void addUser(User user) {
        users.add(user);
    }

    @Override
    public void sendMessage(String message, User sender) {

        for (User user : users) {

            if (user != sender) {
                user.receive(
                    sender.getName() + ": " + message
                );
            }
        }
    }
}
```

### What does `ChatRoom` do?

- Maintains a list of users.
- Receives messages from users.
- Sends messages to all participants except the sender.
- Centralizes the communication logic.

The key point is that the `ChatRoom` knows about the users, but individual users do not need references to each other.

## Step 5: Create the Client

The client creates the mediator and users, registers the users, and starts the communication.

java

```
public class Main {

    public static void main(String[] args) {

        ChatRoom chatRoom = new ChatRoom();

        User alice = new ChatUser(chatRoom, "Alice");
        User bob = new ChatUser(chatRoom, "Bob");
        User charlie = new ChatUser(chatRoom, "Charlie");

        chatRoom.addUser(alice);
        chatRoom.addUser(bob);
        chatRoom.addUser(charlie);

        alice.send("Hello everyone!");
    }
}
```

Output:

text

```
Alice sends: Hello everyone!
Bob receives: Alice: Hello everyone!
Charlie receives: Alice: Hello everyone!
```

### Who is the client?

The `Main` class is the client in this example.

It creates the `ChatRoom`, creates the users, registers them with the mediator, and initiates communication.

The client sets up the objects. The mediator coordinates the communication.

### How does the message travel?

When Alice calls:

java

```
alice.send("Hello everyone!");
```

The flow is:

text

```
1. Main creates Alice, Bob, Charlie, and ChatRoom.
                     |
2. Alice calls send("Hello everyone!").
                     |
3. Alice calls mediator.sendMessage(...).
                     |
4. ChatRoom receives the message.
                     |
5. ChatRoom calls Bob.receive(...).
                     |
6. ChatRoom calls Charlie.receive(...).
```

Alice never directly calls Bob or Charlie.

Interview note: The Mediator Pattern reduces direct dependencies between collaborating objects by centralizing their communication.

# 4. Kotlin Implementation

Kotlin can implement the Mediator Pattern using interfaces and classes.

## Complete implementation

kotlin

```
// Mediator interface
interface ChatMediator {
    fun sendMessage(message: String, sender: User)
    fun addUser(user: User)
}

// Colleague
abstract class User(
    protected val mediator: ChatMediator,
    val name: String
) {
    abstract fun receive(message: String)

    fun send(message: String) {
        println("$name sends: $message")
        mediator.sendMessage(message, this)
    }
}

// Concrete colleague
class ChatUser(
    mediator: ChatMediator,
    name: String
) : User(mediator, name) {

    override fun receive(message: String) {
        println("$name receives: $message")
    }
}

// Concrete mediator
class ChatRoom : ChatMediator {

    private val users = mutableListOf<User>()

    override fun addUser(user: User) {
        users.add(user)
    }

    override fun sendMessage(message: String, sender: User) {
        for (user in users) {
            if (user !== sender) {
                user.receive("${sender.name}: $message")
            }
        }
    }
}

// Client
fun main() {

    val chatRoom = ChatRoom()

    val alice = ChatUser(chatRoom, "Alice")
    val bob = ChatUser(chatRoom, "Bob")
    val charlie = ChatUser(chatRoom, "Charlie")

    chatRoom.addUser(alice)
    chatRoom.addUser(bob)
    chatRoom.addUser(charlie)

    alice.send("Hello everyone!")
}
```

Output:

text

```
Alice sends: Hello everyone!
Bob receives: Alice: Hello everyone!
Charlie receives: Alice: Hello everyone!
```

## Why is this the Mediator Pattern?

Consider:

kotlin

```
protected val mediator: ChatMediator
```

Each user depends on the mediator abstraction rather than on every other user.

Then:

kotlin

```
mediator.sendMessage(message, this)
```

passes the message to the central coordinator.

The mediator decides how the message should be distributed.

Kotlin uses `!==` to check that the recipient is not the same object instance as the sender.

The overall design is the same as the Java implementation.

# 5. How the Mediator Pattern Works

The pattern has three main roles.

| Role | Responsibility | Example |
| --- | --- | --- |
| Mediator | Defines the communication contract | `ChatMediator` |
| Concrete Mediator | Coordinates communication between objects | `ChatRoom` |
| Colleague | Communicates through the mediator | `User`, `ChatUser` |
| Client | Creates and connects the objects | `Main` |

## Class diagram

text

```
                 Client
                   |
                   v
             ChatMediator
              (Interface)
                   ^
                   |
                ChatRoom
              /    |    \
             /     |     \
            v      v      v
          Alice   Bob   Charlie
             \     |     /
              \    |    /
             communicate
              via mediator
```

More precisely:

- `ChatMediator` defines the communication contract.
- `ChatRoom` implements the contract.
- Every `User` holds a reference to the mediator.
- The mediator coordinates communication among users.
- Users do not need direct references to other users.

## Why does the mediator know about all users?

The mediator needs enough information to coordinate communication.

In our example, `ChatRoom` stores the users in a list so it can deliver messages to everyone except the sender.

A different mediator might route messages only to a specific recipient, enforce permissions, or apply notification rules.

## Does the mediator always broadcast messages?

No. That depends on the application.

Common possibilities include:

- Broadcasting a message to all participants.
- Sending a message to one recipient.
- Routing messages based on a user's role.
- Coordinating interactions between UI components.
- Enforcing rules before allowing communication.

The essential idea is that collaborating objects communicate through a central coordinator.

# 6. Types and Common Use Cases

The Mediator Pattern is usually described through its roles and communication behavior rather than a fixed set of formal subtypes.

## 1. Chat room coordination

Users send messages through a chat room rather than directly to one another.

Example: Alice sends a message that the chat room distributes to Bob and Charlie.

## 2. UI component coordination

Multiple UI components interact through a mediator.

Example:

text

```
Checkbox ----\
              \
Text Field ----> FormMediator
              /
Submit Button-/
```

When a checkbox changes, the mediator can enable or disable the submit button and update the form.

The components do not need to reference each other directly.

## 3. Air traffic control

Aircraft communicate with an air traffic control system to coordinate movements and avoid relying solely on direct aircraft-to-aircraft coordination.

This is an analogy for centralized coordination; real aviation communication systems have more complex safety requirements.

## 4. Workflow coordination

A central coordinator manages interactions between multiple services or workflow components.

Example:

text

```
Order Service
      |
      v
Order Mediator
   /       \
  v         v
Payment   Inventory
Service   Service
```

The mediator coordinates when payment and inventory operations should occur.

## 5. Game development

Game objects can communicate through a central event or game mediator.

Example: When a player collects an item, the mediator updates the score, plays a sound, and notifies the UI.

### Common real-world use cases

- Chat applications.
- UI form coordination.
- Dialog boxes and window management.
- Workflow orchestration.
- Game event coordination.
- Communication between loosely coupled components.
- Centralized routing between cooperating objects.

Note that not every centralized service is automatically a Mediator. The pattern is most relevant when it coordinates communication among a group of collaborating objects.

# 7. When to Use the Mediator Pattern

Use this pattern when:

- Many objects communicate directly with one another.
- The number of dependencies between components is becoming difficult to manage.
- You want to centralize communication rules.
- Objects should remain independent of other concrete objects.
- Communication behavior needs to change without modifying every participant.
- You want to coordinate several UI components or workflow components.

### When should you avoid it?

Avoid the Mediator Pattern when:

- Only two objects communicate in a simple, stable relationship.
- The mediator becomes a huge class containing all business logic.
- Communication is already simple and well encapsulated.
- Central coordination introduces unnecessary complexity.
- The mediator becomes a bottleneck or a single point of failure in a distributed system.

Main benefit: Reduces direct dependencies between collaborating objects.

Main limitation: The mediator itself can become overly complex if too much behavior is centralized in one class.

# 8. Mediator vs. Other Patterns

| Feature | Mediator | Observer | Facade | Command | Chain of Responsibility |
| --- | --- | --- | --- | --- | --- |
| Main purpose | Coordinate communication between objects | Notify subscribers of events | Simplify access to a subsystem | Encapsulate an operation | Pass a request through handlers |
| Main focus | Object collaboration | Event notification | Simplified interface | Request representation | Request routing |
| Communication | Through a central coordinator | Publisher notifies observers | Client calls facade | Invoker executes command | Handler forwards request |
| Example | Chat room | Notify users of a ticket update | Simplify access to a payment subsystem | Turn on a light | Escalate a support ticket |

## Mediator vs. Observer

Both patterns help reduce coupling, but they solve different problems.

- Mediator: Coordinates interactions between collaborating objects.
- Observer: Allows one object to notify multiple subscribers when an event occurs.

Example: A chat room routes messages among users through a mediator. A ticket service publishes an update event that notifies subscribed users.

A mediator can use the Observer Pattern internally, but they are not the same pattern.

## Mediator vs. Facade

- Mediator: Coordinates communication between a group of collaborating objects.
- Facade: Provides a simplified interface to a complex subsystem.

Example: A chat room coordinates users. A payment facade provides one simple method to initiate a payment using several internal services.

## Mediator vs. Command

- Mediator: Focuses on communication and coordination between objects.
- Command: Encapsulates an operation as an object.

Example: A chat room routes messages. A `LightOnCommand` represents the operation of turning on a light.

## Mediator vs. Chain of Responsibility

- Mediator: Coordinates communication between participants.
- Chain of Responsibility: Passes a request through a sequence of handlers.

Example: A chat room distributes messages to participants. A support ticket moves from Level 1 to Level 2 if Level 1 cannot handle it.

# 9. Common Interview Questions

### Q1. What is the Mediator Pattern?

It is a behavioral design pattern that centralizes communication between collaborating objects so they do not need direct references to one another.

### Q2. What are the main components?

- Mediator: Defines the communication contract.
- Concrete Mediator: Coordinates the collaborating objects.
- Colleague: Communicates through the mediator.
- Client: Creates and connects the objects.

### Q3. Why do we need the Mediator Pattern?

It reduces tight coupling, centralizes communication rules, and makes collaborating objects easier to maintain independently.

### Q4. Is Mediator a behavioral or structural pattern?

It is a behavioral design pattern because it focuses on how objects communicate and coordinate their behavior.

### Q5. What is a colleague in the Mediator Pattern?

A colleague is an object that communicates with other objects through the mediator.

In our example, Alice, Bob, and Charlie are colleagues.

### Q6. Does the mediator perform all the business logic?

Not necessarily. Its main responsibility is coordinating communication. Business logic should remain in the appropriate components where possible.

### Q7. What are the advantages?

- Reduces direct dependencies between objects.
- Centralizes communication logic.
- Makes individual components easier to reuse.
- Simplifies changes to communication rules.
- Improves testability by isolating coordination behavior.

### Q8. What are the disadvantages?

- The mediator can become a large, complicated class.
- Centralized coordination may become difficult to maintain.
- A poorly designed mediator can become a bottleneck.
- Too much coordination logic can obscure the actual business responsibilities.

### Q9. How is Mediator different from direct communication?

With direct communication, one object calls another object directly. With Mediator, the sender communicates with a central coordinator, which determines how the interaction proceeds.

### Q10. Can there be multiple mediators?

Yes. Different groups of collaborating objects can use different mediators. For example, a chat room can coordinate chat users while a separate form mediator coordinates UI components.

### Q11. Can the Mediator Pattern be implemented in Kotlin?

Yes. Kotlin interfaces, classes, and delegation work well for implementing mediators and colleagues.

### Q12. Is a chat room always an example of the Mediator Pattern?

No. It is an example when the chat room acts as a central coordinator through which users communicate. A chat system that relies entirely on direct communication may use a different design.

### Q13. How does Mediator help with the Open/Closed Principle?

A well-designed mediator can allow new colleague types to participate through an existing abstraction without requiring changes to every other colleague. However, the mediator may still need changes if the new participant introduces new coordination rules.

### Q14. Can the Mediator Pattern be used in distributed systems?

The concept of centralized coordination can be applied in distributed systems, but a classic in-memory Mediator implementation is not automatically a distributed messaging architecture. Distributed implementations must address network failures, concurrency, delivery guarantees, and scalability.

### Q15. Is Mediator the same as a message broker?

No. A mediator coordinates object interactions, typically within an application. A message broker routes messages between applications or services and often provides additional infrastructure such as persistence and delivery guarantees.

### Q16. How can you prevent the mediator from becoming a God object?

- Keep its responsibilities focused on coordination.
- Move business logic into the appropriate services.
- Split unrelated interactions into separate mediators.
- Define clear interfaces for participating objects.
- Test coordination rules independently.

### Q17. Can the Mediator Pattern be combined with Observer?

Yes. A mediator may publish events to observers or use event notifications internally to coordinate participants.

### Q18. Does Mediator improve performance?

Not necessarily. Its main benefit is reducing coupling and simplifying communication. It may add indirection and centralize work, so performance depends on the implementation and workload.

# 10. Quick Revision

- Category: Behavioral design pattern.
- Purpose: Centralize communication between collaborating objects.
- Mediator: Defines the communication contract.
- Concrete Mediator: Coordinates the participants.
- Colleague: Communicates through the mediator.
- Client: Creates and connects the objects.
- Java: Commonly implemented using interfaces and concrete mediator classes.
- Kotlin: Can use interfaces and classes to coordinate communication.
- Main benefit: Reduces direct dependencies between objects.
- Main limitation: The mediator can become overly complex.
- Common examples: Chat rooms, UI coordination, workflow orchestration, and game event coordination.

## One-Line Memory Trick

Mediator Pattern = Instead of objects talking directly to each other, they talk through a central coordinator.
