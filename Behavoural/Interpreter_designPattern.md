# Interpreter Design Pattern — Java & Kotlin

Let's learn the Interpreter Pattern step by step, using a simple example so you can understand what it does, who the client is, and how the Java and Kotlin code works.

## 1. What is the Interpreter Pattern?

The Interpreter Pattern is a behavioral design pattern used to define and evaluate a simple language or set of expressions.

In simple words, you define rules for a language, and your code interprets those rules to produce a result.

Imagine you are building a calculator that understands expressions such as:

- `5 + 3`
- `10 - 4`
- `2 + 6`

You want your program to understand what these expressions mean and calculate the result.

For example:

Expression

# 5 + 3

Interpreter

Understands the expression's meaning

Result

# 8

The Interpreter Pattern represents the parts of an expression as objects, with each object knowing how to evaluate itself.

Memory trick: Interpreter Pattern = Represent the rules of a language as objects and use them to evaluate expressions.

## 2. Understand it with a real example

Suppose you want to build a calculator.

There are three important things:

- The expression — what you want to calculate, such as `5 + 3`.
- The interpreter — the code that understands and evaluates the expression.
- The result — `8`.

In the Interpreter Pattern, we can represent the expression `5 + 3` as a tree:

# +

Addition expression

## 5

Left operand

## 3

Right operand

The addition expression contains two smaller expressions: `5` and `3`.

When evaluated, it asks each child expression for its value and adds the results.

This is called an expression tree.

The same idea can represent more complex expressions, such as:

$$
(5+3)-2=6
$$

The subtraction expression contains an addition expression and a number expression.

## 3. Who are the components?

| Component | Meaning | Example |
| --- | --- | --- |
| Abstract Expression | Defines a common evaluation method | `Expression` |
| Terminal Expression | Represents a basic value | `NumberExpression(5)` |
| Non-terminal Expression | Combines other expressions | `AddExpression` |
| Client | Builds the expression structure and asks for the result | `main()` |

There is no separate invoker like the one in the Command Pattern. Here, the expression objects themselves evaluate the expression.

# 4. Java Implementation

Let's build a small calculator using the Interpreter Pattern.

## Step 1: Create the Expression interface

Every expression must provide an `interpret()` method.

java

```
interface Expression {
    int interpret();
}
```

This is the common contract.

Any expression, whether it represents a number or an addition operation, must implement `interpret()`.

## Step 2: Create a NumberExpression

A number is the simplest expression. It just returns its value.

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

For example:

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

There is no calculation to perform here. The expression simply returns `5`.

## Step 3: Create an AddExpression

An addition expression contains two expressions.

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

Let's understand the important line:

java

```
return left.interpret() + right.interpret();
```

The addition expression:

1. Evaluates the left expression.
2. Evaluates the right expression.
3. Adds both results.
4. Returns the final result.

Notice that `left` and `right` are of type `Expression`, not `NumberExpression`.

That means they can represent numbers, additions, subtractions, or other expression types that implement the interface.

## Step 4: Create a SubtractExpression

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

This follows the same structure but performs subtraction instead of addition.

## Step 5: Create the Client — main()

Now let's build the expression `5 + 3`.

java

```
public class Main {

    public static void main(String[] args) {

        // Create basic expressions
        Expression five = new NumberExpression(5);
        Expression three = new NumberExpression(3);

        // Combine them into an addition expression
        Expression addition =
            new AddExpression(five, three);

        // Interpret the complete expression
        System.out.println(addition.interpret());
    }
}
```

Output:

text

```
8
```

Who is the client? It is `Main`, specifically the `main()` method in this example.

The client creates the expression objects, connects them together, and calls `interpret()` on the root expression.

The expression objects handle the actual evaluation.

## Step 6: Evaluate a more complex expression

What if we want to calculate:

$$
(5+3)-2
$$

We can reuse our existing classes.

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

The client does not calculate `5 + 3 - 2` itself. It builds the expression tree and asks the root expression to interpret it.

# 5. How does the Java code actually work?

Let's trace the expression `(5 + 3) - 2`.

Step 1Client builds the expression tree

`Main` creates the number expressions, the addition expression, and the subtraction expression.

