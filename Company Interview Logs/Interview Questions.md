Lenskart & Visa:
1. Java:
	1. Completable Future
		- What is completable future?
		- What are the production issues one would face?
    2. Java 21 Related Concepts:
	    - Records
	    - Atomic Class & Variables
	    - RentrantLocks 
	    - Synchronised
	    - Concurrent HashMap
	    - JVM Memory
	    - Thread pool 
	    - Thread Local
	    - Garbage Collection (tuning)
	    - Volatile Keyword
	    - Equals vs Hashcode
	    - Spring Boot Lifecycle
	    - Concurrency - Related Question
	    - Interface & Abstract Classes
	    - Volatile & Atomic Keyword
	    - Java 21 Realted Features(Heavy on this)
	3. Spring Boot
		- Transactional 
		- Controller 
		- RestController
		- Different Annotations
		- Exception handling without try-catch
		- RestAdviceController
		- N+1 problem
		- IOC/DI
___

1. Production & Debugging Scenarios:
	1. [[Production Debugging — OOM after an old commit]]
	2. How to trace and RCA (logging, metric, recent deployment, transient errors)
___

1. Data Structure & Algo
	- [[Max Flip to make Domino Numerator = Target Sum]]
	- Longest Consecutive Subsequnce
___

1. System Design
	 1. HLD
		- Design the notifcation system
		- Design the Vending Machine

___ 


Personal Notes:

1. Question regarding resume, especially on Java 21 
2. What are the most important problems that you have faced while working on it and how to explain RFE in such a way that it is a deep engineering project rather than just a migration?
3. Java backend work that I've done in my current company, Project Runway 

___

## Kotak Mahindra Bank — SDE (Bar Raiser Round)

### DSA
- **Longest Common Prefix** — standard string problem
- **Palindrome Number** — constraint: cannot convert to string, must solve with pure math (digit extraction via modulo/division)

### LLD
- **Shipment System** — design entities, relationships, DB schemas, working code
	- Implemented schema + entity + relationships but execution was not very smooth
	- Full question to be added (pending paste)

### Distributed Systems / HLD
- **Strong consistency under network partition** — how do you ensure strong consistency when a network partition failure occurs?
- **Double-booking prevention** — bookstore/seat-booking system: two contenders, one seat — how to guarantee the seat is not booked twice?
- **Payment system / BookMyShow click-by-click flow** — end-to-end internals: what happens at each click, what is the **idempotency key**, how does it work under the hood?

> [!note] Round character
> Bar raiser round — tested breadth across DSA + LLD + distributed systems in a single round. LLD expected working code with DB schema, not just class diagrams.

___

## Oracle Health (Cerner) — IC2 Associate Software Developer (Final Round)

### Round structure
- Final round (round 4 of 4) — "coding round around problem solving"
- **Implicit evaluation criteria** — interviewer did not explicitly ask about these, but judged:
	- Coding style & cleanliness
	- Edge case handling
	- Exception handling
	- Input handling
	- **Code modularity**
- Very few requirements given — candidate had to self-identify what to implement

### DSA — Tree
- **Print ancestors of a person in a family tree** — medium-level tree problem
	- Print father's side ancestors first, then mother's side
	- Essentially a path-finding / ancestor-traversal problem with ordering constraint

> [!tip] Takeaway
> Oracle IC2 final round tests engineering maturity more than algorithmic difficulty — clean code, modularity, and self-driven edge-case coverage matter more than the problem's complexity.
