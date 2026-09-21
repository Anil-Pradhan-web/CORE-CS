# Testing, Code Smells, and Refactoring in OOP

## What is it?

Writing clean object-oriented software requires three interrelated disciplines: writing deterministic **unit tests** using test doubles, identifying **code smells** (symptoms of poor design or structural decay), and applying disciplined **refactorings** to improve internal structure without altering external behavior.

## Simple Explanation

- **Code Smell**: A squeaking car engine. The car still drives, but the squeak warns of friction that will cause an engine breakdown if ignored.
- **Refactoring**: Replacing worn engine belts and re-tuning the spark plugs while parked safely in the garage, ensuring the car still drives exactly the same way—only smoother and more reliably.
- **Unit Testing with Test Doubles**: Practicing emergency maneuvers on a flight simulator rather than crashing a real commercial aircraft.

## Hinglish Explanation

Code smells bugs nahi hote—woh bad design ke symptoms hote hain jo future modifications ko mushkil bana dete hain (jaise God Class, Feature Envy, Shotgun Surgery). Refactoring ka matlab hai bina external behavior change kiye code ko clean aur maintainable banana. Unit testing mein hum real slow dependencies (jaise Database, Payment Gateway) ki jagah **Mocks**, **Stubs**, ya **Fakes** use karte hain taaki tests fast aur isolated rahein.

## Why Do We Need It?

1. **Agility**: Refactored code with comprehensive tests allows teams to deploy features quickly without fear of regression bugs.
2. **Speed of Verification**: Mocking external services allows thousands of unit tests to execute in seconds in CI/CD pipelines.
3. **Prevent Technical Debt**: Eliminating code smells early prevents monolithic code decay.

## How It Works

### The 5 Types of Test Doubles (Gerard Meszaros Taxonomy)

```text
┌─────────────────────────────────────────────────────────────┐
│                        Test Double                          │
├──────────────┬──────────────┬──────────────┬────────────────┤
│    Dummy     │     Stub     │     Spy      │  Mock / Fake   │
├──────────────┼──────────────┼──────────────┼────────────────┤
│ Passed just  │ Returns      │ Records      │ Mock: Verifies │
│ to fill a    │ canned       │ calls/params │ expectations   │
│ parameter    │ responses    │ for later    │ Fake: Working  │
│ (never used) │ for tests    │ inspection   │ lightweight DB │
└──────────────┴──────────────┴──────────────┴────────────────┤
```

### 1. Testing with Mocks: Java (Mockito) vs. Python (unittest.mock)

#### Java (JUnit 5 + Mockito)
```java
import org.junit.jupiter.api.Test;
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

class OrderServiceTest {
    @Test
    void shouldChargeCardWhenOrderPlaced() {
        PaymentGateway mockGateway = mock(PaymentGateway.class);
        when(mockGateway.charge(100.0)).thenReturn(true); // Stub response

        OrderService service = new OrderService(mockGateway);
        boolean success = service.checkout(new Order(100.0));

        assertTrue(success);
        verify(mockGateway, times(1)).charge(100.0); // Verify interaction
    }
}
```

#### Python (pytest + unittest.mock)
```python
from unittest.mock import MagicMock
import pytest

def test_order_checkout():
    mock_gateway = MagicMock()
    mock_gateway.charge.return_value = True # Stub response

    service = OrderService(mock_gateway)
    success = service.checkout(Order(amount=100.0))

    assert success is True
    mock_gateway.charge.assert_called_once_with(100.0) # Verify interaction
```

---

## Important Terminology: Key Code Smells

| Code Smell | What It Looks Like | Refactoring Fix |
|---|---|---|
| **God Class** | Monolithic class with 3,000+ lines doing DB, business logic, UI | **Extract Class** / Split into cohesive domain services |
| **Feature Envy** | A method in Class A accesses data from Class B more than its own | **Move Method** into Class B |
| **Shotgun Surgery** | Making 1 business change requires touching 15 different files | Consolidate related logic into a single cohesive module |
| **Divergent Change** | Class A is repeatedly changed for 5 completely different business reasons | Apply SRP: split class into distinct single-purpose classes |
| **Primitive Obsession**| Passing raw strings (`String zipCode`, `String email`, `double amount`) everywhere | **Introduce Value Object** (`ZipCode`, `Email`, `Money`) |
| **Long Parameter List**| Method taking 8+ parameters (`p1, p2, p3, p4...`) | **Introduce Parameter Object** / Builder pattern |

