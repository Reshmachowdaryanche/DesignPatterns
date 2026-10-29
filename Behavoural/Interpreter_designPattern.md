# Interpreter Design Pattern — Java & Kotlin

An interview-focused guide to the Interpreter Pattern, using a calculator example with expression evaluation, Java and Kotlin implementations, class relationships, comparisons, and common interview questions.

## Table of Contents

1. What Is the Interpreter Pattern?
2. Why Do We Need It?
3. Java Implementation
4. Kotlin Implementation
5. How the Interpreter Pattern Works
6. Types and Common Use Cases
7. When to Use the Interpreter Pattern
8. Interpreter vs. Other Patterns
9. Common Interview Questions
10. Quick Revision

# 1. What Is the Interpreter Pattern?

The Interpreter Pattern is a behavioral design pattern that defines a representation for a language's grammar and provides a way to evaluate expressions in that language.

In simple words, the Interpreter Pattern allows us to represent rules or expressions as objects and evaluate them to produce a result.

### Real-world example: Calculator

Imagine you are building a calculator that understands expressions such as:

- `5 + 3`
- `10 - 4`
- `(5 + 3) - 2`

The calculator needs to understand these expressions and calculate their results.

For example:

text

```
Expression: 5 + 3

Interpreter evaluates the expression.

Result: 8
```

In the Interpreter Pattern, we represent the expression using objects.

For example, `5 + 3` can be represented using:

- A `NumberExpression` representing `5`.
- A `NumberExpression` representing `3`.
- An `AddExpression` combining the two numbers.

The addition expression evaluates its children and returns their sum.

Memory trick: Interpreter Pattern = Represent language rules as objects and evaluate those objects to get a result.

# 2. Why Do We Need It?

Suppose you implement a calculator using `if-else` statements.

java

```
class Calculator {

    public int calculate(String operator, int a, int b) {

        if (operator.equals("+")) {
            return a + b;

        } else if (operator.equals("-")) {
            return a - b;

        } else if (operator.equals("*")) {
            return a * b;
        }

        throw new IllegalArgumentException(
            "Unknown operator"
        );
    }
}
```

This works for simple calculations, but as the application grows, you may need to support:

- Complex mathematical expressions.
- Boolean expressions.
- Business rules.
- Search filters.
- Custom configuration languages.
- Nested expressions using parentheses.

As the rules become more complex, the central conditional logic can become difficult to maintain.

## With the Interpreter Pattern

Instead of putting every rule inside one calculator class, we create separate expression classes.

For example:

text

```
             AddExpression
              /         \
             /           \
   NumberExpression   NumberExpression
          (5)                (3)
```

Each expression knows how to evaluate itself.

The flow is:

1. The client creates expression objects.
2. The client combines them into an expression tree.
3. The client calls `interpret()` on the root expression.
4. The root evaluates its child expressions.
5. Each child returns its result.
6. The root combines the results and returns the final answer.

Important: The Interpreter Pattern represents and evaluates expressions. Converting raw input text such as `"5 + 3"` into an expression tree requires a parser, which is a separate responsibility unless you implement it as part of the system.

# 3. Java Implementation

Let's build a calculator using the Interpreter Pattern.

## Step 1: Create the Expression interface

The interface defines a common method for evaluating expressions.

java

```
interface Expression {
    int interpret();
}
```

### What does `Expression` do?

- Defines the common evaluation contract.
- Allows different expression types to be used interchangeably.
- Enables complex expressions to contain other expressions.

Every expression class must implement the `interpret()` method.

## Step 2: Create NumberExpression

A number is the simplest expression. It returns its value without performing any calculation.

java

```
class NumberExpression implements Expression {

    private final int number;

    public NumberExpression(int number) {
        this.number = number;
    }

    @Override
    public int interpret() {
        return number;
    }
}
```

Example:

java

```
Expression five = new NumberExpression(5);

System.out.println(five.interpret());
```

Output:

text

```
5
```

`NumberExpression` is called a Terminal Expression because it represents a basic value and does not contain other expressions.

## Step 3: Create AddExpression

An addition expression contains two child expressions and adds their results.

java

```
class AddExpression implements Expression {

    private final Expression left;
    private final Expression right;

    public AddExpression(
            Expression left,
            Expression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public int interpret() {
        return left.interpret() + right.interpret();
    }
}
```

