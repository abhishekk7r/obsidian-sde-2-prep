---
tags: [interview, amex, checklist]
interview: 2026-10-04
---

# Amex SDE2 prep checklist

> [!note] Rules
> Tick a topic only after you answer it correctly from a cold start.
> Tier 1 = actually asked at Amex (from 12 reports). Tier 2 = your weak spots. Do Tier 1 first.

## Tier 1: asked at Amex

### Java
- [ ] T1.1 HashMap internals (notes done, cold re-test pending)
- [ ] T1.2 Multithreading: thread creation, lifecycle, deadlock, race conditions
- [ ] T1.3 Java memory model and collections (ArrayList vs array, fail-fast, immutable lists)
- [ ] T1.4 Streams coding and functional interfaces (Supplier, Predicate)
- [ ] T1.5 Java 8 features and abstract class vs interface
- [ ] T1.6 Exceptions: throw vs throws, handling
- [ ] T1.7 Thread-safe Singleton and design patterns, SOLID

### Spring Boot
- [ ] T1.8 @Bean vs @Component and stereotype annotations
- [ ] T1.9 REST controller: @PathVariable vs @RequestParam, build a /api/v1/employee endpoint
- [ ] T1.10 Actuator
- [ ] T1.11 JPA: persist vs save
- [ ] T1.12 Microservices vs monolith
- [ ] T1.13 JUnit and Mockito

### Kafka and design
- [ ] T1.14 Kafka: topics, partitions, consumer groups
- [ ] T1.15 Payment gateway design: retries, failure handling, idempotency
- [ ] T1.16 Behavioral: project deep dive, disagreement, deadline, failure

### DSA
- [ ] T1.17 Stack: celebrity problem
- [ ] T1.18 Sliding window: Fruit Into Baskets, longest substring without repeating
- [ ] T1.19 Backtracking: Combination Sum
- [ ] T1.20 3Sum / unique triplets
- [ ] T1.21 Merge sort

## Tier 2: your weak spots
- [ ] T2.1 synchronized reentrancy and ConcurrentHashMap locking
- [ ] T2.2 Static initialization order, records, switch patterns
- [ ] T2.3 Serialization
- [ ] T2.4 Bean lifecycle, @Transactional traps
- [ ] T2.5 JPA N+1 and lazy loading
- [ ] T2.6 GC basics and memory leaks
- [ ] T2.7 Spring Security and JWT
- [ ] T2.8 Final miss-list re-test
