# Load Balancing

## What is this concept?

**Load Balancing** is the process of distributing incoming network traffic across multiple servers to ensure no single server becomes overwhelmed.

**Core Idea**: Instead of one server handling all requests, spread the load across many servers.

**Think of it as**: Multiple checkout counters at a supermarket instead of just one - customers are distributed to avoid long wait times.

## Why does this exist? / Problem it solves

- **Single server limits**: One server can only handle limited requests (e.g., 1,000 req/s)
- **High availability**: If one server crashes, others continue serving
- **Scalability**: Add more servers to handle more traffic
- **Performance**: Distribute load prevents server overload
- **Geographic distribution**: Route users to nearest server for lower latency

Without load balancing, your single server would crash under high traffic (Black Friday, viral posts).

## Real-World Analogy

**Airport Security Checkpoints**

```
Without Load Balancing (1 checkpoint):
100 passengers → 1 Security Lane
Wait time: 50 minutes ❌
If lane closes: Complete shutdown ❌

With Load Balancing (5 checkpoints):
100 passengers → 5 Security Lanes (20 each)
Wait time: 10 minutes ✓
If 1 lane closes: Redistribute to 4 lanes ✓
```

**Load Balancer** = Security officer directing passengers to shortest line.

## How it works (High Level)

### Basic Architecture

```
                    Load Balancer
                         |
        +----------------+----------------+
        |                |                |
    Server 1         Server 2         Server 3
    (10.0.1.5)      (10.0.1.6)      (10.0.1.7)
```

### Request Flow

```
1. User requests: www.example.com
2. DNS resolves to: Load Balancer IP (203.0.113.50)
3. Load Balancer receives request
4. Load Balancer chooses server based on algorithm
5. Forwards request to chosen server (e.g., Server 2)
6. Server 2 processes and returns response
7. Load Balancer sends response back to user
```

## Types of Load Balancers

### 1. Hardware Load Balancers

**Physical devices** dedicated to load balancing.

**Examples:**
- F5 BIG-IP
- Citrix ADC (NetScaler)
- A10 Thunder

**Characteristics:**
- Extremely high performance (millions of connections)
- Expensive ($10,000 - $100,000+)
- Complex setup and maintenance
- Dedicated hardware appliances

**Pros:**
- Best performance and throughput
- Advanced features (DDoS protection, WAF)
- Vendor support and SLAs

**Cons:**
- Very expensive
- Difficult to scale (need to buy more hardware)
- Vendor lock-in
- Over-provisioned (pay for peak capacity)

**When to use:** Large enterprises, financial institutions, telecom

---

### 2. Software Load Balancers

**Software running on standard servers** (virtual or physical).

**Examples:**
- **Nginx** (most popular, used by Netflix, Airbnb)
- **HAProxy** (high performance, used by Reddit, GitHub)
- **Apache mod_proxy**
- **Traefik** (cloud-native, container-aware)
- **Envoy** (modern, used in service meshes)

**Characteristics:**
- Run on commodity hardware or VMs
- Cost-effective (open-source options)
- Flexible and programmable
- Easy to scale (spin up more instances)

**Pros:**
- Much cheaper (often free/open-source)
- Easy to scale horizontally
- Flexible configuration
- Run anywhere (on-premise, cloud, containers)

**Cons:**
- Lower performance than hardware (but still very high)
- Requires more operational expertise
- Need to manage servers yourself

**When to use:** Startups, mid-size companies, modern cloud architectures

---

### 3. Cloud Load Balancers (Managed Services)

**Fully managed load balancing** provided by cloud platforms.

**AWS Options:**
- **Application Load Balancer (ALB)** - Layer 7, HTTP/HTTPS
- **Network Load Balancer (NLB)** - Layer 4, TCP/UDP
- **Classic Load Balancer (CLB)** - Legacy, both L4/L7

**Google Cloud:**
- **HTTP(S) Load Balancer** - Global, Layer 7
- **Network Load Balancer** - Regional, Layer 4
- **Internal Load Balancer** - Private networks

**Azure:**
- **Azure Load Balancer** - Layer 4
- **Application Gateway** - Layer 7
- **Traffic Manager** - DNS-based global routing

**Characteristics:**
- Fully managed (no servers to manage)
- Auto-scaling built-in
- High availability by default
- Pay-per-use pricing

