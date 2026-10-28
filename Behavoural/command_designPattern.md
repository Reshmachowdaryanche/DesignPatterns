# Command Design Pattern — Java & Kotlin

An interview-focused guide to the Command Pattern, using a real-world home automation example with request encapsulation, Java and Kotlin implementations, class relationships, comparisons, and common interview questions.

## Table of Contents

1. What Is the Command Pattern?
2. Why Do We Need It?
3. Java Implementation
4. Kotlin Implementation
5. How the Command Pattern Works
6. Types and Common Use Cases
7. When to Use the Command Pattern
8. Command vs. Other Patterns
9. Common Interview Questions
10. Quick Revision

# 1. What Is the Command Pattern?

The Command Pattern is a behavioral design pattern that encapsulates a request as an object. It separates the object that requests an operation from the object that actually performs it.

Instead of calling a receiver's method directly, the client creates a command object and gives it to an invoker. The invoker executes the command without needing to know the details of the operation.

### Real-world example: Home automation system

Imagine you are building a smart home application.

Different devices support different operations:

- Light can be turned on or off.
- Fan can be turned on or off.
- Air conditioner can be turned on or off.

Without the Command Pattern, a remote control might need to know the details of every device.

With the Command Pattern, each operation is represented by a command object.

Lutron Pico Smart Remote, 4-Button 4-Group Toggle (Light Icons) | PJ2-4B-XX-L41 – Simply Controlled

atomi smart AT1823 Manuals, Certifications & Specs

MOES Smart Fan Light Switch, WiFi Speed Control RF Remote Works With Alexa Google Tuya Smart Life | Shopee Singapore

For example, the client creates a command to turn on a light:

java

```
Command command = new LightOnCommand(light);
remote.setCommand(command);
remote.pressButton();
```

The remote control does not need to know how the light turns on. It simply executes the command.

The command delegates the actual operation to the light.

Memory trick: Command Pattern = Turn a request into an object that can be executed, stored, queued, or undone.

## Main idea

The pattern separates four responsibilities:

- Client: Creates and configures commands.
- Command: Defines a common execution contract.
- Concrete Command: Connects a request to a receiver operation.
- Receiver: Performs the actual business operation.
- Invoker: Triggers command execution.

# 2. Why Do We Need It?

Suppose you implement a remote control without the Command Pattern.

java

```
class RemoteControl {

    private Light light;
    private Fan fan;

    public RemoteControl(Light light, Fan fan) {
        this.light = light;
        this.fan = fan;
    }

    public void pressLightButton() {
        light.turnOn();
    }

    public void pressFanButton() {
        fan.turnOn();
    }
}
```

This works for a small application, but the remote control is tightly coupled to individual device types.

As the application grows, you might add:

- Air conditioners.
- Television systems.
- Garage doors.
- Smart locks.
- Scheduled device operations.
- Undo and redo functionality.

The remote control would need more fields and methods for each new device.

This can lead to:

- Tight coupling between the invoker and receivers.
- Increasing numbers of conditional statements.
- Difficulty adding new operations.
- Difficulty queuing or logging requests.
- Complicated undo and redo functionality.
- Difficulty reusing the same invoker with different operations.

## With the Command Pattern

Each operation is represented by an independent command object.

Client

Creates the command

Concrete Command

Encapsulates the operation

Invoker — Remote Control

Calls execute()

Receiver — Light / Fan

Performs the actual operation

The flow is:

1. The client creates a receiver, such as a light.
2. The client creates a command, such as `LightOnCommand`.
3. The command stores a reference to the receiver.
4. The client assigns the command to the remote control.
5. The invoker calls `execute()`.
6. The command invokes the receiver's operation.

Important: The invoker knows the command interface, not the specific device implementation.

# 3. Java Implementation

Let's build a home automation system using the Command Pattern.

## Step 1: Create the Command interface

The interface defines the operation that every command must implement.

java

```
interface Command {
    void execute();
}
```

### What does `Command` do?

- Defines a common execution contract.
- Allows different commands to be used interchangeably.
- Hides the details of the actual operation from the invoker.

For example, the remote control can execute a light command or a fan command through the same `execute()` method.

## Step 2: Create the Receiver

The receiver contains the actual business logic.

java

