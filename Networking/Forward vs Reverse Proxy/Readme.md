# Proxy vs Reverse Proxy

## What is this concept?

**Proxy (Forward Proxy)**: Acts on behalf of **clients** to access servers.
- Client → Proxy → Internet
- Hides client identity from servers

**Reverse Proxy**: Acts on behalf of **servers** to handle client requests.
- Client → Reverse Proxy → Servers
- Hides server identity from clients

**Key Difference**: 
- **Forward Proxy** = Client's representative
- **Reverse Proxy** = Server's representative

## Why does this exist? / Problem it solves

### Forward Proxy Problems Solved:
- **Privacy**: Hide client IP from websites
- **Access control**: Block certain websites (corporate firewalls)
- **Bypass restrictions**: Access geo-blocked content
- **Caching**: Speed up browsing by caching common requests
- **Anonymity**: Browse without revealing identity

### Reverse Proxy Problems Solved:
- **Load balancing**: Distribute traffic across multiple servers
- **SSL termination**: Handle HTTPS encryption/decryption
- **Caching**: Cache responses to reduce server load
- **Security**: Hide backend server IPs, prevent direct access
- **DDoS protection**: Filter malicious traffic before it reaches servers

## Real-World Analogy

### Forward Proxy (Receptionist Making Calls)

```
You (Client) want to call a company:

YOU → RECEPTIONIST (Proxy) → COMPANY
       ↓
    "I'm calling on behalf of someone"
    
Company sees: Receptionist's number (not yours)
Company doesn't know: Who actually called

Use case: Privacy, company doesn't know your identity
```

### Reverse Proxy (Company Receptionist Receiving Calls)

```
You (Client) call a company:

YOU → RECEPTIONIST (Reverse Proxy) → EMPLOYEES
       ↓
    "Let me transfer you to the right person"
    
You see: Receptionist's number (1-800-COMPANY)
You don't know: Which employee actually helps you

Use case: Company hides internal structure, distributes calls
```

## How it works (High Level)

### Forward Proxy Flow

```
1. Client (You) configured to use proxy
2. Browser sends request to proxy: "Get google.com"
3. Proxy fetches google.com on your behalf
4. Proxy returns response to you
5. Google sees proxy IP, not your IP

Client IP: 192.168.1.10 (hidden)
Proxy IP: 203.0.113.50 (visible to Google)
Server IP: 142.250.185.46 (Google)
```

### Reverse Proxy Flow

```
1. Client requests: www.example.com
2. DNS resolves to reverse proxy: 203.0.113.50
3. Reverse proxy receives request
4. Proxy forwards to backend server (internal IP)
5. Backend processes and returns to proxy
6. Proxy returns response to client

Client IP: 14.98.52.10
Reverse Proxy IP: 203.0.113.50 (public)
Backend Servers: 10.0.1.5, 10.0.1.6, 10.0.1.7 (hidden)
```

## Simple Example

### Forward Proxy: School/Corporate Network

**Scenario: Students accessing internet**

```
STUDENTS → SCHOOL PROXY → INTERNET
            ↓
         - Blocks Facebook, YouTube
         - Logs all traffic
         - Caches Google, Wikipedia

Student tries: facebook.com
Proxy: "Access denied" ✗

Student tries: wikipedia.org
Proxy: "Serving from cache" ✓ (fast)

Facebook sees: School proxy IP, not student IP
```

### Reverse Proxy: Netflix Architecture

**Scenario: Users streaming videos**

```
USERS → NGINX (Reverse Proxy) → BACKEND SERVERS
         ↓
      - Terminates SSL
      - Load balances across 100 servers
      - Caches popular content
      - Rate limiting

User in Mumbai → Nginx routes to Mumbai server (10.0.1.5)
User in Delhi → Nginx routes to Delhi server (10.0.2.8)

Users see: netflix.com (52.84.124.80)
Users don't know: Which of 100 servers handled request
```

## Code Snippet

