### Intro (aim for 30-45 sec spoken)

> Hi, I'm Abhishek. I'm a backend engineer with about 3 years of experience. I started my career at TCS, where I worked on TCS BaNCS — a Java and Spring Boot backend for SBI General Insurance. I've been at Amazon for the last year, on the Appstore and Fire TV org.
> 
> Over this last year, my focus has shifted from feature development to systems engineering — specifically AWS infrastructure and traffic engineering for Amazon Appstore. We're making our services **region-flexible** — the ability to run any service in any region, on any compute — and I currently own how we move live production traffic safely between AWS regions.

**Note:** this is punchier and interview-ready, but double check "own how we move traffic" isn't overclaiming beyond your actual scope — Round 3 (managerial) may probe ownership boundaries specifically.

### Product Explanation

> This is part of a company-wide region-flexibility initiative. A lot of our services are concentrated in a single AWS region, which now has capacity constraints — it can't scale to absorb peak events like Diwali, Christmas, or Black Friday. Our job is to migrate these services, and their live traffic, to a new region.
> 
> What makes this a genuine engineering problem rather than just an ops migration comes down to three things:
>-  How do you ensure the set of connected services do not encure latency during?
>-  Second would be Appstore runs on physical devices so it's not like moving a backend stateless service. So,  so we typically push the end points of the new regions where the OTA (Over the Air), in a similar way that you get updates on your phone. 
>- We have a config service, which devices poll in every ~6hr to get updated endpoints/url.
>- The problem with the config services: since it takes six hours to get the new URLs, let's say we want to rollback. It would again take at least six hours to get the updated endpoints 
>- We had to figure out a way to 
>
> **First, rollback difficulty.** Appstore runs on physical devices, so this isn't like moving a stateless backend service. We typically push endpoint updates to devices via over-the-air config — devices poll a config service and pick up new endpoints. But that propagation takes 6 to 12 hours, sometimes up to 24. So if we've already pushed new endpoints to devices and something goes wrong, we can't just instantly roll back — customers would face a real outage. We had to design a separate rollback path that doesn't depend on that slow device-side propagation.
> 
> **Second, partial-state correctness.** Our data stores live in one region. If a customer writes data there and then a later request routes to a different region, that data has to already be there — otherwise they hit a broken or stale state. So we have to guarantee cross-region data replication and consistency at the state boundary. That's a much harder problem than just moving compute — compute is stateless, data isn't.
> 
> **Third, region readiness validation.** You can't shift traffic to a new region on faith — you need continuous, active proof that the new region is actually correct: serving the right resources, handling real requests properly, before you trust it with production traffic.