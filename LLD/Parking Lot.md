# Problem Statement (as an interviewer would give it)

Design a parking lot system.

The parking lot has multiple floors, each with a number of parking spots. Spots differ in size/type (e.g. motorcycle, compact/car, large/bus). Vehicles of different types arrive and need to be assigned an available, appropriately-sized spot. When a vehicle leaves, the system should calculate the parking fee based on duration parked, using a rate structure that may vary by vehicle type.

Design the core classes and their relationships (not full production code) to support this. Be ready to discuss:
- How you'd extend it to support additional vehicle types or spot types
- What happens when the lot (or a floor) is full
- How you'd choose which spot to assign to an incoming vehicle

(Deliberately open-ended, the way it'd typically be given verbally — the rest gets clarified through your own requirements-gathering questions, not handed to you as a spec.)

### Requirements
1. The Vehicle should be able to from the entry gate.
2. Upon Entry, Each Vehicle should be provided with a TICKET.
3. A ticket would contain the parking