```java
// Forward Proxy (Java HTTP Client with Proxy)
import java.net.*;
import java.io.*;

class ForwardProxyExample {
    public static void main(String[] args) throws Exception {
        // Configure proxy
        Proxy proxy = new Proxy(
            Proxy.Type.HTTP,
            new InetSocketAddress("proxy.company.com", 8080)
        );
        
        // Make request through proxy
        URL url = new URL("https://api.example.com/data");
        HttpURLConnection conn = (HttpURLConnection) url.openConnection(proxy);
        
        // Server sees proxy IP, not your IP
        InputStream response = conn.getInputStream();
        // Process response...
    }
}

// Nginx Reverse Proxy Configuration
/*
# /etc/nginx/nginx.conf

upstream backend_servers {
    # Load balance across 3 servers
    server 10.0.1.5:8080;
    server 10.0.1.6:8080;
    server 10.0.1.7:8080;
}

server {
    listen 443 ssl;
    server_name example.com;
    
    # SSL Termination
    ssl_certificate /etc/ssl/cert.pem;
    ssl_certificate_key /etc/ssl/key.pem;
    
    location / {
        # Reverse proxy to backend
        proxy_pass http://backend_servers;
        
        # Pass client info to backend
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        
        # Caching
        proxy_cache my_cache;
        proxy_cache_valid 200 60m;
    }
}
*/

// Simple Reverse Proxy in Node.js
const http = require('http');
const httpProxy = require('http-proxy');

const proxy = httpProxy.createProxyServer({});

// Backend servers
const backends = [
    'http://10.0.1.5:8080',
    'http://10.0.1.6:8080',
    'http://10.0.1.7:8080'
];
let current = 0;

// Reverse proxy server
http.createServer((req, res) => {
    // Round-robin load balancing
    const target = backends[current];
    current = (current + 1) % backends.length;
    
    console.log(`Forwarding to ${target}`);
    proxy.web(req, res, { target });
}).listen(80);

console.log('Reverse proxy running on port 80');
```

## Comparison Table

| Feature | Forward Proxy | Reverse Proxy |
|---------|--------------|---------------|
| **Represents** | Client | Server |
| **Hides** | Client IP | Server IP |
| **Direction** | Client → Internet | Internet → Servers |
| **Use Case** | Privacy, access control | Load balancing, security |
| **Example** | Corporate proxy, VPN | Nginx, CloudFlare |
| **Client Knows?** | Yes (configured) | No (transparent) |
| **SSL** | Client to proxy | Proxy to client (SSL termination) |
| **Caching** | Client-side (browser cache) | Server-side (response cache) |
| **Who Benefits** | Client (privacy, access) | Server (performance, security) |

## Where it is used in real systems

### Forward Proxy
- **Corporate Networks**: Filter/monitor employee internet access
- **VPN Services**: NordVPN, ExpressVPN (hide client IP)
- **Squid Proxy**: Caching proxy for web content
- **Tor Network**: Anonymous browsing
- **Cloud NAT Gateway**: AWS NAT Gateway for private subnets

### Reverse Proxy
- **Nginx**: Web server + reverse proxy (Netflix, Airbnb)
- **HAProxy**: High-performance load balancer
- **AWS ALB/NLB**: Application/Network Load Balancer
- **Cloudflare**: CDN + reverse proxy + DDoS protection
- **Apache mod_proxy**: Reverse proxy module
- **Traefik**: Modern reverse proxy for containers/microservices
- **Envoy**: Service mesh proxy (used by Istio)

### Real-World Architectures

**E-commerce Site (Reverse Proxy):**
```
Users → Cloudflare (CDN + Reverse Proxy)
         ↓
      Nginx (Load Balancer + SSL Termination)
         ↓
      App Servers (10.0.1.x)
         ↓
      Database
```

**Corporate Office (Forward Proxy):**
```
Employees → Proxy Server (Squid)
             ↓
          Internet
```

## Common Mistakes / Misconceptions

1. **"Proxy always means forward proxy"** ❌
   - Specify: forward proxy or reverse proxy
   - Very different purposes

