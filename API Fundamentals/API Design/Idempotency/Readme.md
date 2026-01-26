# Idempotency

## What is this concept?

**Idempotency**: An operation that produces the same result whether executed once or multiple times.

**Simple definition**: Doing something once = doing it 100 times (same outcome).

**Think of it as**: Light switch - pressing it when it's already ON keeps it ON. Multiple presses don't change the state.

## Why does this exist? / Problem it solves

**Problem: Double Payment Scenario**
```
User clicks "Pay $100"
→ Network timeout (no response)
→ User clicks "Pay" again
→ Charged twice ($200) ❌

With idempotency:
→ First request: Charge $100 ✓
→ Second request: Detect duplicate, ignore ✓
→ Total charged: $100 ✓
```

**Problems solved:**
- **Network retries**: Timeouts cause duplicate requests
- **User error**: Accidental double-clicks
- **System failures**: Crash after processing, before response
- **Message queues**: Duplicate message delivery

## Real-World Analogy

**Elevator Button**

```
Press "5th floor" once → Elevator goes to 5th floor
Press "5th floor" 10 times → Elevator still goes to 5th floor (same result)

Not idempotent:
Press elevator call button 10 times → 10 elevators come ❌
```

## Idempotent vs Non-Idempotent Operations

### Idempotent ✓

**SET operations (absolute value):**
```
user.status = 'active'
user.status = 'active'  (same result)
```

**DELETE:**
```
DELETE user with ID 123
DELETE user with ID 123  (already deleted, no change)
```

**UPDATE with absolute value:**
```sql
UPDATE users SET age = 30 WHERE id = 123;
(Run 100 times → age is still 30)
```

---

### Non-Idempotent ✗

**INCREMENT operations (relative change):**
```
user.login_count += 1
user.login_count += 1  (different result each time!)
```

**INSERT without check:**
```sql
INSERT INTO orders VALUES (...)
(Run 100 times → 100 duplicate orders)
```

**POST request:**
```
POST /create-order
(Each request creates new order)
```

---

## HTTP Methods: Idempotent or Not?

| Method | Idempotent? | Example |
|--------|-------------|---------|
| **GET** | ✓ Yes | Get user profile (read-only) |
| **PUT** | ✓ Yes | Update user to status='active' (same final state) |
| **DELETE** | ✓ Yes | Delete item (deleting deleted item = no change) |
| **PATCH** | ⚠️ Depends | If absolute values: yes; if relative: no |
| **POST** | ✗ No | Create new order (each creates new resource) |

**Examples:**
```
GET /users/123           → Idempotent ✓
PUT /users/123 {age:30}  → Idempotent ✓ (always age=30)
DELETE /users/123        → Idempotent ✓
POST /orders             → Not idempotent ✗ (creates new order each time)
```

---

## Implementation Strategies

### 1. Idempotency Key (Most Common)

**How it works:**
```
Client generates unique ID (UUID)
Sends with request: Idempotency-Key: abc-123-xyz

Server:
1. Check if key exists in processed_requests table
2. If exists → Return cached response (duplicate)
3. If new → Process request, store key + response
```

**Example: Payment API**
```
POST /payments
Headers:
  Idempotency-Key: uuid-abc-123

Request 1: Process payment → Store key ✓
Request 2 (retry): Check key exists → Return "Already processed" ✓
```

**Storage:**
```
Table: processed_requests
- idempotency_key (unique)
- response (cached)
- created_at
```

---

### 2. Database Upsert (UPDATE or INSERT)

**SQL Example:**
```sql
-- Idempotent: Insert or update
INSERT INTO inventory (item_id, stock)
VALUES (1, 100)
ON CONFLICT (item_id) 
DO UPDATE SET stock = 100;

Run once: stock = 100 ✓
Run 10 times: stock = 100 ✓ (idempotent)
```

**Use unique constraints:**
```sql
CREATE UNIQUE INDEX ON orders(idempotency_key);

-- Duplicate insert fails, prevents double orders
```

---

### 3. Message Deduplication (Queues)

**Track processed message IDs:**
```
Message Queue (Kafka, RabbitMQ):
  Message ID: msg-123
  Content: "Process order"

Consumer:
1. Check if msg-123 in processed_messages
2. If yes → Skip (already processed)
3. If no → Process, add msg-123 to set
```

---

### 4. Versioning / Optimistic Locking

