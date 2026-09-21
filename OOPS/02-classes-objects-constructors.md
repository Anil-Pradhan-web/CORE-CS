# Classes, Objects, Constructors, and Object State

## What is it?

A class specifies fields and methods; an object is an instance created at runtime. A constructor establishes a valid initial state before ordinary methods are used.

## Simple Explanation

A class is an apartment floor plan. Each object is one built apartment with its own resident data. The constructor is the handover checklist that makes sure electricity, door, and address are ready.

## Hinglish Explanation

Class blueprint hai, object actual runtime instance. Constructor object banate time required fields validate/set karta hai so object invalid state mein start na ho.

## Why Do We Need It?

Objects make ownership and lifecycle explicit. Constructors prevent partially initialized values leaking into application logic and reduce later null/invalid-state checks.

## How It Works

1. Runtime allocates object storage.
2. Fields receive default/initializer values.
3. Base-class constructor runs before derived constructor where inheritance applies.
4. Constructor validates inputs and establishes invariants.
5. Reference to object is returned; callers use public behaviour.

```java
final class Money {
  private final long paise;
  Money(long paise) {
    if (paise < 0) throw new IllegalArgumentException("negative");
    this.paise = paise;
  }
}
```

## Important Terminology

| Term | Meaning |
|---|---|
| Instance field | Per-object state |
| Static field | State shared by class, not each object |
| Constructor | Initialization operation called during creation |
| Default constructor | No-argument constructor supplied/declared under language rules |
| Immutable object | State cannot change after construction |
| `this` | Current object reference in many languages |

## Example

A `User` constructor can require validated email and ID. Optional profile data can be represented deliberately with defaults, builder, or nullable/optional type rather than a half-ready object.

## Real-World Example

Immutable configuration objects are constructed once at service startup and safely shared across request threads. A builder keeps constructor readable when many optional settings exist.

## Important Properties

| Choice | Benefit | Trade-off |
|---|---|---|
| Constructor validation | Valid objects only | Can be verbose for many options |
| Builder | Readable optional fields | More code/another object |
| Immutable fields | Thread-safe/simple reasoning | New object needed for changes |
| Public mutable fields | Simple syntax | Breaks encapsulation/invariants |

### Common Misconceptions

* Constructor is not an ordinary method; it has language-specific creation rules and no normal return type in Java/C++.
* Object creation does not always mean memory is immediately physical RAM; runtime/OS allocation mechanisms differ.
* Static data belongs to class-level context, not every individual object.

### What Interviewers Usually Test

Initialization order, constructor validation, static-vs-instance state, immutability, and use of builders for API clarity.

### Common Follow-up Questions

1. Constructor versus method?
2. Can constructor be private?
3. Why use a builder?
4. Static versus instance member?
5. What is immutability?

### Follow-up Answers

1. Constructor initializes a new object under special language rules; a method is an ordinary operation on an existing class/object.
2. Yes; private constructors can control creation through factories or prevent instantiation of utility classes.
3. It makes many optional parameters readable and can validate once before building an immutable object.
4. Static member belongs to class-level context; instance member belongs to each separate object.
5. Immutable means externally observable state cannot change after construction, simplifying sharing and reasoning.
