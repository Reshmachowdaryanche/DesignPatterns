# Chain of Responsibility Design Pattern — Java & Kotlin

An interview-focused guide to the Chain of Responsibility Pattern, using a real-world support ticket example with request handling, Java and Kotlin implementations, class relationships, comparisons, and common interview questions.

## Table of Contents

1. What Is the Chain of Responsibility Pattern?
2. Why Do We Need It?
3. Java Implementation
4. Kotlin Implementation
5. How Chain of Responsibility Works
6. Types and Common Use Cases
7. When to Use Chain of Responsibility
8. Chain of Responsibility vs. Other Patterns
9. Common Interview Questions
10. Quick Revision

# 1. What Is the Chain of Responsibility Pattern?

The Chain of Responsibility Pattern is a behavioral design pattern that passes a request along a chain of handlers. Each handler decides whether to process the request, pass it to the next handler, or stop the chain.

The sender of the request does not need to know which handler will process it.

### Real-world example: Customer support system

Imagine you are building a customer support application.

Different support agents handle different types of issues:

- Level 1 Support handles basic issues.
- Level 2 Support handles technical issues.
- Level 3 Support handles critical or advanced issues.

When a customer submits a ticket, the application sends it to the first handler. If that handler cannot resolve it, the request moves to the next handler.

CCI

IT Help Desk Course Certificate | Free & Fast Course

Cisco warns of active exploit in high-severity SD-WAN vulnerability with no patch yet | Noah Intelligence

For example:

java

```
SupportHandler support =
    new Level1Support();

support.handleRequest("CRITICAL");
```

The request may travel through Level 1, Level 2, and Level 3 until an appropriate handler processes it.

Memory trick: Chain of Responsibility = Pass a request along a chain until a handler takes responsibility for it.

# 2. Why Do We Need It?

Suppose you implement a support system using multiple `if-else` statements.

java

```
class SupportSystem {

    public void handleRequest(String type) {

        if (type.equals("BASIC")) {
            System.out.println(
                "Level 1 handles the issue"
            );

        } else if (type.equals("TECHNICAL")) {
            System.out.println(
                "Level 2 handles the issue"
            );

        } else if (type.equals("CRITICAL")) {
            System.out.println(
                "Level 3 handles the issue"
            );
        }
    }
}
```

This works initially, but as the application grows, more conditions and responsibilities are added.

For example, you might need to support billing, security, networking, and database issues.

This can lead to:

- Large conditional blocks.
- Tight coupling between request routing and business logic.
- Difficult maintenance when handlers change.
- More complex testing.
- Difficulty reordering or extending request processing.

## With Chain of Responsibility

Instead of placing all logic in one class, create separate handlers and connect them into a chain.

text

```
             Client
                |
                v
         Level 1 Support
                |
                v
         Level 2 Support
                |
                v
         Level 3 Support
                |
                v
            End of Chain
```

Each handler either processes the request or forwards it to the next handler.

The flow is:

1. The client creates a request.
2. The request enters the first handler.
3. The handler checks whether it can process the request.
4. If it can, it handles the request.
5. Otherwise, it forwards the request to the next handler.
6. The chain ends when a handler processes the request or no handlers remain.

Important: Unlike a simple `if-else` chain, each handler owns its own processing logic and does not need to know the internal details of the other handlers.

# 3. Java Implementation

Let's build a customer support system using the Chain of Responsibility Pattern.

## Step 1: Create the Base Handler

The base handler defines a common interface for processing requests and stores a reference to the next handler.

java

```
abstract class SupportHandler {

    protected SupportHandler nextHandler;

    public void setNextHandler(
            SupportHandler nextHandler) {
        this.nextHandler = nextHandler;
    }

    public abstract void handleRequest(
            String issueType);
}
```

### What does `SupportHandler` do?

- Defines the common request-handling method.
- Stores the next handler in the chain.
- Allows concrete handlers to implement their own logic.
- Provides a common abstraction for all support levels.

The important field is:

java

```
protected SupportHandler nextHandler;
```