### How does it work?

Consider:

java

```
return left.interpret() + right.interpret();
```

The expression:

1. Evaluates the left child.
2. Evaluates the right child.
3. Adds both results.
4. Returns the final result.

Notice that `left` and `right` are of type `Expression`, not `NumberExpression`.

This allows them to contain other operations as well as numbers.

## Step 4: Create SubtractExpression

We can implement subtraction in the same way.

java

```
class SubtractExpression implements Expression {

    private final Expression left;
    private final Expression right;

    public SubtractExpression(
            Expression left,
            Expression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public int interpret() {
        return left.interpret() - right.interpret();
    }
}
```

This class evaluates both child expressions and subtracts the right result from the left result.

## Step 5: Create the Client

Now let's evaluate the expression `5 + 3`.

java

```
public class Main {

    public static void main(String[] args) {

        // Create number expressions
        Expression five = new NumberExpression(5);
        Expression three = new NumberExpression(3);

        // Combine them into an addition expression
        Expression addition =
            new AddExpression(five, three);

        // Evaluate the complete expression
        System.out.println(addition.interpret());
    }
}
```

Output:

text

```
8
```

### Who is the client?

The `Main` class, specifically its `main()` method, is the client in this example.

The client creates the expression objects, connects them together, and calls `interpret()` on the root expression.

The expression objects perform the actual evaluation.

## Step 6: Evaluate a complex expression

Suppose we want to calculate:

text

```
(5 + 3) - 2
```

We can reuse the expression classes we already created.

java

```
public class Main {

    public static void main(String[] args) {

        Expression five = new NumberExpression(5);
        Expression three = new NumberExpression(3);
        Expression two = new NumberExpression(2);

        Expression addition =
            new AddExpression(five, three);

        Expression result =
            new SubtractExpression(addition, two);

        System.out.println(result.interpret());
    }
}
```

Output:

text

```
6
```

### How does this work?

The expression tree is:

text

```
          SubtractExpression
             /          \
            /            \
    AddExpression    NumberExpression
       /      \              (2)
      /        \
    (5)        (3)
```

The subtraction expression evaluates its left child first:

text

```
5 + 3 = 8
```

Then it evaluates the right child:

text

```
2
```

Finally:

text

```
8 - 2 = 6
```

Interview note: The client builds the expression tree and asks the root expression to evaluate it. Each expression is responsible for its own evaluation logic.

# 4. Kotlin Implementation

Kotlin supports the Interpreter Pattern using interfaces and classes.

## Complete implementation

kotlin

```
// Abstract expression
interface Expression {
    fun interpret(): Int
}

// Terminal expression
class NumberExpression(
    private val number: Int
) : Expression {

    override fun interpret(): Int = number
}

// Non-terminal expression
class AddExpression(
    private val left: Expression,
    private val right: Expression
) : Expression {

    override fun interpret(): Int =
        left.interpret() + right.interpret()
}

// Non-terminal expression
class SubtractExpression(
    private val left: Expression,
    private val right: Expression
) : Expression {

    override fun interpret(): Int =
        left.interpret() - right.interpret()
}

fun main() {

    val five = NumberExpression(5)
    val three = NumberExpression(3)
    val two = NumberExpression(2)

    val addition = AddExpression(five, three)

    val result = SubtractExpression(addition, two)

    println(result.interpret())
}
```

Output:

text

```
6
```

## Why is this the Interpreter Pattern?

Consider:

kotlin

```
interface Expression {
    fun interpret(): Int
}
```

This defines the common evaluation contract.

Now consider:

kotlin

```
left.interpret() + right.interpret()
```

The addition expression doesn't need to know whether its children are numbers, additions, or other expressions. It simply asks them to evaluate themselves.

Kotlin's concise syntax makes the implementation shorter, but the underlying design is the same as in Java.

Kotlin can also use sealed classes and data classes to model expression trees, especially when the set of expression types is known in advance.

# 5. How the Interpreter Pattern Works

The pattern has three main structural roles.

| Role | Responsibility | Example |
| --- | --- | --- |
| Abstract Expression | Defines the common evaluation contract | `Expression` |
| Terminal Expression | Represents a basic value or symbol | `NumberExpression` |
| Non-terminal Expression | Combines other expressions according to a rule | `AddExpression`, `SubtractExpression` |
| Client | Builds the expression structure and requests evaluation | `Main` |

