# 03. SOLID Principles and Design Patterns

SOLID is a set of five design principles. They help code stay understandable and easier to change.

## Hinglish introduction

SOLID principles code ko flexible aur maintainable banate hain. Inko blindly follow mat karo; simple code ko complex banana bhi ek problem ho sakti hai.

## 1. S — Single Responsibility Principle

A class should have one main reason to change, or one clear business responsibility.

A `User` should not also create PDF files, send emails, and write SQL queries.

**Good split:**

```text
User             -> user data and user rules
UserRepository   -> storage operations
MailService      -> email operations
```

**Hinglish:** Ek class ka ek main kaam hona chahiye. “Ek method” hona zaroori nahi; ek cohesive responsibility hona zaroori hai.

## 2. O — Open/Closed Principle

Code should be open for extension and closed for repeated modification when a new variant is added.

Instead of this:

```text
if paymentType == "CARD" ...
else if paymentType == "UPI" ...
else if paymentType == "WALLET" ...
```

Use separate implementations behind one contract:

```text
PaymentGateway
  ├── CardPayment
  ├── UpiPayment
  └── WalletPayment
```

**Hinglish:** Naya payment type add karne ke liye purane service mein naye branches na likhne paden. Naya implementation add karo.

OCP does not mean “never edit code”. Bug fixes and real requirement changes still need edits.

## 3. L — Liskov Substitution Principle

A child type should be usable anywhere its parent type is expected without breaking the parent's contract.

Example: A class that promises `send()` but throws `UnsupportedOperationException` is not a proper substitute.

**Hinglish:** Child class parent ki jagah safely use honi chahiye. Sirf naam badalna substitution nahi hota; behaviour bhi match karna chahiye.

## 4. I — Interface Segregation Principle

Clients should not be forced to implement methods they do not need.

Instead of one large `Worker` interface:

```text
Worker
  ├── work()
  ├── eat()
  └── sleep()
```

Use focused contracts:

```text
Workable
  └── work()

Restable
  └── rest()
```

**Hinglish:** Bada interface sabko force karta hai. Chhote, purpose-based interfaces better hote hain.

## 5. D — Dependency Inversion Principle

High-level policy should not depend on low-level details. Both should depend on an abstraction owned by the policy.

```text
OrderService -> PaymentGateway interface -> StripePayment
```

**Hinglish:** Business logic ko database ya payment provider ki concrete class ka pata nahi hona chahiye. Usko sirf contract chahiye.

## DIP, IoC, and DI

These terms are related but different:

- **DIP:** Design principle—depend on abstractions.
- **IoC:** Broader pattern—control is inverted, often by a framework or container.
- **DI:** Technique—pass a dependency from outside through a constructor or setter.

```python
class MessageSender:
    def send(self, message: str) -> None:
        raise NotImplementedError

class EmailSender(MessageSender):
    def send(self, message: str) -> None:
        print(f"Email: {message}")

class NotificationService:
    def __init__(self, sender: MessageSender) -> None:
        self.sender = sender

    def notify(self, message: str) -> None:
        self.sender.send(message)
```

## 6. Common design patterns

A design pattern is a reusable solution to a recurring design problem. It is not a magic formula; use it only when the problem really matches.

### Strategy

Changes the algorithm or behaviour behind one interface.

```text
CompressionStrategy
  ├── ZipStrategy
  └── GzipStrategy
```

Use when: one operation has interchangeable behaviours.

### Factory

Creates the correct object without exposing construction details to the client.

Use when: client should not know which concrete class to instantiate.

### Observer

Notifies many listeners when something changes.

```text
Order -> EmailNotifier
      -> SmsNotifier
      -> AuditLogger
```

Use when: one event affects many optional listeners.

### Decorator

Adds behaviour by wrapping an existing object.

```text
DataSource
  ├── FileDataSource
  ├── EncryptingDataSource
  └── CompressingDataSource
```

Use when: combinations of optional behaviour are needed.

### Adapter

Converts one interface into another expected interface.