It connects one handler to another.

## Step 2: Create Level 1 Support

Level 1 handles basic issues. Other issues are passed to the next handler.

java

```
class Level1Support extends SupportHandler {

    @Override
    public void handleRequest(String issueType) {

        if (issueType.equals("BASIC")) {

            System.out.println(
                "Level 1 handled the basic issue"
            );

        } else if (nextHandler != null) {

            nextHandler.handleRequest(issueType);

        } else {

            System.out.println(
                "No handler available for: " + issueType
            );
        }
    }
}
```

### What happens here?

- If the issue is `BASIC`, Level 1 handles it.
- Otherwise, it forwards the request to the next handler.
- If there is no next handler, the request remains unhandled.

## Step 3: Create Level 2 Support

Level 2 handles technical issues.

java

```
class Level2Support extends SupportHandler {

    @Override
    public void handleRequest(String issueType) {

        if (issueType.equals("TECHNICAL")) {

            System.out.println(
                "Level 2 handled the technical issue"
            );

        } else if (nextHandler != null) {

            nextHandler.handleRequest(issueType);

        } else {

            System.out.println(
                "No handler available for: " + issueType
            );
        }
    }
}
```

Level 2 follows the same structure but has different processing logic.

## Step 4: Create Level 3 Support

Level 3 handles critical issues.

java

```
class Level3Support extends SupportHandler {

    @Override
    public void handleRequest(String issueType) {

        if (issueType.equals("CRITICAL")) {

            System.out.println(
                "Level 3 handled the critical issue"
            );

        } else if (nextHandler != null) {

            nextHandler.handleRequest(issueType);

        } else {

            System.out.println(
                "No handler available for: " + issueType
            );
        }
    }
}
```

Each handler is responsible for its own issue type.

## Step 5: Connect the Chain

Create the handlers and connect them in order.

java

```
public class Main {

    public static void main(String[] args) {

        SupportHandler level1 =
            new Level1Support();

        SupportHandler level2 =
            new Level2Support();

        SupportHandler level3 =
            new Level3Support();

        level1.setNextHandler(level2);
        level2.setNextHandler(level3);

        level1.handleRequest("BASIC");

        level1.handleRequest("TECHNICAL");

        level1.handleRequest("CRITICAL");

        level1.handleRequest("BILLING");
    }
}
```

Output:

text

```
Level 1 handled the basic issue
Level 2 handled the technical issue
Level 3 handled the critical issue
No handler available for: BILLING
```

### How does the request move?

For `BASIC`:

text

```
Client → Level 1 → Handled
```

For `TECHNICAL`:

text

```
Client → Level 1 → Level 2 → Handled
```

For `CRITICAL`:

text

```
Client → Level 1 → Level 2 → Level 3 → Handled
```

For `BILLING`:

text

```
Client → Level 1 → Level 2 → Level 3 → Unhandled
```

Interview note: The client starts with the first handler and does not need to know which handler will ultimately process the request.

# 4. Kotlin Implementation

Kotlin supports the Chain of Responsibility Pattern using abstract classes or interfaces and references to subsequent handlers.

## Complete Implementation

kotlin

```
// Base handler
abstract class SupportHandler {

    protected var nextHandler: SupportHandler? = null

    fun setNextHandler(handler: SupportHandler) {
        nextHandler = handler
    }

    abstract fun handleRequest(issueType: String)
}

// Level 1 handler
class Level1Support : SupportHandler() {

    override fun handleRequest(issueType: String) {

        if (issueType == "BASIC") {
            println("Level 1 handled the basic issue")
        } else {
            nextHandler?.handleRequest(issueType)
                ?: println("No handler available for: $issueType")
        }
    }
}

// Level 2 handler
class Level2Support : SupportHandler() {

    override fun handleRequest(issueType: String) {

        if (issueType == "TECHNICAL") {
            println("Level 2 handled the technical issue")
        } else {
            nextHandler?.handleRequest(issueType)
                ?: println("No handler available for: $issueType")
        }
    }
}

// Level 3 handler
class Level3Support : SupportHandler() {

    override fun handleRequest(issueType: String) {

        if (issueType == "CRITICAL") {
            println("Level 3 handled the critical issue")
        } else {
            nextHandler?.handleRequest(issueType)
                ?: println("No handler available for: $issueType")
        }
    }
}

fun main() {

    val level1 = Level1Support()
    val level2 = Level2Support()
    val level3 = Level3Support()

    level1.setNextHandler(level2)
    level2.setNextHandler(level3)

    level1.handleRequest("BASIC")
    level1.handleRequest("TECHNICAL")
    level1.handleRequest("CRITICAL")
    level1.handleRequest("BILLING")
}
```

