# Consistency Models

## What is this concept?

**Consistency Models** are rules that define **how and when changes to data become visible** across multiple replicas in a distributed system.

They answer:
- When will other nodes see my update?
- Will all nodes show the same value at the same time?
- What happens during network failures?
- Can I read stale data?

**Think of it as**: A contract between the system and users about data synchronization guarantees.

## Why does this exist? / Problem it solves

- **Concurrent updates**: Two users edit same data simultaneously—who wins?
- **Stale reads**: Reading old data after someone updated it
- **Performance vs correctness trade-off**: Strong consistency is slow, weak is fast
- **Network partition handling**: What to do when nodes can't communicate?
- **User expectations**: Banking needs accuracy, social media tolerates delays

Real-world impact: Facebook uses eventual consistency for likes (slight delay acceptable). Banks use strong consistency for transfers (accuracy critical). Wrong choice = lost money or poor UX.

## Real-World Analogy

**Shared Google Doc with Three People**

### Strong Consistency (Linearizability)
```
You type "Hello" →
System blocks everyone until ALL see "Hello" →
Then you can type next word →
Everyone always sees exact same document

Result: Slow but perfect sync
```

### Eventual Consistency
```
You type "Hello" →
Shows to you immediately →
Alice sees it after 1 second →
Bob sees it after 2 seconds →
Eventually everyone sees "Hello"

Result: Fast but temporary differences
```

### Causal Consistency
```
You type "How are you?" (Message A)
Alice replies "I'm good!" (Message B - caused by A)

Everyone sees A before B (causality maintained)
But might see other unrelated messages in any order

Result: Logical order preserved
```

## How it works (High Level)

### The Consistency Spectrum

```
Strongest ←―――――――――――――――――――――――――→ Weakest

Linearizable → Sequential → Causal → Eventual
(Strictest)                              (Loosest)
```

### Strong Consistency (Linearizability)

**Guarantee**: All operations appear to happen in real-time order

**How it works:**
1. Client writes to primary
2. Primary replicates to ALL replicas synchronously
3. Returns success only after ALL confirm
4. Any read returns the latest write

**Trade-off**: Slow, blocks during network issues (sacrifices availability)

### Eventual Consistency

**Guarantee**: All replicas will eventually be the same (given no new updates)

**How it works:**
1. Client writes to primary
2. Primary returns success immediately
3. Replicas sync asynchronously in background
4. Reads might return stale data

**Trade-off**: Fast, always available, but temporarily inconsistent

### Causal Consistency

**Guarantee**: Causally related operations maintain order

**How it works:**
1. Track causality with vector clocks/timestamps
2. Ensure dependent operations maintain order
3. Independent operations can be reordered

**Trade-off**: Middle ground between strong and eventual

## Simple Example

### User Profile Update System

**Strong Consistency:**
```
User in India updates email →
System waits for replicas in US, EU, Asia to update →
200ms total latency →
Returns success →
Anyone reading worldwide immediately sees new email

Pros: Always correct
Cons: Slow (200ms wait)
```

**Eventual Consistency:**
```
User in India updates email →
India replica updates instantly →
20ms latency, returns success →
US replica updates after 50ms →
EU replica updates after 100ms →
Asia replica updates after 80ms

Pros: Fast (20ms)
Cons: For 100ms, different regions see different emails
```

**Causal Consistency:**
```
User posts status: "I got promoted!" (Event A)
Then updates job title to "Senior Engineer" (Event B)

B depends on A (causality)
Everyone sees A before B (order maintained)
But different users might see both at different times
```

### E-commerce Shopping Cart

**Strong Consistency:**
```
User adds item →
Wait for all regions to sync →
Then show "Added to cart"
Slow but cart is always accurate across devices
```

**Eventual Consistency:**
```
User adds item →
Update local cache immediately →
Show "Added to cart" instantly →
Sync to other regions in background
Fast but phone and laptop might show different carts briefly
```

## Code Snippet