2. **"VPN and proxy are the same"** ❌
   - VPN: Encrypts all traffic, tunnels network layer
   - Forward Proxy: HTTP/HTTPS only, application layer
   - VPN is more comprehensive

3. **"Reverse proxy is just a load balancer"** ❌
   - Load balancing is ONE feature
   - Also does: SSL termination, caching, security, compression

4. **"Client knows about reverse proxy"** ❌
   - Reverse proxy is transparent to client
   - Client only sees one IP/domain

5. **"Forward proxy makes you completely anonymous"** ❌
   - Websites can detect proxies
   - Need VPN + Tor for strong anonymity

6. **"Reverse proxy is slower"** ❌
   - Can be faster due to caching, compression
   - SSL termination offloads work from backend servers

## Interview Explanation (2-minute answer)

*"A forward proxy acts on behalf of clients, while a reverse proxy acts on behalf of servers.*

*In a forward proxy setup, clients send requests to the proxy, which then fetches content from the internet on their behalf. The server sees the proxy's IP, not the client's. This is common in corporate networks where a proxy server controls internet access, blocks certain sites, caches content, and logs activity. VPNs are a type of forward proxy that encrypts traffic for privacy.*

*A reverse proxy sits in front of backend servers and handles incoming client requests. When you visit netflix.com, you're actually hitting a reverse proxy—probably Nginx. The proxy then forwards your request to one of many backend servers, performs SSL termination, caches responses, and load balances traffic. The client only knows about the reverse proxy's IP; the actual backend servers remain hidden.*

*The key benefits differ: forward proxies provide client-side benefits like privacy and access control, while reverse proxies provide server-side benefits like load balancing, security, and performance.*

*In real systems, we use both. Nginx is the most popular reverse proxy, handling SSL termination and load balancing for millions of websites. Cloudflare acts as both a CDN and reverse proxy, providing DDoS protection and caching. On the forward proxy side, corporate networks use Squid or similar proxies to control employee internet access.*

*A common interview scenario is designing an e-commerce site: users hit Cloudflare (reverse proxy for DDoS protection), which forwards to Nginx (reverse proxy for load balancing and SSL), which distributes to backend application servers. This architecture hides infrastructure, improves performance through caching, and provides security."*

## Top Interview Questions

### 1. What's the main difference between forward and reverse proxy?
**Answer:**

**Forward Proxy:**
- Acts for **clients**
- Client → Proxy → Internet
- Hides client IP from servers
- Example: VPN, corporate proxy

**Reverse Proxy:**
- Acts for **servers**
- Client → Proxy → Backend Servers
- Hides server IP from clients
- Example: Nginx, load balancer

**Memory trick**: 
- Forward = Forward your request (you use it)
- Reverse = Reverses direction (server uses it)

### 2. Why use a reverse proxy?
**Answer:**

**Benefits:**
1. **Load Balancing**: Distribute traffic across servers
2. **SSL Termination**: Handle HTTPS encryption/decryption
3. **Caching**: Cache responses, reduce backend load
4. **Security**: Hide backend IPs, DDoS protection
5. **Compression**: Gzip responses
6. **URL Rewriting**: Clean URLs, API versioning

**Example**: Netflix uses Nginx to handle millions of requests, terminate SSL, and route to appropriate backend services.

### 3. What is SSL termination and why use it?
**Answer:**

**SSL Termination**: Reverse proxy decrypts HTTPS, forwards HTTP to backend.

```
Client (HTTPS) → Reverse Proxy → Backend (HTTP)
              ↓
         Decrypts SSL here
```

**Why:**
- **Performance**: SSL is CPU-intensive, offload from app servers
- **Simplicity**: Backend servers don't need SSL certificates
- **Centralized**: Manage certificates in one place
- **Cost**: Need fewer SSL certificates

**Security note**: Backend traffic should still be encrypted in production (mTLS).

### 4. How does a forward proxy provide privacy?
**Answer:**

