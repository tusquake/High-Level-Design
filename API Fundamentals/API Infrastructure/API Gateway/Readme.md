# API Gateway

## What is this concept?

**API Gateway** is a server that acts as a **single entry point** for all client requests to backend microservices. It sits between clients and backend services, handling routing, security, and other cross-cutting concerns.

**Simple definition**: A centralized layer that manages, routes, and secures API traffic.

**Think of it as**: A hotel receptionist - guests (clients) don't go directly to different departments (services); they ask the receptionist who routes them to the right place.

## Why does this exist? / Problem it solves

**Without API Gateway (E-commerce Example):**
```
Mobile App needs to:
- Know URLs of User Service, Payment Service, Inventory Service
- Handle authentication for each service separately
- Implement rate limiting in each service
- Manage different protocols (REST, gRPC, SOAP)

Result: Complex client, duplicated logic, hard to maintain
```

**With API Gateway:**
```
Mobile App → API Gateway → Routes to correct service
                ↓
        - Centralized authentication
        - Single rate limiting
        - Protocol translation
        - One URL to remember

Result: Simple client, centralized management
```

**Problems solved:**
- **Complexity**: Clients don't need to know multiple service URLs
- **Security**: Centralized authentication/authorization
- **Duplication**: No need for auth in every service
- **Monitoring**: Single place to log and track all requests
- **Protocol mismatch**: Gateway translates between protocols

## Real-World Analogy

**Airport Security Checkpoint**

```
Passengers (Clients) → Security (API Gateway) → Gates (Services)
                              ↓
                    - Check ID (Authentication)
                    - Verify ticket (Authorization)
                    - Scan bags (Validation)
                    - Direct to gate (Routing)
                    - Limit queue (Rate limiting)
                    - Log everyone (Monitoring)

Without security: Chaos - passengers flood gates directly,
                  no checks, security nightmare
```

## How it works (Food Delivery App Example)

**Scenario**: User taps "Place Order" on food delivery app

### Step 1: Request Reception

API Gateway receives the request as the **single entry point**.

```
POST /api/v1/orders
{
  "userId": "user123",
  "restaurantId": "rest456",
  "items": ["Burger", "Fries"],
  "deliveryAddress": "123 Main St",
  "paymentMethod": "credit_card"
}
Headers:
  Authorization: Bearer eyJhbGc...
  Content-Type: application/json
```

---

### Step 2: Request Validation

Gateway validates format and required fields.

```
Checks:
✓ Content-Type is application/json
✓ All required fields present (userId, items, address)
✓ Data format correct (valid JSON)

If invalid:
  ✗ Return 400 Bad Request
  ✗ Error: "Missing required field: deliveryAddress"
```

---

### Step 3: Authentication & Authorization

Gateway verifies user identity and permissions.

```
1. Extract JWT token from Authorization header
2. Verify token signature
3. Decode user info: { userId: "user123", role: "customer" }
4. Check permission: Can this user place orders?

If valid:
  ✓ Continue to next step

If invalid:
  ✗ 401 Unauthorized: "Invalid or expired token"
  ✗ 403 Forbidden: "User not authorized to place orders"
```

---

### Step 4: Rate Limiting

Gateway checks request frequency to prevent abuse.

```
Check Redis:
  Key: rate_limit:order:user123
  Value: 8 (requests in last minute)

Limit: 10 requests per minute

Current: 8 requests
Action: Allow request (increment to 9)

If user exceeded limit (e.g., 10 requests already):
  ✗ 429 Too Many Requests
  ✗ "Rate limit exceeded. Try again in 30 seconds."
```

**Why important:**
- Prevents accidental spam (user clicking "Place Order" repeatedly)
- Protects against DDoS attacks
- Ensures fair resource usage

---

### Step 5: Request Transformation

Gateway transforms data to match backend requirements.

```
Mobile app sends:
  "deliveryAddress": "123 Main St, New York"

Delivery Service needs GPS coordinates:
  "deliveryLocation": {
    "latitude": 40.7128,
    "longitude": -74.0060
  }

Gateway transformation:
1. Call geocoding API: getCoordinates("123 Main St, New York")
2. Receive: { lat: 40.7128, lng: -74.0060 }
3. Transform request body for Delivery Service
```

**Other transformations:**
- XML → JSON (legacy service integration)
- Add/remove headers
- Rename fields
- Aggregate multiple requests

---

### Step 6: Service Discovery & Routing

Gateway finds and routes to appropriate services.

```
Order requires 4 services:

1. Order Service → Create order record
   Route to: http://order-service:8080/orders

2. Inventory Service → Check item availability
   Route to: http://inventory-service:8081/check

3. Payment Service → Process payment
   Route to: http://payment-service:8082/charge

4. Delivery Service → Assign driver
   Route to: http://delivery-service:8083/assign

Service Discovery:
- Query service registry (Consul, Eureka)
- Find healthy instances
- Load balance across instances (round-robin)
```