**Pros:**
- Zero operational overhead
- Automatically redundant and scalable
- Integrated with cloud services
- Global reach (geo-distribution)
- Pay only for what you use

**Cons:**
- Vendor lock-in
- Can be expensive at scale
- Less control over configuration
- Limited customization

**When to use:** Most cloud-based applications, startups, teams without ops expertise

---

## Layer-Based Load Balancer Types

### Layer 4 Load Balancer (Network/Transport Layer)

**What it does:** Routes traffic based on **IP address and port** only.

**How it works:**
```
Client: 14.98.52.10:52341 → Server: 203.0.113.50:80
Load balancer sees: Source IP, Dest IP, Port
Routes to: Backend server based on these alone
```

**Characteristics:**
- Doesn't inspect packet content
- Works with any protocol (HTTP, FTP, SMTP, gaming)
- Very fast (no content processing)
- Simple routing decisions

**Examples:**
- AWS Network Load Balancer (NLB)
- HAProxy in TCP mode
- Azure Load Balancer

**Pros:**
- Extremely fast (millions of req/s)
- Protocol-agnostic (works with anything)
- Low latency
- Simple and reliable

**Cons:**
- Can't route based on URL/content
- No SSL termination
- Limited routing intelligence
- No content-based decisions

**Use cases:**
- Gaming servers (low latency critical)
- Database connections
- Non-HTTP protocols
- Raw performance needed

---

### Layer 7 Load Balancer (Application Layer)

**What it does:** Routes traffic based on **HTTP content** (URL, headers, cookies).

**How it works:**
```
Request: GET /api/users HTTP/1.1
         Host: example.com
         Cookie: session=abc123

Load balancer inspects:
- URL path: /api/users → Route to API servers
- Host header: example.com → Route to example backend
- Cookie: session=abc123 → Sticky session to specific server
```

**Characteristics:**
- Inspects HTTP requests
- Content-based routing
- SSL termination
- Advanced features

**Examples:**
- AWS Application Load Balancer (ALB)
- Nginx
- HAProxy in HTTP mode
- Google Cloud HTTP(S) Load Balancer

**Pros:**
- Smart routing (URL, headers, cookies)
- SSL termination (offload from backends)
- Content caching
- Request modification (add headers)
- Path-based routing (/api → API servers, /static → CDN)

**Cons:**
- Slower than Layer 4 (content inspection overhead)
- HTTP/HTTPS only
- More complex configuration
- Higher CPU usage

**Use cases:**
- Web applications
- REST APIs
- Microservices routing
- Multi-tenant applications

---

### Comparison: Layer 4 vs Layer 7

| Feature | Layer 4 (NLB) | Layer 7 (ALB) |
|---------|---------------|---------------|
| **OSI Layer** | Transport | Application |
| **Speed** | Very fast (~millions req/s) | Slower (~100k req/s) |
| **Routing** | IP + Port only | URL, headers, cookies |
| **Protocols** | Any (TCP, UDP) | HTTP, HTTPS |
| **SSL Termination** | No (pass-through) | Yes |
| **Content Inspection** | No | Yes |
| **Latency** | Ultra-low (<1ms) | Low (~5-10ms) |
| **Cost** | Lower | Higher |
| **Complexity** | Simple | Complex |
| **Use Case** | Gaming, databases | Web apps, APIs |

---

## Global Load Balancing (DNS-based)

**What it does:** Routes users to nearest data center using DNS.

**How it works:**
```
User in India: "What's netflix.com IP?"
DNS: "Use 52.84.124.80 (Mumbai region)"

User in USA: "What's netflix.com IP?"
DNS: "Use 54.239.28.85 (Virginia region)"
```

**Examples:**
- AWS Route 53
- Cloudflare Load Balancing
- Azure Traffic Manager
- Google Cloud Load Balancer (global)

**Routing strategies:**
1. **Geolocation**: Based on user's physical location
2. **Latency-based**: Lowest latency to data center
3. **Weighted**: Percentage split (90% US, 10% EU)
4. **Failover**: Primary region, fallback to secondary

**Pros:**
- Global scale
- Disaster recovery (region failover)
- Lowest latency for users
- Geographic compliance (data residency)

