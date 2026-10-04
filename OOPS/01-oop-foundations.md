# 01. OOP Foundations

## 1. Class and object

- **Class** is a blueprint or definition.
- **Object** is a real instance created from that definition.

Example: `Car` is a class. A red car with number `MH12AB1234` is an object.

### Hinglish

Class ek blueprint hai. Object us blueprint se bana hua real instance hai. Ek class se many objects ban sakte hain, lekin har object ka apna state alag hota hai.

```python
class Student:
    def __init__(self, name: str, roll: int) -> None:
        self.name = name
        self.roll = roll

student1 = Student("Aman", 1)
student2 = Student("Riya", 2)
```

Here, `Student` is the class. `student1` and `student2` are objects with different state.

## 2. State and behaviour

- **State** tells us what an object currently has.
- **Behaviour** tells us what an object can do.

```java
class Account {
    private double balance;

    Account(double openingBalance) {
        if (openingBalance < 0) {
            throw new IllegalArgumentException("Balance cannot be negative");
        }
        this.balance = openingBalance;
    }

    void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
        balance += amount;
    }

    double getBalance() {
        return balance;
    }
}
```

`balance` is state. `deposit()` is behaviour.

**Hinglish:** State object ka current data hai; behaviour object ka action hai. Data ko blindly change nahi karna chahiye—valid operations ke through change karna best hai.

## 3. The four pillars of OOP

### 3.1 Encapsulation

Encapsulation means keeping data and the methods that protect that data together. Callers should use safe methods instead of directly changing important fields.

Example: A bank account should not allow `balance = -100`. `withdraw()` should check the balance and reject an invalid request.

**Hinglish:** Encapsulation data ko protect karta hai. Isse har caller ko data directly change nahi karta; allowed method ko call karta hai.

```java
class BankAccount {
    private double balance;

    BankAccount(double openingBalance) {
        if (openingBalance < 0) {
            throw new IllegalArgumentException("Invalid opening balance");
        }
        balance = openingBalance;
    }

    void withdraw(double amount) {
        if (amount <= 0 || amount > balance) {
            throw new IllegalArgumentException("Invalid withdrawal");
        }
        balance -= amount;
    }
}
```

**Important:** Private fields plus public setters do not automatically create good encapsulation. The setters must enforce useful rules.

### 3.2 Abstraction

Abstraction means showing the essential operation and hiding unnecessary implementation details.

A caller only needs to know:

```text
paymentGateway.charge(order)
```

The caller does not need to know every HTTP request, retry rule, or provider detail.

**Hinglish:** Abstraction complex implementation chhupata hai aur simple contract dikhata hai. Client ko sirf yeh pata chalna chahiye ki service kya kar sakti hai.

### 3.3 Inheritance

Inheritance creates a new class from an existing class. It represents a real `is-a` relationship.

Example: `ElectricCar is-a Car`.

**Hinglish:** Inheritance parent class ke behaviour ko child class mein extend karta hai. Isko sirf code copy karne ke liye use mat karo; `is-a` relationship honi chahiye.

### 3.4 Polymorphism

Polymorphism means using one common contract while different objects perform different implementations.

```text
Notifier
  ├── EmailNotifier
  ├── SmsNotifier
  └── WhatsAppNotifier
```

A service can call `notifier.send(message)` without checking the concrete type.

**Hinglish:** Polymorphism ka matlab hai ki ek hi interface ke through alag-alag objects ko call kiya ja sake, aur har object apna behaviour de.

## 4. Encapsulation versus abstraction

| Question | Encapsulation | Abstraction |
|---|---|---|
| Main idea | Protect data and rules | Hide complexity |
| Focus | Internal state | External contract |
| Example | Private `balance` plus `withdraw()` | `PaymentGateway.charge()` |
| Hinglish | Data ki protection | Details ki chhupai |

Both are useful, but they solve different problems.

## 5. Inheritance versus composition

### Inheritance

```text
Vehicle
  └── Car
```

Use it when the child really is a specialized parent.

### Composition

```text
Car
  ├── Engine
  ├── Wheel
  └── Battery
```

Use it when an object has a part or a replaceable behaviour.

**Hinglish:** Inheritance parent-child relationship ke liye, composition has-a relationship aur flexible reuse ke liye best hai. Aapke design mein aksar composition safer aur simpler hota hai.

