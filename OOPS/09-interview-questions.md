# 09. Most Asked OOP Interview Questions and Simple Answers

## How to use this file

Answer in this order: **definition -> why -> example -> trade-off**. Keep each answer between 30 and 60 seconds. After speaking, stop and let the interviewer ask a follow-up.

**Hinglish:** Definition aur example bolo, but trade-off ke bina answer adhoora lagta hai. Ratta lagane ke bajaye real example do.

---

## Section A: Basics

### 1. What is OOP?

OOP organizes code around objects that hold state and provide behaviour. It helps us protect rules, split responsibilities, and change a system in a controlled way.

**Hinglish:** OOP code ko objects ke around organize karta hai jismein data aur uska behaviour hota hai.

### 2. What is the difference between a class and an object?

A class is a blueprint or definition. An object is a real instance created from it with its own state.

**Hinglish:** Class blueprint hai; object uska actual instance hai.

### 3. What are state and behaviour?

State is the data an object currently holds. Behaviour is what the object can do with that data.

**Hinglish:** State object ka current data hai; behaviour us data par chalne wala action hai.

### 4. What is encapsulation?

Encapsulation keeps state and the methods that protect it together. It prevents callers from putting an object into an invalid state.

**Hinglish:** Data aur uske rules ko ek jagah bundle karke invalid changes rokta hai.

### 5. How is encapsulation implemented?

Use private state and small public methods that validate and perform allowed changes. A setter is not useful if it allows every invalid value.

**Hinglish:** Fields private aur public methods validated rakhna best hai.

### 6. What is abstraction?

Abstraction shows the essential contract and hides implementation details that the client does not need.

**Hinglish:** Client ko simple interface dikhao, complicated implementation chhupa do.

### 7. What is the difference between encapsulation and abstraction?

Encapsulation protects data and rules. Abstraction hides complexity and exposes a simple capability.

**Hinglish:** Encapsulation data protection hai; abstraction implementation hiding hai.

### 8. What is inheritance?

Inheritance lets a new type derive behaviour and data from an existing type. Use it for a genuine `is-a` relationship.

**Hinglish:** Inheritance parent se child type banata hai, but sirf code copy karne ke liye use na ho.

### 9. What is the difference between `is-a` and `has-a`?

`Is-a` means the child is a specialized version of the parent. `Has-a` means an object contains or uses another object.

**Hinglish:** `Is-a` specialization hai; `has-a` parts ya collaborators ka relationship hai.

### 10. Why do many designs prefer composition over inheritance?

Composition keeps parts replaceable and avoids a fragile deep hierarchy. It also makes small dependencies easy to fake in tests.

**Hinglish:** Composition flexible reuse aur testing ke liye safer hai.

### 11. What is polymorphism?

Polymorphism lets one contract work with different implementations. A caller does not need a long `if-else` for every concrete type.

**Hinglish:** Ek interface ke through alag objects ka different behaviour call karna polymorphism hai.

### 12. What is the difference between compile-time and runtime polymorphism?

Compile-time polymorphism, such as Java method overloading, is selected by the compiler. Runtime polymorphism, such as overriding, is selected using the actual object.

**Hinglish:** Compile-time polymorphism compiler decide karta hai; runtime polymorphism actual object decide karta hai.

### 13. What is overloading versus overriding?

Overloading uses the same method name with different parameter signatures. Overriding replaces an inherited implementation in a child class.

**Hinglish:** Overloading mein signature badalti hai; overriding mein child implementation change karta hai.

### 14. What is an abstract class?

An abstract class is declared with the `abstract` modifier and cannot be instantiated directly. It may contain shared state, constructors, and concrete methods, and it does not have to declare any abstract methods.

**Hinglish:** Abstract class common base aur shared code ke liye hai.

### 15. What is an interface?

An interface defines a capability contract. A class promises to provide the required operations without the interface prescribing the internal implementation.

**Hinglish:** Interface batata hai “yeh kaam kar sakta hai”, implementation chhupata hai.

### 16. When should you use an abstract class instead of an interface?

Use an abstract class when closely related subclasses need shared state, constructors, or a stable algorithm. Use an interface for independent capabilities.

**Hinglish:** Shared state/algorithm chahiye to abstract class; capability contract chahiye to interface.

### 17. What is a constructor?

A constructor creates or initializes a new object. It should establish a valid initial state and reject invalid input early.

**Hinglish:** Constructor object banata hai aur uski shuruaati validity check karta hai.

### 18. What is `this` in Java and `self` in Python?

Both refer to the current instance. Java supplies `this` implicitly; Python passes `self` explicitly as the first instance-method parameter.

**Hinglish:** Dono current object ko point karte hain; Python mein `self` explicitly pass hota hai.

