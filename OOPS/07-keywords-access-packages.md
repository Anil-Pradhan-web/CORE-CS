# Keywords, Access Modifiers, and Package/Module Scopes

## What is it?

Access modifiers and structural keywords define visibility, mutability, lifetime, and namespace boundaries for classes, methods, and variables. They enforce encapsulation at compile-time (in Java) or through runtime conventions and name mangling (in Python).

## Simple Explanation

- **Access Modifiers**: A house has areas: the front garden is **public** (anyone can see/walk), the living room is **protected/package** (friends and family), and your personal diary is **private** (only you).
- **`static` / Class level**: The street address belongs to the building itself, not to each individual tenant inside.
- **`final` / Const**: A birth certificate stamp: once written, it cannot be changed or overridden.

## Hinglish Explanation

Access modifiers batate hain ki class ke variables aur methods ko kaun-kaun access kar sakta hai. Java mein compiler strictly enforce karta hai (`private`, `protected`, `public`, `default`). Python mein "we are all consenting adults here" philosophy hai: `_single_underscore` convention se protected signal hota hai, aur `__double_underscore` se Python name mangling karta hai. Java ka `static` class-level memory share karta hai, jabki Python mein class attributes aur `@classmethod`/`@staticmethod` use hote hain.

## Why Do We Need It?

Without visibility controls and namespaces:
1. Callers tamper with private internal state, breaking invariants.
2. Global namespaces collide when multiple libraries define a class with the same name.
3. Subclasses accidentally override methods they were never intended to touch (prevented by `final` in Java or `@final` in Python).

## How It Works

### Java Visibility & Keywords

```java
package com.app.banking;

public class BankAccount {
    public String accountNumber;            // Accessible everywhere
    protected double baseFee;              // Accessible in same package & subclasses
    int transactionCount;                   // Default/Package-private: same package only
    private double balance;                 // Accessible ONLY within BankAccount

    public static final double MIN_BALANCE = 500.0; // Class-level constant

    public final void lockAccount() {       // Cannot be overridden by subclasses
        this.balance = 0;                   // "this" refers to current instance
    }
}
```

### Python Visibility & Conventions

```python
from typing import final

class BankAccount:
    MIN_BALANCE: float = 500.0   # Class attribute (shared like static)

    def __init__(self, account_number: str, balance: float):
        self.account_number = account_number   # Public attribute
        self._base_fee = 10.0                  # Protected by convention (internal use)
        self.__balance = balance               # Private via Name Mangling: _BankAccount__balance

    @classmethod
    def get_min_balance(cls) -> float:         # Operates on class, not instance
        return cls.MIN_BALANCE

    @staticmethod
    def is_valid_account(acc_no: str) -> bool: # Utility method, no self/cls
        return len(acc_no) == 10

    @final
    def lock_account(self):                    # Signals to type checker: do not override
        self.__balance = 0
```

## Important Terminology

| Term | Java | Python | Meaning |
|---|---|---|---|
| **Public** | `public` | `name` | Accessible from anywhere |
| **Protected** | `protected` | `_name` | Accessible in package & subclasses (Java); convention (Python) |
| **Package / Default** | *(no keyword)* | Module level | Visible only within the same package/module |
| **Private** | `private` | `__name` | Restricted to class; Python uses name mangling (`_Class__name`) |
| **Class Member** | `static` | Class attribute / `@classmethod` | Bound to class, not individual instance |
| **Non-overridable** | `final` | `@typing.final` | Prevents method overriding or class inheritance |
| **Instance Pointer** | `this` | `self` | Reference to the current instance invoking the method |
| **Super Reference** | `super` | `super()` | Accesses parent class methods/constructors |

## Example

Python's Name Mangling in action:
```python
acc = BankAccount("ACC123", 1000.0)
# acc.__balance  -> AttributeError: 'BankAccount' object has no attribute '__balance'
print(acc._BankAccount__balance)  # 1000.0 (mangled name can still be reached!)
```
In Java, trying to access `acc.balance` directly from another class triggers a compilation error: `'balance' has private access in 'BankAccount'`.

## Real-World Example

- **Frameworks (Spring, Django, FastAPI)** use package/module structures to organize layers: `controllers`, `services`, `repositories`.
- **Database Connection Pools**: The pool size is declared `static final` in Java or class-level in Python so thousands of request threads share a single pool configuration without duplicating it per object.

## Important Properties

| Feature | Java | Python |
|---|---|---|
| **Enforcement** | Strict compile-time checks | Runtime convention + type checker hints |
| **Default Access** | Package-private (same package) | Public by default |
| **Name Mangling** | Not present (true compiler visibility restriction) | `__var` becomes `_ClassName__var` |
| **Constant Declaration**| `static final` (immutable reference) | Capital letters convention (`MAX_RETRIES`) or `Final[int]` |
| **Namespace Boundary** | `package` mapped to directory folders | `__init__.py` modules and packages |

### Common Misconceptions

* In Python, `__var` is not true cryptographic security or memory-level protection; it prevents accidental subclass collisions via name mangling.
* In Java, `protected` is broader than many think: it allows access to subclasses **and** any class within the exact same package.
* A `final` variable in Java means the reference cannot point to another object; the internal state of the referenced object can still be mutated unless it is immutable.

### What Interviewers Usually Test

1. Difference between `default` (package-private) and `protected` in Java.
2. How Python handles private attributes (`_` vs `__` and name mangling).
3. Difference between `@staticmethod` and `@classmethod` in Python.
4. Java `final` keyword applied to variables, methods, and classes.
5. Why `static` methods cannot access `this` or instance fields directly.

### Common Follow-up Questions

1. Can you override a `static` method in Java or Python?
2. What is the difference between `@classmethod` and `@staticmethod` in Python?
3. What is package-private in Java, and when should you prefer it over `public`?
4. What happens when a class is declared `final` in Java?
5. How does Python's `super()` work compared to Java's `super` keyword?

### Follow-up Answers

1. No. In Java, static methods are resolved at compile-time (method hiding, not dynamic dispatch overriding). In Python, defining a static method in a subclass shadows the parent's definition, but dynamic polymorphic dispatch does not apply in the same way.
2. `@classmethod` takes `cls` as its first parameter and can inspect/modify class state or act as an alternative constructor. `@staticmethod` takes neither `self` nor `cls`; it is a plain function scoped inside the class namespace.
3. Package-private (default) is accessible only by classes in the same package. It prevents leaking internal implementation classes across public API boundaries.
4. A `final` class cannot be extended/subclassed (e.g., `java.lang.String`). This guarantees immutability, prevents security exploits, and allows compiler optimizations.
5. In Java, `super` directly calls the immediate superclass. In Python, `super()` follows the Method Resolution Order (MRO) via C3 Linearization, which is critical for correctly cooperative multiple inheritance.
