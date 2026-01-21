# Availability
<img width="1024" height="1024" alt="Gemini_Generated_Image_v5udx9v5udx9v5ud" src="https://github.com/user-attachments/assets/e89db9a3-06d4-41d3-993d-d72da1d07a48" />

## What is this concept?

**Availability** is the percentage of time a system is operational and accessible to users. It measures how reliably your service stays up and running, even when things go wrong (hardware failures, network issues, bugs, traffic spikes).

In simple terms: Can users access your service when they need it?

Availability is usually expressed as "nines" — 99.9% (three nines), 99.99% (four nines), etc.

## Why does this exist? / Problem it solves

Without high availability:
- **Revenue loss**: E-commerce sites lose millions per hour of downtime
- **User trust damaged**: Users abandon apps that frequently crash or are slow
- **SLA violations**: You break contracts with enterprise customers
- **Competitive disadvantage**: Users switch to more reliable competitors
- **Brand reputation**: "That app is always down" spreads quickly

Real-world impact: Amazon found that every 100ms of latency costs them 1% in sales. When AWS goes down, thousands of services worldwide break.

## Real-World Analogy

**A 24/7 Hospital Emergency Room**

A hospital ER must be available 24/7 because lives depend on it. To ensure availability:
- **Redundancy**: Multiple doctors on shift (if one is sick, others cover)
- **Backup equipment**: Spare ventilators, generators for power outages
- **Failover**: If main power fails, backup generators kick in automatically
- **Monitoring**: Constant vital sign monitoring to detect issues early
- **Geographic distribution**: Multiple hospitals in different locations

Your system needs the same thinking—redundancy, backups, and automatic failover.

## How it works (High Level)

Availability is achieved through multiple strategies:

### 1. Eliminate Single Points of Failure (SPOF)
- Every component should have redundancy
- If one server fails, others take over

### 2. Replication
- Keep multiple copies of data and services
- Distribute across different servers, data centers, regions

### 3. Failover Mechanisms
- Automatic detection of failures
- Quick switching to backup systems
- Health checks and monitoring

### 4. Load Balancing
- Distribute traffic across multiple servers
- If one server dies, traffic routes to healthy ones

### 5. Graceful Degradation
- System continues operating with reduced functionality
- Non-critical features disabled during issues

**Flow:**
1. Monitor all components (servers, databases, networks)
2. Detect failure through health checks
3. Mark failed component as unavailable
4. Automatically route traffic to healthy components
5. Alert operations team
6. Fix/replace failed component
7. Restore to full capacity

## Simple Example

**Online Banking App**

**Scenario without High Availability:**
- Single database server
- Single API server
- Power outage → entire app down for 2 hours
- Availability: 99.77% (20 hours downtime/year)
- Users can't access money → panic and complaints

**Scenario with High Availability:**
- 3 API servers behind load balancer in different availability zones
- Primary database + 2 read replicas
- Auto-failover to replica if primary fails
- Multiple data centers (Mumbai, Bangalore)
- Backup power generators

**What happens when primary database fails:**
1. Health check detects database is down (within 30 seconds)
2. Automatic failover promotes replica to primary
3. Application reconnects to new primary
4. Users experience 30-second delay, not 2-hour outage
5. Availability: 99.95% (4 hours downtime/year)

## Code Snippet

```java
// Health check and failover logic (simplified)
class DatabaseFailover {
    Database primary;
    Database replica1;
    Database replica2;
    
    Database getActiveDatabase() {
        // Check primary health
        if (isHealthy(primary)) {
            return primary;
        }
        
        // Primary failed, try replica1
        if (isHealthy(replica1)) {
            promoteToPrimary(replica1);
            alertOps("Primary DB failed, replica1 promoted");
            return replica1;
        }
        
        // Replica1 also failed, try replica2
        if (isHealthy(replica2)) {
            promoteTorimary(replica2);
            alertOps("Primary and replica1 failed, replica2 promoted");
            return replica2;
        }
        
        // All failed - critical alert
        alertOps("ALL DATABASES DOWN - CRITICAL");
        throw new ServiceUnavailableException();
    }
    
    boolean isHealthy(Database db) {
        try {
            return db.ping() && db.getLatency() < 100; // ms
        } catch (Exception e) {
            return false;
        }
    }
}
```

## Where it is used in real systems

- **Netflix**: 99.99% availability through multi-region deployment, uses AWS across multiple availability zones
- **Gmail**: 99.978% availability (2 hours downtime/year) through massive replication
- **AWS S3**: 99.99% availability SLA, stores data across multiple availability zones
- **WhatsApp**: Uses Erlang for fault tolerance, automatic process recovery
- **Payment Gateways** (Stripe, Razorpay): 99.99%+ availability because downtime = lost revenue
- **Kubernetes**: Self-healing, automatically restarts failed pods
- **Load Balancers** (AWS ELB, Nginx): Route traffic away from unhealthy instances
- **Databases**: PostgreSQL streaming replication, MySQL master-slave setup
- **CDNs** (Cloudflare, Akamai): Distributed globally to ensure content availability