**Load Balancing:**
```
Inventory Service instances:
  - Instance 1: 10.0.1.5:8081 (healthy, 50 connections)
  - Instance 2: 10.0.1.6:8081 (healthy, 30 connections)
  - Instance 3: 10.0.1.7:8081 (unhealthy, down)

Gateway routes to Instance 2 (least connections)
```

---

### Step 7: Response Handling

Gateway receives responses and transforms for client.

```
Backend response:
{
  "order_reference": "ORD789",
  "eta": "2024-01-15T18:30:00Z",
  "current_status": "CONFIRMED"
}

Gateway transforms to client format:
{
  "orderId": "ORD789",
  "estimatedDelivery": "6:30 PM",
  "status": "confirmed"
}

Caching (if applicable):
- Cache product catalogs (frequently accessed)
- TTL: 5 minutes
- Next request for same data → serve from cache
```

---

### Step 8: Logging & Monitoring

Gateway logs request details for analytics.

```
Log entry:
{
  "timestamp": "2024-01-15T18:00:00Z",
  "userId": "user123",
  "endpoint": "/api/v1/orders",
  "method": "POST",
  "responseTime": "250ms",
  "statusCode": 201,
  "servicesCalled": ["order", "inventory", "payment", "delivery"],
  "clientIP": "14.98.52.10"
}

Metrics tracked:
- Request count per endpoint
- Average response time
- Error rate (4xx, 5xx)
- Rate limit hits
- Service health
```

---

## Core Features of API Gateway

### 1. Authentication & Authorization

**What:** Centralized security for all services.

**How it works:**
- Verify JWT tokens, API keys, OAuth tokens
- Check user permissions
- Block unauthorized access

**Benefits:**
- Backend services don't handle auth
- Consistent security across all APIs
- Easy to update security policies

**Example:** All services trust gateway - if request reaches them, user is authenticated.

---

### 2. Rate Limiting

**What:** Control request frequency per client.

**Limits:**
```
Free tier: 100 requests/hour
Pro tier: 10,000 requests/hour
Enterprise: Unlimited
```

**Implementation:**
- Track requests in Redis
- Sliding window or token bucket algorithm
- Return 429 when exceeded

**Prevents:**
- DDoS attacks
- Accidental abuse
- Unfair resource usage

---

### 3. Load Balancing

**What:** Distribute requests across service instances.

**Algorithms:**
- Round-robin: Rotate evenly
- Least connections: Route to least busy
- Weighted: Based on instance capacity

**Benefits:**
- No single instance overloaded
- Automatic failover (skip unhealthy instances)
- Better resource utilization

---

### 4. Caching

**What:** Store frequently requested data.

**What to cache:**
- Product catalogs
- User profiles
- Static content
- API responses (GET requests)

**Benefits:**
- Faster response times (cache hit in 5ms vs 100ms backend call)
- Reduced backend load
- Lower costs

**Example:**
```
GET /api/products → Cache for 5 minutes
Next request → Serve from cache (instant)
```

---

### 5. Request/Response Transformation

**What:** Convert data formats between client and services.

**Use cases:**
- Mobile app (JSON) → Legacy service (XML)
- External API (v1 format) → Internal API (v2 format)
- Add default values
- Rename fields

**Example:**
```
Client sends: { "phone": "1234567890" }
Service needs: { "phoneNumber": "+91-1234567890" }
Gateway transforms: Adds country code, renames field
```

---

### 6. Service Discovery

**What:** Dynamically find service instances.

**How:**
- Services register with service registry (Consul, Eureka)
- Gateway queries registry for available instances
- Routes to healthy instances only

**Benefits:**
- Services can scale up/down dynamically
- No hardcoded URLs
- Automatic failover

---

### 7. Circuit Breaking

**What:** Stop sending requests to failing services.

**States:**
```
CLOSED (normal):
  - Send requests normally
  - Track failures

OPEN (service failing):
  - Stop sending requests
  - Return error immediately
  - Prevents cascading failures

HALF-OPEN (testing):
  - Send limited requests
  - If successful → CLOSED
  - If failed → OPEN
```

**Example:**
```
Payment Service failing (5xx errors)
After 10 failures → Circuit OPEN
Return to client: "Payment service temporarily unavailable"
Prevents: Wasting time on failing service
```

---

### 8. Logging & Monitoring

**What:** Track all API activity.

**Logs:**
- Request/response details
- Response times
- Errors and exceptions
- User actions

**Metrics:**
- Requests per second
- Error rates (4xx, 5xx)
- Latency (p50, p95, p99)
- Top endpoints