```
class Light {

    public void turnOn() {
        System.out.println("Light is ON");
    }

    public void turnOff() {
        System.out.println("Light is OFF");
    }
}
```

The `Light` class knows how to turn the light on and off.

It does not need to know anything about commands or remote controls.

## Step 3: Create the Concrete Commands

Concrete commands implement the `Command` interface and delegate the work to the receiver.

### LightOnCommand

java

```
class LightOnCommand implements Command {

    private final Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.turnOn();
    }
}
```

### LightOffCommand

java

```
class LightOffCommand implements Command {

    private final Light light;

    public LightOffCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.turnOff();
    }
}
```

### What is happening here?

Consider:

java

```
private final Light light;
```

The command holds a reference to the receiver.

When the invoker calls:

java

```
command.execute();
```

the concrete command calls the corresponding receiver method.

The command encapsulates the relationship between the request and the operation that fulfills it.

## Step 4: Create the Invoker

The invoker triggers a command without knowing its implementation.

java

```
class RemoteControl {

    private Command command;

    public void setCommand(Command command) {
        this.command = command;
    }

    public void pressButton() {
        if (command != null) {
            command.execute();
        }
    }
}
```

### Why is this important?

The remote control depends only on the `Command` interface.

It does not need separate methods such as `pressLightButton()` or `pressFanButton()`.

Any object implementing `Command` can be assigned to it.

## Step 5: Create the Client

The client creates the receiver and command objects, connects them, and configures the invoker.

java

```
public class Main {

    public static void main(String[] args) {

        // Receiver
        Light light = new Light();

        // Concrete commands
        Command lightOn =
            new LightOnCommand(light);

        Command lightOff =
            new LightOffCommand(light);

        // Invoker
        RemoteControl remote = new RemoteControl();

        // Execute light-on command
        remote.setCommand(lightOn);
        remote.pressButton();

        // Execute light-off command
        remote.setCommand(lightOff);
        remote.pressButton();
    }
}
```

Output:

text

```
Light is ON
Light is OFF
```

### What makes this the Command Pattern?

The client configures the command, but the remote control only invokes `execute()`.

The receiver performs the actual operation.

The remote control can now execute different operations without needing to understand their implementation.

## Step 6: Add another receiver — Fan

Let's add a fan without modifying the `RemoteControl` class.

java

```
class Fan {

    public void turnOn() {
        System.out.println("Fan is ON");
    }

    public void turnOff() {
        System.out.println("Fan is OFF");
    }
}
```

Create a command:

java

```
class FanOnCommand implements Command {

    private final Fan fan;

    public FanOnCommand(Fan fan) {
        this.fan = fan;
    }

    @Override
    public void execute() {
        fan.turnOn();
    }
}
```

Use it with the same remote control:

java

```
Fan fan = new Fan();

Command fanOn = new FanOnCommand(fan);

remote.setCommand(fanOn);
remote.pressButton();
```

Output:

text

```
Fan is ON
```

Interview takeaway: We extended the application by adding a new receiver and command without changing the invoker. This demonstrates loose coupling and extensibility.

# 4. Kotlin Implementation

Kotlin can implement the Command Pattern using interfaces, classes, and function types. Let's first implement the traditional object-oriented version.

## Complete implementation

kotlin

```
// Command interface
interface Command {
    fun execute()
}

// Receiver
class Light {

    fun turnOn() {
        println("Light is ON")
    }

    fun turnOff() {
        println("Light is OFF")
    }
}

// Concrete command
class LightOnCommand(
    private val light: Light
) : Command {

    override fun execute() {
        light.turnOn()
    }
}

// Concrete command
class LightOffCommand(
    private val light: Light
) : Command {

    override fun execute() {
        light.turnOff()
    }
}

// Invoker
class RemoteControl {

    private var command: Command? = null

    fun setCommand(command: Command) {
        this.command = command
    }

    fun pressButton() {
        command?.execute()
    }
}

// Client
fun main() {

    val light = Light()

    val lightOn = LightOnCommand(light)
    val lightOff = LightOffCommand(light)

    val remote = RemoteControl()

    remote.setCommand(lightOn)
    remote.pressButton()

    remote.setCommand(lightOff)
    remote.pressButton()
}
```

Output:

text

```
Light is ON
Light is OFF
```

## Why is this Command Pattern?

Consider the following Kotlin property:

kotlin