## Class diagram

text

```
                   Client
                     |
                     v
                 Expression
                 (Interface)
                     ^
                     |
          +----------+-----------+
          |          |           |
          v          v           v
       Number      Add       Subtract
     Expression  Expression  Expression
                     |           |
                     +-----+-----+
                           |
                  Child Expressions
```

More precisely:

- Every expression implements the `Expression` interface.
- `NumberExpression` represents a terminal expression.
- `AddExpression` and `SubtractExpression` contain references to other expressions.
- The client invokes `interpret()` on the root expression.

## Execution flow

For the expression `(5 + 3) - 2`:

1. The client creates the number expressions.
2. The client creates an addition expression using `5` and `3`.
3. The client creates a subtraction expression using the addition expression and `2`.
4. The client calls `result.interpret()`.
5. The subtraction expression evaluates its child expressions.
6. The final result is returned as `6`.

## Does the Interpreter Pattern automatically understand a string?

No. Our implementation does not automatically parse strings such as `"5 + 3 - 2"`.

We manually created the expression tree using Java objects.

A complete calculator would typically have two stages:

text

```
Input String
     |
     v
   Parser
     |
     v
Expression Tree
     |
     v
  Interpreter
     |
     v
    Result
```

The parser converts text into an expression tree. The interpreter evaluates that tree.

For a large language, a parser generator or a dedicated parsing library may be more appropriate.

# 6. Types and Common Use Cases

The Interpreter Pattern is generally described by how expressions represent and evaluate a grammar, rather than by a fixed set of formal subtypes.

## 1. Mathematical expressions

Represent and evaluate arithmetic operations.

Example:

text

```
(5 + 3) - 2
```

A calculator can represent each number and operation as an expression object.

## 2. Boolean expressions

Evaluate logical conditions.

Example:

text

```
TRUE AND FALSE
```

An `AndExpression` can evaluate two child expressions and combine their Boolean results.

## 3. Business rules

Represent business conditions as composable expressions.

Example:

text

```
CustomerAge > 18 AND HasValidMembership
```

A rule engine can evaluate each condition and combine the results.

## 4. Search and filtering

Interpret simple search expressions.

Example:

text

```
price < 100 AND category = "Books"
```

A filtering system can represent these conditions as an expression tree and evaluate them against data.

## 5. Domain-specific languages (DSLs)

A domain-specific language is a small language designed for a particular purpose.

Examples include:

- Pricing rules.
- Configuration expressions.
- Automation rules.
- Permission expressions.
- Query conditions.

Note that not every rule engine or calculator necessarily uses the Interpreter Pattern. The pattern applies when the language rules are represented and evaluated through expression structures.

# 7. When to Use the Interpreter Pattern

Use this pattern when:

- You have a small, well-defined language or grammar.
- Expressions need to be represented and evaluated repeatedly.
- Rules can be modeled as composable objects.
- You want to add new expression types without rewriting existing ones.
- You need to evaluate business or logical rules dynamically.

### When should you avoid it?

Avoid the Interpreter Pattern when:

- The grammar is large or highly complex.
- The language requires advanced parsing, optimization, or error recovery.
- The expression tree would become difficult to maintain.
- A mature parser or expression engine already solves the problem.

Main benefit: Makes language rules explicit, composable, and easier to extend.

Main limitation: Creating a class for every grammar rule can lead to too many classes and make complex languages difficult to maintain.

# 8. Interpreter vs. Other Patterns

| Feature | Interpreter | Command | Strategy | Chain of Responsibility | Composite |
| --- | --- | --- | --- | --- | --- |
| Main purpose | Evaluate expressions in a language | Encapsulate a request | Choose an algorithm | Pass a request through handlers | Represent part-whole hierarchies |
| Main focus | Meaning of expressions | Executing an operation | Selecting behavior | Routing responsibility | Organizing objects into trees |
| Typical method | `interpret()` | `execute()` | Depends on the strategy | `handleRequest()` | Depends on the component |
| Example | Calculate `5 + 3` | Turn on a light | Choose a payment algorithm | Escalate a support ticket | Represent folders and files |

## Interpreter vs. Command

- Interpreter evaluates an expression according to language rules.
- Command encapsulates an operation so it can be executed by an invoker.

