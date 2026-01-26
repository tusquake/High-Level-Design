# Rate Limiting Algorithms

## What is this concept?

**Rate Limiting** controls how many requests a client can make to a service within a time window. It prevents abuse and ensures fair resource usage.

**Rate Limiting Algorithms** are different strategies to implement this control, each with trade-offs in accuracy, memory usage, and burst handling.

**Think of it as**: A water tap - you control how fast water flows. Different algorithms are different ways to control the flow rate.

## Why does this exist? / Problem it solves

- **Prevent abuse**: Stop malicious users from overwhelming system
- **Fair resource allocation**: Ensure all users get fair access
- **Cost control**: Limit expensive operations (API calls, DB queries)
- **DDoS protection**: Block excessive traffic
- **SLA enforcement**: Different limits for free vs paid tiers

Without rate limiting: Single user can consume all resources, causing service degradation for everyone.

## Real-World Analogy

**Toll Booth with Different Collection Methods**

**Token Bucket**: Drivers get tokens every minute. 10 tokens in pocket → pass through 10 times quickly.

**Leaky Bucket**: Cars queue, exit at constant rate (1 car/minute). Queue full → reject new cars.

**Fixed Window**: Count cars per hour (0-60 min). Hour resets → count starts at zero.

**Sliding Window**: Look at last 60 minutes continuously. If 100 cars passed in rolling window → reject.

## The 5 Rate Limiting Algorithms

### 1. Token Bucket (Most Popular)

**How it works:**
```
Bucket capacity: 10 tokens
Refill rate: 2 tokens/second

Time 0s: Bucket has 10 tokens
Request 1: Take 1 token → 9 left ✓
Request 2: Take 1 token → 8 left ✓
...
Request 11: No tokens → Reject ✗

After 1 second: Add 2 tokens → 10 tokens (capped at capacity)
```

**Visual:**
```
Bucket [🪙🪙🪙🪙🪙🪙🪙🪙🪙🪙] Capacity: 10
       ↑ Refill: 2 tokens/sec
       ↓ Request: -1 token

Request arrives:
  If tokens ≥ 1: Allow, remove token
  If tokens < 1: Reject
```

**Characteristics:**
- Allows **bursts** up to bucket capacity
- Tokens refill at constant rate
- Good for handling traffic spikes

**Pros:**
✓ Handles burst traffic (10 quick requests if bucket full)
✓ Simple to implement
✓ Flexible (burst + sustained rate)

**Cons:**
✗ Memory per user (track tokens for each)
✗ Can allow sudden bursts

**Use cases:**
- AWS API Gateway (default algorithm)
- Most public APIs (allow bursts)
- General-purpose rate limiting

**Example:**
```
API limit: 100 requests/minute
Implementation: Bucket capacity = 100, refill = 100 tokens/60s
User can send 100 requests instantly (burst)
Then 1-2 requests per second sustained
```

---

### 2. Leaky Bucket

**How it works:**
```
Bucket capacity: 10 requests
Leak rate: 2 requests/second

Requests enter from top → Queue in bucket
Requests leave from bottom at constant rate

Bucket [Req1, Req2, Req3]
       ↑ Input (variable rate)
       ↓ Output (constant 2/sec)

If bucket full → Reject new requests
```

**Visual:**
```
     ↓ Requests arrive (bursty)
   ┌─────────┐
   │ Req5    │
   │ Req4    │  ← Bucket (Queue)
   │ Req3    │
   │ Req2    │
   │ Req1    │
   └────┬────┘
        ↓ Leak at constant rate (2/sec)
```

**Characteristics:**
- Processes at **constant rate**
- Smooths bursty traffic
- Queue-based

**Pros:**
✓ Constant output rate (predictable load)
✓ Smooth traffic processing
✓ Good for protecting downstream services

**Cons:**
✗ Doesn't handle bursts well (drops excess)
✗ Requests can be delayed (queued)
✗ More complex than token bucket