**Track version number:**
```sql
UPDATE users 
SET status = 'active', version = version + 1
WHERE id = 123 AND version = 5;

If version changed (concurrent update), update fails
Prevents lost updates
```

---

## Real-World Examples

### Payment Processing (Stripe)

```
POST /charges
Headers:
  Idempotency-Key: order_12345

First call: Charge $100 ✓
Retry (timeout): Detect duplicate key → Return original response ✓
User charged once, not twice
```

### Order Creation (E-commerce)

```
POST /orders
Body: { order_id: "ORD-123", items: [...] }

Using order_id as idempotency key:
- First request: Create order
- Duplicate requests: Return existing order (don't create new)
```

### Message Processing (Kafka)

```
Kafka delivers message twice (at-least-once delivery)
Consumer tracks message offset
Second delivery → Detect duplicate offset → Skip processing
```

---

## Best Practices

1. **Use Idempotency Keys for POST**
   - Client generates UUID
   - Include in header: `Idempotency-Key: <uuid>`

2. **Store Keys with TTL**
   - Don't store forever
   - 24-hour TTL common
   - After expiry, treat as new request

3. **Return Cached Response**
   - Don't just skip processing
   - Return original response to client
   - Status code: 200 OK (not 409 Conflict)

4. **Design for Idempotency from Start**
   - Harder to add later
   - Consider retries in initial design

5. **Use Unique Constraints**
   - Database enforces idempotency
   - Prevents duplicate inserts

6. **Document Idempotent Operations**
   - API docs: "This endpoint is idempotent"
   - Specify how long keys are valid

---

## Common Challenges

**1. Storage Overhead**
- Storing keys costs memory
- Solution: TTL (expire after 24 hours)

**2. Distributed Systems**
- Multiple servers need shared key store
- Solution: Redis for centralized storage

**3. State Management**
- Stateless services harder to implement
- Solution: External store (database, Redis)

**4. Partial Failures**
- Request processed, but response fails
- Solution: Store response with key

---

## Interview Explanation (90-second answer)

*"Idempotency means an operation produces the same result whether executed once or multiple times. It's critical for handling retries safely.*

*The classic example is payment processing. If a user clicks 'Pay $100' and the request times out, they might retry. Without idempotency, they'd be charged twice. With idempotency, the system detects the duplicate request and ignores it.*

*The most common implementation uses idempotency keys. The client generates a unique ID and includes it in the request header. The server stores this key when processing. If the same key arrives again, it's a duplicate—the server returns the cached response instead of reprocessing.*

*HTTP methods have natural idempotency. GET, PUT, and DELETE are idempotent—calling them multiple times produces the same result. POST is not idempotent because each call creates a new resource. That's why POST endpoints often require explicit idempotency keys.*

*In databases, we use techniques like upsert operations or unique constraints. For example, INSERT ... ON CONFLICT ensures you don't create duplicates even if the operation runs multiple times.*

*Real systems like Stripe require idempotency keys for all payment endpoints. Kafka consumers track message offsets to avoid processing duplicate messages. The key is designing for idempotency from the start—it's much harder to add later."*

---

## Top Interview Questions

### 1. What is idempotency and why is it important?
**Answer:**

**What:** Operation with same result when executed 1x or 100x.

**Why important:**
- Network retries (timeouts)
- User double-clicks
- Message queue duplicates
- Prevent duplicate charges, orders, operations

**Example:** DELETE user 123 → Running twice doesn't change result (user still deleted).

### 2. Which HTTP methods are idempotent?
**Answer:**

**Idempotent:**
- **GET**: Read-only, no state change
- **PUT**: Replace resource, same final state
- **DELETE**: Delete resource, already deleted = no change

**Not idempotent:**
- **POST**: Creates new resource each time

**Example:**
```
PUT /users/123 {age: 30}  → Idempotent ✓
POST /orders               → Not idempotent ✗
```

### 3. How do you implement idempotency for POST requests?
**Answer:**

**Use Idempotency Keys:**

```
Client:
POST /orders
Idempotency-Key: uuid-abc-123

Server:
1. Check if key "uuid-abc-123" exists
2. If exists → Return cached response
3. If new → Process, store key + response
```

**Database:**
```sql
CREATE TABLE processed_requests (
  idempotency_key VARCHAR PRIMARY KEY,
  response JSON,
  created_at TIMESTAMP
);
```

**Real example:** Stripe payments require idempotency keys.