Output:

text

```
Level 1 handled the basic issue
Level 2 handled the technical issue
Level 3 handled the critical issue
No handler available for: BILLING
```

## Why is this Chain of Responsibility?

Consider:

kotlin

```
protected var nextHandler: SupportHandler? = null
```

Each handler can reference another handler of the same base type.

The request is forwarded through:

kotlin

```
nextHandler?.handleRequest(issueType)
```

The safe-call operator `?.` prevents a null-pointer exception when the next handler is absent.

The Elvis operator `?:` provides a fallback when no next handler exists.

Kotlin can also implement this pattern using interfaces, function references, or higher-order functions. The essential idea remains the same: each processing step can handle or forward a request.

# 5. How Chain of Responsibility Works

The pattern has three main roles.

| Role | Responsibility | Example |
| --- | --- | --- |
| Handler | Defines the common request-handling contract and often the next-handler reference | `SupportHandler` |
| Concrete Handler | Processes a specific kind of request or forwards it | `Level1Support`, `Level2Support` |
| Client | Starts the request by calling the first handler | `Main` |

## Class diagram

text

```
                   Client
                     |
                     v
              SupportHandler
                (Abstract)
                     ^
                     |
          +----------+----------+
          |          |          |
          v          v          v
       Level1      Level2      Level3
       Support     Support     Support
          |          |          |
          +----------+----------+
               nextHandler
```

More precisely, each concrete handler extends `SupportHandler`, and each instance can reference another handler through `nextHandler`.

For example:

text

```
level1.nextHandler = level2
level2.nextHandler = level3
level3.nextHandler = null
```

## Execution flow

mermaid

```
flowchart TD
    A["Client sends request"] --> B["Level 1 Support"]
    B --> C{"Can Level 1 handle it?"}
    C -->|Yes| D["Process request"]
    C -->|No| E["Level 2 Support"]
    E --> F{"Can Level 2 handle it?"}
    F -->|Yes| D
    F -->|No| G["Level 3 Support"]
    G --> H{"Can Level 3 handle it?"}
    H -->|Yes| D
    H -->|No| I["Request remains unhandled"]
```

### Why does the client call only the first handler?

The client uses:

java

```
level1.handleRequest("CRITICAL");
```

It does not explicitly call Level 2 or Level 3.

Each handler determines whether the request should move forward. This reduces coupling between the client and individual processing steps.

### Does a request always stop at the first handler?

No. It depends on the implementation.

Common approaches include:

- Single-handler processing: Stop when one handler accepts the request.
- Multi-handler processing: Allow multiple handlers to process the same request.
- Pipeline processing: Every handler performs a step and forwards the request.

The pattern is flexible enough to support these variations.

# 6. Types and Common Use Cases

Chain of Responsibility is usually described through its request-processing behavior rather than a fixed set of formal subtypes. These are common ways it is used.

Roqit Fleet Management System

1. Escalation Chain

Requests move through support levels until someone can resolve them.

Example: Level 1 → Level 2 → Level 3 support.

12 Best freelance office automation experts for hire in April 2026

2. Processing Pipeline

Multiple handlers perform sequential processing steps.

Example: Validate input → Authenticate user → Process request.

IT-Sicherheit im Mittelstand: Der vollständige Leitfaden 2025

3. Middleware Chain

