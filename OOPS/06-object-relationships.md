# Object Relationships (Association, Aggregation, Composition, Dependency)

## What is it?

Object relationships describe how classes and objects interact, depend on each other, and manage each other's lifecycles. Rather than defaulting to inheritance ("is-a"), robust software is built primarily on associations ("has-a" and "uses-a") with explicit ownership boundaries.

## Simple Explanation

- **Dependency ("uses-a")**: A person borrows a pen to sign a receipt. Once signed, the pen is returned; the person does not own or keep the pen.
- **Aggregation ("has-a", weak association)**: A university has students. If the university closes down, the students still exist independently.
- **Composition ("has-a", strong association)**: A human body has a heart. If the body dies, the heart cannot survive on its own. The container owns the lifecycle.

## Hinglish Explanation

Object relationships batate hain ki do objects ke beech ka connection kaisa hai. Dependency matlab temporary use ("uses-a"), jaise method parameter. Aggregation matlab container object doosre object ko hold karta hai par dono ki life independent hai ("weak has-a"). Composition mein child object ki life parent object ke sath bandhi hoti hai—parent destroy hua toh child bhi khatam ("strong has-a").

## Why Do We Need It?

Inheritance couples classes tightly to a shared hierarchy. Modeling systems with explicit relationships keeps components modular, prevents memory leaks, makes testing easy with mock dependencies, and enforces clean domain boundaries.

## How It Works

Relationships form a spectrum from loose coupling (dependency) to tightest coupling (composition):

```text
Dependency  ──►  Association  ──►  Aggregation  ──►  Composition
(Method arg)      (Reference)      (Shared life)     (Owned life)
  Loose Coupling ─────────────────────────────────► Tight Coupling
```

### Java Example

```java
// Dependency: OrderService "uses" PaymentGateway temporarily in method
class PaymentGateway { void process(double amt) {} }

// Composition: Engine lifecycle is tied to Car
class Engine { void start() {} }

// Aggregation: Driver can exist without the Car
class Driver { String name; Driver(String name) { this.name = name; } }

class Car {
    private final Engine engine = new Engine(); // Composition: Car creates & owns Engine
    private Driver driver;                      // Aggregation: injected from outside

    void setDriver(Driver d) { this.driver = d; }
    void run(PaymentGateway toll) {             // Dependency: passed as parameter
        engine.start();
        toll.process(50.0);
    }
}
```

### Python Example

```python
class Engine:
    def start(self): pass

class Driver:
    def __init__(self, name: str):
        self.name = name

class PaymentGateway:
    def process(self, amount: float): pass

class Car:
    def __init__(self):
        self._engine = Engine()  # Composition: Engine created & destroyed with Car
        self.driver = None       # Aggregation: Driver assigned externally

    def set_driver(self, driver: Driver):
        self.driver = driver

    def run(self, toll_booth: PaymentGateway):  # Dependency: method parameter
        self._engine.start()
        toll_booth.process(50.0)
```

## Important Terminology

| Term | Meaning | Typical Notation |
|---|---|---|
| **Association** | General link between two independent classes | Solid line (`───`) |
| **Dependency** | Transient relationship where one class uses another (method parameter/local var) | Dashed arrow (`- - - ►`) |
| **Aggregation** | Whole-part relationship where parts survive without the whole (loose ownership) | Hollow diamond (`◇───`) |
| **Composition** | Whole-part relationship where parts cannot exist without the whole (strict ownership) | Filled diamond (`◆───`) |
| **Multiplicity** | Cardinality of relationship (e.g., `1 to 1`, `1 to *`, `* to *`) | Numbers/symbols (`1..*`) |

## Example

In an E-Commerce system:
- `Order` **has-a** list of `OrderItem` (Composition: items make no sense outside their specific order).
- `Order` **has-a** `Customer` (Aggregation: if the order is cancelled, customer profile remains).
- `Order.calculateTax(TaxCalculator calc)` **uses-a** `TaxCalculator` (Dependency: temporary service used in a calculation).

## Real-World Example

Microservices and domain models avoid database cascading deletes when using aggregation (e.g., deleting a `Department` does NOT delete `Employee` records). Conversely, within an Aggregate Root in Domain-Driven Design (DDD), composition guarantees that child entities (e.g., `InvoiceLineItem`) are always saved, updated, and deleted with the root (`Invoice`).

## Important Properties

| Relationship | Relationship Type | Lifecycle Ownership | Can Part Exist Alone? | Coupling |
|---|---|---|---|---|
| **Dependency** | Uses-a | None (method scope) | Yes | Very Low |
| **Association** | Interacts-with | Independent | Yes | Low |
| **Aggregation** | Has-a (Weak) | Shared / External | Yes | Medium |
| **Composition** | Has-a (Strong) | Parent controls part | No | High |
| **Inheritance** | Is-a | Unified object | No (derived needs base) | Very High |

### Common Misconceptions

* Aggregation and Composition are not the same thing; the key differentiator is **lifecycle ownership**.
* Composition does not require child objects to be physically allocated in the same block of memory (in Java/Python, they are separate heap references); it requires semantic lifecycle ownership.
* Not every relationship should be modeled as an instance variable; transient actions belong as method dependencies (`Dependency`).

### What Interviewers Usually Test

1. Distinguishing Aggregation vs. Composition with real-world examples.
2. Explaining why "favor composition over inheritance" is an industry best practice.
3. Identifying relationships from code snippets in Java and Python.
4. Explaining cascading lifecycle and memory leak implications when containers hold references.

### Common Follow-up Questions

1. How do you implement Composition in Java vs Python?
2. What happens to child objects in Composition when the parent is garbage collected?
3. What is the difference between "Association" and "Aggregation"?
4. Why is Dependency considered the loosely coupled relationship?
5. How does the "Law of Demeter" relate to object relationships?

### Follow-up Answers

1. In both Java and Python, composition is implemented by creating the child object internally within the parent's constructor or factory, without exposing raw references that allow external lifecycle management.
2. When the parent object becomes unreachable and is garbage collected, its held references to child objects drop; if no other references exist, the child objects are collected too.
3. Association is any structural link between two classes (e.g., Teacher and Student). Aggregation is a specialized association representing a whole-part hierarchy where the child survives independently.
4. Dependency is transient: class A only receives class B as a parameter, creates it as a local variable, or calls static methods on it, rather than maintaining a persistent field reference.
5. The Law of Demeter ("principle of least knowledge") states that an object should only talk to its immediate friends (own fields, parameters, created objects), avoiding long relationship chains like `order.getCustomer().getAddress().getCity()`.
