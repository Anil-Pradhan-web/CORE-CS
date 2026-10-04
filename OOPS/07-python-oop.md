# 07. Python-Focused OOP

## 1. Python is dynamic

Python classes are objects too. Methods, modules, and functions are objects, so they can be passed around or stored in collections.

**Hinglish:** Python dynamic language hai; type ko runtime par bhi check kiya ja sakta hai. Type hints likhne se IDE aur static checker ko help milti hai, but runtime enforcement automatic nahi hota.

## 2. Instance methods and `self`

```python
class Product:
    def __init__(self, name: str, price: float) -> None:
        if price < 0:
            raise ValueError("price cannot be negative")
        self.name = name
        self.price = price

    def discounted(self, percent: float) -> float:
        return self.price * (1 - percent)
```

`self` refers to the current object. Python passes it explicitly as the first parameter.

## 3. `__new__` versus `__init__`

- `__new__` creates the object.
- `__init__` initializes the already-created object.

Most classes only need `__init__`. Use `__new__` for specialized creation such as immutable types or a carefully controlled singleton.

**Hinglish:** Normal class ke liye `__init__` kaafi hai. `__new__` object creation control karta hai; use it carefully.

## 4. Properties

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
            raise ValueError("balance cannot be negative")
        self._balance = value
```

A property gives a field-like API while allowing validation or computation. It is a clean form of encapsulation, but it is not a security wall.

## 5. Duck typing and polymorphism

Duck typing says: an object can be used when it provides the required operations, even if it does not inherit from a named base class.

```python
class UserService:
    def notify(self, notifier) -> None:
        notifier.send("Welcome")
```

The service only needs `send`. `EmailNotifier`, `SmsNotifier`, or a test double can provide it.

**Hinglish:** Duck typing runtime par behaviour check karta hai. “Aankhein hain toh kaam chal jayega” — required methods hain to object fit hai.

## 6. ABC versus Protocol

### ABC: nominal contract

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def speak(self) -> str:
        raise NotImplementedError
```

Normally a class explicitly inherits from `Animal`. Python can also register a virtual subclass with `ABCMeta.register()` without using normal inheritance.

### Protocol: structural contract

```python
from typing import Protocol

class Speaker(Protocol):
    def speak(self) -> str:
        ...
```

Any class with a compatible `speak` method satisfies the protocol without inheriting from `Speaker`.

**Hinglish:** ABC normally explicit parent maangta hai, `register()` se virtual subclass bhi possible hai. Protocol ke liye compatible methods kaafi hain.

## 7. Dunder methods

Dunder methods are special methods used by Python syntax and the standard library.

| Method | Meaning |
|---|---|
| `__str__` | Human-readable text |
| `__repr__` | Developer/debug representation |
| `__len__` | Enables `len(value)` |
| `__iter__` | Enables iteration |
| `__eq__` | Enables `==` |
| `__add__` | Enables `value1 + value2` |
| `__enter__` / `__exit__` | Context manager protocol |

```python
class Money:
    def __init__(self, amount: int) -> None:
        self.amount = amount

    def __add__(self, other: "Money") -> "Money":
        return Money(self.amount + other.amount)

    def __repr__(self) -> str:
        return f"Money(amount={self.amount})"
```

`__str__` is for users; `__repr__` should make debugging easy and should ideally be unambiguous.

## 8. Multiple inheritance and MRO

```python
class A:
    def ping(self) -> str:
        return "A"

class B(A):
    def ping(self) -> str:
        return "B" + super().ping()

class C(A):
    def ping(self) -> str:
        return "C" + super().ping()

class D(B, C):
    def ping(self) -> str:
        return "D" + super().ping()
```

Python uses C3 linearization to calculate a consistent Method Resolution Order. `D().ping()` follows `D`, `B`, `C`, `A`, `object` for this hierarchy.

Use multiple inheritance only when the relationships are clear. Composition is often easier to understand.

**Hinglish:** Python multiple inheritance allow karta hai, but MRO samajhna zaroori hai. `super()` next class in MRO ko call karta hai, hamesha source-code parent ko blindly nahi.

## 9. `@dataclass`

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class UserId:
    value: int
```

`frozen=True` prevents normal field reassignment and makes the class hashable when the fields are themselves hashable. It is still shallowly immutable if a field points to a mutable object.

## 10. `__slots__`

```python
class Point:
    __slots__ = ("x", "y")
