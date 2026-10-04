# 05. Collections, Equality, and Concurrency

## 1. Choosing a collection

| Need | Typical collection | Example |
|---|---|---|
| Ordered items | List / `ArrayList` | Students in a class |
| Unique items | Set / `HashSet` | User IDs |
| Key-value lookup | Map / `HashMap` / `dict` | User ID to user |
| First-in, first-out | Queue / `deque` | Job processing |
| Last-in, first-out | Stack | Undo operations |

### Average complexity

| Operation | List | Hash-based set/map | Balanced sorted set/map |
|---|---:|---:|---:|
| Read by index | O(1) in many list implementations | Not typical | Not typical |
| Search | O(n) | O(1) average | O(log n) |
| Insert at end | O(1) amortized in many lists | O(1) average | O(log n) |
| Insert at front | O(n) in dynamic arrays | Not typical | Not typical |

**Hinglish:** List order rakhti hai, Set duplicate hatata hai, aur Map key se value dhoondta hai. Complexity table dekh kar data structure choose karo.

## 2. Java generics

Generics let a class or method work with a type parameter while allowing type checking.

```java
class Box<T> {
    private T value;

    void set(T value) {
        this.value = value;
    }

    T get() {
        return value;
    }
}
```

### PECS

- **Producer Extends:** Read from a collection.
- **Consumer Super:** Write into a collection.

```java
List<? extends Number> numbers = List.of(1, 2.5);
double first = numbers.get(0).doubleValue();
```

Java generics are invariant by default: `List<String>` is not the same type as `List<Object>`.

## 3. Python type hints

Python generics help static type checkers and IDEs; Python does not enforce them at runtime by default.

```python
from typing import Generic, TypeVar

T = TypeVar("T")

class Box(Generic[T]):
    def __init__(self, value: T) -> None:
        self.value = value

    def get(self) -> T:
        return self.value
```

**Hinglish:** Java mein compiler generic type check karta hai. Python mein type hints mostly tooling ke liye hain; runtime validation alag se karni padti hai.

## 4. Equality versus identity

- **Identity** asks: “Are these the same object instance?”
- **Equality** asks: “How does this type’s `equals` or `__eq__` contract compare them?” Equality is often value-based, but it can be custom or identity-based.

```python
a = [1, 2]
b = [1, 2]
a == b   # True
a is b   # False
```

An object’s identity normally stays the same. A type’s equality contract may change after mutation if its fields participate in that comparison.

## 5. The equality and hash contract

A hash-based collection first uses a hash to choose a bucket and then checks equality. Java and Python therefore require a strong relationship:

> If two objects are equal, they must have the same hash value.

### Java example

```java
final class UserId {
    private final long value;

    UserId(long value) {
        this.value = value;
    }

    @Override
    public boolean equals(Object other) {
        if (this == other) return true;
        if (!(other instanceof UserId userId)) return false;
        return value == userId.value;
    }

    @Override
    public int hashCode() {
        return Long.hashCode(value);
    }
}
```

### Changing state used by a key

If a field used by `hashCode()` changes while the object is stored in a `HashMap`, the map may no longer find it. Prefer an immutable key when the key is based on field values.

For a custom Python class, defining `__eq__` normally makes it unhashable unless a correct `__hash__` is also provided. Mutating fields used by `__hash__` after insertion can break lookup. Mutability alone is not a problem for a key whose hash and equality use only stable identity.

**Hinglish:** Equal objects ka hash same hona chahiye. Hash mein use hone wala field change mat karo, warna dictionary/hash map ka lookup fail ho sakta hai.

## 6. Copying objects

### Shallow copy

A new outer object is created, but nested references are shared.

### Deep copy

Nested objects are copied too, so changing the copy does not change the original.

Use deep copying only when independence is actually required. It can be slower and can be impossible for resources such as sockets or database connections.

## 7. Serialization

Serialization converts an object into a stored or transmitted representation.

Safety rules:

- Do not deserialize untrusted data.
- Validate allowed classes and fields.
- Never serialize secrets casually.
- Version the format when stored data must survive code changes.

Python `pickle` is powerful but unsafe for untrusted input.

## 8. Concurrency basics

Concurrency means more than one task can make progress over time.

### Race condition

Two tasks read and change shared data without a safe order, so the result depends on timing.

```python
counter += 1
```

This is not automatically safe when many threads or processes share the counter.

### Mutex or lock

A lock allows one task at a time to enter a critical section.

```python
from threading import Lock

counter_lock = Lock()
counter = 0

def increment() -> None:
    global counter
    with counter_lock:
        counter += 1
```

### Atomic operation

One indivisible operation, such as using a concurrent counter API instead of manually doing read, add, and write.

### Deadlock

Two tasks wait for locks held by each other. Keep lock order consistent and hold locks for a short time.

### Immutable data

Immutable data is usually easier to share between threads because it cannot change halfway through an operation.

**Hinglish:** Shared mutable state ko lock ya immutable design se protect karo. Lock ordering consistent rakho, warna deadlock ho sakta hai.

## 9. Java and Python concurrency differences

- Java commonly uses threads, executors, locks, and concurrent collections.
- Python commonly uses threads, `asyncio`, processes, and synchronization primitives.
- The Python GIL affects how some CPU-bound Python code runs, but it is not a universal guarantee that all Python code is single-threaded.
- `volatile` in Java gives visibility and ordering guarantees; it does not make a compound operation such as `count++` atomic.

## 10. Safe cache design

An LRU cache changes recency on both reads and writes. If multiple threads access it, the read also needs a consistent synchronization or concurrency design. A read-only lock is not enough when `get` mutates recency.

**Hinglish:** “Get” bhi state mutate karta hai to use read-only method maan mat karo. Isliye LRU cache ka concurrency design carefully choose karo.

## Quick interview Q&A

### 1. When should we use a `List`?

When order and duplicates matter and we need index-based or sequential access.

### 2. When should we use a `Set`?

When values must be unique and membership lookup is more important than order.

### 3. When should we use a `Map` or dictionary?

When data is naturally represented as key-value pairs and keys are unique.

### 4. What are generics?

Generics provide type parameters so reusable code can work safely with different types.

### 5. What does PECS mean?

Producer Extends, Consumer Super. Use wildcards according to whether a collection is mainly read or written.

### 6. What is the difference between `==` and `equals()` in Java?

`==` compares references for objects, while a correctly overridden `equals()` compares logical value.

### 7. Why must equal objects have the same hash code?

Hash-based collections use the hash to locate candidates. Different hashes can place equal objects into different buckets and break lookup.

### 8. What is the problem with changing state used by a hash key?

Changing a field used by the hash makes the key move logically from one bucket to another, so the collection may not find it.

### 9. What is a shallow copy?

A new outer object is created, but referenced nested objects are shared.

### 10. What is a race condition?

A result depends on unpredictable timing between shared operations.

### 11. What is a deadlock?

A situation where tasks wait for one another’s resources and none can continue.

### 12. Does `volatile` make `count++` thread-safe?

No. It helps visibility and ordering, but a read-modify-write sequence is still compound and must be made atomic or synchronized.

### 13. Why is `pickle` risky for untrusted data?

Deserializing attacker-controlled data can execute unsafe code. Use a safe format and validation.

### 14. What is the average complexity of a hash map lookup?

Usually O(1) on average when the hash distribution and load factor are healthy; worst cases can degrade.

**Hinglish recap:** Data structure performance aur thread safety dono design decisions hain—requirements samajh ke choose karo.
