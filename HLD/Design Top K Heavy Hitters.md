Design top K heavy hitter for products like youtube, shorts, music etc. 

**Functional Requirement:**
1. getTopK(k, time_window)
	1. k <= 1000
	2. last 1 min, last 5 mins, last 60 mins, last 1 day, last 7 days, last 30 days
2. Record Events

**Non Functional Requirements:**
1. Practical Views on these platforms are > 1B unique videos, this translates to **~100 QPS.**
2. Our system should be highly scalable to handle these many events
3. High throughput system
4. Highly available - [[Fours 9's of Availability]] 
5. Low Latency Reads (< 100 ms) - getTopK videos. Especially for smaller durations like 1 min, 5 mins, 60 mins. 
6. For Reads of > 1 Day, `Low Latency cannot be there as the scale is huge, can be discussed in design decisions.` 
7. Accuracy of getTopK for smaller/recent duration
	1. Eventual consistency - approximation vs strong

----

### How we are maintaining the top K elements? 

1. Using Hash-map & Min Heap
	 - Maintain a hashmap -> {eventId, count}
	 - Maintain a min-heap of size k