**Use cases:**
- Network traffic shaping
- When you need constant processing rate
- Protecting rate-sensitive downstream services

**Example:**
```
Video streaming upload: Accept bursts but process at 1 video/second
Bucket queues uploads, processes steadily
Prevents overwhelming video processing service
```

---

### 3. Fixed Window Counter

**How it works:**
```
Window: 1 minute (00:00 - 00:59)
Limit: 100 requests/minute

00:00 - 00:59 → Count requests
00:15: 50 requests so far ✓
00:45: 98 requests → Allow 2 more ✓
00:50: 100 requests → Limit reached ✗
01:00: New window starts → Counter resets to 0
```

**Visual:**
```
Window 1 (00:00-00:59): |||||||||| (10 requests) ✓
Window 2 (01:00-01:59): ||         (2 requests)  ✓
Window 3 (02:00-02:59): Counter resets
```

**Characteristics:**
- Fixed time windows
- Counter resets at window boundary
- Simple to implement

**Pros:**
✓ Very simple (just a counter)
✓ Memory efficient
✓ Easy to understand

**Cons:**
✗ **Boundary problem**: Can allow 2x requests at window edges
✗ Traffic spikes at reset

**Boundary Problem Example:**
```
Limit: 100 req/minute

00:30-00:59: 100 requests ✓ (valid)
01:00-01:29: 100 requests ✓ (valid, new window)

Result: 200 requests in 30 seconds! (00:30-01:29)
This is 2x the intended rate!
```

**Use cases:**
- Simple APIs with low traffic
- When exact accuracy not critical
- Internal rate limiting

**Example:**
```
GitHub API: 60 requests per hour for unauthenticated
Uses fixed window, resets every hour on the hour
```

---

### 4. Sliding Window Log

**How it works:**
```
Window: 60 seconds
Limit: 100 requests
Log: Keep timestamp of every request

Current time: 12:00:45
Window: Last 60 seconds (11:59:45 - 12:00:45)

Request log:
[11:59:50, 11:59:55, 12:00:10, 12:00:20, ...]

New request at 12:00:45:
1. Remove timestamps < 11:59:45 (older than 60s)
2. Count remaining: 95 requests
3. If < 100: Allow, add timestamp
4. If ≥ 100: Reject
```

**Visual:**
```
Timeline: ←────────────60 seconds──────────→
          [removed] [valid requests in window]
                    ↑                      ↑
                  Past                   Now

Valid timestamps in window: 95
New request: 95 < 100 → Allow ✓
```

**Characteristics:**
- Keeps log of all request timestamps
- Accurate sliding window
- Memory intensive

**Pros:**
✓ Very accurate (no boundary issues)
✓ True sliding window
✓ Works for any time range

**Cons:**
✗ Memory intensive (store every timestamp)
✗ Slower (must scan and remove old timestamps)
✗ Not suitable for high-volume APIs

**Use cases:**
- Low-volume critical APIs
- When accuracy is paramount
- Financial/payment APIs

**Example:**
```
Payment API: Max 10 transactions per hour per user
Keep log of all transaction timestamps
Accurate sliding 60-minute window
```

---

### 5. Sliding Window Counter (Hybrid)

**How it works:**
```
Combine Fixed Window + Sliding Window Log

Windows:
  Previous: 10:00-10:59 (60 requests)
  Current:  11:00-11:59 (20 requests so far)

Current time: 11:45 (45% into current window)

Weighted count:
  = (100% - 45%) × Previous + Current
  = 55% × 60 + 20
  = 33 + 20
  = 53 requests

If 53 < 100 → Allow ✓
```

**Visual:**
```
Previous Window    Current Window
[00-59 minutes]   [00-59 minutes]
60 requests       20 requests
    ↓                  ↓
    └──────┬───────────┘
           ↓
    Weighted: 55% of 60 + 100% of 20 = 53

Current position: 45% into current window
Use 55% of previous + 100% of current
```

