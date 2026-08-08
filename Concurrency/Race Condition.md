A race condition occurs when two or more processes or threads access and modify the same data at the same time

**For example:** If two people update the same bank account simultaneously without checking each other’s changes, the final balance may be wrong.

# **Causes of Race Conditions**

- **Simultaneous Access:** When two or more processes try to read or write the same shared resource at the same time.
- **Non-Atomic Updates:** Operations like increment or decrement are not indivisible.
- **Lack of Synchronization**: No mechanisms like locks, semaphores, or monitors are used to control access.
- **Improper Scheduling:** OS scheduler interrupts processes at critical moments.

# **Prevention Techniques**

1. **Mutex (Mutual Exclusion)**: Ensure only one process can enter the critical section at a time.
2. **Semaphores**: Counting or binary semaphores control access to resources.
3. **Monitors**: High-level synchronization constructs that manage shared resources.
4. **Atomic Operations**: Use hardware or software-supported atomic instructions.
![[Pasted image 20260808195437.png]]
