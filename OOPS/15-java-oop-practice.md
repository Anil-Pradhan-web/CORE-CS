# Modern Java OOP in Practice

## What is it?

Modern Java (Java 8 through Java 21+) introduces language features that dramatically modernize Object-Oriented Programming: **Records**, **Sealed Classes**, **Pattern Matching**, **Interface Default Methods**, and **Rich Enums**. These features eliminate ceremonial boilerplate and enable algebraic data types and domain-driven modeling.

## Simple Explanation

- **Old Java**: Writing a simple 3-field data object required 50 lines of boilerplate (constructor, getters, setters, `equals`, `hashCode`, `toString`).
- **Modern Java Record**: A single line: `public record Point(int x, int y) {}`. The compiler automatically creates immutability, getters, constructor, `equals`, `hashCode`, and `toString` with zero ceremony.

## Hinglish Explanation

Pehle Java mein DTOs ya Value Objects banane ke liye Lombok ya boilerplate code likhna padta tha. Modern Java mein:
- **Records**: Pure immutable data carriers hain jo ek line mein ban jaate hain.
- **Sealed Classes**: Class hierarchy ko restrict karte hain (`permits` keyword se)—sirf allowed classes hi inherit kar sakti hain.
- **Pattern Matching**: `instanceof` ke baad explicit casting karne ki zaroorat nahi padti.
- **Rich Enums**: Enums sirf constants nahi, unke paas methods, fields, aur abstract implementations ho sakti hain.

## Why Do We Need It?

1. **Safety via Immutability**: Records provide true immutability by default, preventing bugs in concurrent multi-threaded applications.
2. **Exhaustive Domain Modeling**: Sealed hierarchies allow the compiler to verify that all possible subclasses are handled in `switch` expressions without needing a fallback `default` case.
3. **API Evolution**: Default interface methods let library authors add new methods to existing interfaces without breaking third-party implementations.

## How It Works

### 1. Records and Compact Constructors

```java
public record PaymentRequest(String orderId, double amount, String currency) {
    // Compact Constructor for invariant validation
    public PaymentRequest {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
        if (currency == null || currency.isBlank()) {
            throw new IllegalArgumentException("Currency is required");
        }
    }
}
```

### 2. Sealed Classes and Pattern Matching

```java
// Define a strictly controlled domain hierarchy
public sealed interface PaymentResult permits Success, Failure, Pending {}

public record Success(String transactionId) implements PaymentResult {}
public record Failure(String reason, int errorCode) implements PaymentResult {}
public record Pending(long timeoutMs) implements PaymentResult {}

// Pattern Matching Switch (Exhaustive check by compiler!)
public class PaymentRouter {
    public static String handleResult(PaymentResult result) {
        return switch (result) {
            case Success s -> "Payment succeeded with ID: " + s.transactionId();
            case Failure f -> "Payment failed: " + f.reason() + " (Code " + f.errorCode() + ")";
            case Pending p -> "Waiting for confirmation, timeout in " + p.timeoutMs() + "ms";
            // No default clause needed! The compiler knows all 3 permitted types.
        };
    }
}
```

### 3. Rich Enums (State Pattern via Enum)

Enums in Java are full classes that can define fields, constructors, and polymorphic methods:

```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    MULTIPLY("*") {
        public double apply(double x, double y) { return x * y; }
    };

    private final String symbol;
    Operation(String symbol) { this.symbol = symbol; }
    public String getSymbol() { return symbol; }

    public abstract double apply(double x, double y);
}
```

## Important Terminology

| Feature | Introduced | Primary Purpose |
|---|---|---|
| **Record** | Java 14 / 16 | Transparent, immutable shallow data carrier class |
| **Sealed Types** | Java 17 | Restricts which other classes or interfaces may extend or implement them |
| **Pattern Matching** | Java 16 / 21 | Tests expressions against patterns and binds variables automatically |
| **Default Methods** | Java 8 | Provides default implementation in interfaces for non-breaking API evolution |
| **`Optional<T>`** | Java 8 | Container object representing presence or absence of a non-null value |

## Example: Pattern Matching for `instanceof`

```java
// Traditional Java (Clumsy cast)
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// Modern Java (Pattern matching)
if (obj instanceof String s && !s.isBlank()) {
    System.out.println(s.length()); // Directly usable as variable 's'
}
```

## Real-World Example

In Spring Boot 3+ (Java 17/21):
- DTOs and API request/response payloads are defined as **Records**.
- Domain event models are defined as **Sealed Interfaces** (`DomainEvent permits OrderCreated, OrderShipped, OrderCancelled`).
- Kafka message listeners use pattern-matching `switch` statements to process event types with compile-time exhaustiveness guarantees.

## Important Properties

| Modern Feature | Benefit | Restriction / Anti-pattern |
|---|---|---|
| **Record** | Auto-generates canonical constructor, getters, `equals`, `hashCode` | Cannot extend classes (always extends `Record`); all fields are `final` |
| **Sealed Hierarchy**| Closed polymorphism; compile-time completeness in switch | Subclasses must be in the same package/module and declare `final`, `sealed`, or `non-sealed` |
| **`Optional<T>`** | Explicitly models missing return value without `null` | Never use as class field, method parameter, or wrap collections in it |
| **Interface Defaults**| Backward compatibility in libraries | Overuse creates multiple-inheritance diamond ambiguities in interfaces |

### Common Misconceptions

* Records are NOT just Lombok's `@Data` equivalent; `@Data` generates mutable POJOs with setters, whereas Records are inherently **immutable** and transparent data carriers.
* `Optional.get()` should not be used without checking `isPresent()`; prefer functional methods like `.orElse()`, `.orElseThrow()`, or `.map()`.
* Sealed classes do not eliminate polymorphism; they enable **closed polymorphism**, where all allowed variants are known to the domain model at compile time.

### What Interviewers Usually Test

1. Differences between a standard Java class, an abstract class, and a Record.
2. How Sealed Classes work and why they improve pattern matching with `switch`.
3. How Java resolves multiple inheritance conflicts with interface `default` methods.
4. Correct usage of `Optional` (anti-patterns to avoid).
5. Why Enums with abstract methods are superior to `switch` statements for Strategy patterns.

### Common Follow-up Questions

1. Can a Record implement an interface? Can it extend another class?
2. What are the three modifier options for subclasses of a `sealed` class?
3. How does Java resolve conflicts if two interfaces provide the exact same `default` method?
4. Why is `Optional` not serializable?
5. What is a "Compact Constructor" in a Record?

### Follow-up Answers

1. Yes, a Record can implement multiple interfaces. No, it cannot extend any class because it implicitly extends `java.lang.Record` and is implicitly `final`.
2. A permitted subclass must explicitly be declared as either `final` (no further subclassing), `sealed` (further restricted subclassing), or `non-sealed` (open for unrestricted subclassing).
3. The compiler throws an error ("inherits unrelated defaults"). The implementing class must explicitly override the conflicting method and manually specify which interface implementation to call via `InterfaceName.super.method()`.
4. `Optional` was designed strictly as a method return type to represent absence of value in functional pipelines; making it serializable would add heavy memory footprint and serialization hazards.
5. A constructor syntax in Records where parameter lists are omitted (`public Point { ... }`); it is used to validate or normalize parameters before they are assigned to the record's private fields.