Example: `AddExpression` evaluates `5 + 3`, while `LightOnCommand` represents the action of turning on a light.

## Interpreter vs. Strategy

- Interpreter evaluates a language or expression.
- Strategy selects one of several interchangeable algorithms.

Example: An expression tree evaluates a mathematical formula, while a payment strategy chooses between card and bank transfer processing.

## Interpreter vs. Chain of Responsibility

- Interpreter evaluates an expression.
- Chain of Responsibility forwards a request through handlers that decide whether to process it or pass it along.

Example: An expression tree calculates a result, while a support ticket moves from Level 1 to Level 2 support.

## Interpreter vs. Composite

Both patterns commonly use tree structures.

- Interpreter represents language expressions and defines how they are evaluated.
- Composite represents individual objects and groups through a common interface.

An expression tree can use a Composite-like structure, but Interpreter adds the language-specific evaluation behavior.

# 9. Common Interview Questions

### Q1. What is the Interpreter Pattern?

It is a behavioral design pattern that represents the grammar of a language and provides a way to evaluate expressions in that language.

### Q2. What are the main components?

- Abstract Expression: Defines the evaluation contract.
- Terminal Expression: Represents a basic value or symbol.
- Non-terminal Expression: Combines other expressions according to a rule.
- Client: Builds the expression structure and requests evaluation.

### Q3. What is a terminal expression?

A terminal expression represents a basic element that does not contain other expressions.

In our example, `NumberExpression(5)` is a terminal expression.

### Q4. What is a non-terminal expression?

A non-terminal expression combines one or more expressions.

In our example, `AddExpression` contains left and right expressions and adds their results.

### Q5. Is the Interpreter Pattern behavioral or structural?

It is a behavioral design pattern because it focuses on evaluating expressions according to language rules.

### Q6. What is an expression tree?

An expression tree represents the structure of an expression using nodes for values and operations.

For example, `(5 + 3) - 2` has a subtraction node at the root, an addition node as its left child, and number expressions as leaves.

### Q7. Does the Interpreter Pattern parse strings automatically?

No. Parsing raw text into an expression tree is a separate responsibility unless you implement a parser as part of the application.

### Q8. What are the advantages?

- Makes language rules explicit.
- Supports composable expressions.
- Allows new expression types to be added.
- Makes small rule engines easier to extend.
- Makes individual expression classes easier to test.

### Q9. What are the disadvantages?

- Can create many classes.
- Complex grammars become difficult to maintain.
- Recursive evaluation can have performance costs.
- Parsing and syntax validation require additional implementation.

### Q10. Where is the Interpreter Pattern used?

It can be used in simple calculators, rule engines, filtering systems, Boolean expressions, configuration languages, and small domain-specific languages.

### Q11. Can Interpreter be implemented in Kotlin?

Yes. Kotlin supports interfaces, sealed classes, data classes, and recursive functions, which can all be used to represent and evaluate expressions.

### Q12. Can Interpreter use the Composite Pattern?

Yes. Expression trees often have a Composite-like structure in which individual values form the leaves and operations contain other expressions. Interpreter adds the rules for evaluating those nodes.

### Q13. What is the difference between an interpreter and a compiler?

An interpreter evaluates or executes expressions according to their meaning. A compiler translates source code into another representation, such as machine code or bytecode. Real language implementations can combine compilation and interpretation techniques.

### Q14. When should you avoid Interpreter?

Avoid building your own interpreter for a complex language when a mature parser or expression engine would be more suitable.

# 10. Quick Revision

- Category: Behavioral design pattern.
- Purpose: Represent and evaluate expressions in a language.
- Abstract Expression: Defines the `interpret()` method.
- Terminal Expression: Represents a basic value.
- Non-terminal Expression: Combines other expressions.
- Client: Builds the expression tree and requests evaluation.
- Java: Commonly implemented using an interface and concrete expression classes.
- Kotlin: Can use interfaces, sealed classes, or data classes.
- Expression tree: Represents the structure of an expression.
- Parsing: Converts input text into an expression structure; it is not automatic.
- Main benefit: Makes simple language rules composable and extensible.
- Main limitation: Can become complicated when the grammar grows.

## One-Line Memory Trick

Interpreter Pattern = Represent the rules of a language as objects, then evaluate those objects to get a result.