```
private var command: Command? = null
```

The remote control stores a reference to a command, not directly to a `Light` or `Fan`.

Then:

kotlin

```
command?.execute()
```

uses Kotlin's safe-call operator to execute the command if one has been assigned.

Kotlin's `private val` also makes the receiver reference immutable within each concrete command.

### Kotlin alternative: Function-based commands

Kotlin supports higher-order functions, so a small command abstraction can sometimes be implemented using function types.

kotlin

```
class RemoteControl {

    private var command: (() -> Unit)? = null

    fun setCommand(action: () -> Unit) {
        command = action
    }

    fun pressButton() {
        command?.invoke()
    }
}
```

Usage:

kotlin

```
val light = Light()
val remote = RemoteControl()

remote.setCommand { light.turnOn() }
remote.pressButton()

remote.setCommand { light.turnOff() }
remote.pressButton()
```

Output:

text

```
Light is ON
Light is OFF
```

This uses a function as a request object. It can be a lightweight alternative when commands do not need additional state or a named type.

However, the traditional `Command` interface is often easier to extend with features such as undo, logging, command metadata, and queuing.

# 5. How the Command Pattern Works

The pattern has four main structural roles, plus the client that assembles them.

| Role | Responsibility | Example |
| --- | --- | --- |
| Command | Defines the common operation contract | `Command` |
| Concrete Command | Maps execution to a receiver operation | `LightOnCommand` |
| Receiver | Performs the actual work | `Light` |
| Invoker | Triggers the command | `RemoteControl` |
| Client | Creates and configures the objects | `Main` |

## Class diagram

Client

Creates and wires objects

Invoker

Stores a Command reference

Command interface

execute()

Concrete Command

Delegates to the receiver

Receiver

Performs the operation

The key relationships are:

- `RemoteControl` has a `Command` reference.
- `LightOnCommand` implements `Command`.
- `LightOnCommand` has a `Light` reference.
- `Light` performs the operation.

## Execution flow

1. Client creates a command

`new LightOnCommand(light)`

2. Invoker stores the command

`remote.setCommand(lightOn)`

3. Invoker executes the command

`remote.pressButton()` calls `execute()`

4. Receiver performs the operation

`light.turnOn()`

## Why does the invoker call only `execute()`?

Because it is programmed against the `Command` abstraction.

For example:

java

```
remote.setCommand(lightOn);
remote.pressButton();

remote.setCommand(fanOn);
remote.pressButton();
```

The invoker does not need to know whether the operation controls a light, a fan, or another device.

This separation makes commands interchangeable.

## Advanced feature: Undo operations

One useful extension is to let commands reverse their operations.

For example:

java

```
interface UndoableCommand {
    void execute();
    void undo();
}
```

A light-on command could turn the light off when undone:

java

```
class LightOnCommand implements UndoableCommand {

    private final Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.turnOn();
    }

    @Override
    public void undo() {
        light.turnOff();
    }
}
```

A remote control can keep a history of executed commands and invoke `undo()` on the most recent command.

For more complex systems, undo may need to store the receiver's previous state rather than simply reverse the operation.

Important: Undo is a possible feature of the Command Pattern, not a requirement of every implementation.

# 6. Types and Common Use Cases

The Command Pattern does not have one universally fixed set of formal subtypes. The following are common implementation styles and uses.

## 1. Simple Command

Encapsulates one operation.

Example: Turn on a light or send an email.

java

```
remote.setCommand(new LightOnCommand(light));
remote.pressButton();
```

## 2. Undoable Command

Stores enough information to reverse an operation.

Example: A text editor records an insertion so it can remove the inserted text during undo.

## 3. Queued Command

Stores commands for later execution.

Example: An application queues email-sending jobs and processes them in the background.

## 4. Macro Command

Groups multiple commands and executes them together.

Example: A "Leave Home" action turns off the lights, turns off the fan, and activates the security system.

java

```
class MacroCommand implements Command {

    private final List<Command> commands;

    public MacroCommand(List<Command> commands) {
        this.commands = commands;
    }

    @Override
    public void execute() {
        for (Command command : commands) {
            command.execute();
        }
    }
}
```

This is useful when a single action needs to trigger several independent operations.

## Common real-world applications

