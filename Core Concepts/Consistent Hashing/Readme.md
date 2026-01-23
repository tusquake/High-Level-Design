# Consistent Hashing

## What is this concept?

**Consistent Hashing** is a distributed hashing technique that minimizes data redistribution when nodes (servers) are added or removed from a system.

**Traditional hashing problem:**
```
Server = hash(key) % N

When N changes (add/remove server):
- Almost ALL keys map to different servers
- Massive data movement
- Cache invalidation
```

**Consistent hashing solution:**
- Only ~1/N keys move when adding/removing servers
- Minimal disruption
- Smooth scaling

## Why does this exist? / Problem it solves

- **Massive data reshuffling**: Adding 1 server redistributes 80-90% of data in traditional hashing
- **Cache invalidation**: All cached data becomes invalid when servers change
- **Downtime during scaling**: Can't add/remove servers without major disruption
- **Hotspots**: Poor load distribution across servers
- **Scalability**: Hard to scale up/down dynamically

Real-world impact: Without consistent hashing, Memcached cluster scaling would invalidate entire cache, causing database overload. With it, only affected keys get redistributed.

## Real-World Analogy

**Library Book Distribution Across Shelves**

**Traditional Hashing (Bad):**
```
10 shelves, Book ID = 12345
Shelf = 12345 % 10 = Shelf 5

Add 11th shelf:
Shelf = 12345 % 11 = Shelf 1 ← Changed!

Problem: Must reorganize ~90% of all books
```

**Consistent Hashing (Good):**
```
Imagine shelves and books placed on a circular track (0-360°)
Each book goes to the nearest shelf clockwise

Add new shelf at 225°:
Only books between 180° and 225° move to new shelf

Problem solved: Move only ~9% of books (1/11)
```

## How it works (High Level)

### The Hash Ring

1. **Create a circular hash space** (0 to 2^32-1)
2. **Hash server names** and place them on the ring
3. **Hash data keys** and place them on the ring
4. **Assign each key to the first server found clockwise**

### Adding a Server
```
Before: Server A (90°), Server B (180°), Server C (270°)
Add: Server D at 225°

Keys affected: Only those between 180° and 225°
Keys moved: ~1/4 of total (from C to D)
```

### Removing a Server
```
Remove Server B at 180°
Its keys go to next server clockwise (Server C)
Only Server B's keys move, rest stay in place
```

### Virtual Nodes (Critical Optimization)

Each physical server gets **multiple positions** on the ring:
- Server A: positions at 45°, 120°, 200°, 300° (4 virtual nodes)
- Server B: positions at 60°, 150°, 240°, 330°
- Server C: positions at 90°, 180°, 270°, 350°

**Why?**
- Better load distribution
- Smoother when servers have different capacities
- Avoid clustering (all servers close together on ring)

**Typical: 150-200 virtual nodes per physical server**

## Simple Example

**Cache Server Distribution (3 servers, 6 keys)**

### Traditional Hashing
```
hash(key) % 3

Key1: hash=100 → 100%3 = Server 1
Key2: hash=101 → 101%3 = Server 2
Key3: hash=102 → 102%3 = Server 0
Key4: hash=103 → 103%3 = Server 1
Key5: hash=104 → 104%3 = Server 2
Key6: hash=105 → 105%3 = Server 0

Add Server 3 (now 4 total):
Key1: 100%4 = Server 0 ← MOVED!
Key2: 101%4 = Server 1 ← MOVED!
Key3: 102%4 = Server 2 ← MOVED!
Key4: 103%4 = Server 3 ← MOVED!
Key5: 104%4 = Server 0 ← MOVED!
Key6: 105%4 = Server 1 ← MOVED!

Result: 6/6 keys moved (100%)
```

### Consistent Hashing
```
Ring positions (0-359°):
Server A: 90°
Server B: 180°
Server C: 270°

Key1: 45°  → Server A (nearest clockwise)
Key2: 120° → Server B
Key3: 200° → Server C
Key4: 300° → Server A
Key5: 30°  → Server A
Key6: 150° → Server B

Add Server D at 225°:
Key3: 200° → Server D (was C, now D is closer)

Result: 1/6 keys moved (17%)
```

