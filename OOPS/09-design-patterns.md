# Design Patterns in Object-Oriented Programming

## What is it?

Design patterns are battle-tested, reusable solutions to common recurring architectural problems in software design. First formalized by the "Gang of Four" (GoF), patterns provide a shared vocabulary and structured blueprint for writing flexible, decoupled code.

## Simple Explanation

- **Creational (How to create objects)**: Instead of ordering custom brick-by-brick manufacturing on-site every time, you use a specialized factory or a step-by-step assembly line (Builder).
- **Structural (How to compose objects)**: An electrical adapter lets a 3-pin UK laptop plug fit into a 2-pin European socket (Adapter).
- **Behavioral (How objects communicate)**: A YouTube channel publishes videos; thousands of subscribed users automatically receive alerts without the creator knowing each viewer's home address (Observer).

## Hinglish Explanation

Design patterns ready-made code templates nahi hain, balki recurring design problems ke standard solutions hain:
- **Creational**: Objects kaise bante hain unhe control karna (Singleton, Factory, Builder).
- **Structural**: Classes aur objects ko kaise jodein taaki bada structure flexible rahe (Adapter, Decorator, Facade).
- **Behavioral**: Objects ke beech algorithms aur responsibilities kaise distribute karein (Observer, Strategy, Command).

## Why Do We Need It?

Without patterns:
1. Developers reinvent ad-hoc solutions with hidden bugs (e.g., race conditions in multithreaded singletons).
2. Code reviews lack a shared vocabulary ("use an Adapter here" communicates intent in 4 words vs. 2 pages of explanation).
3. Modifying existing behaviors requires rewriting large blocks of client code.

## How It Works

### The 3 Core Pattern Categories

```text
┌──────────────────────────────────────────────────────────────┐
│                    GoF Design Patterns                       │
├────────────────────┬────────────────────┬────────────────────┤
│     Creational     │     Structural     │     Behavioral     │
├────────────────────┼────────────────────┼────────────────────┤
│ • Singleton        │ • Adapter          │ • Strategy         │
│ • Factory Method   │ • Decorator        │ • Observer         │
│ • Builder          │ • Facade           │ • Command          │
│ • Abstract Factory │ • Proxy            │ • State            │
│ • Prototype        │ • Composite        │ • Chain of Resp.   │
└────────────────────┴────────────────────┴────────────────────┘
```

### 1. Strategy Pattern in Python

Allows swapping algorithms at runtime without changing the context object:

```python
from abc import ABC, abstractmethod

class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount: float) -> None: pass

class UpiPayment(PaymentStrategy):
    def __init__(self, upi_id: str): self.upi_id = upi_id
    def pay(self, amount: float): print(f"Paid ₹{amount} via UPI ({self.upi_id})")

class CreditCardPayment(PaymentStrategy):
    def __init__(self, card_no: str): self.card_no = card_no
    def pay(self, amount: float): print(f"Paid ₹{amount} via Card ({self.card_no[-4:]})")

class ShoppingCart:
    def __init__(self, strategy: PaymentStrategy):
        self._strategy = strategy

    def set_strategy(self, strategy: PaymentStrategy):
        self._strategy = strategy

    def checkout(self, amount: float):
        self._strategy.pay(amount)
```

### 2. Thread-Safe Singleton in Java (Double-Checked Locking)

```java
public final class DatabaseConnection {
    // volatile prevents instruction reordering in JVM
    private static volatile DatabaseConnection instance;

    private DatabaseConnection() {
        // Private constructor prevents direct instantiation
    }

    public static DatabaseConnection getInstance() {
        if (instance == null) {                         // 1st Check (no lock)
            synchronized (DatabaseConnection.class) {
                if (instance == null) {                 // 2nd Check (with lock)
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }
}
```

### 3. Builder Pattern in Java

```java
public class User {
    private final String name;
    private final String email;
    private final int age;

    private User(Builder b) {
        this.name = b.name;
        this.email = b.email;
        this.age = b.age;
    }

    public static class Builder {
        private final String name;    // required
        private String email = "";    // optional
        private int age = 0;          // optional

        public Builder(String name) { this.name = name; }
        public Builder email(String e) { this.email = e; return this; }
        public Builder age(int a) { this.age = a; return this; }
        public User build() { return new User(this); }
    }
}
// Usage: User u = new User.Builder("Anil").email("anil@test.com").age(25).build();
```

