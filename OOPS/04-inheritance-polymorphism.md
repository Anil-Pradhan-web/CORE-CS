# Inheritance and Polymorphism

## What is it?

Inheritance lets a derived type reuse and specialize a base type when there is a genuine substitutable “is-a” relationship. Polymorphism lets code use a common contract while runtime dispatch selects the correct implementation.

## Simple Explanation

A `Vehicle` contract may say `move()`. A `Car` and `Bike` both satisfy it differently. The traffic controller only needs “vehicle,” not a giant chain of `if car else bike` checks.

## Hinglish Explanation

Inheritance tab use karo jab child truly parent ki jagah safely use ho sake. Polymorphism mein caller common interface/base type use karta hai aur actual object ka override runtime par call hota hai.

## Why Do We Need It?

Polymorphism keeps high-level code open to new behaviours without rewriting type checks. Inheritance can capture shared stable behaviour, but composition is safer for changeable features.

## How It Works

1. Define base contract with allowed operations.
2. Derived type inherits/overrides behaviour.
3. Caller holds base/interface reference.
4. Runtime dispatch invokes object’s implementation.

```java
interface Notifier { void send(String message); }
final class EmailNotifier implements Notifier { public void send(String m) {} }
final class SmsNotifier implements Notifier { public void send(String m) {} }
```

## Important Terminology

| Term | Meaning |
|---|---|
| Base/superclass | General parent type |
| Derived/subclass | Specialized child type |
| Override | Replace inherited instance behaviour |
| Overload | Same name, different parameter list; compile-time choice |
| Dynamic dispatch | Runtime selection of overridden method |
| LSP | Subtype should remain usable where base type is expected |

## Example

`List<Notifier>` can hold email, SMS, and push notifiers. A loop calls `send()` without knowing each vendor’s internal code.

## Real-World Example

Payment integrations use polymorphic gateway adapters. Avoid making `CreditCardPayment extends Payment` merely to reuse fields if alternatives differ in lifecycle, security, or settlement behaviour.

## Important Properties

| Reuse method | Strength | Risk |
|---|---|---|
| Inheritance | Shared stable semantics | Tight coupling/fragile base class |
| Composition | Flexible delegated behaviour | More wiring/objects |
| Interface polymorphism | Decoupled clients | Can create too many abstractions |

### Common Misconceptions

* Method overloading is compile-time polymorphism; overriding is runtime polymorphism in common terminology.
* Inheritance is not automatically better than composition.
* A subclass that throws unsupported exceptions for base operations may violate substitutability.

### What Interviewers Usually Test

The is-a test, overriding-vs-overloading, dynamic dispatch, Liskov substitution, and composition-over-inheritance judgment.

### Common Follow-up Questions

1. Override versus overload?
2. What is dynamic dispatch?
3. Why composition over inheritance?
4. What is LSP?
5. Can constructor be overridden?

### Follow-up Answers

1. Override supplies derived implementation of same contract; overload uses same name with different parameter signature, selected at compile time.
2. It is choosing overridden instance method based on actual runtime object rather than reference variable’s declared type.
3. Composition avoids forcing a fragile is-a hierarchy and lets behaviour vary or change independently.
4. Liskov Substitution Principle says clients using a base type should work correctly with its subtype without surprises.
5. No. Constructors initialize a particular class and are not inherited/overridden like instance methods.