**Calculation Example:**
```
Window: 1 minute (60 seconds)
Limit: 100 requests
Time: 11:00:45 (45 seconds into current minute)

Previous window (10:59-10:60): 80 requests
Current window (11:00-11:01): 30 requests

Time passed in window: 45 seconds
Percentage into window: 45/60 = 75%

Weighted count:
  = (100 - 75)% × 80 + 30
  = 25% × 80 + 30
  = 20 + 30
  = 50 requests

New request:
  50 + 1 = 51 < 100 → Allow ✓
```

**Characteristics:**
- Approximates sliding window
- Only tracks two windows
- Balance of accuracy and efficiency

**Pros:**
✓ More accurate than Fixed Window
✓ More efficient than Sliding Log
✓ Smooth boundary transitions
✓ Good accuracy-memory tradeoff

**Cons:**
✗ Slightly complex
✗ Approximation (not 100% accurate)

**Use cases:**
- High-volume APIs
- Production systems (Cloudflare uses this)
- When you need accuracy + efficiency

**Example:**
```
Twitter API: 900 tweets per 15 minutes
Uses sliding window counter
Smooth rate limiting without sharp boundaries
```

---

## Algorithm Comparison

| Algorithm | Accuracy | Memory | Bursts | Complexity | Best For |
|-----------|----------|--------|--------|------------|----------|
| **Token Bucket** | Good | Low | Yes ✓ | Low | General purpose |
| **Leaky Bucket** | Good | Medium | No ✗ | Medium | Constant rate |
| **Fixed Window** | Poor | Very Low | Yes ✓ | Very Low | Simple APIs |
| **Sliding Log** | Excellent | High | No ✗ | High | Low volume |
| **Sliding Counter** | Very Good | Low | Moderate | Medium | High volume |

---

## Visual Comparison

### Traffic Pattern Handling

**Burst of 20 requests in 1 second, then nothing:**

**Token Bucket:**
```
Burst: ✓✓✓✓✓✓✓✓✓✓ (10 allowed instantly)
       ✗✗✗✗✗✗✗✗✗✗ (10 rejected, bucket empty)
Later: Tokens refill → Accept more
```

**Leaky Bucket:**
```
Burst: Queued [20 requests]
       ↓ Process at 2/sec
Output: 2, wait, 2, wait, 2...
Takes 10 seconds to process all
```

**Fixed Window:**
```
00:00-00:59: 20 requests ✓ (under limit)
Allowed all 20 if under window limit
```

**Sliding Window Log:**
```
Keep all 20 timestamps
Future requests: Check if within window
Accurate to the second
```

**Sliding Window Counter:**
```
Count in current window: 20
Weighted with previous window
Smooth rate checking
```

---

## Interview Explanation (2-minute answer)

*"Rate limiting algorithms control request frequency to prevent abuse and ensure fair resource usage. The five main algorithms each have different trade-offs.*

*Token Bucket is the most popular. Imagine a bucket that holds tokens, refilled at a constant rate. Each request consumes a token. If the bucket is full, you can handle bursts—sending many requests quickly. This makes it great for APIs that need to allow occasional spikes. AWS API Gateway uses this by default.*

*Leaky Bucket is similar but focuses on smooth output. Requests enter a queue and are processed at a constant rate, like water leaking from a bucket. This smooths out bursty traffic but can delay requests. It's good when you need to protect downstream services from sudden spikes.*

*Fixed Window Counter is the simplest—count requests in fixed time windows like minutes or hours. When the window ends, the counter resets. The problem is the boundary issue: you can get 2x the rate at window edges. For example, 100 requests at 00:59 and 100 more at 01:00 gives you 200 requests in one minute.*

*Sliding Window Log keeps a timestamp log of every request and checks the last N seconds continuously. It's very accurate but memory-intensive because you store every timestamp. It's only practical for low-volume APIs.*