## Code Snippet

```java
class ConsistentHash {
    private TreeMap<Long, String> ring = new TreeMap<>();
    private int virtualNodes = 150; // per server
    
    // Add server to ring
    public void addServer(String server) {
        for (int i = 0; i < virtualNodes; i++) {
            // Create virtual node identifier
            String virtualNode = server + "#" + i;
            long hash = hash(virtualNode);
            ring.put(hash, server);
        }
    }
    
    // Remove server from ring
    public void removeServer(String server) {
        for (int i = 0; i < virtualNodes; i++) {
            String virtualNode = server + "#" + i;
            long hash = hash(virtualNode);
            ring.remove(hash);
        }
    }
    
    // Get server for a key
    public String getServer(String key) {
        if (ring.isEmpty()) return null;
        
        long hash = hash(key);
        
        // Find first server clockwise (ceiling)
        Map.Entry<Long, String> entry = ring.ceilingEntry(hash);
        
        // Wrap around if at the end
        if (entry == null) {
            entry = ring.firstEntry();
        }
        
        return entry.getValue();
    }
    
    private long hash(String key) {
        // Use MD5 or SHA-1 for uniform distribution
        return Math.abs(key.hashCode());
    }
}

// Usage Example
ConsistentHash ch = new ConsistentHash();
ch.addServer("Server1");
ch.addServer("Server2");
ch.addServer("Server3");

// Keys automatically distributed
String s1 = ch.getServer("user:1234"); // → Server2
String s2 = ch.getServer("user:5678"); // → Server1

// Add new server - minimal redistribution
ch.addServer("Server4");
// Only ~25% of keys move

// Server failure - automatic redistribution
ch.removeServer("Server2");
// Server2's keys go to next server clockwise
```

## Where it is used in real systems

- **Amazon DynamoDB**: Partitions data across nodes
- **Apache Cassandra**: Ring-based cluster topology
- **Memcached clients**: Distribute keys across cache servers
- **Redis Cluster**: Sharding with hash slots (similar concept)
- **Akamai CDN**: Route requests to edge servers
- **Discord**: Distribute guilds across servers
- **Chord DHT**: Peer-to-peer distributed hash table
- **Riak**: Distributed database partitioning
- **Nginx**: Consistent hash load balancing

## Pros and Cons

**Pros:**
- **Minimal data movement**: Only ~1/N keys redistributed
- **Smooth scaling**: Add/remove servers with minimal impact
- **No downtime**: Changes don't require cluster restart
- **Better load distribution**: With virtual nodes
- **Fault tolerance**: Failed server's keys automatically redistributed

**Cons:**
- **More complex**: Than simple modulo hashing
- **Virtual nodes overhead**: Memory for ring positions
- **Still possible hotspots**: Popular keys can overload server
- **Rebalancing needed**: When servers have different capacities

## Common Mistakes / Misconceptions

1. **"No data movement when adding servers"** ❌
   - Wrong. ~1/N keys move (much better than 100%, not zero)

2. **"Don't need virtual nodes"** ❌
   - Without them, uneven distribution especially with few servers

3. **"All hash functions work equally"** ❌
   - Need good distribution (MD5, SHA-1, MurmurHash)
   - Poor hash = clustering on ring

4. **"Consistent hashing means strong consistency"** ❌
   - No relation! Different concepts entirely
   - This is about distribution, not data consistency

5. **"Works perfectly for all use cases"** ❌
   - Hot keys still overload single server
   - Need additional strategies (key splitting)

## Interview Explanation (2-minute answer)

*"Consistent Hashing solves the problem of data redistribution when servers are added or removed. With traditional hashing using modulo, if you have 3 servers and add a 4th, almost all keys get remapped to different servers because the modulo value changes from 3 to 4.*

*Consistent Hashing uses a hash ring—imagine a circle from 0 to 2^32. Both servers and keys are hashed to positions on this ring. Each key is assigned to the first server found going clockwise around the ring.*

*When you add a server, only keys between the new server and the next server clockwise need to move. This is approximately 1/N of total keys, where N is the number of servers. When you remove a server, only its keys move to the next server in the ring.*

