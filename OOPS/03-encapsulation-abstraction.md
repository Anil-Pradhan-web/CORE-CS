# Encapsulation and Abstraction

## What is it?

Encapsulation keeps state with operations that protect its rules. Abstraction exposes a useful contract while hiding irrelevant implementation details. They work together but solve different problems.

## Simple Explanation

Encapsulation is locking the car engine so drivers use pedals safely. Abstraction is showing a steering wheel and pedals without teaching every driver how the engine works.

## Hinglish Explanation

Encapsulation data ko direct wrong update se protect karta hai. Abstraction caller ko sirf required operations dikhata hai; internal algorithm, database ya API details hide rehte hain.

## Why Do We Need It?

Without encapsulation, every caller can violate business rules. Without abstraction, callers depend on changing implementation details, making refactors risky and tests difficult.

## How It Works

1. Keep fields private or otherwise controlled.
2. Publish intent-based methods such as `withdraw(amount)`.
3. Validate state transitions inside owner object.
4. Define interfaces/abstract types around caller needs.
5. Inject implementation behind the contract.

```text
CheckoutService → TaxCalculator interface
                      ├── IndiaGstCalculator
                      └── TestTaxCalculator
```

## Important Terminology

| Term | Meaning |
|---|---|
| Information hiding | Hide implementation decisions likely to change |
| API/contract | Promised observable behaviour |
| Invariant | Condition object keeps true |
| Interface | Behavioural contract without concrete implementation details |
| Getter/setter | Accessor/mutator; not always good design |

## Example

`BankAccount.deposit` rejects negative values and changes balance atomically. Exposing `setBalance` would allow invalid negative balance and spreads business rules across callers.

## Real-World Example

A payment service depends on `PaymentGateway.charge()`, not a specific Stripe/UPI SDK. The adapter implementation can be swapped or tested with a fake.

## Important Properties

| Concept | Answers | Typical mechanism |
|---|---|---|
| Encapsulation | Who may change state and how? | Private fields + methods |
| Abstraction | What capability does caller need? | Interface/abstract type |
| Information hiding | What details may change freely? | Module/private implementation |

### Common Misconceptions

* Abstraction is not only abstract classes; modules and interfaces can abstract too.
* Encapsulation is not simply “make every field private then create setter.”
* Hiding code is not useful if contract is unclear.

### What Interviewers Usually Test

Whether you distinguish the ideas, protect a realistic invariant, and define a narrow dependency contract.

### Common Follow-up Questions

1. Encapsulation versus abstraction?
2. Are getters/setters always good?
3. What is information hiding?
4. Why depend on interface?
5. What makes an API contract good?

### Follow-up Answers

1. Encapsulation controls state access and invariants; abstraction presents essential capability while hiding implementation details.
2. No. A setter that exposes arbitrary state mutation can break invariants; prefer meaningful operations where appropriate.
3. It hides change-prone implementation choices so clients depend only on stable behaviour.
4. It reduces coupling, enables substitution/testing, and lets implementation change without changing caller code.
5. It is small, coherent, explicit about inputs/outputs/errors, and stable around client needs rather than implementation accidents.