### 19. What is the difference between a static method and an instance method?

An instance method uses the current object’s state. A static method does not need an object instance and is usually a helper related to the class.

**Hinglish:** Instance method object state use karta hai; static method object ke bina helper hai.

### 20. What is `final` in Java, and what is a constant?

In Java, `final` prevents reassignment of a variable, overriding of a method, and inheritance of a class. A constant is conventionally `static final` with an immutable value. Python's `typing.final` is mainly a type-checker signal, not runtime enforcement.

**Hinglish:** Java `final` location ke hisaab se reassignment, overriding, ya inheritance rokta hai; constant `static final` hota hai. Python `typing.final` mostly type checker ke liye hai.

### 21. What is coupling and cohesion?

Coupling is how much one component depends on another. Cohesion is how well a component’s responsibilities belong together. We usually want low coupling and high cohesion.

**Hinglish:** Kam coupling aur zyada cohesion maintainable code dete hain.

---

## Section B: Relationships, SOLID, and patterns

### 22. What is association?

Association is a general collaboration between objects where neither necessarily owns the other object’s lifecycle.

**Hinglish:** Association do independent objects ka use/connection hai.

### 23. What is aggregation?

Aggregation is a weak whole-part relationship. The part can survive without the whole.

**Hinglish:** Aggregation mein part independently reh sakta hai.

### 24. What is composition?

Composition is a strong whole-part relationship where the parent manages the child’s lifecycle.

**Hinglish:** Composition mein parent part ka owner hota hai.

### 25. What is Dependency Inversion?

High-level business code depends on an abstraction it owns, not directly on a low-level concrete implementation.

**Hinglish:** Business code ko contract chahiye, database/provider ki concrete class nahi.

### 26. What is the difference between DIP and Dependency Injection?

DIP is the design principle. Dependency Injection is the technique of supplying the dependency from outside, usually through a constructor.

**Hinglish:** DIP principle hai; DI us principle ko implement karne ka tarika hai.

### 27. What is the SRP?

Single Responsibility Principle says a class should have one main reason to change or one cohesive business responsibility.

**Hinglish:** Ek class ka ek clear kaam hona chahiye; “ek method” hona zaroori nahi.

### 28. What is the OCP?

Open/Closed Principle means new variants should be added through new implementations and abstractions instead of repeatedly editing the main workflow.

**Hinglish:** Naya behaviour add karne ke liye purane core flow ko naye branches se na todlo.

### 29. What is the LSP?

Liskov Substitution Principle says a child must be safely usable wherever the parent is expected, including its behaviour and contract.

**Hinglish:** Child parent ki jagah safely fit hona chahiye.

### 30. What is the ISP?

Interface Segregation Principle says clients should depend on small interfaces with only the methods they use.

**Hinglish:** Bada interface sabko force mat karo; focused interfaces banao.

### 31. What is the Strategy pattern?

Strategy defines a family of interchangeable algorithms behind one contract. A client can select or replace the algorithm.

**Hinglish:** Strategy ek operation ke algorithms ko interchangeable banata hai.

### 32. What is the Factory pattern?

Factory centralizes creation of objects so the client does not need to know every concrete construction detail.

**Hinglish:** Factory object banane ki details client se chhupata hai.

### 33. What is the Observer pattern?

Observer defines a one-to-many notification relationship between an event source and listeners.

**Hinglish:** Ek event change ho to multiple listeners notify hote hain.

### 34. What is the Decorator pattern?

Decorator wraps an object to add behaviour while keeping the original interface.

**Hinglish:** Decorator extra behaviour layering karta hai bina original class badle.

### 35. What is the Adapter pattern?

Adapter converts one interface into another interface that a client expects.

**Hinglish:** Adapter do incompatible interfaces ko connect karta hai.

### 36. What is the Singleton pattern?

Singleton restricts a class to one instance. It can be useful for a truly global resource, but global state often makes tests and configuration harder.

**Hinglish:** Singleton ek hi instance deta hai, but global state testing ko complicate kar sakta hai.

### 37. What is the Template Method pattern?

A base class defines a stable algorithm sequence while subclasses provide selected steps.

**Hinglish:** Base flow fixed hota hai, subclasses kuch steps change karte hain.

### 38. What is a code smell?

A code smell is a warning sign that a design may be hard to understand, change, or test. It is not automatically a bug.

**Hinglish:** Smell future problem ka signal hai, proof nahi.

### 39. When should you not use SOLID or a pattern?

Do not add layers to a tiny script or use a complex pattern when a simple function is clearer. The benefit must be real.

**Hinglish:** Chhote problem ko complex architecture se mat banaao.

---

## Section C: Exceptions, memory, collections, and concurrency

### 40. What is an exception?