*Sliding Window Counter combines fixed windows with weighted counting. It tracks the current and previous window, calculating a weighted count based on how far you are into the current window. If you're 75% through, it uses 25% of the previous window plus 100% of the current. This is more accurate than fixed windows and more efficient than logs. Cloudflare uses this approach.*

*For interviews, Token Bucket and Sliding Window Counter are most important. Token Bucket handles bursts well and is widely used. Sliding Window Counter balances accuracy and efficiency for high-scale systems."*

---

## Top Interview Questions

### 1. What are the main rate limiting algorithms?
**Answer:**

1. **Token Bucket**: Tokens refill at rate, consume per request (allows bursts)
2. **Leaky Bucket**: Queue requests, process at constant rate (smooths traffic)
3. **Fixed Window**: Count per time window (simple but boundary issues)
4. **Sliding Window Log**: Keep all timestamps (accurate but memory-heavy)
5. **Sliding Window Counter**: Weighted count across windows (balanced)

**Most popular**: Token Bucket (AWS, Stripe), Sliding Counter (Cloudflare)

### 2. Explain Token Bucket algorithm
**Answer:**

**Concept:**
- Bucket holds tokens (e.g., capacity = 10)
- Tokens refill at rate (e.g., 2/second)
- Request consumes token
- No tokens → reject request

**Example:**
```
Capacity: 10 tokens
Refill: 2 tokens/second

Bucket [🪙🪙🪙🪙🪙🪙🪙🪙🪙🪙]

10 quick requests → All allowed (burst)
Bucket now empty []
After 1 second → 2 tokens added [🪙🪙]
Can handle 2 more requests
```

**Benefits:**
- Allows bursts (good UX)
- Simple implementation
- Widely used (AWS API Gateway)

### 3. What is the boundary problem in Fixed Window?
**Answer:**

**Problem:** Can allow 2x rate at window boundaries.

**Example:**
```
Limit: 100 requests/minute
Windows: 00:00-00:59, 01:00-01:59

Scenario:
00:30-00:59: 100 requests ✓ (valid, end of window)
01:00-01:29: 100 requests ✓ (valid, new window)

Result: 200 requests in 60 seconds (00:30-01:29)
This is 2x the intended 100/minute rate!
```

**Solution:** Use Sliding Window Counter instead.

### 4. When would you use Leaky Bucket vs Token Bucket?
**Answer:**

**Token Bucket:**
- Allow bursts (better UX)
- Variable output rate
- User-facing APIs

**Example:** User uploads 10 photos quickly → Allow (good UX)

**Leaky Bucket:**
- Need constant processing rate
- Protect downstream services
- Traffic shaping

**Example:** Video processing service can only handle 1 video/second → Queue uploads, process steadily

**Key difference:**
- Token Bucket: Variable output (burst then slow)
- Leaky Bucket: Constant output (always steady)

### 5. How does Sliding Window Counter work?
**Answer:**

**Combines Fixed Window + Weighted calculation:**

```
Track:
- Previous window count
- Current window count

Calculate weighted sum:
Weight = (100 - %_in_window) × Previous + Current

Example:
Previous window: 80 requests
Current window: 30 requests  
45 seconds into current (75%)

Weighted = (100 - 75)% × 80 + 30
         = 25% × 80 + 30
         = 20 + 30
         = 50 requests

New request: 50 + 1 = 51 < 100 → Allow ✓
```

**Benefits:**
- Smooth boundaries (no 2x spike)
- Memory efficient (only 2 counters)
- Good accuracy

### 6. Which algorithm is most memory efficient?
**Answer:**

**From most to least efficient:**

1. **Fixed Window Counter**: Just a counter (few bytes)
2. **Token Bucket**: Token count + timestamp (few bytes)
3. **Sliding Window Counter**: 2 counters (previous + current)
4. **Leaky Bucket**: Queue of requests (depends on queue size)
5. **Sliding Window Log**: All timestamps (bytes per request)

