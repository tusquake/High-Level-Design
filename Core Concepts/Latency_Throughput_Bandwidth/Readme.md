# Latency vs Throughput vs Bandwidth
<img width="1024" height="1024" alt="Gemini_Generated_Image_1xolz61xolz61xol" src="https://github.com/user-attachments/assets/d04fcef4-38a9-4dfc-b504-ed161e2dc3d1" />

## What is this concept?

These three metrics measure different aspects of system performance:

**Latency**: How long does ONE request take to complete? (Time delay)
- Measured in: milliseconds (ms)
- Example: "API responds in 200ms"

**Throughput**: How many requests can the system handle per second? (Volume)
- Measured in: requests per second (RPS)
- Example: "Server handles 1000 requests/second"

**Bandwidth**: Maximum data transfer capacity (Pipe size)
- Measured in: Mbps, GB/s
- Example: "Network has 100 Mbps bandwidth"

**Key difference**: Latency = Speed | Throughput = Capacity | Bandwidth = Data transfer limit

## Why does this exist? / Problem it solves

- **Different problems need different solutions**: Slow responses need caching (latency), many users need more servers (throughput)
- **Trade-offs exist**: Batching improves throughput but increases latency
- **User experience**: Latency = how fast it feels; Throughput = how many users you can handle
- **Cost optimization**: Netflix pays millions for bandwidth; over-provisioning wastes money

## Real-World Analogy

**Highway Traffic System**

**Latency** - How long does ONE car take from A to B?
- Empty road: 30 minutes (low latency)
- Traffic jam: 2 hours (high latency)

**Throughput** - How many cars pass per hour?
- 2-lane road: 200 cars/hour (low throughput)
- 10-lane highway: 1000 cars/hour (high throughput)

**Bandwidth** - How wide is the road?
- 2 lanes vs 10 lanes (maximum capacity)

**Key Insight**: Wide highway ≠ fast travel. Many lanes ≠ individual cars go faster. All three are independent!

## How it works (High Level)

### Latency Formula
```
Total Latency = Network Latency + Processing Latency + Queueing Latency
```

**Reduce latency:**
- Caching (Redis, CDN)
- Database optimization (indexes, read replicas)
- Reduce network distance (CDN, multi-region)

### Throughput Formula
```
Throughput = Concurrent Workers / Processing Time

Example: 10 workers, 100ms each = 100 req/s
```

**Increase throughput:**
- Add more servers (horizontal scaling)
- Asynchronous processing
- Batch operations

### Bandwidth
```
Bandwidth = Maximum data transfer rate

Example: 100 Mbps = 12.5 MB/s maximum
```

**Optimize bandwidth:**
- Compression
- CDN
- Upgrade network

## Simple Example

**Video Streaming (YouTube)**

**Bad: High Latency, Low Throughput**
```
User clicks play → Takes 5 seconds to start (high latency)
Server handles only 100 users (low throughput)
Result: Slow and can't scale
```

**Good: Low Latency, High Throughput, High Bandwidth**
```
User clicks play → Starts in 500ms (CDN caching)
Streams 1080p smoothly (high bandwidth)
Handles 1M concurrent users (distributed servers)
```

**E-commerce Checkout**
- **Latency critical**: User clicks "Buy" → needs response <1 second
- **Throughput critical**: Black Friday → 100K simultaneous checkouts

## Code Snippet

```java
// Latency vs Throughput Trade-off

// LOW LATENCY - Process immediately
public Response handleFast(Request req) {
    Result result = processNow(req);
    return new Response(result);
    // Latency: 50ms ✓
    // Throughput: Limited by CPU
}

// HIGH THROUGHPUT - Queue and batch
public Response handleVolume(Request req) {
    queue.add(req);
    return new Response("Queued");
    // Latency: Higher (processed later)
    // Throughput: Much higher (batch efficient) ✓
}

// Key Formula: Throughput = Concurrency / Latency
// 10 workers × 100ms = 100 req/s throughput
```

## Where it is used in real systems

**Latency-Critical:**
- Stock Trading: <1ms (co-located servers)
- Gaming: 20-50ms
- Search Engines: <100ms

**Throughput-Critical:**
- Facebook: Billions of requests/day
- Amazon: Millions of concurrent users