An exception is a typed object representing an abnormal or exceptional situation that interrupts normal control flow.

**Hinglish:** Exception failure ka structured representation hai.

### 41. Why is an exception better than an error code?

It carries a type, message, stack trace, and context. It also usually interrupts the current flow and is harder to ignore than a numeric code, although a careless handler can still swallow it.

**Hinglish:** Exception detail aur stack trace deta hai; error code aksar ignore ho jata hai.

### 42. What is the difference between checked and unchecked exceptions in Java?

Checked exceptions require the compiler to see handling or declaration. Unchecked exceptions are not required to be caught or declared; they include `RuntimeException` and `Error`.

**Hinglish:** Checked compile-time handle karna maangta hai; unchecked ko mandatory handling nahi maangti.

### 43. Why should we avoid catching `Exception` everywhere?

A broad catch hides programming errors and makes recovery unclear. Catch the narrowest exception you can actually handle.

**Hinglish:** Generic catch bugs chhupa deta hai; specific error pakdo.

### 44. What is exception chaining?

It keeps the original low-level error as the cause of a higher-level error so the root cause is not lost.

**Hinglish:** Root cause preserve karna debugging easy banata hai.

### 45. What is `finally`?

It runs cleanup code during normal completion and while an exception is unwinding. For resources, prefer `try-with-resources` in Java and `with` in Python.

**Hinglish:** `finally` cleanup ke liye hai, but resource code context manager se likhna better hai.

### 46. What is a resource leak?

A program keeps a file, socket, database connection, or other resource open longer than needed and may run out of available resources.

**Hinglish:** Resource leak ko close na karna kehte hain.

### 47. What is garbage collection?

Automatic reclamation of memory for unreachable objects. It is not a guarantee that business resources or transactions close immediately.

**Hinglish:** GC memory manage karta hai, business resource cleanup ki guarantee nahi deta.

### 48. What is a memory leak?

A program keeps reachable references to data it no longer needs, so the runtime cannot reclaim that memory.

**Hinglish:** Leak ka matlab hai unwanted object reference taki memory na free ho.

### 49. What is the simple stack-versus-heap answer?

Local call information is often associated with the stack, and objects are commonly allocated on the heap. Exact behaviour is runtime-dependent, so avoid presenting this as a universal rule.

**Hinglish:** Interview ka simple model yaad rakho, but implementation ko language/runtime decide karta hai.

### 50. When should we use a list, set, or map?

Use a list for order and duplicates, a set for unique values, and a map for key-value lookup.

**Hinglish:** List order, Set uniqueness, Map key-value lookup ke liye.

### 51. What is the average complexity of a hash map lookup?

Usually O(1) on average with a good hash distribution. Worst-case behaviour can be worse under collisions or poor load factors.

**Hinglish:** Hash map average O(1) hota hai, but worst case slow ho sakta hai.

### 52. What is the equality and hash-code contract?

If two objects are equal, they must have the same hash value. Hash-based collections use the hash first and equality second.

**Hinglish:** Equal objects ka hash same hona chahiye, warna hash-based lookup fail hota hai.

### 53. Why can mutating a hash-map key be a problem?

Changing a field used by the key’s hash or equality can move the key to another logical bucket, so the collection may not find it with the original key. Mutability alone is not the problem if the key uses only stable identity.

**Hinglish:** Hash ya equality mein use hone wala field change mat karo; value-based immutable key best hai.

### 54. What is the difference between a shallow copy and a deep copy?

A shallow copy shares nested references. A deep copy also creates the nested objects.

**Hinglish:** Shallow copy mein nested objects shared rehte hain; deep copy unko independently banata hai.

### 55. Why is deserializing untrusted data dangerous?

Untrusted formats can create unexpected objects or execute unsafe behaviour. Validate the format and allowed classes.

**Hinglish:** Untrusted data deserialize mat karo; format aur classes validate karo.

### 56. What is a race condition?

It is a bug where the result depends on unpredictable timing between operations on shared state.

**Hinglish:** Shared state par timing ke hisaab se result badalna race condition hai.

### 57. What is a mutex or lock?

It is a synchronization mechanism that allows only a controlled set of tasks to enter a critical section at one time.

**Hinglish:** Lock critical section ko one-at-a-time access karwata hai.

### 58. What is a deadlock?

It is a situation where two or more tasks wait for resources held by one another, so none can progress.

**Hinglish:** Do tasks ek doosre ka lock wait karein to deadlock hota hai.

### 59. Does `volatile` make a counter thread-safe in Java?

No. It helps visibility and ordering, but a read-modify-write sequence such as `count++` is still compound and needs atomic or synchronized handling.

**Hinglish:** `volatile` visibility deta hai, but `count++` ko automatically atomic nahi banata.