```

`__slots__` can reduce per-instance memory by removing a normal instance dictionary. It also prevents arbitrary new attributes, so use it only when that restriction is desired.

## 11. Type hints and protocols

```python
from collections.abc import Iterable

def total(values: Iterable[int]) -> int:
    return sum(values)
```

Good type hints document intent and help static tools. Runtime validation still needs explicit checks for external input.

## 12. Exceptions and context managers

```python
class OrderRepository:
    def __enter__(self) -> "OrderRepository":
        return self

    def __exit__(self, exc_type, exc_value, traceback) -> bool:
        return False
```

A `with` block guarantees `__exit__` is called. Returning `False` does not suppress the exception.

## 13. Python concurrency and the GIL

The Global Interpreter Lock affects how some Python implementations execute Python bytecode. It does not mean every Python program is single-threaded, and it does not remove the need for locks around shared application state. I/O-heavy work can benefit from `asyncio` or threads; some CPU-heavy work may need another approach depending on the runtime and workload.

**Hinglish:** GIL ko simple shortcut ki tarah explain mat karo. Shared data, blocking I/O, aur runtime/version ko consider karke design karo.

## 14. Common Python mistakes

- Calling an instance method without the correct `self` convention.
- Mutating fields used by a custom dictionary key’s hash or equality after insertion.
- Assuming type hints enforce types at runtime.
- Treating `_private` or `__private` as a security mechanism.
- Forgetting that a custom `__eq__` affects hashability.
- Creating deep inheritance trees for simple reuse.
- Returning a mutable internal collection directly and allowing callers to change it.

## 15. Small clean Python example

```python
from dataclasses import dataclass
from typing import Protocol

class PaymentGateway(Protocol):
    def charge(self, amount: float) -> str:
        ...

@dataclass(frozen=True)
class Order:
    order_id: str
    amount: float

    def __post_init__(self) -> None:
        if self.amount <= 0:
            raise ValueError("amount must be positive")

class Checkout:
    def __init__(self, gateway: PaymentGateway) -> None:
        self.gateway = gateway

    def pay(self, order: Order) -> str:
        return self.gateway.charge(order.amount)
```

## Quick interview Q&A

### 1. What is duck typing?

Duck typing uses an object based on the methods it provides, rather than requiring a specific named parent class.

### 2. What is the difference between `__new__` and `__init__`?

`__new__` creates the instance; `__init__` initializes it after creation.

### 3. What is a property?

A property controls attribute access while allowing validation or computed values.

### 4. What is Python name mangling?

A double-leading-underscore attribute is internally renamed with the class name. It reduces accidental collisions but is not secure privacy.

### 5. What is MRO?

Method Resolution Order is the order Python searches classes for a method. Multiple inheritance uses C3 linearization.

### 6. What does `super()` do in Python?

It follows the next class in the current instance’s MRO, enabling cooperative multiple inheritance.

### 7. What is the difference between `__str__` and `__repr__`?

`__str__` is intended for readable display; `__repr__` is intended for an unambiguous debugging representation.

### 8. What is `typing.Protocol`?

It describes a structural contract: compatible methods are enough, and explicit inheritance is not required.

### 9. What is `@dataclass(frozen=True)`?

It generates common data-class methods and prevents normal field reassignment. Nested mutable objects need separate protection.

### 10. What does `__slots__` do?

It removes the normal per-instance attribute dictionary and restricts new attributes, often reducing memory use.

### 11. Does the GIL make all Python code single-threaded?

No. It affects certain execution paths and implementations. Do not use it as a complete concurrency strategy.

### 12. Why can mutating a dictionary key be a problem?

Changing a field used by the key’s hash or equality can make it impossible to find with the original key. Mutability alone is safe when the key uses only stable identity.

### 13. What is the difference between an ABC and a Protocol?

An ABC normally uses explicit nominal inheritance, although `ABCMeta.register()` can register a virtual subclass. A Protocol uses structural compatibility based on required methods.

### 14. Is `private` enforced in Python?

Not by a strict security boundary. Conventions, name mangling, and properties communicate intent.

**Hinglish recap:** Python mein simplicity, protocols, aur clear data models ko prefer karo; magical code se pehle behaviour samjho.