## Important Terminology

| Pattern | Category | Core Intent |
|---|---|---|
| **Singleton** | Creational | Ensures a class has only one instance and provides global access |
| **Factory Method**| Creational | Delegates object instantiation to subclasses or specialized methods |
| **Builder** | Creational | Separates complex object construction from its representation |
| **Adapter** | Structural | Converts the interface of a class into another interface clients expect |
| **Decorator** | Structural | Dynamically attaches additional responsibilities to an object |
| **Facade** | Structural | Provides a simplified, high-level interface to a complex subsystem |
| **Proxy** | Structural | Provides a surrogate or placeholder to control access to another object |
| **Observer** | Behavioral | 1-to-many dependency where state changes notify all observers |
| **Strategy** | Behavioral | Defines a family of algorithms and makes them interchangeable |
| **State** | Behavioral | Allows an object to alter its behavior when its internal state changes |

## Example: Decorator Pattern vs. Inheritance

Instead of creating explosion of subclasses (`CoffeeWithMilk`, `CoffeeWithMilkAndCaramel`, `CoffeeWithSugar`), the **Decorator pattern** wraps a base coffee object dynamically at runtime:
`Beverage order = new CaramelDecorator(new MilkDecorator(new SimpleCoffee()));`

## Real-World Example

- **Java Standard Library**: `java.lang.Runtime` (Singleton), `InputStreamReader(new FileInputStream())` (Decorator/Adapter), `StringBuilder` (Builder).
- **Python Ecosystem**: Django ORM QuerySets (Builder), `@property` / `@functools.wraps` (Decorator), Logging handlers (Observer/Chain of Responsibility).

## Important Properties

| Comparison | Pattern A | Pattern B | Key Difference |
|---|---|---|---|
| **Adapter vs. Decorator** | Adapter | Decorator | Adapter changes the interface; Decorator keeps the interface but adds behavior |
| **Strategy vs. State** | Strategy | State | Strategy is selected by the client externally; State transitions happen internally based on object state |
| **Facade vs. Adapter** | Facade | Adapter | Facade defines a new simpler interface; Adapter makes two existing incompatible interfaces work together |
| **Factory vs. Builder** | Factory | Builder | Factory creates object in a single shot; Builder constructs complex multi-step objects |

### Common Misconceptions

* Singleton is not always a good idea; overusing it introduces hidden dependencies and makes unit testing difficult due to shared global state.
* Design patterns are not to be forced everywhere; applying patterns when simple straightforward code works is known as "Patternitis" (over-engineering).
* In Python, first-class functions and duck typing often eliminate the need for verbose GoF patterns (e.g., passing a function directly instead of creating an explicit Strategy class).

### What Interviewers Usually Test

1. Implementing thread-safe Singleton in Java (explaining why `volatile` and double-checked locking are necessary).
2. Distinguishing between Adapter, Decorator, and Proxy.
3. Choosing the right pattern for a scenario (e.g., notification systems, caching proxies, discount calculators).
4. Explaining the Open/Closed Principle benefits of Factory and Strategy patterns.

### Common Follow-up Questions

1. Why is `volatile` required in Java double-checked locking?
2. How is Singleton implemented idiomatically in Python?
3. What is the difference between Factory Method and Abstract Factory?
4. How does the Observer pattern relate to modern Event-Driven Architecture?
5. When would you prefer the Builder pattern over a constructor with default parameters?

### Follow-up Answers

1. Without `volatile`, the JVM/CPU can reorder instructions (allocating memory, assigning reference, running constructor). Another thread might see `instance != null` before the constructor finishes running, accessing a partially initialized object.
2. In Python, a module is already a cached singleton. Alternatively, overriding `__new__` on the class, using a metaclass, or applying a decorator handles explicit singleton needs.
3. Factory Method creates instances of a single product using inheritance/methods; Abstract Factory creates families of related or dependent products (e.g., `MacUIFactory` creates `MacButton` and `MacCheckbox`).
4. Observer is the in-memory, single-process foundation of Event-Driven Architecture; distributed systems scale this concept across networks using message brokers like Kafka and RabbitMQ.
5. When an object has many optional parameters (telescoping constructor antipattern), complex validation rules before construction, or when creating immutable objects with clean, fluent readability.
