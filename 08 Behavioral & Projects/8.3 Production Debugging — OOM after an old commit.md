
**Question:**  
Application is showing OOM today. A commit was deployed 10 days ago, but there were no OOMs for the last 10 days. How would you diagnose it?

**Answer pattern:**

1. **Check application/service logs first**
    - Identify exact error, timestamp, stack trace and affected service.
    - Determine whether it's JVM OOM, container OOMKilled, or host-level memory exhaustion.
2. **Check metrics around the incident**
    - Heap/RSS memory
    - GC activity
    - CPU
    - Traffic/request volume
    - Container/host memory
3. **Look at the trend**
    - **Gradual memory increase** → possible memory leak / retained objects.
    - **Sudden spike** → traffic spike, large payload, batch job, configuration, dependency, etc.
4. **Correlate with changes**
    - Deployment/commit
    - Configuration
    - Traffic
    - Database/data growth
    - Scheduled jobs
    - Dependencies
5. **Deep dive if JVM heap issue**
    - Heap dump
    - GC logs
    - Identify objects consuming/retaining memory.
6. **Validate hypothesis**
    - Compare with previous version.
    - Controlled rollback/canary if safe.
    - Verify whether memory behavior returns to normal.

**Key takeaway:**

> **Don't assume the old commit is the cause. Start with logs → identify failure → check metrics/trend → correlate changes → investigate → validate.**