Step 2Client calls the root expression

java

```
result.interpret();
```

The root is `SubtractExpression`.

Step 3Subtraction evaluates its children

java

```
left.interpret() - right.interpret()
```

The left child is the addition expression; the right child is the number `2`.

Step 4Addition evaluates its own children

`5 + 3 = 8`

Step 5The final result is calculated

`8 - 2 = 6`

The most important concept is that each expression knows how to evaluate itself.

- `NumberExpression` returns a number.
- `AddExpression` evaluates two expressions and adds them.
- `SubtractExpression` evaluates two expressions and subtracts them.

## Class relationship

text

```
              Expression
                  |
          +-------+--------+
          |                |
   NumberExpression    AddExpression
                           |
                     left, right
                     Expressions

              SubtractExpression
                    |
               left, right
               Expressions
```

All these classes implement `Expression`. The non-terminal expressions also contain references to other expressions.

# 6. Kotlin Implementation

Kotlin implements the same pattern using interfaces and classes.

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

This defines a common contract for every expression.

Now consider:

kotlin

```
left.interpret() + right.interpret()
```

The addition expression doesn't need to know whether its children are numbers or other operations. It simply asks them to evaluate themselves.

This is called recursive evaluation: an expression evaluates its child expressions, which may evaluate their own children.

Kotlin's concise syntax makes this implementation shorter, but the underlying design is the same as Java.

# 7. Does Interpreter automatically understand a string?

This is an important interview point.

Suppose the user enters:

text

```
5 + 3 - 2
```

Our implementation does not automatically convert that text into an expression tree.

We manually created the tree using:

java

```
new SubtractExpression(
    new AddExpression(
        new NumberExpression(5),
        new NumberExpression(3)
    ),
    new NumberExpression(2)
);
```

The Interpreter Pattern defines how expressions are represented and evaluated. Parsing the raw string into an expression tree is a separate step, unless you implement that functionality as part of your system.

A complete calculator could have this architecture:

Input string

## `5 + 3 - 2`

Parser

Converts text into an expression tree

Expression tree

AddExpression, SubtractExpression, NumberExpression

Interpreter

# 6

For a full programming language or complicated mathematical expressions, a parser generator or a dedicated parsing technique may be more appropriate than creating every expression tree manually.

# 8. Common types and use cases

These are common ways the Interpreter Pattern is used, rather than a mandatory list of formal subtypes.

## 1. Mathematical expressions

Evaluate expressions such as addition, subtraction, multiplication, and division.

Example: A calculator or a small formula engine.

## 2. Business rules

Represent rules as expressions that can be combined.

Example:

text

```
CustomerAge > 18 AND HasValidMembership
```

A rule engine could evaluate each condition and combine the results.

## 3. Search and filtering

Interpret simple search conditions.

Example:

text

```
price < 100 AND category = "Books"
```

A filter engine can represent the conditions as expression objects and evaluate them against data.

## 4. Domain-specific languages (DSLs)

A DSL is a small language designed for a specific purpose.

Examples include:

- Pricing rules.
- Configuration expressions.
- Simple automation rules.
- Permission expressions.
- Query conditions.

## 5. Boolean expressions

Represent and evaluate conditions such as:

text

```
TRUE AND FALSE
```

An `AndExpression` could evaluate its two child expressions and combine their results.

## When should you use it?

Use the Interpreter Pattern when:

- You have a small, well-defined language or grammar.
- Expressions need to be represented and evaluated repeatedly.
- The rules can be modeled as composable objects.
- You want to add new expression types without rewriting all existing expressions.

Avoid it when:

- The grammar is large or highly complex.
- The language needs advanced parsing, optimization, or error recovery.
- The expression tree would become difficult to maintain.
- A standard expression engine or parser already solves the problem.

Main benefit: Rules are represented as objects that can be composed and extended.

Main limitation: A class for every grammar rule can create too many classes and become inefficient for complex languages.

# 9. Interpreter vs. Other Design Patterns