---

## Section D: Java and Python language questions

### 60. What is a Java record?

It is a compact class for data-carrying values. The record generates accessors and common value methods, but its immutability is shallow.

**Hinglish:** Record data carrier hai; nested mutable objects ko protect karna ab bhi zaroori hai.

### 61. What is a sealed interface in Java?

It restricts the classes or interfaces that may implement it, making a known set of domain variants explicit.

**Hinglish:** Sealed interface allowed implementations ko limit karta hai.

### 62. What is `Optional` for in Java?

It represents a possibly absent return value in a clear API. It should mainly be used as a return type, not as a field or collection wrapper.

**Hinglish:** `Optional` missing return value ko clear karta hai, har jagah use nahi karna chahiye.

### 63. Why does Java not allow multiple class inheritance?

To avoid ambiguous state and method conflicts. A class can still implement multiple interfaces.

**Hinglish:** Ambiguity avoid karne ke liye Java multiple class inheritance restrict karta hai.

### 64. What is duck typing in Python?

Duck typing uses an object based on the methods it provides, without requiring inheritance from a specific class.

**Hinglish:** Required methods hain to object fit hai; explicit parent ki zaroorat nahi.

### 65. What is the difference between an ABC and a Protocol in Python?

An ABC uses explicit nominal inheritance. A Protocol uses structural compatibility, so any object with the required methods can satisfy it.

**Hinglish:** ABC explicit hai; Protocol duck typing ko static tools ke saath support karta hai.

### 66. What is the difference between `__new__` and `__init__`?

`__new__` creates the object and `__init__` initializes the created object. Most normal classes need only `__init__`.

**Hinglish:** Creation `__new__` karta hai; initialization `__init__` karta hai.

### 67. What is MRO in Python?

Method Resolution Order is the order Python searches classes for a method. Multiple inheritance uses C3 linearization to calculate it.

**Hinglish:** MRO method lookup order define karta hai.

### 68. What is a Python property?

A property controls attribute access while allowing validation or computed values.

**Hinglish:** Property attribute jaisa access deta hai but rules enforce karta hai.

### 69. Does the Python GIL make all Python code single-threaded?

No. It affects certain execution paths and runtimes. Use the right concurrency model for the workload and protect shared state explicitly.

**Hinglish:** GIL ko complete concurrency answer mat samjho; workload dekho.

---

## Section E: Scenario and LLD questions

### 70. Design a parking lot. What classes do you need?

I would clarify size, pricing, payment, and concurrency requirements first. Then I would model `ParkingLot`, `Spot`, `Vehicle`, `Ticket`, and a payment contract, define each state transition, and add rules preventing double assignment.

**Hinglish:** Requirements clarify karo, phir classes, states, aur invariants banao.

### 71. Design an LRU cache. What are the main concerns?

I would decide the capacity and missing-key behaviour, then use a map plus an ordered list or another recency structure. I would also clarify whether reads update recency and whether the cache is shared between threads.

**Hinglish:** Capacity, missing key, recency, aur thread safety poochho.

### 72. How do you test a service without a real payment gateway?

I would define a small gateway contract and inject a fake implementation that records calls and returns a controlled result. The business logic can then be tested without network calls.

**Hinglish:** Real gateway ki jagah fake inject karke service test karo.

### 73. How do you decide whether to use inheritance or composition?

I ask whether the relationship is truly `is-a` and whether the child can honour the parent’s full contract. If the need is only reuse, replaceable parts, or a collaborator, I usually use composition or an injected dependency. Collaboration alone does not always mean composition.

**Hinglish:** `is-a` clear nahi hai to composition ya injected dependency choose karo.

### 74. How would you design an order lifecycle?

I would define explicit states such as `CREATED`, `PAID`, `SHIPPED`, and `CANCELLED`, then allow only valid transitions through methods. Invalid transitions should fail clearly without corrupting the order.

**Hinglish:** Order ka state machine banao; random state changes band karo.

### 75. What makes an OOP design good?

It has clear responsibilities, small contracts, controlled state, replaceable dependencies, readable code, and tests that describe behaviour instead of implementation details.

**Hinglish:** Good design clear responsibility, safe state, aur testable behaviour rakhta hai.

---

## Final revision checklist

- Answer every question with one example and one trade-off.
- Say “not every case” instead of claiming a pattern solves everything.
- Correctly explain `equals` and `hashCode` together.
- Do not claim `volatile` makes `count++` atomic.
- Do not claim garbage collection closes files or sockets.
- Do not claim Python name mangling is security.
- In LLD, discuss failure cases and state transitions, not only class names.
- Keep the answer simple and stop after the main point. A follow-up invitation is better than a long monologue.
