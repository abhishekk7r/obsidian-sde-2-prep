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

# Entities & Relationships
**Entities**

- `ParkingLot` — orchestrator; owns `ParkingSpotManager`, `RateCard`, `EntryGate`, `ExitGate`
- `ParkingSpotManager` — owns the pool of `ParkingSpot`s; find/allocate a free spot of a given type, free a spot. Knows nothing about tickets, vehicles, or rates.
- `ParkingSpot` — id, type, status (`FREE` / `OCCUPIED`)
- `Vehicle` (interface) → `Motorcycle`, `Car`, `Bus`
- `Ticket` — id, vehicle reference, spot reference, entryTime, hourlyRate (locked at issue time)
- `EntryGate` — vehicle in → issues `Ticket`, or rejects if no matching spot
- `ExitGate` — ticket in → calculates cost, frees the spot
- `RateCard` — `VehicleType → hourlyRate` lookup (not a Strategy-pattern hierarchy: the cost formula `hourlyRate × hoursParked` is identical across vehicle types, only the rate value varies, so a single lookup class is the right level of complexity for now. Revisit as Strategy if a future requirement makes the formula itself diverge by type — e.g. flat daily rate for buses.)

**Relationships**

- Parking lot has a ParkingSpotManager
- ParkingLot has a RateCard
- ParkingLot has a EntryGate & Exit Gate (Singelton)
- ParkingLot has a Rate Card - static map 
- ParkingLotManager has many parkingSpots
- Rate Card has many vehicle type with hourly rate. 

> [!note] Why not a Strategy pattern for rates?
> Strategy earns its place when the *algorithm* varies by type, not just a *value*. Here the formula is the same for every vehicle — only the number differs — so a plain lookup (`RateCard`) is the right-sized design. Forcing Strategy now would be solving a problem that doesn't exist yet.