**Integration:**
- Prometheus (metrics)
- Grafana (dashboards)
- ELK Stack (logs)
- AWS CloudWatch

---

## Where it is used in real systems

**Cloud API Gateways:**
- **AWS API Gateway**: Managed service, integrates with Lambda
- **Google Cloud API Gateway**: GCP managed service
- **Azure API Management**: Microsoft's offering

**Open Source:**
- **Kong**: Built on Nginx, plugin-based
- **Apigee**: Google's API management platform
- **Tyk**: Lightweight, Go-based
- **AWS App Mesh**: Service mesh for microservices

**Use Cases:**
- **E-commerce**: Route /users → User Service, /orders → Order Service
- **Banking**: Centralized authentication, rate limiting
- **Social Media**: Load balancing, caching user feeds
- **IoT**: Protocol translation (MQTT → HTTP)

---

## API Gateway vs Reverse Proxy vs Load Balancer

| Feature | API Gateway | Reverse Proxy | Load Balancer |
|---------|-------------|---------------|---------------|
| **Routing** | Advanced (path, header-based) | Basic URL routing | Server selection only |
| **Authentication** | Yes (JWT, OAuth) | No | No |
| **Rate Limiting** | Yes | Limited | No |
| **Transformation** | Yes | No | No |
| **Caching** | Yes | Yes | No |
| **Use Case** | Microservices APIs | Web servers | Traffic distribution |
| **Example** | Kong, AWS API Gateway | Nginx | HAProxy, AWS NLB |

**Relationship:** API Gateway = Reverse Proxy + Authentication + Rate Limiting + More features

---

## Interview Explanation (2-minute answer)

*"An API Gateway is a single entry point for all client requests to backend microservices. Instead of clients calling multiple services directly, they send requests to the gateway which handles routing, security, and other concerns.*

*For example, in a food delivery app, when you tap 'Place Order,' the request goes to the API Gateway. The gateway first validates the request format, then authenticates your JWT token, and checks rate limits to prevent abuse—say, 10 orders per minute maximum.*

*Next, it performs service discovery to find healthy instances of the Order Service, Inventory Service, Payment Service, and Delivery Service. It routes requests to these services using load balancing—perhaps sending to the instance with fewest connections.*

*The gateway can also transform requests. If the mobile app sends an address in plain text but the Delivery Service needs GPS coordinates, the gateway converts it before forwarding. Similarly, it transforms responses—converting the backend's order_reference field to orderId for the client.*

*Throughout this process, the gateway logs everything—request times, endpoints called, errors—enabling monitoring and debugging. If a service starts failing, circuit breaking kicks in, temporarily stopping requests to that service and returning errors immediately instead of waiting for timeouts.*

*Key features include authentication and authorization, rate limiting to prevent abuse, caching for frequently accessed data like product catalogs, and service discovery for dynamic routing. Popular solutions include AWS API Gateway for managed services, and Kong or Apigee for self-hosted options.*

*In system design, API Gateway centralizes cross-cutting concerns. Instead of implementing authentication in every microservice, you do it once at the gateway. This simplifies backend services and provides consistent security, logging, and routing across your entire API ecosystem."*

---

## Top Interview Questions

### 1. What is an API Gateway and why is it needed?
**Answer:**

Single entry point for all client requests to microservices.

**Why needed:**
- **Simplify clients**: One URL instead of many
- **Centralized security**: Auth in one place
- **Cross-cutting concerns**: Rate limiting, logging, caching
- **Service abstraction**: Clients don't know backend structure

**Example:** Food delivery app → API Gateway → Order, Payment, Delivery services

### 2. What are the main features of API Gateway?
**Answer:**

1. **Authentication/Authorization**: Verify users (JWT, OAuth)
2. **Rate Limiting**: Prevent abuse (100 req/min)
3. **Load Balancing**: Distribute across instances
4. **Caching**: Store frequent responses
5. **Request Transformation**: Format conversion (XML→JSON)
6. **Service Discovery**: Find healthy instances
7. **Circuit Breaking**: Stop calling failing services
8. **Logging/Monitoring**: Track all requests

### 3. How does API Gateway handle authentication?
**Answer:**

**Process:**
1. Client sends request with token (JWT, API key)
2. Gateway extracts and verifies token
3. Checks user permissions
4. If valid: Forward to backend
5. If invalid: Return 401/403

**Benefits:**
- Backend services trust gateway
- No auth code in every service
- Easy to update auth logic

**Example:**
```
Authorization: Bearer eyJhbGc...
Gateway validates JWT → Extracts userId
Forwards request with X-User-Id header to backend
```

### 4. Explain rate limiting in API Gateway
**Answer:**

**What:** Limit requests per user/API key in time window.

**Example:**
```
User: 100 requests/minute
Request 101 within minute → 429 Too Many Requests
```

