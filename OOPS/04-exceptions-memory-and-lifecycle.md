# 04. Exceptions, Memory, and Object Lifecycle

## 1. What is an exception?

An exception is an object that represents an abnormal or exceptional situation.

```text
try risky operation
catch understand and handle the specific failure
finally cleanup if needed
```

**Hinglish:** Exception error ka structured message hai. Isse code ko sirf `false` ya `-1` return karne ke bajaye bataya jaata hai ki kya hua aur kahan handle karna hai.

## 2. Java exception types

### Checked exception

The compiler asks the caller to catch it or declare it.

Examples: `IOException`, `SQLException`.

### Unchecked exception

An unchecked exception does not require mandatory catch or declaration. `RuntimeException` and its subclasses are unchecked; `Error` is also unchecked.

Examples: `NullPointerException`, `IllegalArgumentException`.

Do not use checked exceptions for ordinary validation such as “amount must be positive”. A direct validation check is clearer.

**Hinglish:** Java mein checked exception compile-time handling maangta hai. Unchecked exception ko compiler se mandatory handling nahi maangti; failure aksar runtime par dikhta hai. Har failure ko exception banana zaroori nahi.

## 3. Python exceptions

Python user-code exceptions normally inherit from `Exception`.

```python
class InsufficientFundsError(Exception):
    pass

def withdraw(balance: float, amount: float) -> float:
    if amount <= 0:
        raise ValueError("amount must be positive")
    if amount > balance:
        raise InsufficientFundsError("Insufficient funds")
    return balance - amount
```

Python does not have Java's checked/unchecked split.

## 4. Catch the exception you can handle

Bad:

```python
try:
    process_order()
except Exception:
    pass
```

This hides useful errors.

Better:

```python
try:
    process_order()
except InsufficientFundsError as error:
    show_payment_error(str(error))
except TimeoutError:
    retry_later()
```

Never silently swallow a failure.

**Hinglish:** Generic catch aur empty `pass` bugs chhupate hain. Specific error pakdo, handle karo, ya safe boundary tak propagate karo.

## 5. Exception chaining

When a low-level error causes a business error, preserve the original cause.

```python
try:
    charge_gateway()
except GatewayTimeout as error:
    raise PaymentUnavailableError("Payment gateway timed out") from error
```

Java uses the cause constructor:

```java
throw new PaymentProcessingException("Payment failed", originalError);
```

**Hinglish:** Root cause ko preserve karna debugging easy banata hai. Sirf generic message replace karke original error kho dena sahi nahi hai.

## 6. Error handling at the boundary

Business code should raise meaningful domain errors. A web or UI boundary can map them to user-friendly responses.

```text
Domain service -> DomainException
HTTP boundary   -> 400/404/409/500 response
Logger          -> technical details
```

Do not send a database stack trace directly to a user.

## 7. Deterministic resource cleanup

Memory cleanup can be delayed. Files, sockets, and database connections should be closed as soon as their scope ends.

### Java

```java
try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
    String line = reader.readLine();
    System.out.println(line);
}
```

### Python

```python
from pathlib import Path

with Path(path).open(encoding="utf-8") as file:
    first_line = file.readline()
    print(first_line)
```

**Hinglish:** File, socket, ya DB connection ko garbage collector par depend mat chhodo. `with` or `try-with-resources` use karo.

## 8. Object lifecycle

A simple lifecycle is:

```text
created -> initialized -> used -> changed or retired -> collected
```

A good design makes these stages visible:

- Constructor or initializer creates a valid object.
- Methods keep the object valid.
- A close or release method frees external resources.
- Garbage collection handles memory later; it is not a business shutdown protocol.

## 9. Stack, heap, references, and garbage collection

For interviews, use this simple model:

- Local variables and method-call information are often associated with the stack.
- Objects are commonly created on the heap.
- A reference is a value that lets code reach an object. In Java it can be stored in a variable, field, or array element; in Python a variable is a name bound to an object.
- When an object is no longer reachable, the runtime may reclaim its memory.

**Important:** Exact placement and GC behaviour depend on the language and runtime. Do not claim one fixed algorithm for every language.

**Hinglish:** Stack/heap ka simple model yaad rakho, but implementation ko runtime decide karta hai. Object tab collect hota hai jab use reachable na ho.

## 10. Common memory leaks

### Retaining a collection forever

```python
cache = {}
cache["large-result"] = result
```

If the cache grows without limits or invalidation, memory can grow too much.

### Global references

A global list holding every request object prevents those objects from being collected.

### Unclosed resources

Not always a heap leak, but it can exhaust file descriptors, sockets, or database connections.

### Listener registrations

A long-lived publisher that keeps references to short-lived listeners can retain them.

**Hinglish:** Leak ka common matlab hai “object ko abhi bhi koi reference chala raha hai jab use hone ki zaroorat nahi”.

## 11. Immutability and memory safety

An immutable object cannot change after construction. Immutable values are:

- Easier to reason about
- Safer to share between threads
- Suitable as dictionary or hash-map keys, provided referenced mutable state is also handled safely

A Java record is shallowly immutable: its fields cannot be reassigned, but a referenced mutable list can still change.

**Hinglish:** Immutability ka fayda tabhi poora hai jab object ke andar referenced objects bhi safe hon.

## 12. Final interview summary

- Use exceptions for abnormal failures, not normal branching.
- Catch only what you can handle.
- Preserve the root cause.
- Close resources deterministically.
- Treat garbage collection as memory management, not business cleanup.
- Make objects immutable when it simplifies sharing and reasoning.

## Quick interview Q&A

### 1. What is an exception?

It is a typed object representing a failure that interrupts normal flow.

### 2. What is the difference between checked and unchecked exceptions in Java?

Checked exceptions require compile-time handling or declaration. Unchecked exceptions are not required to be caught or declared; they include `RuntimeException` and `Error`.

### 3. Why should we not catch every `Exception`?

Because it hides programming errors and makes recovery unclear. Catch the narrowest exception you can actually handle.

### 4. What is exception chaining?

It preserves a lower-level failure as the cause of a higher-level failure.

### 5. What is fail-fast design?

Rejecting an invalid operation immediately instead of continuing with a broken state.

### 6. What does `finally` do?

It runs cleanup code during normal completion and while an exception is unwinding. For resources, prefer `try-with-resources` in Java and `with` in Python.

### 7. What is garbage collection?

Automatic memory reclamation for unreachable objects. It does not guarantee when a file, socket, or transaction is closed.

### 8. What is a memory leak?

A program keeps references to data it no longer needs, so memory cannot be reclaimed.

### 9. Is `__del__` a good way to close a file in Python?

No. Use a context manager so cleanup happens at a predictable time.

### 10. Is a Java `final` class deeply immutable?

No. It prevents extension, but referenced mutable objects may still change unless you protect or copy them.

### 11. What is a shallow copy?

A new container is created, but referenced objects inside it are shared with the original.

### 12. What is a memory barrier?

A low-level mechanism that controls visibility and ordering between threads. Do not rely on a vague “flushes cache” story when discussing Java `volatile`.

**Hinglish recap:** Error ko clear rakho, resource ko timely close karo, aur garbage ko business lifecycle ka replacement mat samjho.
