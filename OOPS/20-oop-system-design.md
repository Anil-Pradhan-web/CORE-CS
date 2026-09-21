# OOP in System Design and Architecture

## What is it?

Object-Oriented Programming doesn't stop at single-process classes; it forms the foundation of modern high-scale system design through **Domain-Driven Design (DDD)** and **Clean / Hexagonal Architecture**. This bridge shows how OOP principles scale from in-memory objects to microservices, database transactions, and distributed event streams.

## Simple Explanation

If you build a house, the kitchen sink shouldn't care whether the city water comes from a river or a mountain reservoir—it just connects to a standardized water pipe.
In Clean Architecture, your core business logic (e.g., how an interest rate is calculated) shouldn't care whether data comes from PostgreSQL, MongoDB, a REST API, or an automated test—it connects only to abstract domain interfaces ("Ports").

## Hinglish Explanation

Jab systems scale hote hain, tab procedural code ya "Anemic Domain Models" (sirf getters/setters wale dumb objects) maintain karna impossible ho jata hai. Domain-Driven Design (DDD) aur Clean Architecture OOP ke concepts ko enterprise level par le jaate hain:
- **Entities**: Jinhe unique ID se pehchana jata hai (e.g., `User(id=101)`).
- **Value Objects**: Jo immutable hote hain aur apni attributes se define hote hain (e.g., `Money(100, "USD")`).
- **Aggregate Root**: Multiple related entities ka gatekeeper jo transaction aur invariant rules enforce karta hai.
- **Clean / Hexagonal Architecture**: Business logic core mein rehti hai aur database, framework, UI bahar "Adapters" ki tarah plug hote hain via Interfaces ("Ports").

## Why Do We Need It?

1. **Avoid Anemic Domain Models**: Prevents dumb data bags where business logic leaks haphazardly into UI controllers or SQL queries.
2. **Framework Independence**: Allows swapping frameworks (e.g., switching Spring to Quarkus in Java, or Flask to FastAPI in Python) without touching a single line of business logic.
3. **Bounded Contexts**: Defines clear microservice boundaries so independent teams can deploy changes without cross-service merge conflicts.

## How It Works

### 1. Hexagonal Architecture (Ports and Adapters)

```text
               ┌──────────────────────────────────────────┐
               │              Outer Ring                  │
               │   (Frameworks, Web Controllers, DB)      │
               │                                          │
               │     ┌──────────────────────────────┐     │
               │     │       Application Layer      │     │
               │     │      (Use Cases, DTOs)       │     │
               │     │                              │     │
               │     │   ┌──────────────────────┐   │     │
               │     │   │      Domain Core     │   │     │
               │     │   │ (Entities, Aggregates│   │     │
               │     │   │    Value Objects)    │   │     │
               │     │   └──────────────────────┘   │     │
               │     │    ▲ Ports (Interfaces)      │     │
               │     └────┼─────────────────────────┘     │
               │          │ Adapters                      │
               │     Postgres / Kafka / REST              │
               └──────────┴───────────────────────────────┘
```
**The Dependency Rule**: Source code dependencies point strictly inwards. The Domain Core knows nothing about the outer database, network, or UI!

### 2. Rich Domain Entity vs. Anemic Model (Java & Python)

#### The Anemic Anti-Pattern (Bad OOP)
```java
// Dumb data bag: any caller can set an invalid negative balance!
public class AnemicAccount {
    private double balance;
    public double getBalance() { return balance; }
    public void setBalance(double balance) { this.balance = balance; }
}
```

#### Rich Domain Model (Clean OOP & DDD)
```java
// Encapsulated invariants, explicit domain actions
public class BankAccount {
    private final AccountId id;
    private Money balance;

    public BankAccount(AccountId id, Money initialDeposit) {
        if (initialDeposit.isNegative()) throw new DomainException("Initial deposit cannot be negative");
        this.id = id;
        this.balance = initialDeposit;
    }

    public void withdraw(Money amount) {
        if (amount.isGreaterThan(this.balance)) {
            throw new InsufficientFundsException(id, amount, this.balance);
        }
        this.balance = this.balance.subtract(amount);
    }
}
```

```python
# Python DDD: Value Object & Entity
from dataclasses import dataclass

@dataclass(frozen=True) # Value Object: Immutable, equality by value
class Money:
    amount: float
    currency: str

    def subtract(self, other: 'Money') -> 'Money':
        if self.currency != other.currency:
            raise ValueError("Currency mismatch")
        return Money(self.amount - other.amount, self.currency)

class Order: # Entity & Aggregate Root: Identity-based
    def __init__(self, order_id: str, customer_id: str):
        self.id = order_id
        self.customer_id = customer_id
        self._items = []
        self._status = "CREATED"

    def add_item(self, product_id: str, price: Money):
        if self._status != "CREATED":
            raise RuntimeError("Cannot modify finalized order")
        self._items.append((product_id, price))
```

