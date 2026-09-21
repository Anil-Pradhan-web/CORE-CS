# Real-World OOP Scenarios and Low-Level Design (LLD)

## What is it?

Low-Level Design (LLD) translates system requirements into concrete object-oriented blueprints: classes, interfaces, relationships, state transitions, and design patterns. LLD interviews evaluate your ability to produce clean, extensible, bug-free domain models under ambiguity.

## Simple Explanation

If High-Level Design (HLD) is drawing the city map (where the roads, power stations, and water reservoirs go), Low-Level Design (LLD) is architecting the individual house (where the load-bearing walls, plumbing pipes, electrical sockets, and doors are placed).

## Hinglish Explanation

LLD interviews mein interviewer aapko ek scenario deta hai jaise "Design a Parking Lot" ya "Design an Elevator System". Yahan goal database ya network setup karna nahi hota, balki Core OOP concepts use karke clean classes, interfaces, design patterns, aur relationships model karna hota hai. Key rule yeh hai ki code SOLID follow kare, extensible ho, aur concurrency aur edge cases ko handle kare.

## Why Do We Need It?

Without structured LLD:
1. Developers write massive procedural functions with 50 `if-else` branches that break whenever a new requirement arrives.
2. Responsibilities leak across modules (e.g., ticket generation logic mixed with parking spot allocation).
3. The system cannot be extended without rewriting core business workflows.

## How It Works: The 5-Step LLD Interview Framework

```text
Step 1: Clarify Scope & Requirements (Functional & Non-functional)
   │
Step 2: Identify Core Entities & Value Objects (Nouns in requirements)
   │
Step 3: Define Relationships & Class Diagram (Has-a, Is-a, Uses-a)
   │
Step 4: Apply Appropriate Design Patterns (Strategy, State, Factory, Observer)
   │
Step 5: Write Clean, Extensible Code (Handle edge cases & concurrency)
```

---

## Scenario 1: Parking Lot System (Java)

### Core Requirements
- Multiple vehicle types (Car, Bike, Truck).
- Multiple spot types (Compact, Large, Motorcycle).
- Dynamic parking fee strategy (Hourly vs. Flat).
- Ticket generation on entry, payment & spot release on exit.

```java
import java.time.Duration;
import java.time.LocalDateTime;
import java.util.List;

// 1. Vehicle Hierarchy
enum VehicleType { MOTORCYCLE, CAR, TRUCK }

abstract class Vehicle {
    private final String licensePlate;
    private final VehicleType type;
    public Vehicle(String plate, VehicleType type) { this.licensePlate = plate; this.type = type; }
    public VehicleType getType() { return type; }
}

// 2. Parking Spot
enum SpotType { MOTORCYCLE, COMPACT, LARGE }

class ParkingSpot {
    private final int spotId;
    private final SpotType spotType;
    private boolean isOccupied = false;
    private Vehicle currentVehicle;

    public ParkingSpot(int id, SpotType type) { this.spotId = id; this.spotType = type; }
    public synchronized boolean park(Vehicle v) {
        if (isOccupied) return false;
        this.currentVehicle = v;
        this.isOccupied = true;
        return true;
    }
    public synchronized void vacate() { this.currentVehicle = null; this.isOccupied = false; }
    public boolean isAvailable() { return !isOccupied; }
    public SpotType getSpotType() { return spotType; }
}

// 3. Strategy Pattern for Pricing
interface FeeStrategy {
    double calculateFee(Duration duration, VehicleType type);
}

class HourlyFeeStrategy implements FeeStrategy {
    public double calculateFee(Duration duration, VehicleType type) {
        long hours = Math.max(1, duration.toHours());
        return switch (type) {
            case MOTORCYCLE -> hours * 10.0;
            case CAR -> hours * 20.0;
            case TRUCK -> hours * 50.0;
        };
    }
}

// 4. Ticket
class ParkingTicket {
    private final String ticketId;
    private final LocalDateTime entryTime;
    private final ParkingSpot spot;
    private final Vehicle vehicle;

    public ParkingTicket(String id, ParkingSpot spot, Vehicle vehicle) {
        this.ticketId = id;
        this.spot = spot;
        this.vehicle = vehicle;
        this.entryTime = LocalDateTime.now();
    }
    public LocalDateTime getEntryTime() { return entryTime; }
    public ParkingSpot getSpot() { return spot; }
    public Vehicle getVehicle() { return vehicle; }
}
```

---

## Scenario 2: Elevator System (Python)

### Core Requirements
- Multiple elevator cars.
- Internal requests (user presses floor button inside car) and External requests (user calls elevator from floor hall).
- State transitions (IDLE, MOVING_UP, MOVING_DOWN).

```python
from enum import Enum
import heapq

class Direction(Enum):
    UP = 1
    DOWN = -1
    IDLE = 0

class ElevatorCar:
    def __init__(self, car_id: int):
        self.car_id = car_id
        self.current_floor = 0
        self.direction = Direction.IDLE
        self.up_stops = []    # Min-heap for floors above
        self.down_stops = []  # Max-heap (inverted) for floors below

    def add_request(self, floor: int):
        if floor > self.current_floor:
            heapq.heappush(self.up_stops, floor)
            if self.direction == Direction.IDLE:
                self.direction = Direction.UP
        elif floor < self.current_floor:
            heapq.heappush(self.down_stops, -floor) # Inverted for max-heap
            if self.direction == Direction.IDLE:
                self.direction = Direction.DOWN

    def step(self):
        if self.direction == Direction.UP:
            if self.up_stops:
                self.current_floor = heapq.heappop(self.up_stops)
                print(f"Car {self.car_id} reached floor {self.current_floor}")
            else:
                self.direction = Direction.DOWN if self.down_stops else Direction.IDLE
        elif self.direction == Direction.DOWN:
            if self.down_stops:
                self.current_floor = -heapq.heappop(self.down_stops)
                print(f"Car {self.car_id} reached floor {self.current_floor}")
            else:
                self.direction = Direction.UP if self.up_stops else Direction.IDLE
```