## Pros and Cons

### High Availability Design

**Pros:**
- Users can access service 24/7
- Better user experience and trust
- Meets enterprise SLA requirements
- Revenue protection (no downtime = no lost sales)
- Competitive advantage
- Automatic recovery from failures

**Cons:**
- Higher infrastructure costs (redundant servers, multiple regions)
- Increased complexity (failover logic, monitoring, orchestration)
- Data consistency challenges (CAP theorem - can't have perfect consistency and availability)
- More operational overhead (managing multiple components)
- Testing complexity (need to test failure scenarios)
- Potentially higher latency (due to replication)

## Common Mistakes / Misconceptions

1. **"99% availability is good enough"**
   - 99% = 3.65 days of downtime per year. For a payment app, that's disastrous. Understand what each "nine" means in actual downtime.

2. **"High availability = zero downtime"**
   - Even 99.999% (five nines) means 5 minutes of downtime per year. Perfect availability is impossible and infinitely expensive.

3. **"Just add more servers for availability"**
   - If all servers use the same database and it fails, adding API servers doesn't help. You need redundancy at EVERY layer.

4. **"Replication guarantees availability"**
   - Replication helps, but you also need automatic failover. Manual failover can take hours.

5. **"Availability = Performance"**
   - Wrong. A slow system can be highly available (always up but takes 10 seconds to respond). A fast system can have low availability (responds in 100ms but crashes frequently).

6. **"Active-Passive is the same as Active-Active"**
   - Active-Passive: One system handles traffic, backup is idle. Active-Active: Both systems handle traffic simultaneously. Active-Active is more efficient but more complex.

7. **"We need 99.999% availability"**
   - Each additional "nine" exponentially increases cost and complexity. Choose based on business needs, not arbitrary targets.

## Availability Numbers to Remember

| Availability % | Downtime/Year | Downtime/Month | Downtime/Day | Common Name |
|----------------|---------------|----------------|--------------|-------------|
| 90% | 36.5 days | 3 days | 2.4 hours | One nine |
| 99% | 3.65 days | 7.2 hours | 14.4 minutes | Two nines |
| 99.9% | 8.76 hours | 43.8 minutes | 1.44 minutes | Three nines |
| 99.99% | 52.6 minutes | 4.38 minutes | 8.64 seconds | Four nines |
| 99.999% | 5.26 minutes | 26.3 seconds | 0.86 seconds | Five nines |

---

## Interview Explanation (2-minute answer)

*"Availability measures how reliably a system stays operational and accessible to users. It's typically expressed as a percentage—like 99.9% or 99.99%—which translates to allowed downtime per year.*

*High availability is achieved through several key strategies. First, we eliminate single points of failure by adding redundancy at every layer—multiple servers, databases, and even data centers. Second, we implement automatic failover so when a component fails, the system detects it through health checks and immediately switches to a backup without human intervention.*

*For example, in a banking application, we'd have multiple API servers behind a load balancer. If one server crashes, the load balancer detects it within seconds and stops routing traffic there. The other servers continue handling requests, so users don't notice the failure. Similarly, for the database, we'd have a primary with read replicas, and automatic failover to promote a replica if the primary dies.*

*The challenge is balancing availability with cost and complexity. Each additional 'nine' exponentially increases infrastructure costs. We also face trade-offs from the CAP theorem—in distributed systems during network partitions, we must choose between consistency and availability. Payment systems typically choose consistency, while social media feeds often choose availability.*

*In practice, we determine the required availability based on business impact. An internal admin dashboard might be fine with 99% availability, but a payment gateway needs 99.99% or higher because downtime directly costs revenue."*

## Top Interview Questions

### 1. What's the difference between availability and reliability?
**Answer:** 
- **Availability**: Is the system up and accessible? (uptime percentage)
- **Reliability**: Does the system perform correctly without errors? (correctness of results)

A system can be highly available but unreliable (always up but returns wrong data). Example: A weather app that's always accessible but shows incorrect forecasts.

### 2. How do you calculate availability?
**Answer:**
```
Availability = (Total Time - Downtime) / Total Time × 100%

Example:
Total time in a year = 365 × 24 = 8,760 hours
Downtime = 8.76 hours
Availability = (8760 - 8.76) / 8760 × 100 = 99.9%
```

Also measured by: `Uptime / (Uptime + Downtime)`

### 3. Explain Active-Active vs Active-Passive failover
**Answer:**
- **Active-Passive**: Primary system handles all traffic, backup is on standby. When primary fails, backup takes over. Wastes backup resources but simpler to implement.
- **Active-Active**: Both systems handle traffic simultaneously. Better resource utilization, no failover delay, but requires sophisticated load distribution and data synchronization.

Example: Netflix uses Active-Active across multiple AWS regions.

### 4. What is the CAP theorem and how does it affect availability?
**Answer:** CAP theorem states a distributed system can only guarantee two of three:
- **Consistency**: All nodes see the same data
- **Availability**: System always responds to requests
- **Partition Tolerance**: System works despite network failures

In practice, partition tolerance is mandatory (networks do fail), so you choose between:
- **CP**: Consistent but unavailable during partitions (banks, payment systems)
- **AP**: Available but eventually consistent (social media feeds, DNS)

### 5. How would you design for 99.99% availability?
**Answer:**
- **Multi-region deployment**: Deploy across at least 2 geographic regions
- **Redundancy at every layer**: Multiple load balancers, API servers, databases
- **Automatic failover**: Health checks every 10-30 seconds, automatic recovery
- **Database replication**: Primary with at least 2 replicas in different availability zones
- **Monitoring and alerting**: Detect issues before they cause downtime
- **Chaos engineering**: Regularly test failure scenarios (Netflix's Chaos Monkey)
- **Rolling deployments**: Deploy updates gradually, quick rollback capability
- **Graceful degradation**: Non-critical features can fail without bringing down the system

### 6. What are circuit breakers and why are they important for availability?
**Answer:** Circuit breakers prevent cascading failures. When a downstream service fails repeatedly, the circuit breaker "opens" and immediately returns errors instead of waiting for timeouts. This prevents:
- Resource exhaustion (threads waiting for failed service)
- Cascading failures (one service failure bringing down others)
- Allows failed service time to recover

States: Closed (normal), Open (failing fast), Half-Open (testing recovery)

Libraries: Hystrix (Netflix), Resilience4j

### 7. How do you handle database failover without data loss?
**Answer:**
- **Synchronous replication**: Primary waits for replica confirmation before committing (slower but no data loss)
- **Asynchronous replication**: Primary commits immediately, replicates later (faster but potential data loss)
- **Semi-synchronous**: Primary waits for at least one replica (balanced approach)

For critical data (payments, orders), use synchronous replication. For less critical data (analytics, logs), asynchronous is acceptable.

## Cross-Questions Interviewer May Ask

### "Why not just aim for 100% availability?"
"100% availability is theoretically impossible and infinitely expensive. Hardware fails, networks partition, bugs exist, and maintenance requires updates. Even with perfect engineering, external factors like natural disasters or ISP failures are beyond our control. Instead, we define acceptable availability based on business requirements and budget. For example, 99.99% might be sufficient for most businesses while being economically feasible."

### "What happens during maintenance? Doesn't that affect availability?"
"That's why we use rolling deployments. We update servers one at a time or in small batches. The load balancer continues routing traffic to healthy servers while others update. This allows zero-downtime deployments. For database schema changes, we use techniques like blue-green deployments or backward-compatible migrations."

### "How do you handle data consistency with multiple replicas?"
"It depends on the use case. For strong consistency, we use synchronous replication where the primary waits for replicas to confirm before committing—this ensures all replicas have the same data but adds latency. For eventual consistency, we use asynchronous replication where replicas catch up over time—faster but temporarily inconsistent. Payment systems need strong consistency; social media feeds can tolerate eventual consistency."

### "Isn't multi-region deployment very expensive?"
"Yes, but the cost depends on your business model. For a payment gateway where every hour of downtime costs millions, multi-region is essential. For an internal tool, single-region with multiple availability zones might suffice. We make trade-offs based on the cost of downtime versus infrastructure costs. Also, multi-region provides latency benefits—users in India hit the Mumbai region, users in the US hit the Virginia region."

### "How do you test failover mechanisms?"
"Through chaos engineering and disaster recovery drills. Netflix's Chaos Monkey randomly kills production servers to ensure failover works. We conduct scheduled failover tests during low-traffic periods, simulate database failures, test network partitions, and verify monitoring alerts trigger correctly. The key is testing in production-like environments, not just staging."

## Points to Impress the Interviewer

- **"Availability is a business decision, not just a technical one"** — The right availability target depends on the cost of downtime vs. cost of infrastructure.

- **"Every nine is exponentially harder"** — Going from 99% to 99.9% is much easier than going from 99.99% to 99.999%.

- **"Design for failure, not perfection"** — Assume everything will fail and build resilience into the system.

- **"Monitoring is half the battle"** — You can't fix what you can't see. Observability is critical for high availability.

- **"Graceful degradation over complete failure"** — Better to serve core features than crash entirely. YouTube still works if recommendations fail.

- **"RTO and RPO matter as much as availability"** — Recovery Time Objective (how fast you recover) and Recovery Point Objective (how much data you can lose) are critical metrics.

- **"Active-Active is ideal but not always necessary"** — Active-Passive is simpler and sufficient for many use cases.

- **Real-world insight:** "WhatsApp handles 100 billion messages daily with ~50 engineers because they built for reliability from day one using Erlang's fault-tolerance features."

- **Cost awareness:** "AWS offers 99.99% SLA for multi-AZ deployments but only 99.5% for single-AZ. That difference can justify the extra cost for critical services."

- **"Circuit breakers prevent cascading failures"** — One service going down shouldn't bring down the entire system.

- **"Chaos engineering in production builds confidence"** — Netflix's Chaos Monkey approach proves your system can handle failures before customers discover them.


---
