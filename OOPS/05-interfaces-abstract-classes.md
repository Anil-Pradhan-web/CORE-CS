# Interfaces and Abstract Classes

## What is it?

An interface describes capabilities a type promises. An abstract class provides a partially implemented base that cannot be directly instantiated. The correct choice depends on whether you need a contract, shared state/implementation, or both.

## Simple Explanation

An interface is a plug standard: any compatible appliance fits. An abstract class is a partly built appliance frame with shared wiring that specialised products complete.

## Hinglish Explanation

Interface caller-facing contract hota hai—multiple unrelated classes implement kar sakti hain. Abstract class common state/default behavior share karne ke liye useful hai, but inheritance coupling create karta hai.

## Why Do We Need It?

Interfaces support dependency inversion and testing. Abstract classes avoid duplicate stable implementation when types truly share an identity and lifecycle.

## How It Works

1. Identify what a client needs, e.g. `Cache.get/put`.
2. Define interface around that need.
3. Supply implementations: in-memory, Redis, test fake.
4. Use abstract class only if subclasses share protected state/template workflow.

```text
Cache (interface)
  ├── InMemoryCache
  ├── RedisCache
  └── FakeCache

AbstractReportExporter
  ├── CsvReportExporter
  └── PdfReportExporter
```

## Important Terminology

| Term | Meaning |
|---|---|
| Interface | Capability/contract |
| Abstract method | Declared behaviour implemented by subclass |
| Default method | Interface-provided default in languages that support it |
| Template method | Base defines algorithm skeleton; child fills steps |
| Dependency inversion | High-level code depends on abstraction, not details |

## Example

`OrderService` receives a `Clock` interface. Production uses system time; tests use fixed clock, making expiry logic deterministic.

## Real-World Example

Frameworks use interfaces for injectable dependencies and abstract classes for extension points with common lifecycle hooks. Too many inheritance hooks can make framework behaviour hard to follow.

## Important Properties

| Property | Interface | Abstract class |
|---|---|---|
| Multiple implementation/inheritance | Often yes in Java interfaces | Usually one class base |
| Instance state | Usually no/limited contract constants | Yes |
| Shared implementation | Defaults/helper methods | Natural |
| Main intent | Capability contract | Shared base behaviour |

### Common Misconceptions

* Interface is not only for “multiple inheritance”; it expresses client-needed capability.
* Abstract class can contain concrete methods and state.
* Do not create interface for every tiny class without a substitution/test boundary.

### What Interviewers Usually Test

Choice rationale, dependency inversion, testing with fakes, and precise language-specific capabilities.

### Common Follow-up Questions

1. Interface versus abstract class?
2. Can abstract class have constructor?
3. Why use interface in tests?
4. What is template method?
5. Can interface have implementation?

### Follow-up Answers

1. Interface primarily expresses capability contract; abstract class shares base state and implementation within a hierarchy.
2. Yes. Its constructor initializes base state when a concrete subclass is created.
3. A test can provide a fake/stub implementation without real database, clock, network, or vendor dependency.
4. Base class defines a fixed algorithm sequence and subclasses override selected steps.
5. In Java, default/static/private methods can have implementation, but interface remains a contract-first tool.
