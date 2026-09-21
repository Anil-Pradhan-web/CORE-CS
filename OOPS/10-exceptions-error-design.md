# Exceptions and Error Design in Object-Oriented Systems

## What is it?

Exception handling is an object-oriented mechanism for separating normal application logic from anomalous error conditions. Rather than returning cryptic error status codes (like `-1` or `NULL`), errors are represented as rich, typed objects with stack traces, context, and hierarchy.

## Simple Explanation

Imagine sending a courier package.
- **Error Code approach**: The delivery person returns to the office and hands you a piece of paper with "code 42", and you have to look up what 42 means in a giant manual.
- **OOP Exception approach**: The delivery person hands you an `AddressNotFoundException` object containing the incorrect address, the timestamp of the attempt, and a trace of the route taken.

## Hinglish Explanation

Traditional procedural programming mein functions error codes return karte the (jaise 0 success, -1 failure), jisko callers ignore kar dete the. OOP mein errors first-class objects hote hain jo crash ke time call stack capture karte hain. Java mein do types hote hain: **Checked** (compile-time par handle karna compulsory hai) aur **Unchecked** (`RuntimeException`). Python mein saare exceptions unchecked hote hain aur `Exception` class se inherit karte hain.

## Why Do We Need It?

1. **Separation of Concerns**: Business logic isn't cluttered with repetitive `if (res < 0)` checks after every single line.
2. **Error Propagation**: Errors bubble up through the call stack automatically until a component capable of handling or logging them catches them.
3. **Cannot Be Ignored**: Unlike ignored return values, unhandled exceptions safely halt execution instead of corrupting data silently.

## How It Works

### Java Exception Hierarchy

```text
               Throwable
              /         \
          Error          Exception
       (JVM crash)      /         \
             Checked Exceptions   RuntimeException (Unchecked)
             (IOException, etc.)  (NullPointerException, etc.)
```

### Python Exception Hierarchy

```text
             BaseException
            /      |      \
KeyboardInterrupt SystemExit Exception (Standard root for user code)
                             /        \
                    ValueError        CustomApplicationError
```

### Resource Management Comparison

#### Java: `try-with-resources` (`AutoCloseable`)
```java
public void readFile(String path) throws IOException {
    // Automatically calls file.close() even if an exception is thrown
    try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
        String line = reader.readLine();
        System.out.println(line);
    }
}
```

#### Python: Context Manager (`with` statement, `__enter__` / `__exit__`)
```python
class DatabaseSession:
    def __enter__(self):
        print("Opening DB connection")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Closing DB connection safely")
        # Returning False allows exceptions to propagate normally
        return False

# Usage:
with DatabaseSession() as session:
    print("Executing query...")
    # Connection closes automatically even if an error occurs here!
```

### Custom Exception Design

#### Python Custom Exception
```python
class InsufficientFundsError(Exception):
    def __init__(self, current_balance: float, requested_amount: float):
        super().__init__(f"Balance ₹{current_balance} insufficient for withdrawal ₹{requested_amount}")
        self.current_balance = current_balance
        self.requested_amount = requested_amount
```

#### Java Custom Exception
```java
public class InsufficientFundsException extends RuntimeException {
    private final double balance;
    private final double requested;

    public InsufficientFundsException(double balance, double requested) {
        super(String.format("Balance ₹%.2f is less than requested ₹%.2f", balance, requested));
        this.balance = balance;
        this.requested = requested;
    }

    public double getDeficit() { return requested - balance; }
}
```

## Important Terminology

| Term | Meaning |
|---|---|
| **Checked Exception** | Compiler forces method to either catch it or declare it in `throws` (Java only) |
| **Unchecked Exception** | Runtime exceptions that do not require mandatory compile-time catch blocks |
| **RAII / Context Manager**| Deterministic cleanup idiom binding resource lifetime to block scope |
| **Exception Chaining** | Wrapping a low-level exception in a high-level domain exception while preserving cause |
| **Fail-Fast** | Immediately throwing an exception upon detecting invalid state instead of continuing |

## Example

Preserving the root cause (Exception Chaining):
- **Python**: `raise ServiceUnavailableError("Payment service down") from original_network_error`
- **Java**: `throw new OrderProcessingException("Failed order", originalSqlException);`
This prevents losing the underlying low-level stack trace while exposing clean domain errors.

## Real-World Example

REST API Error Handlers (e.g. `@ControllerAdvice` in Spring Boot, `@app.exception_handler` in FastAPI):
Domain services throw high-level exceptions like `UserNotFoundException`. A centralized web layer catches it and maps it to an HTTP `404 Not Found` JSON response with error timestamp and trace ID, avoiding leaking internal DB stack traces to public clients.

## Important Properties

| Feature | Java | Python |
|---|---|---|
| **Checked vs. Unchecked** | Both (Checked requires `throws`; Unchecked extends `RuntimeException`) | Unchecked only (all exceptions inherit from `BaseException` / `Exception`) |
| **Root of User Errors** | `java.lang.Exception` | `Exception` (never inherit directly from `BaseException`) |
| **Cleanup Mechanism** | `try-with-resources` (`AutoCloseable` interface) | `with` statement (`__enter__` and `__exit__` dunder methods) |
| **Control Flow Use** | Strongly discouraged (high stack unwinding overhead) | Occasionally accepted (e.g., `StopIteration` in iterators) |

### Common Misconceptions

* Catching generic `Exception` or `Throwable` and doing nothing (swallowing errors with empty `catch`/`except: pass`) is dangerous; it hides critical production bugs.
* `Error` in Java (like `OutOfMemoryError` or `StackOverflowError`) should generally **not** be caught; it signals unrecoverable JVM failure.
* Exceptions should not replace ordinary conditional validation (e.g., don't wrap a null pointer in a try-catch when a simple `if (user == null)` check suffices).

### What Interviewers Usually Test

1. Difference between Checked and Unchecked exceptions in Java and why modern languages (including Python, Kotlin, C#) avoided Checked exceptions.
2. How `try-with-resources` works under the hood (the `AutoCloseable` contract).
3. How Python's `with` statement and `__enter__`/`__exit__` work.
4. Exception hierarchy and why inheriting from `Exception` is preferred over `BaseException` in Python.
5. Best practices for designing custom business domain exceptions.

### Common Follow-up Questions

1. Why are Checked exceptions widely considered a controversial or failed feature in Java?
2. What is the difference between `finally` and `try-with-resources`?
3. What happens if an exception is thrown in `__exit__` or during `AutoCloseable.close()`?
4. How do you re-raise an exception in Python while preserving the stack trace?
5. When should you use a Result object (e.g., `Optional`, `Result<T, E>`) instead of throwing an exception?

### Follow-up Answers

1. Checked exceptions clutter method signatures, break interface contracts (an implementation cannot easily add new checked exceptions to an existing interface), and often lead developers to write anti-patterns like empty catch blocks.
2. `finally` requires manual nested null-checks and close calls where an exception in closing can suppress the original business exception; `try-with-resources` automatically handles multi-resource closing and uses "suppressed exceptions" to preserve the primary error.
3. In Python, if `__exit__` returns `True`, the exception is suppressed; if it returns `False` or `None`, the exception continues propagating. In Java, any exception during `.close()` is attached to the primary exception as a suppressed exception (`e.getSuppressed()`).
4. In Python, use a bare `raise` inside an `except` block, or use `raise NewException(...) from err` for explicit chaining.
5. For expected, high-frequency domain outcomes (e.g., user not found in search, validation failure on form input), a Result or `Optional` is cleaner and faster because creating exceptions incurs stack trace generation overhead.