## Important Terminology

| DDD Concept | Definition | Example |
|---|---|---|
| **Entity** | Object defined by unique identity that persists across time | `User`, `Order`, `BankAccount` |
| **Value Object** | Immutable object defined entirely by its attribute values | `Money`, `Address`, `DateRange` |
| **Aggregate Root** | The entry-point entity that safeguards consistency across a cluster of objects | `Order` (owns `OrderLineItems`) |
| **Repository** | Abstraction emulating an in-memory collection for persisting aggregate roots | `OrderRepository` (`save`, `findById`) |
| **Domain Event** | Record of an immutable, significant business occurrence in the past | `OrderShippedEvent`, `UserRegistered` |
| **Bounded Context**| Boundary within an organization where a specific domain model applies | "Billing Context" vs. "Inventory Context" |

## Example: Microservice Domain Mapping

In an enterprise e-commerce platform:
- In the **Inventory Service**, a `Product` entity cares about SKU, warehouse location, and stock quantity.
- In the **Catalog Service**, a `Product` entity cares about title, images, customer reviews, and SEO tags.
Instead of building one monstrous "God Product" shared across all databases, each microservice defines its own clean, focused Bounded Context and domain model!

## Real-World Example

- **Uber Driver Dispatch**: The `Ride` entity is the Aggregate Root. Riders and Drivers cannot mutate ride status directly. All state transitions (`REQUESTED` -> `ACCEPTED` -> `IN_PROGRESS` -> `COMPLETED`) are strictly guarded by `Ride` methods, publishing `RideAcceptedEvent` to Apache Kafka for asynchronous notifications.
- **Netflix Microservices**: Uses Hexagonal Architecture. If Netflix changes their playback telemetry database from Cassandra to ScyllaDB, only the outer repository Adapter is rewritten; zero business domain logic is touched.

## Important Properties

| Architectural Layer | Responsibilities | Allowed Dependencies |
|---|---|---|
| **Domain Core** | Entities, Value Objects, Domain Events, Invariants | **None** (pure Java/Python, no external libraries) |
| **Application Layer** | Use cases, orchestrating domain objects, transaction boundaries | Domain Core |
| **Infrastructure Layer**| SQL queries, Kafka producers, REST controllers, 3rd-party SDKs | Application & Domain Core (via Port interfaces) |

### Common Misconceptions

* Value Objects do not have IDs; if two `Address` objects have the exact same street, city, and zip code, they are identical.
* An Aggregate Root should only reference other Aggregate Roots by their **ID**, not by direct object references; direct object nesting across aggregate boundaries breaks transactional isolation.
* Clean Architecture is not overkill for production enterprise backends; it is what prevents applications from becoming legacy unmaintainable monoliths.

### What Interviewers Usually Test

1. Difference between an Entity and a Value Object in Domain-Driven Design.
2. Explaining the Anemic Domain Model anti-pattern and how to fix it.
3. How the Dependency Inversion Principle forms the core of Hexagonal Architecture.
4. What an Aggregate Root is and why external objects cannot mutate child entities directly.
5. Defining Bounded Contexts when decomposing monoliths into microservices.

### Common Follow-up Questions

1. Why must Value Objects be immutable?
2. What is an Anemic Domain Model, and why is it considered an anti-pattern?
3. How does Hexagonal Architecture make automated testing easier?
4. What is the role of a Domain Event?
5. How do you maintain transactional consistency across multiple aggregates?

### Follow-up Answers

1. Because Value Objects represent pure descriptive values (like numbers or colors). If a shared `Address` or `Money` object could be mutated, changing it in one place would silently corrupt all other entities referencing that value.
2. It is an anti-pattern where classes contain only getters and setters with zero business rules, while business logic is scattered across giant procedural service classes, breaking object encapsulation.
3. Because the domain logic depends solely on abstract interfaces (Ports), unit tests can run 100% in-memory using mock or fake adapters without spinning up databases, web servers, or cloud dependencies.
4. It decouples side effects: when an aggregate mutates state, it records a Domain Event (e.g., `OrderPaid`). Other independent systems listen to this event and trigger emails, analytics, and shipping asynchronously.
5. Within a single aggregate, use database ACID transactions. Across multiple aggregates or microservices, use Eventual Consistency and the Saga Pattern (orchestrated or choreographed events with compensating transactions).