Use when: two incompatible components must work together.

### Singleton

Restricts a class to one instance.

Use sparingly. A global instance can make tests and dependencies harder. Prefer a controlled factory or a simple object instance unless one global instance is truly required.

### Template Method

A base class defines the algorithm steps; subclasses provide selected steps.

Use when: the sequence is stable but some steps vary.

## 7. Pattern selection cheat sheet

| Problem | Possible direction |
|---|---|
| Swap payment algorithm | Strategy |
| Hide object creation | Factory |
| Notify many listeners | Observer |
| Add optional layers | Decorator |
| Connect incompatible APIs | Adapter |
| Share a stable algorithm skeleton | Template Method |

## 8. Code smells

Watch for these signs:

- A class that handles UI, validation, database, email, and business rules.
- A long `if-else` that checks every possible type.
- A subclass that throws because a parent method does not apply.
- A large interface with methods most clients never call.
- A service that creates its own database, clock, network client, and random generator.
- Public mutable fields everywhere.

**Hinglish:** Smell ka matlab immediate bug nahi, but design future mein problem de sakta hai.

## 9. When not to use SOLID or patterns

Do not over-engineer a ten-line script. A simple function and a dictionary may be clearer than five interfaces and three factories. Apply a pattern when the change, reuse, or testing benefit is real.

## 10. A small Strategy example

Before:

```python
def calculate_price(kind: str, amount: float) -> float:
    if kind == "student":
        return amount * 0.8
    if kind == "staff":
        return amount * 0.9
    return amount
```

After:

```python
from typing import Protocol

class Discount(Protocol):
    def apply(self, amount: float) -> float:
        ...

class NoDiscount:
    def apply(self, amount: float) -> float:
        return amount

class StudentDiscount:
    def apply(self, amount: float) -> float:
        return amount * 0.8

class StaffDiscount:
    def apply(self, amount: float) -> float:
        return amount * 0.9

def calculate_price(discount: Discount, amount: float) -> float:
    return discount.apply(amount)
```

`NoDiscount` keeps the old no-discount result. This is an intentional API change, not a fully behaviour-preserving refactoring. Keep the old public wrapper if existing callers must keep calling `calculate_price(kind, amount)`. The new design is useful when discount strategies need independent testing or more variants are expected.

## Quick interview Q&A

### 1. What is SOLID?

SOLID is five principles for creating maintainable, flexible, and understandable object-oriented code.

### 2. What is SRP?

A class should have one main reason to change. It can have many methods, but they should serve one cohesive responsibility.

### 3. What is OCP?

New variants should be added through abstractions and new implementations without repeatedly changing the main workflow.

### 4. What is LSP?

A subtype must be safely replaceable wherever its base type is expected.

### 5. What is ISP?

Clients should depend on small interfaces that contain only the operations they need.

### 6. What is DIP?

High-level code should depend on abstractions, not directly on low-level concrete details.

### 7. What is the difference between DIP and DI?

DIP is the principle; DI is the technique of supplying a dependency from outside.

### 8. What is the Strategy pattern?

Strategy changes an operation's algorithm through interchangeable implementations.

### 9. What is the Factory pattern?

Factory centralizes object creation so clients do not need to know every construction detail.

### 10. What is the Observer pattern?

Observer defines a one-to-many notification relationship between an event source and listeners.

### 11. What is the Decorator pattern?

Decorator wraps an object to add behaviour without changing its original class.

### 12. What is the Adapter pattern?

Adapter converts one interface into another interface expected by a client.

### 13. When is Singleton risky?

When global state makes tests, configuration, and multiple environments difficult to control.

### 14. Does SOLID mean every class needs an interface?

No. Use abstractions where they provide a real boundary or substitution point.

### 15. What is a code smell?

A sign that a design may be difficult to understand, change, or test. It is a warning, not proof that a bug exists.

**Hinglish recap:** SOLID ko design guide samjho, mandatory recipe nahi. Pattern tab use karo jab real problem ho.
