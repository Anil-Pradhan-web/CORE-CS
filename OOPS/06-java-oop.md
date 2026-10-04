# 06. Java-Focused OOP

## 1. A normal Java class

```java
public final class Product {
    private final String id;
    private String name;
    private double price;

    public Product(String id, String name, double price) {
        if (id == null || id.isBlank()) {
            throw new IllegalArgumentException("id is required");
        }
        if (price < 0) {
            throw new IllegalArgumentException("price cannot be negative");
        }
        this.id = id;
        this.name = name;
        this.price = price;
    }

    public void updateName(String name) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("name is required");
        }
        this.name = name;
    }

    public String id() {
        return id;
    }

    public String name() {
        return name;
    }

    public double price() {
        return price;
    }
}
```

**Hinglish:** Java mein fields ko `private` rakho aur valid changes controlled methods se karo. `final` field constructor ke baad reassign nahi hota.

## 2. Abstract class versus interface

```java
abstract class Notification {
    protected final String recipient;

    Notification(String recipient) {
        this.recipient = recipient;
    }

    public abstract void send(String message);

    protected void validateMessage(String message) {
        if (message == null || message.isBlank()) {
            throw new IllegalArgumentException("message is required");
        }
    }
}
```

An interface is better when the contract describes a capability. An abstract class is useful when related subclasses share state or a stable algorithm.

## 3. Records

A record is a compact data carrier.

```java
public record Money(long amount, String currency) {
    public Money {
        if (amount < 0) {
            throw new IllegalArgumentException("amount cannot be negative");
        }
        if (currency == null || currency.isBlank()) {
            throw new IllegalArgumentException("currency is required");
        }
    }
}
```

Records provide accessors and value-based `equals`, `hashCode`, and `toString` automatically.

**Important:** Record immutability is shallow. If a component points to a mutable list, the list can still change.

**Hinglish:** Record boilerplate kam karta hai, but nested mutable object ko protect karna ab bhi zaroori hai.

## 4. Sealed types

```java
sealed interface PaymentResult permits PaymentSuccess, PaymentFailure, PaymentPending {}

record PaymentSuccess(String transactionId) implements PaymentResult {}

record PaymentFailure(String reason) implements PaymentResult {}

record PaymentPending(long timeoutMs) implements PaymentResult {}
```

A sealed interface limits the permitted implementations. This is useful when a domain has a known, closed set of outcomes.

A permitted subtype must be declared `final`, `sealed`, or `non-sealed`.

## 5. Pattern matching

```java
String describe(Object value) {
    if (value instanceof String text && !text.isBlank()) {
        return "text: " + text;
    }
    return "other";
}
```

With a sealed hierarchy, a modern `switch` expression can make the allowed outcomes explicit.

## 6. Rich enums

```java
public enum Operation {
    ADD("+") {
        @Override
        public double apply(double left, double right) {
            return left + right;
        }
    },
    MULTIPLY("*") {
        @Override
        public double apply(double left, double right) {
            return left * right;
        }
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    public abstract double apply(double left, double right);

    public String symbol() {
        return symbol;
    }
}
```

An enum can carry data and behaviour. It is useful for a fixed set of domain values.

## 7. `Optional`

Use `Optional` mainly as a return type when a method may naturally have no result.

```java
Optional<User> findById(String id) {
    return repository.findById(id);
}
```

Prefer `map`, `orElse`, or `orElseThrow` over calling `get()` without checking. Do not normally use `Optional` as a field, method parameter, or a wrapper for a collection.

**Hinglish:** `Optional` missing return value ko represent karta hai. Isse har field ya collection ko wrap mat karo.

## 8. Java collections

- `List` keeps order and allows duplicates.
- `Set` keeps unique values.
- `Map` stores key-value pairs.
- `Deque` supports efficient insertion and removal at both ends.
- `ConcurrentHashMap` is a concurrent map, not a universal replacement for synchronizing a whole business operation.

### PECS example

```java
double total(List<? extends Number> numbers) {
    double sum = 0;
    for (Number number : numbers) {
        sum += number.doubleValue();
    }
    return sum;
}
```

The method reads elements from the list and does not add elements, so the list is a **producer** and `? extends Number` is the correct wildcard.

## 9. Interface default methods

```java
public interface Named {
    String name();

    default String displayName() {
        return name().trim();
    }
}
```

Default methods can make interface evolution easier, but conflicting defaults still need an explicit choice in the implementing class.

## 10. Java exception style

```java
try {
    repository.save(order);
} catch (DataAccessException error) {
    throw new OrderStorageException("Could not save order " + order.id(), error);
}
```

Use domain-specific exceptions at important boundaries. Keep technical details out of public user messages.

## 11. Pass-by-value

Java passes arguments by value. For an object argument, the value is the reference value, so the method can mutate the referenced object but cannot make the caller’s reference point to a different object by assigning to the parameter.

## 12. Common Java interview corrections

- `final` does not automatically create deep immutability.
- A record cannot extend an arbitrary class; it implicitly extends `Record` and is final.
- A class can extend one class but implement many interfaces.
- `hashCode` and `equals` must be written together when value equality is needed.
- `volatile` is not a replacement for synchronization of compound operations.
- `List<?>` is a reference to a list of an unknown type. You can read elements as `Object` and can use structural operations such as `remove` or `clear`, but you cannot add a non-null element; it is different from `List<Object>`.

## 13. Java design mini-example

```java
public interface UserRepository {
    Optional<User> findById(String id);
    void save(User user);
}

public final class UserService {
    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }

    public User requireById(String id) {
        return repository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
}
```

The service owns business behaviour. The repository contract hides database details and can be replaced in tests.

## Quick interview Q&A

### 1. What is a record in Java?

A record is a compact, transparent data carrier whose fields are final and whose basic value methods are generated.

### 2. Are records deeply immutable?

No. They are shallowly immutable because referenced mutable objects can still change.

### 3. What is a sealed interface?

It is an interface that limits which classes or interfaces may implement it.

### 4. Why use sealed types?

They model a closed set of domain outcomes and can make exhaustive handling clearer.

### 5. What is pattern matching?

It lets code test and bind a value’s type without a separate cast.

### 6. What is a rich enum?

An enum with fields, constructors, or methods, allowing each constant to carry data and behaviour.

### 7. What is `Optional` for?

It represents a possibly absent return value in a clear API. It is not a replacement for every nullable value or collection.

### 8. Can a record implement an interface?

Yes. A record cannot extend a normal class because it implicitly extends `Record`.

### 9. Can a class implement multiple interfaces?

Yes. Java does not allow multiple class inheritance, but multiple interfaces are allowed.

### 10. What is the Java PECS rule?

Producer Extends, Consumer Super. It describes how wildcards should be chosen for safe reading or writing.

### 11. What does Java pass by value?

Arguments are copied into the method. An object parameter is a copied reference value, not the object itself.

### 12. When should a Java class use composition?

When it has a part, a dependency, or a replaceable behaviour rather than a true `is-a` relationship.

### 13. What is a default interface method?

It is an interface method with an implementation, commonly used to evolve an interface without forcing every implementation to change.

### 14. Why should `Optional.get()` be used carefully?

It can throw if the value is absent. Prefer safe operations such as `orElse`, `map`, or `orElseThrow`.

**Hinglish recap:** Java mein record, sealed, enum, aur interface modern design ke tools hain—simple problem ke liye overkill mat karo.
