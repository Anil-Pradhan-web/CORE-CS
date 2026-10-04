# 08. Testing, LLD, UML, and Architecture

## 1. Testing layers

### Unit test

Tests one small unit, such as `BankAccount.withdraw()`. It should be fast and independent.

### Integration test

Checks that real components work together, such as a service and a database.

### End-to-end test

Checks a user or API journey through the system. It is useful but slower and harder to diagnose.

**Hinglish:** Unit test chhote logic ke liye, integration test components ke connection ke liye, aur end-to-end test complete user flow ke liye hota hai.

## 2. Test structure: Arrange, Act, Assert

1. **Arrange:** Create valid input and dependencies.
2. **Act:** Call the behaviour under test.
3. **Assert:** Check the observable result.

```python
def test_withdraw_rejects_too_much_money() -> None:
    account = BankAccount(100)

    try:
        account.withdraw(150)
    except ValueError:
        return

    raise AssertionError("Expected ValueError")
```

The exact assertion style can use the testing library available in the project.

## 3. Test doubles

- **Stub:** Returns prepared answers.
- **Fake:** A lightweight working implementation, such as an in-memory repository.
- **Mock:** Verifies that a call happened.
- **Spy:** Records calls for later inspection.

Do not mock everything. Mocking a value object such as a simple `Money` class often makes the test harder to understand.

**Hinglish:** External dependency ko fake ya mock karo; apne simple domain objects ko normally real rakho. Test behaviour par focus karo, implementation details par nahi.

## 4. Test object invariants

Test both successful operations and invalid operations:

- Deposit a valid amount.
- Reject zero, negative, or excessive withdrawal.
- Check that a failed operation leaves the old state unchanged.
- Check equality and hash-based collection behaviour when the object is used as a key.

## 5. Refactoring versus rewriting

**Refactoring** changes the internal structure without changing observable behaviour. Tests protect the behaviour before and after the change.

A rewrite is a new implementation and needs its own plan, tests, migration, and risk review.

## 6. LLD workflow

For a Low-Level Design question:

1. Clarify requirements and constraints.
2. Identify actors, use cases, and nouns.
3. Define the smallest class model.
4. Assign responsibilities and ownership.
5. Model states and transitions.
6. Add collection and performance choices.
7. Add error cases and thread-safety concerns.
8. Test the important invariants.
9. Explain trade-offs.

**Hinglish:** LLD mein pehle problem samjho, fir classes banao. Har class ka ek clear responsibility hona chahiye.

## 7. Parking-lot design

### Requirements

- A parking spot has an available or occupied state.
- A ticket is created when a vehicle enters.
- Ticket is closed when the vehicle exits.
- Spot capacity and charges must be controlled.

### Small model

```text
ParkingLot
  └── manages Spot and Ticket

Spot
  ├── id
  ├── size
  └── isAvailable

Ticket
  ├── id
  ├── vehicleNumber
  ├── spot
  ├── entryTime
  └── exitTime

ParkingLot
  ├── findSpot(vehicleSize)
  ├── park(vehicle)
  └── unpark(ticket)
```

### Important rules

- A spot cannot be assigned to two active tickets.
- A closed ticket must not be used for a second exit.
- A vehicle needs a compatible spot size.
- A failed unpark operation must not corrupt the ticket or spot state.

## 8. Elevator design

Separate responsibilities:

```text
ElevatorController -> chooses requests
Elevator          -> moves between floors
Door              -> opens and closes safely
RequestQueue      -> orders requests fairly
```

Do not put scheduling, motor control, display formatting, and door safety in one giant `Elevator` class.

## 9. LRU cache design

```text
LRUCache
  ├── capacity
  ├── get(key)
  ├── put(key, value)
  └── recency structure
```

Questions to ask:

- What happens when capacity is zero?
- What is returned for a missing key?
- Does `get` update recency?
- Is the cache shared between threads?

