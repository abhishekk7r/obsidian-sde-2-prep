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
4. Highly available - Fours 9's of Ava