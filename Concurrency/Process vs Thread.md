### Process:

- A process is an **independent program in execution.** It has its own memory space, resources, file descriptors, and at least one thread.
- Processes are isolated.

Every process has it’s own:

```java
Memory
Heap
Stack
CPU Registers
Files
Network Connections
```

### Threads:

- A thread is the smallest unit of execution scheduled by the operating system.
- Thread share same memory inside a process.

```java
Spotify Process
Thread 1 → Play music
Thread 2 → Download songs
Thread 3 → Update UI
Thread 4 → Notifications
```

### How the memory is shared?

```java
												               Heap
												
												        +---------------+
												        | Counter       |
												        | count = 0     |
												        +---------------+
												
												          ▲           ▲
												          │           │
												
														   Thread A     Thread B
```

- Each thread has it’s own stack memory.
- Variables lives inside threads, and objects in memory.

## Interview Question:

1. What is shared between threads?
    - Heap
    - Objects
    - Static variables
2. What is not shared between threads?
    - Stacks
    - Local variables
    - Program Counter Register (stores memory address & next instructions)

## Thread Life Cycle

![image.png](attachment:fca3d1b3-c843-4fa4-9d2c-71f36e6faa5a:image.png)

```css
New -> Created
Runnable -> The thread is ready to run
Running -> CPU Starts executing it
Waiting -> The thread voluntarily waits
Blocked -> Thread becomes BLOCKED, waiting to acquire the monitor lock.
```