**Bandwidth-Critical:**
- Netflix: 25 Mbps per 4K stream
- File Sharing: Large uploads/downloads

## Pros and Cons

### Low Latency
**Pros**: Better UX, real-time features
**Cons**: Expensive (CDN, premium infra), may reduce throughput

### High Throughput  
**Pros**: Handle more users, cost-effective at scale
**Cons**: May increase latency (queuing), complexity

### High Bandwidth
**Pros**: Large data transfers, high-quality media
**Cons**: Expensive, doesn't improve latency

## Common Mistakes / Misconceptions

1. **"High bandwidth = low latency"** ❌
   - Satellite internet: 1 Gbps bandwidth, 500ms latency

2. **"Latency and throughput are the same"** ❌
   - Latency = time per request | Throughput = requests per second

3. **"More servers always = higher throughput"** ❌
   - Not if database is the bottleneck

4. **"Throughput = 1 / Latency"** ❌
   - Only for serial processing. With concurrency: `Throughput = Concurrency / Latency`

## Interview Explanation (2-minute answer)

*"Latency, throughput, and bandwidth are three different performance metrics.*

*Latency is how long ONE request takes—like an API responding in 200ms. Throughput is how many requests you handle per second—like 1000 req/s. Bandwidth is maximum data transfer rate—like 100 Mbps.*

*Think of a highway: Latency is how long one car takes to travel. Throughput is how many cars pass per hour. Bandwidth is how many lanes the highway has.*

*They're independent. You can have high bandwidth but high latency—satellite internet has 1 Gbps speed but 500ms delay. You can have low latency but low throughput—a single-threaded server responds in 10ms but only handles 100 req/s.*

*The relationship is: Throughput = Concurrency / Latency. With 10 workers and 100ms latency, you get 100 req/s throughput.*

*Different systems optimize differently. Stock trading needs ultra-low latency. Facebook needs high throughput. Netflix needs high bandwidth for 4K streaming."*

## Top Interview Questions

### 1. What's the difference between latency and throughput?
**Answer:** Latency = time for ONE operation. Throughput = operations per second.

Example: One chef makes a burger in 5 minutes (latency). Five chefs make 60 burgers/hour (throughput).

### 2. Can you have high throughput with high latency?
**Answer:** Yes! Through parallelism.

Example: Each query takes 1 second (high latency), but 1000 queries run in parallel = 1000 req/s throughput.

### 3. What's the difference between bandwidth and throughput?
**Answer:** 
- **Bandwidth** = Maximum possible (100 Mbps connection)
- **Throughput** = Actual usage (80 Mbps download speed)

Throughput < Bandwidth due to network overhead, packet loss, etc.

### 4. How do you reduce latency?
**Answer:**
- Caching (Redis, CDN)
- Database optimization (indexes, read replicas)
- Geographic distribution (multi-region)
- Asynchronous processing

### 5. How do you increase throughput?
**Answer:**
- Horizontal scaling (more servers)
- Asynchronous processing (message queues)
- Batching
- Load balancing

### 6. Formula for throughput?
**Answer:**
```
Throughput = Concurrency / Latency

Example:
10 concurrent workers
100ms per request
= 10 / 0.1s = 100 req/s
```

## Cross-Questions Interviewer May Ask

**"Can you improve both latency and throughput?"**
"Yes, with caching—reduces latency (faster responses) and increases throughput (more capacity). But some optimizations conflict: batching increases throughput but adds latency."

**"What's more important: latency or throughput?"**
"Depends on use case. User-facing features need low latency (<200ms). Background jobs need high throughput. Payment systems need both."

**"How do you measure these in production?"**
"Use percentiles (p95, p99) for latency, not averages. Track requests/second for throughput. Monitor with tools like Prometheus, Grafana, DataDog."

## Points to Impress the Interviewer

- **"Optimize for percentiles, not averages"** — p99 latency matters more than average
- **"Throughput = Concurrency / Latency"** — Shows mathematical understanding
- **"Batching increases throughput but adds latency"** — Classic trade-off
- **"CDNs solve latency, horizontal scaling solves throughput"** — Right tool for right problem
- **"Bandwidth is rarely the bottleneck for APIs"** — For typical APIs, latency/throughput matter more


---