- GUI buttons and menu actions.
- Undo and redo in editors.
- Task queues and background jobs.
- Job schedulers.
- Transaction or operation histories.
- Smart home automation.
- Remote procedure invocation.
- Batch operations and macro commands.
- Game actions and replay systems.

A queue of command objects can also support retries, persistence, or delayed execution, provided the application implements those capabilities.

# 7. When to Use the Command Pattern

Use this pattern when:

- You want to separate the requester from the object performing the operation.
- You need to parameterize objects with different operations.
- Requests must be stored, queued, or scheduled.
- You need to log or audit operations.
- You want to support undo and redo.
- You need to group multiple operations into one command.
- You want to reuse a generic invoker with different actions.

## When should you avoid it?

Avoid introducing the pattern when:

- An operation is simple and has only one fixed caller.
- Creating many command classes adds unnecessary complexity.
- Requests do not need to be stored, switched, queued, or independently managed.
- A direct method call already provides sufficient separation.

### Main benefit

The requester does not need to know how an operation is performed.

### Main limitation

The pattern can introduce many small classes and additional indirection. Undo, persistence, retries, and failure recovery also require explicit design; they do not come automatically with a command interface.

# 8. Command vs. Other Patterns

These patterns can appear similar because they use abstraction and delegation, but they address different problems.

| Feature | Command | Strategy | Chain of Responsibility | Observer | State |
| --- | --- | --- | --- | --- | --- |
| Main purpose | Encapsulate a request as an object | Encapsulate interchangeable algorithms | Pass a request through handlers | Notify interested subscribers | Change behavior based on state |
| Main focus | Representing and executing an operation | Selecting an algorithm | Routing or forwarding a request | Broadcasting an event | State-dependent behavior |
| Typical flow | Invoker calls command | Context delegates to strategy | Handler processes or forwards | Publisher notifies observers | Current state controls behavior |
| Example | `LightOnCommand` | Card vs. bank transfer | Support escalation | Notify users of a ticket update | Order status transitions |

## Command vs. Strategy

Both patterns can use interfaces and interchangeable implementations.

- Command: Represents an operation that can be invoked independently.
- Strategy: Represents an algorithm chosen to perform a task.

Example:

- `SendEmailCommand` represents the action of sending an email.
- `CompressionStrategy` selects how a file is compressed.

A command can be queued or logged as an operation. A strategy is primarily concerned with choosing an algorithm.

## Command vs. Chain of Responsibility

- Command: Encapsulates a request so an invoker can execute it.
- Chain of Responsibility: Passes a request through handlers that decide whether to process or forward it.

Example:

- `TurnOnLightCommand` represents an operation to execute.
- A support ticket chain forwards an issue through Level 1, Level 2, and Level 3.

These patterns can be combined: a handler in a chain may decide to execute a command.

## Command vs. Observer

- Command: Represents an operation to be invoked.
- Observer: Notifies multiple subscribers when an event occurs.

Example:

- A command tells a system to publish a report.
- An observer system notifies subscribed users when the report is ready.

## Command vs. Decorator

- Command: Encapsulates a request as an object.
- Decorator: Adds behavior to an object by wrapping it.

Example:

- `SaveFileCommand` represents the save operation.
- A logging decorator adds logging around a service or command execution.

## Command vs. Factory

- Command: Encapsulates an operation.
- Factory: Encapsulates object creation.

Example:

- `LightOnCommand` executes an operation on a light.
- `CommandFactory` could create a command based on a button identifier.

A factory can create commands, but the factory and Command Pattern solve different problems.

# 9. Common Interview Questions

### Q1. What is the Command Pattern?

It is a behavioral design pattern that encapsulates a request as an object, separating the requester from the object that performs the operation.

### Q2. What are the main components?

- Command: Defines an execution contract.
- Concrete Command: Implements the contract and delegates to a receiver.
- Receiver: Performs the actual operation.
- Invoker: Triggers the command.
- Client: Creates and connects the objects.

### Q3. Why do we need the Command Pattern?

It reduces coupling between requesters and receivers and makes operations easier to parameterize, store, queue, log, and potentially undo.

### Q4. Is the Command Pattern behavioral or structural?

It is a behavioral design pattern because it focuses on how requests and operations are represented and executed.

### Q5. What is the difference between a command and a receiver?

A command represents the request and coordinates its execution. The receiver contains the business logic that actually performs the operation.

