# 02. Object Relationships and Language Basics

## 1. Association

Association means two classes work together, but neither object owns the other object's lifecycle.

Example: A `Doctor` works with a `Patient`. Deleting the doctor record does not automatically delete the patient.

**Hinglish:** Association do independent objects ka connection hai. Ek ke delete hone se doosra automatically delete nahi hota.

## 2. Aggregation

Aggregation is a weak `has-a` relationship. The child can exist without the parent.

Example: A `Department` contains `Professor` objects. If the department is removed, a professor can move to another department.

UML uses a hollow diamond at the whole side.

## 3. Composition

Composition is a strong `has-a` relationship. The parent owns the child and usually creates and destroys it.

Example: A `House` owns its `Room` objects.

UML uses a filled diamond at the whole side.

**Hinglish:** Aggregation mein part independently reh sakta hai. Composition mein part ka lifecycle parent se linked hota hai.

## 4. Dependency

A dependency exists when a class uses another class, method, or value in its implementation.

Example: `OrderService` uses a `PaymentGateway`. The service needs the gateway, but it does not have to own it.

**Hinglish:** Dependency ka matlab hai “mujhe iska use chahiye”, ownership ka nahi.

## Easy relationship table

| Relationship | Meaning | Lifecycle |
|---|---|---|
| Association | Uses or knows another object | Independent |
| Aggregation | Weak whole-part | Child may survive |
| Composition | Strong whole-part | Parent manages child |
| Dependency | Uses a required collaborator | No ownership |

## 5. Why composition is often safer

Inheritance creates a strong connection to the parent implementation. If the parent changes, children may also need changes. Composition keeps the parts behind smaller contracts.

Example:

```text
Car has-a Engine
Car has-a Wheel
```

A car can be tested with a fake engine without inheriting from a real engine class.

**Hinglish:** Inheritance mein parent ka implementation child ko affect karta hai. Composition mein parts ko independently replace karna easier hota hai.

## 6. Java access modifiers

| Modifier | Meaning |
|---|---|
| `private` | Only the same class |
| default/no keyword | Same package |
| `protected` | Same package or subclass rules |
| `public` | Accessible from anywhere allowed by the project/module boundary |

A common design is to keep fields private and expose small public methods.

## 7. Python visibility

Python uses naming conventions more than strict access rules.

- `_value` means “use this internally, please”.
- `__value` triggers name mangling, such as `_ClassName__value`.
- A property can validate reads and writes.

```python
class Account:
    def __init__(self, balance: float) -> None:
        self._balance = balance

    @property
    def balance(self) -> float:
        return self._balance

    @balance.setter
    def balance(self, value: float) -> None:
        if value < 0:
            raise ValueError("Balance cannot be negative")
        self._balance = value
```

Name mangling is not a security feature. It mainly avoids accidental name collisions in inheritance.

**Hinglish:** Python mein `_` internal hone ka signal hai aur `__` name mangling karta hai, but Python security boundary nahi banata.

## 8. `this`, `self`, `cls`, and `super`

- Java `this` means the current object.
- Python `self` is the explicit first parameter of an instance method.
- Python `cls` is the first parameter of a class method.
- `super()` follows the parent or the next class in the MRO.

```python
class User:
    def __init__(self, name: str) -> None:
        self.name = name

    def greet(self) -> str:
        return f"Hello, {self.name}"
```

## 9. Static, class, and instance methods

### Instance method

Uses one object's state.

```python
class User:
    def display(self) -> None:
        print(self.name)
```

### Class method

Receives the class and can create alternatives or access class-level behaviour.

```python
class User:
    @classmethod
    def from_dict(cls, data: dict[str, str]) -> "User":
        return cls(data["name"])
```

### Static method

Does not need the object or class. It is often a helper logically related to the class.

```python
class User:
    @staticmethod
    def is_valid_name(name: str) -> bool:
        return bool(name.strip())
```

**Hinglish:** `self` instance ke liye, `cls` class ke liye, aur `staticmethod` helper function jaisa hai.

## 10. `final`, constants, and immutability

In Java, `final` can mean a variable cannot be reassigned, a class cannot be extended, or a method cannot be overridden. It does not automatically make every referenced object deeply immutable.

In Python, `typing.final` is mainly a type-checker signal. It is not a strong runtime security lock.

## 11. Packages and modules

A module or package groups related code. Good boundaries make navigation and testing easier.

- **High-level module:** business rules; should not know every database or UI detail.
- **Low-level module:** database, network, file system, or framework details.
- **Dependency direction:** high-level code should normally depend on a contract owned by the business side.

**Hinglish:** Packages code ko logical groups mein divide karte hain. Boundary sahi ho to test karna aur change karna easy hota hai.

## 12. Small Java relationship example

```java
class Engine {
    void start() {
        System.out.println("Engine started");
    }
}

class Car {
    private final Engine engine;

    Car(Engine engine) {
        this.engine = engine;
    }

    void start() {
        engine.start();
    }
}
```

`Car` has-a `Engine`. The `Engine` is passed from outside, so it can be replaced in a test.

## 13. Design questions to ask

Before choosing a relationship, ask:

1. Is this really an `is-a` relationship, or only code reuse?
2. Who owns the child object's lifecycle?
3. Can the dependency be replaced with a fake?
4. Does the public contract contain only what the client needs?
5. Will the relationship remain understandable when requirements change?

## Quick interview Q&A

### 1. What is association?

Association is a general relationship where objects collaborate but do not necessarily own one another.

### 2. What is aggregation?

Aggregation is a weak whole-part relationship. The part can exist independently of the whole.

### 3. What is composition?

Composition is a strong whole-part relationship where the parent manages the child lifecycle.

### 4. What is dependency?

A dependency is a use relationship where one component needs another component to work.

### 5. What does `has-a` mean?

It means an object contains or uses another object as a part or collaborator.

### 6. Why prefer composition over inheritance?

Composition makes parts replaceable, reduces deep coupling, and is often easier to test.

### 7. What is `this` in Java?

`this` refers to the current instance of the class.

### 8. What is `self` in Python?

`self` is the explicit first parameter of an instance method and refers to the current object.

### 9. What is the difference between a class method and a static method?

A class method conventionally receives `cls`; a static method does not receive an implicit instance or class reference, so it behaves like a helper function. Any parameters are ordinary explicit parameters.

### 10. Is a Python `__private` variable truly private?

No. It is name-mangled, but a determined caller can still access the generated name. It is a design warning, not security.

### 11. What is Dependency Inversion in one line?

High-level business code should depend on its own abstraction instead of a low-level concrete database or network class.

### 12. What is loose coupling?

Loose coupling means one component can change without forcing many unrelated components to change.

**Hinglish recap:** Relationship tabhi clear hota hai jab ownership, lifecycle, aur replacement ka answer pata ho.