Each component processes or examines an incoming request before forwarding it.

Example: Logging → Authentication → Authorization → Endpoint.

Internal Control of Procurement: Strategies for Risk Management

4. Approval Workflow

Requests move through approval handlers based on conditions.

Example: Manager approval → Department head → Director approval.

### Common real-world examples

- Web middleware pipelines, such as request filters and interceptors.
- Logging systems, where log records pass through configured handlers.
- Customer support escalation.
- Expense and purchase approval workflows.
- Event processing and message handling.
- Input validation pipelines.
- Authentication and authorization processing.

Note that these applications use the pattern when requests are explicitly forwarded through handlers. Not every collection of filters or workflow steps automatically qualifies as Chain of Responsibility.

# 7. When to Use Chain of Responsibility

Use this pattern when:

- Multiple handlers may be able to process the same request.
- The sender should not know the exact handler.
- You want to decouple request creation from request processing.
- The handler order needs to be configurable.
- You need to add or remove processing steps without heavily modifying the client.
- Requests need to move through a series of checks or processing stages.

### When should you avoid it?

Avoid Chain of Responsibility when:

- There is only one fixed handler and no benefit from indirection.
- Every request must be handled by a specific component, but the chain does not guarantee that.
- An unhandled request would silently cause serious problems.
- The chain is so long that tracing requests becomes difficult.
- The processing order is complex enough to require a dedicated workflow engine.

Main benefit: Decouples the sender from the handler that processes the request.

Main limitation: A request may reach the end of the chain without being handled, so fallback behavior and observability matter.

# 8. Chain of Responsibility vs. Other Patterns

These patterns can look similar because they distribute work across multiple objects, but they solve different problems.

| Feature | Chain of Responsibility | Decorator | Strategy | Command | Observer |
| --- | --- | --- | --- | --- | --- |
| Main purpose | Pass requests between handlers | Add behavior to an object | Choose an algorithm | Encapsulate a request as an object | Notify multiple subscribers |
| Flow | Handler to handler | Wrapped object to wrapped object | Client chooses a strategy | Invoker calls a command | Publisher notifies observers |
| Responsibility | A handler may accept or forward | Each wrapper adds behavior | One strategy performs the chosen algorithm | Command represents an operation | Observers react to events |
| Example | Support escalation | Add logging around a service | Choose a payment algorithm | Queue a payment request | Notify users of an update |

## Chain of Responsibility vs. Decorator

Both patterns can use linked objects and delegation.

- Chain of Responsibility: Each handler decides whether to handle or forward the request.
- Decorator: Each wrapper adds behavior around a wrapped object, often delegating before or after its own behavior.

Example: A support handler forwards an unresolved issue to another support level. A logging decorator records information around a service call.

## Chain of Responsibility vs. Strategy

- Chain of Responsibility: A request can travel through multiple handlers until one handles it, or through several handlers in a pipeline.
- Strategy: The client or context selects an algorithm to perform a particular task.

Example: An issue moves from Level 1 to Level 3 support. A payment service chooses either a card-payment strategy or a bank-transfer strategy.

## Chain of Responsibility vs. Command

- Chain of Responsibility: Focuses on who handles a request.
- Command: Encapsulates a request or operation as an object.

Example: Support handlers route an issue. A `SendEmailCommand` encapsulates the operation of sending an email so it can be queued, logged, or executed later.

## Chain of Responsibility vs. Observer

- Chain of Responsibility: Requests move between handlers according to the chain's rules.
- Observer: One publisher notifies multiple interested subscribers.

Example: An issue moves from one support level to another. A ticket update event notifies the customer, support agent, and monitoring service.

# 9. Common Interview Questions

### Q1. What is the Chain of Responsibility Pattern?

It is a behavioral design pattern that passes a request along a chain of handlers until a handler processes it or the chain ends.

### Q2. What are the main components?

- Handler: Defines the handling contract.
- Concrete Handler: Processes or forwards requests.
- Client: Initiates the request through the chain.

### Q3. Why do we need this pattern?