## Example Refactoring: Replace Conditional with Polymorphism

### Before (Code Smell: Massive Switch / OCP Violation)
```python
class Bird:
    def get_speed(self, bird_type: str) -> float:
        if bird_type == "EUROPEAN": return 20.0
        elif bird_type == "AFRICAN": return 30.0
        elif bird_type == "NORWEGIAN_BLUE": return 0.0
        raise ValueError("Unknown bird")
```

### After (Clean Polymorphic Refactoring)
```python
from abc import ABC, abstractmethod

class Bird(ABC):
    @abstractmethod
    def get_speed(self) -> float: pass

class EuropeanSwallow(Bird):
    def get_speed(self) -> float: return 20.0

class AfricanSwallow(Bird):
    def get_speed(self) -> float: return 30.0

class NorwegianBlue(Bird):
    def get_speed(self) -> float: return 0.0
```

## Real-World Example

- **Value Objects in Fintech**: A banking system replacing `double balance` with a `Money(amount, currency)` Value Object. This prevents catastrophic currency mismatch bugs (e.g., adding 100 USD to 100 INR without conversion).
- **In-Memory Fakes**: Integration tests using an in-memory `FakeUserRepository` instead of spinning up a heavyweight Docker PostgreSQL container, speeding up test suite execution from 10 minutes to 3 seconds.

## Important Properties

| Test Double | Has Real Logic? | Used For |
|---|---|---|
| **Dummy** | No | Parameter filling only (e.g. logger or unused argument) |
| **Stub** | No (hardcoded return) | State verification (supplying predictable inputs to caller) |
| **Spy** | Wraps real or stubbed object | Recording interactions (number of invocations, arguments) |
| **Mock** | Pre-programmed expectations | Behavior verification (asserting exact method calls occurred) |
| **Fake** | Yes (simplified shortcut) | Realistic simulation (e.g., in-memory SQLite / fake repository) |

### Common Misconceptions

* Refactoring does NOT mean rewriting the system from scratch; refactoring happens in small, verifiable, behavior-preserving increments while tests continuously pass.
* Over-mocking is an anti-pattern: if your test mocks every single class boundary, the test becomes brittle and tests implementation details rather than observable behavior.
* "If it ain't broke, don't touch it" is false economy in software: ignoring code smells guarantees catastrophic bugs and exponentially higher maintenance costs later.

### What Interviewers Usually Test

1. Distinguishing between a Mock, a Stub, and a Fake.
2. Identifying code smells from a code snippet (Feature Envy, God Class, Primitive Obsession).
3. Demonstrating "Replace Conditional with Polymorphism" refactoring.
4. Explaining the difference between Shotgun Surgery and Divergent Change.
5. Designing testable classes using Dependency Injection.

### Common Follow-up Questions

1. What is the difference between Divergent Change and Shotgun Surgery?
2. Why is Primitive Obsession considered dangerous in domain modeling?
3. What is the difference between state verification and behavior verification?
4. When should you prefer a Fake over a Mock?
5. What is the "Extract Class" refactoring recipe?

### Follow-up Answers

1. **Divergent Change** is 1 class modified for many different reasons (violates SRP). **Shotgun Surgery** is 1 single change forcing simultaneous modifications across many different classes (spread-out responsibility).
2. Primitive Obsession treats domain concepts as raw types (e.g. `String email`), which allows invalid states (`""` or `"not-an-email"`) to permeate throughout the system without centralized validation. Value Objects encapsulate invariants safely.
3. State verification asserts that the returned value or final object state matches expectations. Behavior verification asserts that specific collaborators were invoked with expected arguments (using mocks).
4. Fakes are preferred when testing complex multi-step business flows where multiple methods interact with a repository or cache; a Fake behaves like the real service without brittle mock stubbing on every call.
5. Create a new class, move the cohesive subset of fields and methods from the old class to the new class, establish an association between them, and update client callers to interact with the new abstraction.