**Mechanism:**
1. You send request to proxy
2. Proxy forwards to website
3. Website sees proxy IP, not yours
4. Response goes back through proxy
5. Your IP remains hidden

**Privacy level:**
- **Basic proxy**: Hides IP but not encrypted
- **VPN**: Hides IP + encrypts traffic
- **Tor**: Multi-layer proxies, strong anonymity

**Use cases:**
- Access geo-restricted content
- Bypass corporate firewalls
- Anonymous browsing

### 5. Can you have both forward and reverse proxy?
**Answer:**

**Yes!** Common in enterprise setups.

```
Employee → Forward Proxy (outbound control)
              ↓
           Internet
              ↓
Customer → Reverse Proxy (inbound load balancing)
              ↓
          App Servers
```

**Example:**
- Employees use **forward proxy** to access internet (filtered)
- Customers hit **reverse proxy** to access company website (load balanced)

### 6. What's the difference between reverse proxy and API Gateway?
**Answer:**

**Reverse Proxy:**
- Basic routing and load balancing
- SSL termination
- Caching
- Examples: Nginx, HAProxy

**API Gateway:**
- Reverse proxy + advanced features
- Authentication/Authorization
- Rate limiting
- Request/Response transformation
- API versioning
- Analytics
- Examples: Kong, AWS API Gateway, Apigee

**Relationship**: API Gateway is a reverse proxy with API management features.

### 7. How do you configure Nginx as reverse proxy?
**Answer:**

```nginx
# Basic reverse proxy
server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://backend_server:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# Load balancing
upstream backend {
    server 10.0.1.5:8080;
    server 10.0.1.6:8080;
}

server {
    location / {
        proxy_pass http://backend;
    }
}
```

## Cross-Questions Interviewer May Ask

**"How does reverse proxy improve performance?"**
"Three ways: First, caching—frequently accessed content is served from cache without hitting backend servers. Second, SSL termination—the proxy handles expensive encryption/decryption, freeing up backend CPU. Third, compression—proxy can gzip responses before sending to clients. For example, Nginx can serve cached static assets at 10,000 req/s vs 1,000 req/s hitting the backend."

**"What happens if the reverse proxy fails?"**
"It's a single point of failure. Solutions: Run multiple reverse proxies with DNS round-robin or use a load balancer (like AWS ELB) that's itself redundant. Cloud providers handle this automatically. For on-premise, tools like Keepalived provide high availability with floating IPs—if the primary proxy fails, backup takes over the IP."

**"How does forward proxy help with caching?"**
"Corporate proxies cache frequently accessed content. If 100 employees visit cnn.com, the proxy fetches it once, caches it, and serves subsequent requests from cache. This saves bandwidth and improves speed. Squid proxy can achieve 30-60% cache hit rates in corporate environments, significantly reducing internet traffic costs."

**"Can reverse proxy do A/B testing?"**
"Yes! You can route a percentage of traffic to different backends. For example, 90% to version A, 10% to version B. Nginx can route based on cookies, headers, or random selection. This is common for canary deployments and feature flags. More advanced proxies like Envoy support sophisticated traffic shaping for this purpose."

## Points to Impress the Interviewer

- **"Forward = client-side, Reverse = server-side"** — Clear distinction
- **"SSL termination offloads backend CPU"** — Performance benefit
- **"Reverse proxy enables zero-downtime deployments"** — Route traffic during updates
- **"Nginx handles 10K+ concurrent connections"** — Industry standard performance
- **"API Gateway is reverse proxy + features"** — Shows architectural knowledge
- **"Both can cache, different purposes"** — Forward caches for clients, reverse for servers

**Real-world insight:** "Netflix uses Zuul as an API gateway (reverse proxy). It handles authentication, rate limiting, and routes requests to hundreds of microservices. Each service doesn't need to implement auth—Zuul centralizes it."

**System design context:** "In microservices, we use service mesh (Envoy, Istio) as reverse proxies between services. They handle load balancing, retries, circuit breaking, and mTLS—all without changing application code. This is the modern evolution of reverse proxies."

---