**Cons:**
- DNS caching delays (changes take time to propagate)
- Coarse-grained (can't do per-request routing)
- Limited to DNS TTL for failover speed

**Use cases:**
- Global applications (Netflix, Facebook)
- Multi-region deployments
- Disaster recovery
- CDN routing

---

## Load Balancing Algorithms

### 1. Round Robin (Default)

**How**: Distribute requests in circular order.

```
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1 (repeat)
```

**Pros:** Simple, fair distribution
**Cons:** Ignores server load
**Best for:** Uniform servers, similar request times

---

### 2. Least Connections

**How**: Send to server with fewest active connections.

```
Server 1: 10 active connections
Server 2: 5 active connections  ← Choose this
Server 3: 8 active connections
```

**Pros:** Balances actual load
**Cons:** Requires connection tracking
**Best for:** Long-lived connections (WebSockets, DB)

---

### 3. Weighted Round Robin

**How**: Servers get traffic proportional to their capacity.

```
Server 1 (weight=5): Gets 50% of traffic
Server 2 (weight=3): Gets 30% of traffic
Server 3 (weight=2): Gets 20% of traffic
```

**Pros:** Handles heterogeneous servers
**Cons:** Manual weight configuration
**Best for:** Mixed server capacities

---

### 4. IP Hash (Sticky Sessions)

**How**: Hash client IP to determine server.

```
hash(14.98.52.10) % 3 = 2 → Server 2 (always)
hash(203.0.113.5) % 3 = 1 → Server 1 (always)
```

**Pros:** Session persistence without cookies
**Cons:** Uneven distribution possible
**Best for:** Stateful applications, shopping carts

---

### 5. Least Response Time

**How**: Send to server with fastest response.

```
Server 1: 50ms average
Server 2: 30ms average ← Choose this
Server 3: 45ms average
```

**Pros:** Optimizes user experience
**Cons:** Complex monitoring needed
**Best for:** Geo-distributed servers

---

### 6. Random

**How**: Randomly select a server.

```
Random() → Server 2
Random() → Server 1
Random() → Server 3
```

**Pros:** Simple, no state needed
**Cons:** Less predictable
**Best for:** Stateless, homogeneous servers

---

## Health Checks

**Purpose:** Detect and remove unhealthy servers automatically.

### Active Health Checks

Load balancer actively probes servers:

```
Every 10 seconds:
Load Balancer → GET /health → Server

Response 200 OK → Healthy ✓
Timeout or 5xx → Unhealthy ✗

After 3 consecutive failures → Remove from pool
After 2 consecutive successes → Add back to pool
```

**Configuration:**
- **Interval**: How often to check (5-30 seconds)
- **Timeout**: Max wait time (2-10 seconds)
- **Unhealthy threshold**: Failures before marking down (2-3)
- **Healthy threshold**: Successes before marking up (2-3)

### Passive Health Checks

Monitor actual request responses:

```
Track real traffic:
5xx errors → Count failures
Timeouts → Count failures
200 responses → Count successes

If failure rate > 50% → Mark unhealthy
```

**Benefits:**
- No extra traffic (uses real requests)
- Faster detection (immediate on errors)

---

## Session Persistence (Sticky Sessions)

**Problem:** User session on Server 1, next request to Server 2 → session lost.

### Solution 1: Cookie-Based Stickiness

```
1. User's first request → Server 1
2. Load balancer sets cookie: LB_SERVER=server1
3. Future requests → Read cookie → Always route to Server 1
```

**Pros:** Reliable, survives server restart
**Cons:** Reduces load balancing effectiveness

### Solution 2: IP-Based Stickiness

```
Hash(client_ip) → Always same server
```

**Pros:** No cookies needed
**Cons:** NAT causes many users → same server

### Solution 3: Shared Session Store (Best)

```
All servers → Redis/Memcached for sessions
Any server can handle any request
No stickiness needed
```

**Pros:** True load balancing, no affinity needed
**Cons:** Extra infrastructure (Redis)

---

## SSL/TLS Termination

**What:** Load balancer decrypts HTTPS, forwards HTTP to backends.

```
Client (HTTPS) → Load Balancer (decrypt) → Backend (HTTP)
```

**Benefits:**
- **Performance**: Offload CPU-intensive encryption from app servers
- **Simplicity**: Backend doesn't need SSL certificates
- **Centralized**: Manage certificates in one place
- **Cheaper**: Fewer SSL certificates needed

**Security note:** Use encrypted connection (TLS) between LB and backend in production.

---

## Where it is used in real systems

### Cloud Providers
- **AWS**: ALB (Layer 7), NLB (Layer 4), Route 53 (DNS)
- **Google Cloud**: HTTP(S) LB (global), Network LB (regional)
- **Azure**: Application Gateway (L7), Load Balancer (L4)

### Software
- **Nginx**: Netflix, Airbnb, WordPress.com
- **HAProxy**: Reddit, GitHub, Stack Overflow
- **Envoy**: Uber, Lyft (service mesh)
- **Traefik**: Kubernetes, Docker Swarm

### Hardware
- **F5 BIG-IP**: Banks, telecom, large enterprises
- **Citrix ADC**: Healthcare, government, finance

---

## Common Mistakes / Misconceptions

1. **"Load balancer solves all scaling problems"** ❌
   - Database can still be bottleneck
   - Need to scale entire stack

2. **"One load balancer is enough"** ❌
   - LB itself is single point of failure
   - Need redundancy or managed service

3. **"Layer 7 is always better"** ❌
   - Layer 4 is faster, use when appropriate
   - Gaming, databases benefit from L4

4. **"Sticky sessions are required"** ❌
   - Better to use shared session store (Redis)
   - Sticky sessions reduce flexibility

5. **"All algorithms work the same"** ❌
   - Round robin ≠ Least connections
   - Choose based on workload

---

## Interview Explanation (2-minute answer)

*"Load balancing distributes traffic across multiple servers to prevent overload and ensure high availability. There are several types to understand.*

*First, we have hardware, software, and cloud load balancers. Hardware load balancers like F5 are physical devices—extremely fast but very expensive, used by large enterprises. Software load balancers like Nginx and HAProxy run on standard servers—cheaper and flexible, used by most companies. Cloud load balancers like AWS ALB are fully managed services—zero operations, pay-per-use, most common today.*

*Second, there's Layer 4 versus Layer 7. Layer 4 load balancers operate at the transport layer, routing based only on IP and port—very fast, works with any protocol, used for gaming or databases. Layer 7 load balancers operate at the application layer, routing based on HTTP content like URLs and headers—enables smart routing and SSL termination, used for web applications.*

*Common algorithms include Round Robin for simple circular distribution, Least Connections for routing to the server with fewest active connections, and IP Hash for session stickiness. The choice depends on your workload characteristics.*

*Health checks are critical—load balancers periodically probe servers and automatically remove unhealthy ones. This provides automatic failover without manual intervention.*

*For global applications, DNS load balancing routes users to the nearest region. AWS Route 53 or Cloudflare can direct users in India to Mumbai servers and US users to Virginia servers based on geolocation or latency.*

*A complete architecture might look like: DNS load balancing for regions → Layer 7 ALB for smart routing → backend servers. This provides geographic distribution, intelligent routing, and automatic failover at multiple levels."*

---

## Top Interview Questions

### 1. What are the main types of load balancers?
**Answer:**

**By Implementation:**
- **Hardware**: F5, Citrix (expensive, high performance)
- **Software**: Nginx, HAProxy (flexible, cost-effective)
- **Cloud**: AWS ALB/NLB (managed, zero ops)

**By OSI Layer:**
- **Layer 4**: Network/Transport (IP+Port, fast, any protocol)
- **Layer 7**: Application (HTTP, smart routing, SSL termination)

**By Scope:**
- **Local**: Single data center
- **Global**: DNS-based, multi-region

### 2. When would you use Layer 4 vs Layer 7 load balancer?
**Answer:**

**Use Layer 4 when:**
- Need raw speed (gaming servers)
- Non-HTTP protocols (databases, email)
- Simple routing sufficient
- Ultra-low latency required

**Use Layer 7 when:**
- Need content-based routing (/api → API servers)
- SSL termination required
- Microservices architecture
- Path/host-based routing

**Example:** Use NLB (L4) for database connections, ALB (L7) for web application.

### 3. Explain common load balancing algorithms.
**Answer:**

1. **Round Robin**: Circular distribution (simple, default)
2. **Least Connections**: Route to server with fewest connections (WebSockets)
3. **Weighted Round Robin**: Based on capacity (3x traffic to powerful server)
4. **IP Hash**: Same client → same server (sticky sessions)
5. **Least Response Time**: Fastest server (geo-distributed)

**Choose based on:**
- Uniform servers → Round Robin
- Long connections → Least Connections
- Mixed capacities → Weighted
- Sessions needed → IP Hash

### 4. How do health checks work?
**Answer:**

**Active Health Checks:**
```
Every 10s: LB → GET /health → Server
200 OK → Healthy ✓
Timeout/5xx → Unhealthy ✗

3 failures → Remove from pool
2 successes → Add back
```

**Configuration:**
- Interval: 5-30 seconds
- Timeout: 2-10 seconds
- Unhealthy threshold: 2-3 failures
- Healthy threshold: 2 successes

**Benefit:** Automatic failover without manual intervention.

### 5. What is SSL termination and why use it?
**Answer:**

**SSL Termination**: Load balancer decrypts HTTPS, forwards HTTP to backend.

```
Client (HTTPS) → LB (decrypt SSL) → Backend (HTTP)
```

**Benefits:**
- Offload CPU-intensive encryption from app servers
- Centralized certificate management
- Fewer SSL certificates needed
- Backend servers simpler

**Note:** In production, use TLS between LB and backend for security.

### 6. How do you handle session persistence?
**Answer:**

**Problem:** User session on Server 1, next request to Server 2 → lost session.

**Solutions:**

**1. Sticky Sessions (Cookie):**
- LB sets cookie with server ID
- Routes to same server based on cookie

**2. IP Hash:**
- Hash client IP → always same server

**3. Shared Session Store (Best):**
- All servers use Redis/Memcached
- Any server can handle any request
- True load balancing maintained

**Recommendation:** Use shared session store for better scalability.

### 7. What if the load balancer itself fails?
**Answer:**

**Problem:** Load balancer is single point of failure.

**Solutions:**

**1. Multiple Load Balancers:**
- DNS round-robin between 2+ LBs
- If one fails, DNS routes to others

**2. Managed Cloud Services:**
- AWS ELB, Cloudflare automatically redundant
- Built-in high availability

**3. Active-Passive with Floating IP:**
- Primary LB handles traffic
- Secondary takes over if primary fails (keepalived)

**Best practice:** Use managed cloud load balancers for automatic redundancy.

### 8. What are the differences between hardware, software, and cloud load balancers?
**Answer:**

| Aspect | Hardware | Software | Cloud |
|--------|----------|----------|-------|
| **Cost** | Very high ($10K-$100K+) | Low (open-source) | Pay-per-use |
| **Scalability** | Buy more hardware | Add more VMs | Auto-scales |
| **Performance** | Best (millions req/s) | High (100K+ req/s) | High (managed) |
| **Flexibility** | Limited | Very flexible | Moderate |
| **Operations** | Complex | Manual setup | Zero ops |
| **Examples** | F5, Citrix | Nginx, HAProxy | AWS ALB/NLB |
| **Best for** | Banks, telecom | Custom needs | Most use cases |

**Modern trend:** Cloud load balancers for most applications, software for special requirements.

### 9. How does global load balancing work?
**Answer:**

**DNS-based routing** directs users to nearest/best data center.

```
User in India:
DNS query: netflix.com
Response: 52.84.124.80 (Mumbai region)

User in USA:
DNS query: netflix.com
Response: 54.239.28.85 (Virginia region)
```

**Routing strategies:**
1. **Geolocation**: Based on user's country/region
2. **Latency-based**: Lowest network latency
3. **Weighted**: Percentage distribution (90% US, 10% EU)
4. **Failover**: Primary region, secondary if down

**Examples:** AWS Route 53, Cloudflare, Azure Traffic Manager

**Use cases:** 
- Global applications (Netflix, Facebook)
- Disaster recovery (region failover)
- Compliance (data residency requirements)

### 10. Explain Round Robin vs Least Connections algorithm.
**Answer:**

**Round Robin:**
```
Req 1 → Server 1
Req 2 → Server 2
Req 3 → Server 3
Req 4 → Server 1 (cycle repeats)
```
- **Pros**: Simple, predictable, fair
- **Cons**: Ignores server load
- **Best for**: Similar request times, stateless apps

**Least Connections:**
```
Server 1: 15 connections
Server 2: 8 connections  ← Choose this (least)
Server 3: 12 connections
```
- **Pros**: Balances actual load
- **Cons**: Requires connection tracking
- **Best for**: Long-lived connections (WebSockets, streaming)

**Example:** Chat app with WebSockets → use Least Connections (connections last minutes/hours).

### 11. What is the difference between Active and Passive health checks?
**Answer:**

**Active Health Checks:**
- Load balancer **actively probes** servers
- Sends synthetic requests (GET /health)
- Configured intervals (e.g., every 10s)
- Proactive detection

```
LB → GET /health → Server
Response 200 → Healthy ✓
Timeout → Unhealthy ✗
```

**Passive Health Checks:**
- Monitor **real traffic** responses
- No extra probe requests
- React to actual failures
- Faster detection (immediate)

```
Real request → 5xx error → Count failure
Multiple failures → Mark unhealthy
```

**Best practice:** Use both together for comprehensive monitoring.

### 12. How do you distribute traffic 80-20 between two versions?
**Answer:**

**Use Weighted Round Robin for A/B testing or canary deployments:**

```
Version A (stable): Weight = 80
Version B (new): Weight = 20

Result: 80% traffic to A, 20% to B
```

**Implementation:**
- **Nginx**: `server backend-a weight=80; server backend-b weight=20;`
- **AWS ALB**: Target group weights (80% to TG-A, 20% to TG-B)
- **HAProxy**: `weight 80` and `weight 20` in backend servers

**Use cases:**
- Canary deployments (test new version with small traffic)
- Blue-green deployments (gradual traffic shift)
- A/B testing (different features to user segments)

**Example:** Deploy new payment service version to 10% users, monitor errors, then increase to 100%.

### 13. What metrics should you monitor for load balancers?
**Answer:**

**Traffic Metrics:**
- **Request count**: Total requests per second
- **Active connections**: Current concurrent connections
- **New connections/sec**: Rate of new connections

**Performance Metrics:**
- **Latency**: Response time (p50, p95, p99)
- **Queue depth**: Requests waiting to be processed
- **Backend response time**: How long backends take

**Health Metrics:**
- **Healthy host count**: Number of healthy backends
- **Unhealthy host count**: Failed health checks
- **Health check failures**: Rate of health check fails

**Error Metrics:**
- **4xx errors**: Client errors (bad requests)
- **5xx errors**: Backend/LB errors (system issues)
- **Connection errors**: Failed connections to backends
- **Timeout errors**: Backends not responding in time

**Alerts to set:**
- Healthy hosts < 50% of total → Critical
- p99 latency > SLA threshold → Warning
- 5xx error rate > 1% → Critical
- All backends unhealthy → Critical

**Tools:** CloudWatch (AWS), Prometheus, Datadog, New Relic

### 14. How does SSL/TLS termination work at load balancer?
**Answer:**

**Without SSL Termination:**
```
Client (HTTPS) → LB (pass-through) → Backend (HTTPS)
- Backend handles SSL (CPU intensive)
- Need SSL cert on every backend
```

**With SSL Termination:**
```
Client (HTTPS) → LB (terminates SSL) → Backend (HTTP)
- LB decrypts HTTPS
- Forwards plain HTTP to backend
- Backend doesn't need SSL
```

**Benefits:**
1. **Performance**: Offload CPU-intensive encryption from app servers
2. **Simplicity**: Backends don't need SSL certificates
3. **Centralized certs**: Manage certificates in one place
4. **Cost**: Need fewer SSL certificates

**Security consideration:**
- In production, use TLS between LB and backend (end-to-end encryption)
- Or ensure backends are on private network

**Modern approach:** SSL termination at LB, mTLS (mutual TLS) between LB and backends.

### 15. What is the difference between stateful and stateless load balancing?
**Answer:**

**Stateful Load Balancing:**
- Maintains connection/session information
- Tracks which client goes to which server
- Requires memory/storage
- Enables sticky sessions

```
Client A → Server 1 (remembered)
Client A again → Server 1 (same)
```

**Examples:** IP Hash, cookie-based stickiness

**Stateless Load Balancing:**
- No memory of past requests
- Each request independent
- No state stored
- Purely algorithmic

```
Client A → Server 1 (round robin)
Client A again → Server 2 (next in rotation)
```

**Examples:** Pure Round Robin, Random

**Trade-offs:**
- Stateful: Better for sessions, less flexible scaling
- Stateless: Better for scaling, requires external session store

**Modern best practice:** Use stateless LB + Redis for sessions (best of both).

### 16. How would you design load balancing for a microservices architecture?
**Answer:**

**Multi-layer approach:**

```
External Users
    ↓
API Gateway / Edge Load Balancer (Layer 7)
  - SSL termination
  - Rate limiting
  - Authentication
  - Routing: /users → User Service
            /orders → Order Service
    ↓
Service Mesh (Envoy/Istio)
  - Service-to-service load balancing
  - Circuit breaking
  - Retries and timeouts
  - mTLS between services
    ↓
Individual Microservices
```

**Key components:**

1. **External LB (AWS ALB/Nginx):**
   - Public-facing
   - SSL termination
   - Path-based routing to services

2. **API Gateway (Kong/Apigee):**
   - Authentication/Authorization
   - Rate limiting per client
   - Request transformation

3. **Service Mesh (Envoy/Istio):**
   - Internal service-to-service LB
   - Advanced traffic management
   - Observability and tracing

4. **Service Discovery (Consul/Eureka):**
   - Dynamic service registration
   - LB automatically updates backend list

**Example flow:**
```
User → ALB → API Gateway → 
  → User Service (via service mesh) → Database
  → User Service calls Order Service (via service mesh)
```

**Benefits:**
- Centralized routing and policies
- Service-to-service resilience
- No hardcoded service URLs
- Automatic failover at every level

---

## Cross-Questions Interviewer May Ask

**"How does DNS load balancing differ from L4/L7?"**
"DNS load balancing happens at domain resolution—different users get different IPs based on location. It's coarse-grained and slow to update (DNS caching). L4/L7 load balancers operate per-request—every request can go to a different server. They're complementary: DNS routes to nearest region, then L7 load balances within that region."

**"What's the difference between load balancing and auto-scaling?"**
"Load balancing distributes traffic across existing servers. Auto-scaling adds/removes servers based on load. They work together: auto-scaling creates servers, load balancer distributes traffic to them. AWS Auto Scaling Groups automatically register new instances with the load balancer."

**"How do you choose health check intervals?"**
"Balance between detection speed and overhead. Too frequent (1s) wastes resources. Too infrequent (60s) delays failure detection. Standard is 10-30s. For critical services, use 5-10s. Consider: how quickly you need to detect failures vs. how much probe traffic is acceptable."

**"Can load balancers handle WebSocket connections?"**
"Yes, but use Layer 7 load balancers with WebSocket support (AWS ALB, Nginx). Use Least Connections algorithm since WebSockets are long-lived. Enable sticky sessions or use consistent hashing so the same client maintains connection to same server. Health checks must account for idle connections."

**"What happens when you add a new server to the pool?"**
"With Round Robin, new server immediately gets 1/N traffic. With Weighted, assign appropriate weight (start low, increase). Health checks verify it's healthy before routing traffic. Some load balancers support 'warm-up' where new servers get gradually increasing traffic to build caches and connections."

---

## Points to Impress the Interviewer

- **"Layer 4 for speed, Layer 7 for intelligence"** — Know trade-offs
- **"Hardware is legacy, cloud is modern"** — Understand industry shift
- **"Health checks enable automatic failover"** — Zero manual intervention
- **"Shared session store > sticky sessions"** — Better scalability
- **"Multi-layer LB for global apps"** — DNS → L7 → servers
- **"LB itself needs redundancy"** — Avoid SPOF

**Real-world insight:** "Netflix uses multi-tier load balancing: Route 53 for geo-routing → ELB per region → Zuul (API gateway) for microservices → thousands of servers. Each layer serves a purpose: global distribution, regional failover, intelligent routing, and server distribution."

**System design context:** "For e-commerce design: Cloudflare (DDoS + global LB) → AWS ALB (SSL termination, path routing /api vs /static) → Application servers. This protects against attacks, provides geographic distribution, and enables microservices architecture."

---