| Feature | Interpreter | Command | Strategy | Chain of Responsibility |
| --- | --- | --- | --- | --- |
| Main purpose | Evaluate expressions in a language | Encapsulate a request | Select an algorithm | Pass a request through handlers |
| Main focus | Meaning of expressions | Executing an operation | Choosing behavior | Routing responsibility |
| Key method | `interpret()` | `execute()` | Depends on the strategy | `handleRequest()` |
| Example | Calculate `5 + 3` | Turn on a light | Choose a payment algorithm | Escalate a support ticket |

## Interpreter vs. Command

- Interpreter: Evaluates an expression according to language rules.
- Command: Encapsulates an operation so it can be executed by an invoker.

Example: `AddExpression` evaluates `5 + 3`, while `LightOnCommand` represents the action of turning on a light.

## Interpreter vs. Strategy

- Interpreter: Evaluates a language or expression.
- Strategy: Selects one of several interchangeable algorithms.

Example: An expression tree evaluates a mathematical formula, while a payment strategy chooses between card and bank transfer processing.

## Interpreter vs. Composite

Both patterns commonly use tree structures.

- Interpreter: Each node represents a language expression and knows how to evaluate it.
- Composite: Each node represents an object in a part-whole hierarchy and provides a uniform way to work with individual objects and groups.

An expression tree can use the Composite structure, but Interpreter adds the language-specific evaluation behavior.

# 10. Common Interview Questions

### Q1. What is the Interpreter Pattern?

It is a behavioral design pattern that represents the grammar of a simple language and provides a way to evaluate expressions in that language.

### Q2. What are the main components?

- Abstract Expression: Defines the interpretation contract.
- Terminal Expression: Represents a basic value or symbol.
- Non-terminal Expression: Combines other expressions according to a rule.
- Client: Creates the expression structure and requests evaluation.

### Q3. What is a terminal expression?

A terminal expression represents a basic element that does not contain other expressions.

In our example, `NumberExpression(5)` is a terminal expression.

### Q4. What is a non-terminal expression?

A non-terminal expression combines one or more expressions.

In our example, `AddExpression` contains left and right expressions and adds their results.

### Q5. Is Interpreter a behavioral pattern?

Yes. It focuses on how language expressions are represented and evaluated.

### Q6. What is an expression tree?

It is a tree structure in which nodes represent values or operations. For example, `(5 + 3) - 2` can be represented using subtraction at the root, addition as its left child, and number expressions as leaves.

### Q7. Does the Interpreter Pattern parse a string automatically?

No. Parsing raw text into an expression tree is a separate responsibility unless you implement a parser as part of the application.

### Q8. What are the advantages?

- Makes language rules explicit.
- Supports composable expressions.
- Allows new expression types to be added.
- Makes small rule engines easier to extend.
- Can make individual expression classes easy to test.

### Q9. What are the disadvantages?

- Can create many classes.
- Complex grammars become difficult to maintain.
- Recursive evaluation can have performance costs.
- Parsing and syntax validation may require additional implementation.

### Q10. Where is the Interpreter Pattern used?

It can be used in simple calculators, rule engines, filtering systems, Boolean expressions, configuration languages, and small domain-specific languages.

### Q11. Can Interpreter be implemented in Kotlin?

Yes. Kotlin supports interfaces, sealed classes, data classes, and recursive functions, all of which can be used to represent and evaluate expressions.

### Q12. Can Interpreter use the Composite Pattern?

Yes. Expression trees often have a composite-like structure: individual values form the leaves, and operations contain other expressions. Interpreter adds the rules for evaluating those nodes.

### Q13. What is the difference between an interpreter and a compiler?

An interpreter evaluates or executes expressions according to their meaning. A compiler translates source code into another representation, such as machine code or bytecode. Real language implementations can combine compilation and interpretation techniques.

### Q14. When should you avoid Interpreter?

Avoid building your own interpreter for a complex language when a mature parser or expression engine would be more suitable.

# 11. Quick Revision

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

## One-line memory trick

Interpreter Pattern = Represent the rules of a language as objects, then evaluate those objects to get a result.

The easiest way to remember it is to compare these three examples:

- Command: Execute an instruction — `LightOnCommand`.
- Interpreter: Evaluate an expression — `5 + 3`.
- Chain of Responsibility: Forward a request — Level 1 to Level 2 support.