**Rule:** Inheritance ko sirf tab use karo jab child parent ki har important contract ko safely follow kar sakta ho.

## 6. Overloading versus overriding

### Overloading

Same method name with a different parameter list. Java usually resolves it at compile time.

```java
class Calculator {
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }
}
```

### Overriding

A child class provides a new implementation of an inherited method. Java usually resolves it at runtime using the actual object.

```java
class Parent {
    String speak() {
        return "parent";
    }
}

class Child extends Parent {
    @Override
    String speak() {
        return "child";
    }
}
```

**Hinglish:** Overloading mein signature change hoti hai. Overriding mein child parent ke method ka implementation change karta hai.

## 7. Interface versus abstract class

| Feature | Interface | Abstract class |
|---|---|---|
| Main purpose | Capability contract | Shared base with common code/state |
| Java state | No normal instance fields | Can have instance fields |
| Multiple implementation | A class can implement many interfaces | A class extends one class |
| Best use | “Can do this” | “Is a kind of this” and shared code |
| Test replacement | Very easy | Also possible |

Example:

```text
PaymentGateway (interface)
  ├── CardPayment
  ├── UpiPayment
  └── TestPayment
```

A service should depend on `PaymentGateway`, not directly on `CardPayment`.

**Hinglish:** Interface bolta hai “yeh kaam kar sakta hai”. Abstract class bolta hai “yeh ek common base type hai aur isme shared state/code hai”.

## 8. Constructor and initialization

A constructor creates or initializes an object. A valid object should start in a valid state.

```python
class Rectangle:
    def __init__(self, width: float, height: float) -> None:
        if width <= 0 or height <= 0:
            raise ValueError("Dimensions must be positive")
        self.width = width
        self.height = height
```

### Default constructor

- Java creates a no-argument constructor only when no constructor is written.
- If a class has a parameterized constructor, add a no-argument constructor yourself if callers need it.
- Python uses the inherited initializer if `__init__` is not defined.

## 9. Invariant

An **invariant** is a rule that must remain true while an object is valid.

Examples:

- Bank balance cannot be negative.
- An order cannot be shipped before payment.
- A rectangle cannot have a negative width.

Encapsulation helps protect invariants because callers cannot bypass the validating methods.

## 10. Real-world design example

```python
from typing import Protocol

class PaymentGateway(Protocol):
    def charge(self, amount: float) -> str:
        ...

class FakePayment:
    def charge(self, amount: float) -> str:
        return "fake-payment-id"

class Checkout:
    def __init__(self, gateway: PaymentGateway) -> None:
        self.gateway = gateway

    def pay(self, amount: float) -> str:
        if amount <= 0:
            raise ValueError("Amount must be positive")
        return self.gateway.charge(amount)
```

The checkout service does not know whether the payment is real or fake. That makes testing easy.

## Quick interview Q&A

### 1. What is OOP?

OOP organizes software around objects that combine state and behaviour. It helps us protect rules and change one part of a system without rewriting everything.

### 2. What is the difference between a class and an object?

A class is a definition or blueprint. An object is a runtime instance with actual state.

### 3. What is encapsulation?

Encapsulation keeps state and the methods that protect it together. It prevents callers from creating invalid state.

### 4. What is abstraction?

Abstraction exposes an essential contract while hiding implementation details.

### 5. What is inheritance?

Inheritance derives a new type from an existing type. It should model a meaningful `is-a` relationship.

### 6. What is polymorphism?

Polymorphism lets different objects respond through one common contract, each with its own implementation.

### 7. What is overloading?

Overloading uses the same method name with different parameter signatures. In Java, it is normally compile-time polymorphism.

### 8. What is overriding?

Overriding replaces an inherited method implementation in a child class. It is normally runtime polymorphism.

### 9. Interface or abstract class?

Use an interface for a capability contract. Use an abstract class when related subclasses need shared state or common implementation.

### 10. Why prefer composition?

Composition keeps parts replaceable and avoids a fragile deep inheritance tree. It is often easier to test.

### 11. What is an invariant?

It is a condition that must always remain true for an object to be valid, such as a non-negative account balance.

### 12. Why is OOP not only inheritance?

OOP also includes encapsulation, abstraction, objects, composition, contracts, and good responsibility boundaries.

**Hinglish recap:** OOP ka core hai state + behaviour + rules. Inheritance kaam ka tool hai, solution nahi.