```java
// Strong Consistency - Synchronous replication
class StrongConsistencyDB {
    Node primary;
    List<Node> replicas;
    
    public void write(String key, String value) {
        List<Node> allNodes = new ArrayList<>();
        allNodes.add(primary);
        allNodes.addAll(replicas);
        
        // Write to ALL nodes synchronously
        List<Future<Boolean>> results = new ArrayList<>();
        for (Node node : allNodes) {
            Future<Boolean> result = node.writeAsync(key, value);
            results.add(result);
        }
        
        // Wait for ALL confirmations
        for (Future<Boolean> result : results) {
            if (!result.get()) {
                throw new WriteException("Replication failed");
            }
        }
        
        // Only return after all replicas confirm
    }
    
    public String read(String key) {
        // Always read from primary for latest data
        return primary.read(key);
    }
}

// Eventual Consistency - Asynchronous replication
class EventualConsistencyDB {
    Node primary;
    List<Node> replicas;
    
    public void write(String key, String value) {
        // Write to primary only
        primary.write(key, value);
        
        // Return immediately, replicate async
        CompletableFuture.runAsync(() -> {
            for (Node replica : replicas) {
                try {
                    replica.write(key, value);
                } catch (Exception e) {
                    // Retry later, don't block user
                    retryQueue.add(new WriteOp(replica, key, value));
                }
            }
        });
    }
    
    public String read(String key) {
        // Read from any node (might be stale)
        return pickRandomNode().read(key);
    }
}

// Causal Consistency - With vector clocks
class CausalConsistencyDB {
    Map<String, VectorClock> clocks = new ConcurrentHashMap<>();
    
    public void write(String key, String value, VectorClock clock) {
        VectorClock existing = clocks.get(key);
        
        // Check causality
        if (existing != null && !clock.isAfter(existing)) {
            // Conflict: concurrent writes
            resolveConflict(key, value, clock);
        } else {
            // Causal order maintained
            primary.write(key, value);
            clocks.put(key, clock);
        }
        
        // Replicate with clock
        replicateWithCausality(key, value, clock);
    }
}
```

## Where it is used in real systems

### Strong Consistency
- **Google Spanner**: Global distributed database
- **Apache Zookeeper**: Distributed coordination
- **etcd**: Kubernetes configuration store
- **Banking systems**: Account transfers
- **Inventory management**: Stock tracking
- **Booking systems**: Seat reservations
- **Traditional SQL** (PostgreSQL, MySQL): ACID transactions

### Eventual Consistency
- **Amazon DynamoDB**: Default mode
- **Apache Cassandra**: Default mode
- **Amazon S3**: Object storage
- **DNS**: Domain name resolution
- **Shopping carts**: Amazon, Flipkart
- **Social media feeds**: Facebook, Twitter, Instagram
- **Analytics systems**: Data warehouse

### Causal Consistency
- **MongoDB**: Causal consistency sessions (optional)
- **Riak**: Configurable
- **Collaborative apps**: Google Docs (operational transforms)
- **Chat applications**: Message ordering

### Tunable Consistency
- **Cassandra**: Choose consistency level per query (ONE, QUORUM, ALL)
- **DynamoDB**: Strong or eventual per read
- **MongoDB**: Read/write concern levels

## Pros and Cons

### Strong Consistency
**Pros:**
- Always correct, latest data
- Simple application logic
- No conflict resolution needed
- Predictable behavior

**Cons:**
- Slow (synchronous replication)
- Lower availability (blocks during partition)
- Higher latency (wait for all replicas)
- Reduced throughput

**Use when**: Correctness critical (banking, inventory, bookings)

### Eventual Consistency
**Pros:**
- Fast (no waiting)
- High availability (always responsive)
- Low latency
- High throughput
- Scalable

**Cons:**
- Temporary inconsistency
- Complex application logic
- Conflict resolution needed
- Stale reads possible

**Use when**: Availability > Correctness (social media, caching, analytics)

### Causal Consistency
**Pros:**
- Better than eventual (maintains logical order)
- Faster than strong (no global sync)
- Good for collaborative apps

**Cons:**
- Still has conflicts
- More complex than eventual
- Overhead of tracking causality

**Use when**: Need logical ordering (chat, collaborative editing)

## Common Mistakes / Misconceptions

1. **"Eventual consistency means always inconsistent"** ❌
   - Wrong. "Eventual" means temporary (usually seconds)
   - Eventually becomes consistent

2. **"Strong consistency is always better"** ❌
   - Depends on use case
   - Social media doesn't need it, wastes performance

3. **"Eventual consistency = data loss"** ❌
   - No data loss, just temporary staleness
   - All writes are preserved

4. **"Consistency models = Consistent hashing"** ❌
   - Completely different concepts
   - Consistent hashing = data distribution
   - Consistency models = data synchronization

5. **"You must choose one for entire system"** ❌
   - Can mix: strong for payments, eventual for comments
   - Modern DBs are tunable

6. **"Causal consistency needs global clock"** ❌
   - Uses vector clocks or logical timestamps
   - No synchronized physical clocks needed

## Interview Explanation (2-minute answer)

*"Consistency models define how and when data updates become visible across replicas in distributed systems.*

*Strong consistency, or linearizability, means all replicas always show the same data. When you write, the system waits for all replicas to confirm before returning success. Any subsequent read returns the latest write. This is slow because of synchronous replication but guarantees correctness. Banks use this for account transfers.*