**For high-scale:**
- Use Sliding Window Counter (good efficiency + accuracy)
- Avoid Sliding Window Log (too memory-intensive)

### 7. How do you implement rate limiting in a distributed system?
**Answer:**

**Challenge:** Multiple servers need shared rate limit state.

**Solution: Use Redis**

```
Key: rate_limit:user123
Value: {count: 95, window_start: timestamp}

Each server:
1. Increment Redis counter
2. Check if exceeded limit
3. Return allow/deny

Atomic operations:
INCR rate_limit:user123
EXPIRE rate_limit:user123 60
```

**Algorithms in distributed systems:**

**Token Bucket:**
```
Store in Redis:
- tokens_remaining
- last_refill_time

On request:
1. Calculate tokens to add (time elapsed × rate)
2. Update tokens in Redis (atomic)
3. Decrement if allow
```

**Sliding Window Counter:**
```
Store in Redis:
- current_window:user123 → count
- previous_window:user123 → count

Calculate weighted sum
Check against limit
```

**Tools:** Redis, Memcached, DynamoDB

### 8. What rate limiting algorithm does AWS API Gateway use?
**Answer:**

**Token Bucket algorithm**

**Why:**
- Allows bursts (better user experience)
- Simple to configure
- Industry standard

**Configuration:**
```
Burst limit: 5000 requests
Steady-state rate: 10000 requests/second

Means:
- Can burst up to 5000 requests instantly
- Then sustained at 10000/sec
```

**Other examples:**
- **Stripe**: Token Bucket
- **GitHub**: Fixed Window
- **Cloudflare**: Sliding Window Counter
- **Twitter**: Sliding Window Counter

### 9. How do you handle rate limit responses?
**Answer:**

**HTTP Status Code:**
```
429 Too Many Requests
```

**Response Headers:**
```
X-RateLimit-Limit: 100         (total allowed)
X-RateLimit-Remaining: 23      (requests left)
X-RateLimit-Reset: 1625097600  (when it resets)
Retry-After: 30                (seconds to wait)
```

**Response Body:**
```json
{
  "error": "Rate limit exceeded",
  "message": "Too many requests. Limit: 100/hour",
  "retry_after": 3600
}
```

**Client handling:**
- Respect Retry-After header
- Exponential backoff
- Show user-friendly message

### 10. How do you choose which algorithm to use?
**Answer:**

**Consider:**

**1. Traffic pattern:**
- Bursty → Token Bucket
- Steady → Leaky Bucket

**2. Scale:**
- Low volume → Sliding Window Log (accuracy)
- High volume → Sliding Window Counter (efficiency)

**3. Requirements:**
- Need exact accuracy → Sliding Window Log
- Allow bursts → Token Bucket
- Constant rate → Leaky Bucket

**4. Simplicity:**
- Simple API → Fixed Window
- Production system → Sliding Window Counter

**Decision tree:**
```
High volume API?
├─ Yes → Sliding Window Counter
└─ No → Need bursts?
    ├─ Yes → Token Bucket
    └─ No → Need constant rate?
        ├─ Yes → Leaky Bucket
        └─ No → Fixed Window (simple)
```

---

## Points to Impress the Interviewer

- **"Token Bucket allows bursts, Leaky Bucket smooths them"** — Key difference
- **"Fixed Window has boundary problem"** — Can allow 2x rate
- **"Sliding Window Counter = accuracy + efficiency"** — Production choice
- **"Sliding Window Log = accurate but memory-intensive"** — Trade-off
- **"Use Redis for distributed rate limiting"** — Practical implementation
- **"Return 429 with retry headers"** — Proper API design

**Real-world insight:** "Cloudflare processes 40+ million requests/second with Sliding Window Counter. It's accurate enough to prevent abuse while being memory-efficient enough for global scale."

**System design context:** "For API Gateway design, use Token Bucket for external APIs (allow client bursts) and Leaky Bucket for internal service protection (constant processing rate)."

---