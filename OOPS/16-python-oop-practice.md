# Python OOP in Practice: Dunders, MRO, and Protocols

## What is it?

Python is a dynamic, object-oriented language where "everything is an object"—including functions, modules, and classes themselves. Python OOP blends traditional class hierarchies with duck typing, metaprogramming, dunder (magic) methods, and structural subtyping (`typing.Protocol`).

## Simple Explanation

- In traditional languages (like Java), an object is defined strictly by its declared ancestry (`implements Interface`).
- In Python, if it walks like a duck and quacks like a duck, it is a duck (**Duck Typing**). By implementing standard double-underscore methods ("dunders" like `__len__`, `__iter__`, `__getitem__`), your custom class effortlessly plugs into Python's native operators (`len()`, `for x in obj`, `obj[key]`).

## Hinglish Explanation

Python mein OOP bahut dynamic aur expressive hai:
- `__init__` constructor nahi balki initializer hai, jabki actual memory allocation `__new__` karta hai.
- `self` current instance ko point karta hai, aur `cls` class object ko point karta hai.
- Multiple inheritance allowed hai aur Diamond Problem ko solve karne ke liye Python **C3 Linearization (MRO - Method Resolution Order)** use karta hai.
- `@property` se aap bina existing callers ka code break kiye variable access ko method validation se wrap kar sakte ho.
- `abc.ABC` explicit contracts enforce karta hai, jabki modern Python `typing.Protocol` se duck typing ko static type safety deta hai.

## How It Works

### 1. `__new__` vs. `__init__` (True Creation vs. Initialization)

```python
class Singleton:
    _instance = None

    def __new__(cls, *args, **kwargs):
        # __new__ is responsible for creating and returning the instance
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self, value: int):
        # __init__ initializes the already-created instance
        self.value = value
```

### 2. Method Resolution Order (MRO) & Cooperative `super()`

Python resolves multiple inheritance using the C3 Linearization algorithm:

```python
class A:
    def ping(self): print("Ping A")

class B(A):
    def ping(self):
        print("Ping B")
        super().ping()

class C(A):
    def ping(self):
        print("Ping C")
        super().ping()

class D(B, C):
    def ping(self):
        print("Ping D")
        super().ping()

# MRO order: D -> B -> C -> A -> object
d = D()
d.ping()
# Output:
# Ping D
# Ping B
# Ping C  <-- super() in B called C, NOT directly A! Cooperative MRO in action.
# Ping A
print(D.mro()) # [<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>]
```

### 3. `@property` for Pythonic Encapsulation

```python
class Temperature:
    def __init__(self, celsius: float):
        self._celsius = celsius

    @property
    def celsius(self) -> float:
        return self._celsius

    @celsius.setter
    def celsius(self, value: float):
        if value < -273.15:
            raise ValueError("Temperature below absolute zero is impossible!")
        self._celsius = value

    @property
    def fahrenheit(self) -> float:
        return (self._celsius * 9/5) + 32
```

### 4. Protocols (Structural Subtyping) vs. ABCs

```python
from typing import Protocol, runtime_checkable
from abc import ABC, abstractmethod

# 1. Nominal subtyping via ABC (Requires explicit inheritance)
class Animal(ABC):
    @abstractmethod
    def speak(self) -> str: pass

# 2. Structural subtyping via Protocol (Duck typing with type checks)
@runtime_checkable
class Renderable(Protocol):
    def render(self) -> str: ...

class WebPage:
    def render(self) -> str: return "<html>Home</html>"

# WebPage matches Renderable without inheriting from it!
def display(item: Renderable):
    print(item.render())

display(WebPage()) # Passes mypy & runtime checks!
```

## Important Terminology

| Term | Meaning | Typical Use Case |
|---|---|---|
| **Dunder Methods** | Double-underscore special methods (`__name__`) | Operator overloading, lifecycle hooks, protocols |
| **MRO** | Method Resolution Order (C3 algorithm) | Deterministic lookup order in multiple inheritance |
| **`__slots__`** | Restricts allowed attributes on instances | Saves huge memory by skipping default `__dict__` |
| **Duck Typing** | "If it walks like a duck, it's a duck" | Dynamic polymorphism without strict inheritance |
| **Protocol** | `typing.Protocol` for static duck typing | Interface verification without explicit inheritance |
| **`dataclass`** | Decorator generating init, repr, eq | Clean data modeling with zero boilerplate |