*Eventual consistency means replicas will eventually sync, but not immediately. When you write, the system returns success right away and replicates asynchronously. Reads might return stale data temporarily—usually for just seconds. This is fast and highly available, used by DynamoDB and social media platforms.*

*Causal consistency is a middle ground. It maintains the order of causally related events. If event A causes event B, everyone sees A before B. But independent events can appear in any order. This is useful for chat apps where message order matters.*

*The choice depends on business needs. Payment processing needs strong consistency—you can't have duplicate charges. Social media likes can use eventual consistency—a 2-second delay in like counts is acceptable for better user experience. Many modern systems like Cassandra and DynamoDB are tunable, letting you choose consistency level per operation."*

## Top Interview Questions

### 1. What's the difference between strong and eventual consistency?
**Answer:**

**Strong Consistency:**
- All replicas always synchronized
- Read returns latest write
- Slow, blocks during replication

**Eventual Consistency:**
- Replicas sync asynchronously
- Read might return stale data
- Fast, always available

**Example:**
- Bank transfer: Strong (can't show wrong balance)
- Instagram likes: Eventual (2-second delay acceptable)

### 2. When would you choose eventual over strong consistency?
**Answer:**

**Choose eventual when:**
- Availability > Correctness
- Performance critical (low latency needed)
- Stale data acceptable (analytics, social feeds)
- High write throughput needed

**Don't choose when:**
- Financial transactions
- Inventory/booking systems
- Data correctness critical

### 3. What is causal consistency?
**Answer:**

Maintains order of **causally related** events. If A causes B, everyone sees A before B.

**Example:**
- Message: "What's for dinner?" (A)
- Reply: "Pizza!" (B - caused by A)
- Everyone sees A before B
- But unrelated messages can appear in any order

**Middle ground**: Weaker than strong, stronger than eventual

### 4. How long does eventual consistency take?
**Answer:**

**Typical**: Seconds (1-10 seconds)
**Fast systems**: Sub-second
**During issues**: Minutes

**Examples:**
- DynamoDB: Usually <1 second
- DNS: Minutes to hours (due to TTL caching)
- S3: Typically a few seconds

Not hours or days!

### 5. How do you handle conflicts in eventual consistency?
**Answer:**

**Strategies:**
1. **Last Write Wins (LWW)**: Use timestamps, latest wins
2. **Vector Clocks**: Track causality, detect conflicts
3. **Application-level**: Let app decide (merge shopping carts)
4. **CRDTs**: Conflict-free replicated data types (auto-merge)

**Example:** Two users add items to cart → Merge both additions

### 6. What is read-your-writes consistency?
**Answer:**

A user always sees their own updates immediately (even with eventual consistency).

**How:**
- Track user's session
- Route reads to replica that has user's writes
- Or force read from primary for user's own data

**Example:** Post tweet → Immediately see it in your feed (even if others see it later)

## Cross-Questions Interviewer May Ask

**"Can you have strong consistency with high availability?"**
"No, this is the CAP theorem trade-off. During network partitions, strong consistency requires blocking requests to maintain correctness, which sacrifices availability. You must choose CP (consistency + partition tolerance) or AP (availability + partition tolerance)."

**"How does DynamoDB provide both strong and eventual consistency?"**
"DynamoDB is tunable. By default, reads are eventually consistent (faster). You can request strongly consistent reads, which read from the primary replica and wait for latest data. This is slower but guarantees no stale data. You choose per operation based on needs."

**"What happens if two users update same data simultaneously?"**
"Depends on consistency model. Strong consistency: One wins (serialized writes). Eventual consistency: Both succeed, conflict resolved later using strategies like Last Write Wins, vector clocks, or application-level merging. Example: Two users add items to cart → System merges both."

**"Is eventual consistency eventual or can it take forever?"**
"It's guaranteed to become consistent if no new updates arrive. In practice, it takes seconds. The 'eventual' part accounts for network delays and replication lag, not infinite time. Systems monitor replication lag and alert if it exceeds thresholds (e.g., >10 seconds)."

## Points to Impress the Interviewer

- **"It's a spectrum, not binary"** — Strong → Causal → Eventual
- **"Business needs drive the choice"** — Not purely technical
- **"Modern systems are tunable"** — Cassandra, DynamoDB let you choose per query
- **"Eventual ≠ inconsistent forever"** — Usually seconds, not hours
- **"Different ops can have different guarantees"** — Critical writes strong, casual reads eventual
- **"Read-your-writes consistency"** — Users see their own updates immediately

**Real-world insight:** "Amazon shopping cart uses eventual consistency with conflict-free merging. If you add items on phone and laptop simultaneously, both items appear in cart—system merges rather than choosing one."

**Links to CAP:** "Strong consistency sacrifices availability during partitions (CP). Eventual consistency maintains availability but sacrifices consistency (AP)."

---