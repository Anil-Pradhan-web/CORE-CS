# OOP Basics and the Four Pillars

## What is it?

Object-Oriented Programming (OOP) organizes software around objects that combine state and behaviour. It helps code model real responsibilities and change without exposing every implementation detail.

## Simple Explanation

An object is like a coffee machine: it has state (water level, beans) and actions (`brew`, `clean`). Users press safe buttons instead of reaching inside the machine.

## Hinglish Explanation

OOP mein data aur us data ko operate karne wale methods ek object ke andar rakhte hain. Isse code real-world entity jaisa model hota hai aur implementation change karne par har caller ko change nahi karna padta.

## Why Do We Need It?

Large applications need boundaries: who owns data, which operations are valid, and where changes belong. OOP can reduce coupling and make behaviours easier to test, replace, and extend when used with composition and interfaces.

## How It Works

1. Define a class as a reusable blueprint and behaviour contract.
2. Create objects with individual state.
3. Expose meaningful methods rather than public mutable internals.
4. Use abstraction/interfaces for stable contracts.
5. Reuse or substitute behaviour through composition, inheritance, and polymorphism.

```text
Client → PaymentService (interface)
              ├── CardPayment
              ├── UpiPayment
              └── WalletPayment
```

## Important Terminology

| Term | Meaning |
|---|---|
| Class | Blueprint defining data and behaviour |
| Object | Runtime instance of a class |
| State | Data currently held by object |
| Method | Operation an object performs |
| Encapsulation | Control access to state/invariants |
| Abstraction | Show essential contract, hide details |
| Inheritance | Derive specialized type from base type |
| Polymorphism | One contract, different implementations |

## Example

`Order` owns its valid state transitions: `pay()`, `ship()`, and `cancel()`. A caller should not directly set `order.status = SHIPPED` when payment has not happened.

## Real-World Example

Backend services often use interfaces for external dependencies: `EmailSender`, `PaymentGateway`, or `Clock`. Production and test implementations can change without changing business logic.

## Important Properties

| Pillar | Main value | Common misuse |
|---|---|---|
| Encapsulation | Protect invariants | Getters/setters for every field without rules |
| Abstraction | Hide accidental complexity | Vague or oversized interfaces |
| Inheritance | True “is-a” specialization | Reuse-only deep hierarchies |
| Polymorphism | Substitutable behaviour | Type checks instead of dispatch |

### Common Misconceptions

* OOP is not only classes and inheritance; composition is often safer reuse.
* Private fields plus trivial setters do not automatically give useful encapsulation.
* OOP does not guarantee good design without cohesive responsibilities.

### What Interviewers Usually Test

Whether you can define the four pillars accurately, identify responsibilities, and choose composition versus inheritance for a real feature.

### Common Follow-up Questions

1. Class versus object?
2. Why prefer composition sometimes?
3. Can OOP have functions too?
4. What is an invariant?
5. Is OOP only for Java/C++?

### Follow-up Answers

1. A class is the definition/blueprint; an object is a runtime instance with its own state.
2. Composition reuses behaviour without tightly coupling to a fragile base-class hierarchy and allows runtime replacement.
3. Yes. Methods are functions attached to types/objects, and many OOP languages also support standalone functions.
4. An invariant is a condition that must remain true for an object to be valid, such as nonnegative account balance.
5. No. Python, C#, Kotlin, Ruby, TypeScript, and many other languages support object-oriented features.
