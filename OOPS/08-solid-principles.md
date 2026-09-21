# SOLID Principles in Object-Oriented Design

## What is it?

SOLID is a set of five foundational design principles for writing maintainable, understandable, and flexible object-oriented software. Introduced by Robert C. Martin ("Uncle Bob"), these principles guide the design of classes, interfaces, and module boundaries.

## Simple Explanation

- **S (Single Responsibility)**: A chef cooks; an accountant handles bills. Don't make the chef balance the restaurant's taxes.
- **O (Open/Closed)**: A smartphone is open to adding new apps via the app store, but its internal hardware and OS core are closed for direct modification.
- **L (Liskov Substitution)**: If an electric car replaces a petrol car, the driver shouldn't have to relearn how to steer or brake.
- **I (Interface Segregation)**: Don't force a simple user to sign a 50-page legal document when they only need a library membership card.
- **D (Dependency Inversion)**: A wall plug has a standard outlet; your lamp plugs into the outlet, not directly soldered into the building's electrical wiring.

## Hinglish Explanation

SOLID principles code ko spaghetti banne se rokte hain:
- **S**: Ek class ka sirf ek hi reason hona chahiye change hone ka.
- **O**: Existing code ko bina modify kiye naye features add hone chahiye (via abstraction/inheritance/polymorphism).
- **L**: Derived class ko base class ki jagah safely use kiya ja sake bina kisi runtime crash ya unexpected behavior ke.
- **I**: Large "fat" interfaces mat banao; chote, focused interfaces banao taaki clients un methods ko implement na karein jinki unhe zaroorat nahi.
- **D**: High-level modules low-level concrete classes par depend na karein; dono abstractions (interfaces/protocols) par depend karein.

## Why Do We Need It?

Without SOLID, software rots:
- **Rigidity**: A single small change triggers a cascade of modifications across unrelated files.
- **Fragility**: Changing code in module A unexpectedly breaks functionality in module B.
- **Immobility**: Components are so tightly tangled that they cannot be extracted or reused in other projects.

## How It Works

### The 5 Principles Overview

```text
S ──► Single Responsibility   ──► One reason to change per class
O ──► Open/Closed             ──► Open for extension, closed for modification
L ──► Liskov Substitution     ──► Subtypes must be substitutable for base types
I ──► Interface Segregation   ──► Client-specific small interfaces
D ──► Dependency Inversion    ──► Depend upon abstractions, not concretions
```

### Python Implementation: Before vs. After (DIP & OCP)

```python
from abc import ABC, abstractmethod

# VIOLATION: NotificationService is tightly coupled to concrete EmailSender
class EmailSender:
    def send_email(self, msg: str): print(f"Email: {msg}")

# CLEAN (DIP & OCP): Depend on abstraction
class MessageSender(ABC):
    @abstractmethod
    def send(self, message: str) -> None: pass

class EmailSenderImpl(MessageSender):
    def send(self, message: str) -> None: print(f"Email: {message}")

class SmsSenderImpl(MessageSender):
    def send(self, message: str) -> None: print(f"SMS: {message}")

class NotificationService:
    def __init__(self, sender: MessageSender):  # Injected abstraction
        self._sender = sender

    def notify(self, message: str) -> None:
        self._sender.send(message)
```

### Java Implementation: LSP & ISP

```java
// ISP: Segregate large worker interface into focused contracts
interface Workable { void work(); }
interface Eatable { void eat(); }

class HumanWorker implements Workable, Eatable {
    public void work() { System.out.println("Working"); }
    public void eat() { System.out.println("Eating lunch"); }
}

class RobotWorker implements Workable {
    public void work() { System.out.println("Operating 24/7"); }
    // No need to implement eat() and throw UnsupportedOperationException (which violates LSP!)
}
```

## Important Terminology

| Principle | Primary Goal | Smells When Violated |
|---|---|---|
| **SRP** | Cohesion | "God classes" with 2000+ lines doing DB, validation, UI |
| **OCP** | Extensibility | Massive `switch`/`if-else` blocks checking type strings |
| **LSP** | Correct Subtyping | Subclass overrides method with `throw new UnsupportedOperationException()` |
| **ISP** | Decoupling | Empty method stubs in implementing classes |
| **DIP** | Inversion of Control | Using `new ConcreteRepository()` inside business service constructor |

## Example: Classic LSP Violation (Rectangle vs. Square)

A classic mistake: making `Square extends Rectangle`.
If `Rectangle` has `setWidth(w)` and `setHeight(h)`, setting width on a `Square` also mutates height. A function expecting a `Rectangle` with independent dimensions will fail test assertions when passed a `Square`.
**Solution**: Both should implement a common `Shape` interface with `area()`, rather than inheriting mutable geometry from each other.

## Real-World Example

In Spring Boot (Java) and FastAPI / Dependency Injector (Python):
Controllers do not instantiate SQL databases directly. They receive an interface (`UserRepository`). When testing, a `FakeUserRepository` is injected; in production, `PostgresUserRepository` is injected. This satisfies DIP, OCP, and SRP cleanly.

## Important Properties

| Principle | Mechanism in Java | Mechanism in Python |
|---|---|---|
| **SRP** | Small single-purpose classes | Single-purpose modules and classes |
| **OCP** | Interfaces & Strategy pattern | Abstract Base Classes (`abc.ABC`) / Duck typing |
| **LSP** | Compiler subtype checks + contract tests | Contract tests & consistent method signatures |
| **ISP** | Multiple focused interfaces (`implements A, B`) | Multiple small ABCs or typing `Protocol`s |
| **DIP** | Constructor Injection via Interfaces / Spring `@Autowired` | Dependency Injection / Passing Protocols or Callables |

### Common Misconceptions

* SRP does not mean a class must only have one single method; it means it should have only one *actor* or *business responsibility*.
* OCP does not mean you can never modify an existing file (bug fixes and requirement changes still require edits); it means new variants should be added via new implementations, not by hacking existing branching logic.
* SOLID is not a dogma to over-engineer trivial scripts; applying all 5 principles to a 10-line utility script adds accidental complexity.

### What Interviewers Usually Test

1. Asking you to explain the classic `Square vs. Rectangle` LSP violation.
2. Spotting SOLID violations in a provided messy code snippet.
3. Explaining the difference between Dependency Inversion (principle), Inversion of Control (pattern), and Dependency Injection (technique).
4. Refactoring an `if-else` chain using OCP and Strategy Pattern.

### Common Follow-up Questions

1. What is the difference between DIP, IoC, and DI?
2. How does LSP relate to method preconditions and postconditions?
3. Can a class have multiple methods and still satisfy SRP?
4. How do Python Protocols help achieve ISP?
5. When should you choose NOT to apply SOLID?

### Follow-up Answers

1. **DIP** is the high-level design principle (depend on abstractions). **IoC** is the architectural pattern (inverting control flow to a container/framework). **DI** is the concrete implementation mechanism (passing dependencies via constructor/setter).
2. By Bertrand Meyer's Design by Contract: a subtype may weaken preconditions (demand less) but cannot strengthen them; it may strengthen postconditions (guarantee more) but cannot weaken them.
3. Yes. A `UserRepository` can have `findById`, `save`, and `delete`. All these methods serve a single cohesive responsibility: persistence of user entities.
4. Python's `typing.Protocol` enables structural subtyping (static duck typing). Clients define only the exact subset of methods they need, and any class implementing those methods matches the protocol automatically without explicit inheritance.
5. In simple scripts, prototypes, rapidly evolving exploratory spikes, or performance-critical embedded loops where indirection and multiple interface lookups create measurable overhead.