## Example: Essential Dunder Methods

```python
class CustomVector:
    def __init__(self, x: int, y: int):
        self.x, self.y = x, y

    def __str__(self) -> str:        # User-facing string: print(v)
        return f"({self.x}, {self.y})"

    def __repr__(self) -> str:       # Developer debug string: v in REPL
        return f"CustomVector(x={self.x}, y={self.y})"

    def __add__(self, other):        # Operator overloading: v1 + v2
        return CustomVector(self.x + other.x, self.y + other.y)

    def __len__(self) -> int:        # len(v)
        return int((self.x**2 + self.y**2) ** 0.5)

    def __call__(self, factor: int): # Callable object: v(5)
        return CustomVector(self.x * factor, self.y * factor)
```

## Real-World Example

- **FastAPI and Pydantic**: Pydantic models leverage Python's metaclasses and `__init_subclass__` to provide runtime schema validation, serialization, and OpenAPI generation.
- **SQLAlchemy ORM**: Uses Python descriptors and dunder methods (`__get__`, `__set__`) to map class attributes directly to SQL table columns.

## Important Properties

| Feature | Java Equivalent | Python Characteristic |
|---|---|---|
| **Class Methods** | `static` method | `@classmethod` (receives `cls`, knows its actual subclass) |
| **Static Methods** | `static` utility method | `@staticmethod` (plain function inside class namespace) |
| **Interface** | `interface` keyword | `abc.ABC` (nominal) or `typing.Protocol` (structural) |
| **Operator Overload**| Not supported (except `+` for String) | Supported extensively via dunder methods (`__add__`, etc.) |
| **Data Container** | `record` | `@dataclass(frozen=True)` or `NamedTuple` |

### Common Misconceptions

* `__init__` does NOT create the object; `__new__` creates it, and `__init__` merely customizes attributes on the already-allocated instance.
* `__str__` and `__repr__` are not identical: `__str__` is for human-readable output, while `__repr__` should be unambiguous and ideally valid Python code to recreate the object.
* Multiple inheritance in Python does not cause chaotic crashes like in unmanaged C++; Python's C3 Linearization enforces a strictly defined, monotonic search path.

### What Interviewers Usually Test

1. Differences between `__new__` and `__init__`.
2. How Python's Method Resolution Order (MRO) handles the Diamond Problem.
3. Difference between `@staticmethod` and `@classmethod`.
4. Difference between `__str__` and `__repr__`.
5. Why and when to use `__slots__`.

### Common Follow-up Questions

1. What happens if a class defines both `__init__` and `__new__`?
2. How does `super()` determine which class method to invoke in Python?
3. How does `__slots__` reduce memory consumption?
4. What is the difference between `abc.ABC` and `typing.Protocol`?
5. How can you make a Python `@dataclass` immutable?

### Follow-up Answers

1. `__new__` is called first with the class as its first argument; if it returns an instance of that class, Python automatically invokes `__init__` on that new instance with the same arguments.
2. `super()` does not simply point to the parent class written in code; it looks up the next class in the runtime `__mro__` list of the calling instance (`self`), enabling cooperative multiple inheritance.
3. By default, every Python instance stores attributes in a dynamic dictionary (`__dict__`). Defining `__slots__ = ('x', 'y')` prevents dictionary creation and allocates fixed-size C array pointers, saving significant RAM when creating millions of instances.
4. `abc.ABC` requires explicit inheritance (`class Dog(Animal)`) checked at runtime. `Protocol` enables duck typing: any class that implements the required methods satisfies the protocol without ever inheriting from it.
5. Decorate the class with `@dataclass(frozen=True)`. Any attempt to assign to an attribute after initialization raises a `FrozenInstanceError`.