*To ensure better load distribution, we use virtual nodes. Each physical server gets multiple positions on the ring—typically 150-200 virtual nodes. This prevents scenarios where all servers cluster together on one part of the ring, leaving other parts empty.*

*This is used by DynamoDB for partitioning data, Cassandra for cluster topology, and Memcached for cache distribution. It enables smooth scaling without massive data reshuffling."*

## Top Interview Questions

### 1. Why is consistent hashing better than modulo hashing?
**Answer:**

**Traditional (modulo):**
```
server = hash(key) % N

Add 1 server: N changes, ~90-100% keys remapped
```

**Consistent hashing:**
```
Only ~1/N keys move when adding/removing servers

Example: 100 servers
- Add 1 server: ~1% keys move
- Traditional: ~99% keys move
```

### 2. What are virtual nodes and why use them?
**Answer:**

Each physical server gets **multiple positions** on the ring (150-200 typically).

**Why needed:**
- **Load balancing**: Without them, servers might cluster together
- **Uniform distribution**: Spreads data evenly
- **Heterogeneous servers**: High-capacity server gets more virtual nodes

**Example:**
```
Without virtual nodes: Server positions might be 10°, 15°, 20° (clustered)
With virtual nodes: Positions spread across entire ring
```

### 3. How does consistent hashing handle server failure?
**Answer:**

Failed server's keys **automatically go to the next server clockwise** on the ring.

**Example:**
```
Server A (90°), Server B (180°), Server C (270°)

Server B fails:
- Keys from B (180° to 270°) move to C
- Rest unchanged
- No manual intervention needed
```

### 4. What happens when you add a new server?
**Answer:**

1. Hash new server to position on ring
2. Keys between new server and next server clockwise move to new server
3. All other keys stay in place

**Data moved: ~1/(N+1) where N is existing servers**

### 5. How do you calculate which keys move?
**Answer:**

Keys that hash to positions between:
- Previous server (counterclockwise) to new server

**Example:**
```
Servers at: A(90°), B(180°), C(270°)
Add D at 225°:

Keys from 180° to 225° move from C to D
All other keys unchanged
```

### 6. Can you still have hotspots?
**Answer:**

Yes! If certain keys are very popular:
- One server handles all requests for that key
- Virtual nodes don't help with single hot key

**Solutions:**
- **Key splitting**: Split hot key into multiple keys
- **Read replicas**: Replicate hot data
- **Caching layer**: Add cache before consistent hash ring

## Cross-Questions Interviewer May Ask

**"How many virtual nodes should you use?"**
"Typically 150-200 per physical server. Too few (10-20) = uneven distribution. Too many (1000+) = memory overhead and slower lookups in the ring. 150 is a good balance used by production systems."

**"What hash function should you use?"**
"Need uniform distribution across the ring. MD5, SHA-1, or MurmurHash work well. Avoid simple hashCode() as it might cluster. The goal is random, even distribution around the ring."

**"How do you handle servers with different capacities?"**
"Use weighted virtual nodes. A high-capacity server gets 300 virtual nodes, a low-capacity one gets 100. This assigns proportionally more keys to powerful servers."

**"What if multiple servers hash to same position?"**
"Extremely rare with large hash space (2^32). If it happens, use a different hash function or add server identifier to create unique position. In practice, with good hash function, collision probability is negligible."

## Points to Impress the Interviewer

- **"Only ~1/N keys move, not 100%"** — Shows you understand the math
- **"Virtual nodes prevent clustering"** — Understanding beyond basics
- **"Typically 150-200 virtual nodes per server"** — Real-world knowledge
- **"Used by DynamoDB, Cassandra for partitioning"** — Production systems
- **"Not the same as data consistency models"** — Avoids common confusion
- **"Still need to handle hot keys separately"** — Know the limitations

**Real-world insight:** "When Discord scaled, they used consistent hashing to distribute guilds across servers. Adding new servers only required moving ~10% of guilds, not all of them."

**Cost awareness:** "Consistent hashing enables auto-scaling without massive cache invalidation, reducing database load and infrastructure costs."

---