Because `get` can update recency, it is a mutating operation from a concurrency perspective.

## 10. UML essentials

| Symbol | Meaning |
|---|---|
| `+` | Public |
| `-` | Private |
| `#` | Protected |
| Solid line | Association; an arrow can show navigability |
| `◇` | Aggregation: weak ownership |
| `◆` | Composition: strong ownership |
| Hollow triangle at a class | Generalization or inheritance |
| Hollow triangle at an interface | Interface realization |
| Dashed line with an open arrowhead | Dependency |

UML is a communication tool. The exact notation can vary by tool; always explain the relationship in words too.

## 11. Entity, value object, and aggregate

### Entity

Has a stable identity even when its values change, such as a customer with an ID.

### Value object

Is defined by its values, such as an immutable money value or an address pair.

### Aggregate

A consistency boundary around an entity and related objects. Changes inside one aggregate are managed as one unit.

**Hinglish:** Entity ki identity important hai, value object ki values important hain, aggregate state ko consistent rakhne ka boundary hai.

## 12. Layered architecture

```text
Controller / API
       ↓
Application service
       ↓
Domain model
       ↓
Repository or port
       ↓
Database or external adapter
```

The domain should not know that a controller is HTTP or that a repository is SQL. Ports and adapters keep those details replaceable.

## 13. Dependency direction

A common rule is that higher-level business policy depends on interfaces it owns. Low-level database and network code implements those interfaces and is injected from outside.

This is not a law that every small program needs many layers. Apply boundaries where they improve testability or protect important business rules.

## 14. Architecture trade-offs

- More layers can improve separation but add navigation and debugging cost.
- A repository can hide SQL but cannot make a slow query fast.
- Microservices do not automatically make a system scalable; they add network, deployment, and consistency problems.
- DDD is useful when the domain is complex, but a tiny CRUD app may not need every DDD concept.
- Caching improves speed but introduces invalidation, staleness, and concurrency concerns.

## 15. Interview LLD template

> First, I will clarify the requirements and constraints. Then I will model the core entities and state transitions. I will assign one responsibility to each class, choose the smallest collection needed, define failure cases, and discuss consistency and concurrency. Finally, I will test the important invariants.

## Quick interview Q&A

### 1. What is a unit test?

A test of one small unit with controlled dependencies and no need for a full system.

### 2. What is the difference between a fake and a mock?

A fake is a working simplified implementation; a mock records or verifies calls.

### 3. What is Arrange-Act-Assert?

It is a test structure: set up inputs, perform one action, and check the result.

### 4. What is refactoring?

Changing code structure without changing observable behaviour.

### 5. What is a code smell?

A sign that a design may be costly to understand, change, or test.

### 6. What is LLD?

Low-Level Design is the class, object, relationship, state, and implementation design for a system or feature.

### 7. What should you clarify in an LLD interview?

Actors, use cases, constraints, state transitions, data size, error cases, consistency, and concurrency.

### 8. What is aggregation in UML?

A whole-part relationship with weaker ownership, shown by a hollow diamond.

### 9. What is composition?

A strong whole-part relationship where the parent manages the child lifecycle, shown by a filled diamond.

### 10. What is an entity?

An object with a stable identity whose attributes can change.

### 11. What is a value object?

An object defined by its values, ideally immutable and compared by those values.

### 12. What is an aggregate?

A consistency boundary that groups an entity with related objects and controls state changes.

### 13. What is Dependency Inversion in architecture?

Business policy owns an abstraction such as a repository port; database code supplies the implementation.

### 14. Why avoid a God class?

It takes too many responsibilities, so unrelated changes, tests, and ownership become difficult.

### 15. Is every LRU cache read read-only?

No. If a read updates recency, it mutates state and needs a concurrency-safe design.

**Hinglish recap:** LLD ka answer “classes ki list” nahi hona chahiye. Requirements, state, ownership, failure, aur trade-offs explain karo.
