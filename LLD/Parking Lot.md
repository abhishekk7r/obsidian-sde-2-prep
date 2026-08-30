# Problem Statement (as an interviewer would give it)

Design a parking lot system.

## Requirements

1. A vehicle enters through a single entry gate and is issued a **Ticket**.
2. The ticket contains: spot number, entry time, and hourly rate (locked in at the time of entry).
3. Supported vehicle types: Motorcycle, Car, Bus — the design should be extensible to more types in the future.
4. The parking lot has a fixed spot pool per vehicle type: 50 motorcycle spots, 100 car spots, 50 bus spots (200 total).
5. Strict type matching: a vehicle can only occupy a spot of its own type — no overflow into another type's pool.
6. If no spot of the matching type is available, the vehicle is rejected at the entry gate.
7. On exit, the parking cost is calculated using the hourly rate locked on the ticket and the duration the vehicle was parked.

## Out of Scope

1. Multi-level parking.
2. Spot allocation *algorithm* — i.e., which specific empty spot within a type's pool gets assigned. (The *policy* of fixed pools per type is in scope; the assignment *strategy* is not.)
3. Payment collection.
4. Multiple exit gates.
5. Concurrency / thread-safety — single-threaded, single-gate assumption for this design.

---

## Entities & Relationships

1. Parking Spot
2. Parking Spot Manager
3. Vehicle
4. Entry/Exit Class
5. 