---

## Scenario 3: LRU Cache (Python & Java)

### Core Design
- O(1) `get(key)` and O(1) `put(key, value)`.
- Hash Table (O(1) lookup) + Doubly Linked List (O(1) node removal and insertion at head).

```python
class Node:
    def __init__(self, key: int, val: int):
        self.key, self.val = key, val
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity: int):
        self.cap = capacity
        self.cache = {} # key -> Node
        # Dummy head and tail to eliminate null-checks
        self.head, self.tail = Node(0, 0), Node(0, 0)
        self.head.next = self.tail
        self.tail.prev = self.head

    def _remove(self, node: Node):
        node.prev.next = node.next
        node.next.prev = node.prev

    def _add_to_head(self, node: Node):
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node

    def get(self, key: int) -> int:
        if key in self.cache:
            node = self.cache[key]
            self._remove(node)
            self._add_to_head(node)
            return node.val
        return -1

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self._remove(self.cache[key])
        node = Node(key, value)
        self._add_to_head(node)
        self.cache[key] = node
        if len(self.cache) > self.cap:
            lru = self.tail.prev
            self._remove(lru)
            del self.cache[lru.key]
```

## Important Terminology

| Term | Meaning |
|---|---|
| **Aggregate Root** | The master entity that coordinates all state mutations across child entities |
| **State Pattern** | Models entity state transitions (e.g., Ticket: `ISSUED` -> `PAID` -> `EXPIRED`) |
| **Strategy Pattern** | Pluggable business rules (e.g., Pricing calculations, Spot allocation algorithms) |
| **Concurrency Guard** | Using mutex locks or thread-safe maps to prevent race conditions during booking/parking |

## Real-World Example

- **Amazon / Flipkart Order Checkout**: Involves a complex state machine (`OrderPlaced`, `PaymentAuthorized`, `FulfillmentStarted`, `OutForDelivery`, `Delivered`). Each state defines valid next transitions using the State Pattern.
- **Uber / Lyft Ride Matching**: `RideMatchingService` uses the Strategy Pattern to select matching algorithms (closest driver vs. highest-rated driver vs. batch auction) without altering the ride dispatch lifecycle.

## Important Properties

| Scenario | Primary Design Pattern | Key Concurrency Consideration |
|---|---|---|
| **Parking Lot** | Strategy (Pricing), Factory (Vehicle) | Locking individual parking spots to prevent double booking |
| **Elevator System** | State (Direction), PriorityQueue (Scheduling) | Thread-safe request dispatching across multiple elevator cars |
| **LRU Cache** | Doubly Linked List + HashMap | Mutex locking or ReadWriteLock for concurrent access |
| **Tic-Tac-Toe** | Board Game State, Rule Validator | Turn-based synchronization |

### Common Misconceptions

* LLD is not about writing 5,000 lines of code; it's about setting up clean interfaces, identifying entities, and demonstrating design patterns in 30-45 minutes.
* Do not jump into coding immediately in an interview; always spend the first 5-10 minutes gathering requirements and agreeing on scope.
* Never use a single monolithic "Manager" class that handles database, networking, validation, and printing; adhere strictly to Single Responsibility.

### What Interviewers Usually Test

1. Can you extract correct entities and value objects from conversational requirements?
2. Do you follow SOLID principles under pressure?
3. Can you identify and cleanly implement appropriate GoF design patterns?
4. How do you handle concurrency, race conditions, and edge cases?

### Common Follow-up Questions

1. How would you handle multiple entry and exit gates concurrently in a Parking Lot?
2. How would you support VIP or Reserved parking spots?
3. How does the Elevator algorithm handle starvation (e.g., a request at the top floor waiting indefinitely)?
4. Why use dummy head and tail nodes in the LRU Cache doubly linked list?
5. How would you scale an LLD design into a distributed microservice?

### Follow-up Answers

1. Guard spot reservation with synchronized methods, database row locks (`SELECT ... FOR UPDATE`), or atomic distributed locks (Redis Redlock), ensuring no two gates allocate the same spot simultaneously.
2. Add a `SpotCategory` enum (`REGULAR`, `VIP`, `EV_CHARGING`) to `ParkingSpot` and update the allocation strategy to filter spots matching the vehicle's credentials.
3. Use the LOOK or SCAN (Elevator) algorithm: the elevator completes all requests in the current travel direction to the furthest floor before reversing direction, preventing starvation.
4. Dummy head and tail nodes eliminate edge-case null checks when inserting the very first node or removing the very last node from the list.
5. Encapsulate the domain logic into an Aggregate Root, replace in-memory references with database IDs, use message queues (Kafka) for asynchronous state events, and expose operations via REST/gRPC endpoints.