It decouples the sender from individual handlers and makes request-processing logic easier to extend and organize.

### Q4. Is Chain of Responsibility a behavioral or structural pattern?

It is a behavioral design pattern because it focuses on how responsibilities and requests are distributed among objects.

### Q5. Does every handler have to process the request?

No. A handler can process the request, forward it, reject it, or terminate processing, depending on the design.

### Q6. Can multiple handlers process the same request?

Yes. In a pipeline-style implementation, every handler may perform its own operation and forward the request. In a first-match implementation, processing normally stops when a handler accepts the request.

### Q7. What happens when no handler can process the request?

The request may remain unhandled. A robust implementation should define a fallback, such as returning an error, logging the failure, or invoking a default handler.

### Q8. What are the advantages?

- Reduces coupling between sender and receiver.
- Supports extensible request processing.
- Allows handler ordering to be configurable.
- Encourages single-responsibility handler classes.
- Makes individual handlers easier to test.
- Can reduce large conditional blocks.

### Q9. What are the disadvantages?

- Requests may remain unhandled.
- Debugging can be harder when chains become long.
- Incorrect handler ordering can produce unexpected behavior.
- Excessive delegation may add complexity and overhead.
- Cyclic handler references can cause infinite recursion.

### Q10. How is the pattern different from a simple `if-else` statement?

A simple `if-else` centralizes decisions in one place. Chain of Responsibility distributes handling decisions across separate handler objects that can be connected, replaced, or reordered.

### Q11. Is Chain of Responsibility related to middleware?

Yes. Middleware is a common application of the pattern when each middleware component can process a request and pass control to the next component.

However, some middleware frameworks enforce a fixed pipeline or have additional semantics beyond the classic pattern.

### Q12. Can we dynamically change the chain at runtime?

Yes. If the implementation exposes appropriate configuration methods, handlers can be added, removed, or reordered at runtime.

### Q13. Can Chain of Responsibility be implemented in Kotlin without inheritance?

Yes. Kotlin can use interfaces, function types, and higher-order functions to represent handlers and compose processing steps. Inheritance is one implementation option, not a requirement.

### Q14. Is the order of handlers important?

Usually, yes. If several handlers can accept the same request, the first eligible handler may determine the result. The order is especially important in validation, authorization, and middleware pipelines.

### Q15. What is the difference between Chain of Responsibility and a pipeline?

The concepts overlap. A classic Chain of Responsibility often allows a handler to decide whether to process or forward a request. A pipeline commonly expects every stage to perform a processing step before passing the result onward.

### Q16. Is Chain of Responsibility used in Spring?

Yes. Spring applications commonly use handler chains and interceptor or filter pipelines. Servlet filters and Spring MVC interceptors are useful examples of sequential request processing, although their exact behavior depends on the framework and configuration.

### Q17. How can you prevent infinite loops in a handler chain?

- Build the chain with a clear termination point.
- Avoid cyclic references.
- Validate chain configuration where appropriate.
- Use suitable limits or safeguards if the chain is dynamically assembled.

### Q18. Does the pattern improve performance?

Not necessarily. It can improve maintainability and extensibility, but each additional handler may introduce processing overhead. Performance depends on the length of the chain and the work performed by each handler.

# 10. Quick Revision

- Category: Behavioral design pattern.
- Purpose: Pass requests along a chain of handlers.
- Handler: Defines the request-handling contract.
- Concrete Handler: Processes a request or forwards it.
- Client: Starts processing through the first handler.
- Java: Commonly implemented using an abstract class or interface and a next-handler reference.
- Kotlin: Can use interfaces, abstract classes, nullable references, or function-based handlers.
- Escalation chain: Moves a request to increasingly specialized handlers.
- Pipeline: Applies multiple processing steps in sequence.
- Main benefit: Decouples the sender from the handler that ultimately processes the request.
- Main limitation: Requests may reach the end without being handled.

## One-Line Memory Trick

Chain of Responsibility = Pass the request along a chain; each handler decides whether to handle it or pass it on.