### 4. What's the difference between idempotent and safe HTTP methods?
**Answer:**

**Safe**: Doesn't change server state (read-only)
- Example: GET

**Idempotent**: Multiple calls = same result (may change state)
- Example: DELETE (changes state once, then no more changes)

**Table:**
| Method | Safe | Idempotent |
|--------|------|------------|
| GET | ✓ | ✓ |
| POST | ✗ | ✗ |
| PUT | ✗ | ✓ |
| DELETE | ✗ | ✓ |

### 5. How long should you store idempotency keys?
**Answer:**

**Common: 24 hours**

**Why not forever:**
- Storage costs
- Keys accumulate infinitely
- Unlikely retry after 24h

**Implementation:**
```sql
-- Auto-delete old keys
DELETE FROM processed_requests 
WHERE created_at < NOW() - INTERVAL '24 hours';
```

**Or use Redis with TTL:**
```
SET idempotency:key123 "response" EX 86400  # 24 hours
```

### 6. How do you handle idempotency in distributed systems?
**Answer:**

**Challenge:** Multiple servers need shared key storage.

**Solution: Centralized store (Redis)**

```
Request 1 → Server A → Redis (check key) → Process
Request 2 → Server B → Redis (key exists) → Return cached

Redis:
  idempotency:key123 → {response: {...}, timestamp: ...}
```

**Alternative: Database with unique constraint**
```sql
CREATE UNIQUE INDEX ON requests(idempotency_key);
-- Duplicate insert fails → handle gracefully
```

### 7. What are common idempotency patterns in databases?
**Answer:**

**1. Upsert (ON CONFLICT):**
```sql
INSERT INTO users (id, status) VALUES (123, 'active')
ON CONFLICT (id) DO UPDATE SET status = 'active';
-- Idempotent: Run 10 times, status still 'active'
```

**2. Unique Constraints:**
```sql
CREATE UNIQUE INDEX ON orders(idempotency_key);
-- Prevents duplicate inserts
```

**3. Conditional Updates:**
```sql
UPDATE inventory SET stock = 100 WHERE id = 1;
-- Absolute value, not increment
```

**Not idempotent:**
```sql
UPDATE inventory SET stock = stock + 1 WHERE id = 1;
-- Increment ✗
```

### 8. How does Stripe implement idempotency?
**Answer:**

**Requires idempotency keys for all write operations:**

```
POST https://api.stripe.com/v1/charges
Headers:
  Idempotency-Key: order_12345_attempt1

First call: Creates charge
Retry: Returns same charge (no duplicate)
```

**Storage:**
- Keys stored for 24 hours
- Returns cached response for duplicates
- Status 200 (not 409)

**Prevents:** Double charging customers on network errors.

### 9. How do you test idempotency?
**Answer:**

**Test scenarios:**

**1. Duplicate requests:**
```
Send same request twice with same idempotency key
Verify: Second request returns same result, no side effects
```

**2. Timeout retry:**
```
Request 1: Timeout before response
Request 2: Retry with same key
Verify: Only one operation executed
```

**3. Concurrent requests:**
```
Send same key from 2 clients simultaneously
Verify: Only one processes, other gets cached response
```

**4. Database level:**
```
Run INSERT/UPDATE multiple times
Verify: Final state is same
```

### 10. When is idempotency NOT needed?
**Answer:**

**Skip idempotency when:**

1. **Read-only operations**: GET requests (naturally idempotent)
2. **Internal services**: Guaranteed single execution
3. **Analytics/logging**: Duplicate logs acceptable
4. **Non-critical operations**: Newsletter signup (duplicate tolerable)

**Always need for:**
- Payment processing
- Order creation
- Account updates
- Financial transactions

---

## Points to Impress the Interviewer

- **"POST is not idempotent, needs explicit keys"** — Know HTTP methods
- **"Store keys with 24h TTL"** — Practical consideration
- **"Return cached response, not error"** — Proper behavior
- **"Use Redis for distributed idempotency"** — Scalable solution
- **"Design for retries from day one"** — Best practice
- **"Stripe requires idempotency keys"** — Real-world example

**Real-world insight:** "Amazon's order system uses idempotency keys. If you click 'Place Order' and the page reloads, you don't get two orders. The order ID acts as the idempotency key."

**System design context:** "In payment gateway design, idempotency is mandatory. Use client-generated UUID as key, store in Redis with 24h TTL, return cached response for duplicates. This prevents double-charging during network errors."

---