For example, `LightOnCommand` is the command, and `Light` is the receiver.

### Q6. What is an invoker?

An invoker triggers a command without knowing its concrete implementation.

Examples include a remote control, a GUI button, or a job scheduler.

### Q7. Can a command contain multiple operations?

Yes. A macro command can contain a collection of commands and execute them in sequence.

### Q8. How does the Command Pattern support undo?

Commands can implement an `undo()` method and store the information needed to reverse an operation. The invoker can maintain a history of commands to support undo and redo.

### Q9. Does the Command Pattern automatically support undo?

No. Undo requires additional methods and, often, state management. Some operations are difficult or impossible to reverse reliably.

### Q10. Can commands be queued?

Yes. Commands can be stored in a queue and executed later by a worker or scheduler. Reliable processing may also require persistence, retry policies, and error handling.

### Q11. What are the advantages?

- Reduces coupling between invoker and receiver.
- Makes operations interchangeable.
- Supports command queues and scheduling.
- Can support undo and redo.
- Enables logging and auditing.
- Makes it easier to group operations.
- Encourages separation of responsibilities.

### Q12. What are the disadvantages?

- May introduce many small command classes.
- Adds layers of indirection.
- Requires explicit failure and retry handling.
- Undo logic can be difficult for operations with side effects.
- Persisting commands requires serialization or another representation of the requested operation.

### Q13. How is Command different from a simple method call?

A direct method call invokes an operation immediately on a specific object. A command object represents the operation separately, allowing it to be passed around, stored, scheduled, or executed by a generic invoker.

### Q14. Can the Command Pattern be implemented in Kotlin without interfaces?

Yes. Kotlin function types and lambdas can represent commands.

For example:

kotlin

```
val command: () -> Unit = {
    light.turnOn()
}

command()
```

However, a dedicated command interface or class may be more appropriate when commands need undo behavior, metadata, or additional state.

### Q15. Can commands have parameters?

Yes. A command can store parameters in its constructor or receive them through its execution method.

For example, a `SetVolumeCommand` might store a volume level and apply it to an audio receiver.

### Q16. Is the Command Pattern used in Spring?

Command-like objects are common in Spring applications, including task execution, scheduled jobs, and application operations. However, not every Spring service method or scheduled task is automatically an implementation of the classic Command Pattern.

### Q17. What is a macro command?

A macro command combines multiple commands into one higher-level command. When executed, it invokes its constituent commands, typically in sequence.

### Q18. Can the Command Pattern be used with the Strategy Pattern?

Yes. A command may use a strategy to decide how to perform its operation. The command represents the requested action, while the strategy provides an algorithm for carrying it out.

### Q19. How would you implement redo?

Maintain a history of executed commands and a redo stack. When undoing, move the command to the redo stack; when redoing, execute it again and return it to the history.

This requires careful handling of new operations, failures, and commands whose results cannot safely be repeated.

### Q20. Does the Command Pattern improve performance?

Not necessarily. Its primary purpose is flexibility and decoupling. Additional objects and method calls can introduce small overhead, which is usually less important than the architectural benefits when the pattern is genuinely needed.

# 10. Quick Revision

- Category: Behavioral design pattern.
- Purpose: Encapsulate a request as an object.
- Command: Defines the execution contract.
- Concrete Command: Connects a request to a receiver operation.
- Receiver: Performs the actual business logic.
- Invoker: Triggers the command.
- Client: Creates and configures commands.
- Java: Commonly uses a `Command` interface and concrete command classes.
- Kotlin: Can use interfaces, classes, or function types.
- Undo/redo: Possible when commands retain the necessary state and implement reversal logic.
- Queueing: Commands can be stored and executed later.
- Macro command: Groups multiple commands into one operation.
- Main benefit: Decouples the invoker from the receiver.
- Main limitation: Adds extra objects and indirection.

## One-Line Memory Trick

Command Pattern = Encapsulate a request as an object so it can be executed, stored, queued, logged, or undone.

## Quick self-test

1. Which component performs the actual business operation?

Invoker

Receiver

Command interface

2. What does the invoker know about a command?

The receiver's internal implementation

Only the command abstraction

Every concrete receiver type

3. Which feature can be built using a history of command objects?

Undo and redo

Automatic inheritance

Database normalization