**Implementation:**
- Track in Redis: `rate_limit:user:123 = 95`
- Increment on each request
- Reset after time window

**Why:**
- Prevent DDoS
- Fair resource usage
- Enforce pricing tiers (free vs paid)

### 5. What is circuit breaking and why is it important?
**Answer:**

**What:** Stop calling failing services temporarily.

**States:**
```
CLOSED → Service healthy, send requests
  ↓ (After 10 failures)
OPEN → Service failing, reject immediately
  ↓ (After cooldown period)
HALF-OPEN → Test with limited requests
  ↓
CLOSED or OPEN (based on result)
```

**Why important:**
- Prevents cascading failures
- Fast fail (don't wait for timeout)
- Gives service time to recover

**Example:** Payment service down → Circuit OPEN → Return "Service unavailable" immediately

### 6. How does API Gateway differ from a Load Balancer?
**Answer:**

| Feature | API Gateway | Load Balancer |
|---------|-------------|---------------|
| **Purpose** | API management | Traffic distribution |
| **Authentication** | Yes | No |
| **Rate Limiting** | Yes | No |
| **Routing** | Path/header-based | Server selection only |
| **Transformation** | Yes | No |
| **Layer** | Layer 7 (Application) | Layer 4/7 |

**Use together:** Load Balancer → API Gateway → Backend services

### 7. What is request transformation in API Gateway?
**Answer:**

**What:** Modify request/response format.

**Use cases:**

**1. Format conversion:**
```
Client (JSON) → Gateway → Legacy service (XML)
```

**2. Field transformation:**
```
Client: { "phone": "1234567890" }
Service needs: { "phoneNumber": "+1-1234567890" }
Gateway adds country code
```

**3. Protocol translation:**
```
HTTP REST → gRPC
```

**Example:** Mobile app sends address text → Gateway converts to GPS coordinates for Delivery Service

### 8. How does service discovery work with API Gateway?
**Answer:**

**Process:**
1. Services register with registry (Consul, Eureka)
2. Gateway queries registry: "Where is Order Service?"
3. Registry returns: `[10.0.1.5:8080, 10.0.1.6:8080]`
4. Gateway load balances across instances
5. Routes request to selected instance

**Benefits:**
- Services scale dynamically
- No hardcoded URLs
- Automatic failover

**Example:**
```
Order Service scales from 3 to 5 instances
Gateway automatically discovers new instances
Starts routing traffic to all 5
```

### 9. What metrics should you monitor in API Gateway?
**Answer:**

**Traffic Metrics:**
- Requests per second
- Requests per endpoint
- Top clients (by API key/IP)

**Performance Metrics:**
- Latency (p50, p95, p99)
- Response times per service
- Cache hit rate

**Error Metrics:**
- 4xx errors (client errors)
- 5xx errors (server errors)
- Rate limit hits
- Circuit breaker triggers

**Health Metrics:**
- Healthy service instances
- Failed health checks
- Service availability

**Tools:** Prometheus, Grafana, CloudWatch, ELK Stack

### 10. How would you design API Gateway for microservices?
**Answer:**

**Architecture:**
```
Clients → API Gateway (Kong/AWS API Gateway)
            ↓
      Path-based routing:
      /users/* → User Service
      /orders/* → Order Service
      /payments/* → Payment Service
            ↓
      Service Registry (Consul)
            ↓
      Backend Services (multiple instances)
```

**Key decisions:**

**1. Authentication:**
- JWT tokens
- Validate at gateway
- Pass user context to services

**2. Rate Limiting:**
- Per API key: 1000 req/hour
- Per IP: 100 req/min

**3. Caching:**
- Cache GET requests
- TTL based on endpoint (products: 5min, orders: no cache)

**4. Load Balancing:**
- Round-robin for stateless services
- Sticky sessions for stateful

**5. Monitoring:**
- Log all requests
- Track latency, errors
- Dashboards for real-time monitoring

---

## Points to Impress the Interviewer

- **"API Gateway is reverse proxy + authentication + rate limiting + more"** — Know the relationship
- **"Circuit breaking prevents cascading failures"** — Critical for resilience
- **"Service discovery enables dynamic scaling"** — No hardcoded URLs
- **"Centralized concerns simplify microservices"** — Auth once, not in every service
- **"Cache at gateway for performance"** — Reduce backend load
- **"Transform for compatibility"** — Mobile (JSON) → Legacy (XML)

**Real-world insight:** "Netflix's Zuul API Gateway handles billions of requests daily, routing to 500+ microservices. It provides authentication, dynamic routing, circuit breaking, and monitoring—all without touching individual services."

**System design context:** "In e-commerce design: Cloudflare (DDoS protection) → AWS API Gateway (auth, rate limiting) → Kong (internal routing) → Microservices. Multi-layer approach for security and performance."

---