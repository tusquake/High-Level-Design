# DNS (Domain Name System)

## What is this concept?

**DNS (Domain Name System)** is like the phonebook of the internet. It translates human-readable domain names (like `www.google.com`) into machine-readable IP addresses (like `142.250.185.46`).

**Without DNS**: You'd need to remember `142.250.185.46` to visit Google.
**With DNS**: You just type `google.com` and DNS finds the IP for you.

**Think of it as**: Contact list on your phone - you click "Mom" instead of dialing the phone number.

## Why does this exist? / Problem it solves

- **Human memory limitation**: Can't remember IP addresses for every website
- **IP addresses change**: Domain stays same even if server IP changes
- **Load balancing**: One domain can map to multiple IPs
- **Service discovery**: Find the right server for email, web, etc.
- **Scalability**: Distributed system handles billions of queries daily

Real-world impact: Without DNS, the internet would be unusable. Imagine memorizing `172.217.14.206` instead of `youtube.com`.

## Real-World Analogy

**Postal Address System**

```
You want to visit: "John's House"
↓
You ask locals (DNS resolver): "Where's John's house?"
↓
They check the city directory (Root server): "Check downtown area"
↓
Downtown directory (TLD server): "Check Oak Street"
↓
Street directory (Authoritative server): "John lives at 123 Oak St"
↓
You arrive at: 123 Oak Street (IP address)

Domain Name = "John's House" (easy to remember)
IP Address = "123 Oak Street" (actual location)
```

## How it works (High Level)

### DNS Resolution Process (8 Steps)

```
1. User types www.google.com in browser

2. Browser checks local cache
   - If cached, return IP (done!)
   - If not, continue...

3. Browser queries DNS Resolver (ISP or 8.8.8.8)
   - Resolver checks its cache
   - If not cached, continue...

4. Resolver queries Root Server
   Root: "I don't know google.com, but ask .com TLD server at X.X.X.X"

5. Resolver queries .com TLD Server
   TLD: "I don't know google.com, but ask ns1.google.com at Y.Y.Y.Y"

6. Resolver queries Authoritative Server (ns1.google.com)
   Authoritative: "google.com is 142.250.185.46"

7. Resolver caches result and returns to browser

8. Browser connects to 142.250.185.46
```

### DNS Hierarchy (Tree Structure)

```
                    . (Root)
                    |
        ┌───────────┼───────────┐
        |           |           |
       .com        .org        .net    ← TLD (Top-Level Domain)
        |
    ┌───┴───┐
  google  amazon                       ← Second-Level Domain
    |
  ┌─┴─┐
 www mail                               ← Subdomain
```

## Simple Example

### Resolving www.netflix.com

**Step-by-step:**

```
1. You type: www.netflix.com
   Browser: "What's the IP?"

2. Check Browser Cache
   Cache: Empty
   
3. Check OS Cache
   OS: Empty
   
4. Query DNS Resolver (8.8.8.8 - Google DNS)
   Resolver: "Let me find it..."
   
5. Resolver → Root Server (.)
   Root: "For .com domains, ask 192.5.6.30"
   
6. Resolver → .com TLD Server
   TLD: "For netflix.com, ask 198.41.0.4"
   
7. Resolver → netflix.com Authoritative Server
   Auth: "www.netflix.com = 52.84.124.80"
   
8. Resolver → Browser
   "The IP is 52.84.124.80" (caches for 300 seconds)
   
9. Browser → 52.84.124.80
   "GET /index.html"
   
10. Netflix loads!
```

**Time taken**: 
- **Without cache**: 100-200ms (multiple server queries)
- **With cache**: <10ms (instant lookup)

## Code Snippet

```java
import java.net.InetAddress;
import java.util.Arrays;

class DNSDemo {
    
    // Simple DNS lookup
    public static void dnsLookup(String domain) throws Exception {
        // Java automatically uses system DNS resolver
        InetAddress address = InetAddress.getByName(domain);
        
        System.out.println("Domain: " + domain);
        System.out.println("IP Address: " + address.getHostAddress());
    }
    
    // Get all IPs (for load-balanced domains)
    public static void getAllIPs(String domain) throws Exception {
        InetAddress[] addresses = InetAddress.getAllByName(domain);
        
        System.out.println("Domain: " + domain);
        System.out.println("All IPs:");
        for (InetAddress addr : addresses) {
            System.out.println("  - " + addr.getHostAddress());
        }
    }
    
    // Reverse DNS lookup (IP to domain)
    public static void reverseLookup(String ip) throws Exception {
        InetAddress address = InetAddress.getByName(ip);
        String hostname = address.getCanonicalHostName();
        
        System.out.println("IP: " + ip);
        System.out.println("Hostname: " + hostname);
    }
    
    public static void main(String[] args) throws Exception {
        // Forward lookup
        dnsLookup("google.com");
        // Output: IP Address: 142.250.185.46
        
        // Multiple IPs (load balancing)
        getAllIPs("google.com");
        // Output: Multiple IPs returned
        
        // Reverse lookup
        reverseLookup("8.8.8.8");
        // Output: Hostname: dns.google
    }
}

// Simple DNS Cache Implementation
class DNSCache {
    private Map<String, CacheEntry> cache = new ConcurrentHashMap<>();
    
    static class CacheEntry {
        String ipAddress;
        long expiryTime;
        
        CacheEntry(String ip, int ttl) {
            this.ipAddress = ip;
            this.expiryTime = System.currentTimeMillis() + (ttl * 1000);
        }
        
        boolean isExpired() {
            return System.currentTimeMillis() > expiryTime;
        }
    }
    
    public String lookup(String domain) {
        CacheEntry entry = cache.get(domain);
        
        // Check cache first
        if (entry != null && !entry.isExpired()) {
            System.out.println("Cache HIT: " + domain);
            return entry.ipAddress;
        }
        
        // Cache miss - query DNS
        System.out.println("Cache MISS: " + domain);
        String ip = queryDNSServer(domain);
        
        // Cache with 300 second TTL
        cache.put(domain, new CacheEntry(ip, 300));
        
        return ip;
    }
    
    private String queryDNSServer(String domain) {
        // Simulate DNS query
        try {
            InetAddress addr = InetAddress.getByName(domain);
            return addr.getHostAddress();
        } catch (Exception e) {
            return null;
        }
    }
}
```

## DNS Record Types

Common DNS records you'll encounter:

| Record Type | Purpose | Example |
|-------------|---------|---------|
| **A** | Maps domain to IPv4 address | `example.com → 93.184.216.34` |
| **AAAA** | Maps domain to IPv6 address | `example.com → 2606:2800:220:1:...` |
| **CNAME** | Alias one domain to another | `www.example.com → example.com` |
| **MX** | Mail server for domain | `example.com → mail.example.com` |
| **NS** | Nameserver for domain | `example.com → ns1.example.com` |
| **TXT** | Text info (SPF, verification) | `example.com → "v=spf1 ..."` |
| **SOA** | Start of Authority (admin info) | Zone metadata |

## Where it is used in real systems

**Public DNS Resolvers:**
- **Google DNS**: `8.8.8.8`, `8.8.4.4` (2 billion+ queries/day)
- **Cloudflare DNS**: `1.1.1.1`, `1.0.0.1` (fastest, privacy-focused)
- **OpenDNS**: `208.67.222.222`, `208.67.220.220`

**Enterprise DNS:**
- **Amazon Route 53**: AWS managed DNS service
- **Azure DNS**: Microsoft's DNS hosting
- **Cloudflare DNS**: Enterprise DNS with DDoS protection
- **Akamai**: CDN with integrated DNS

**Internal DNS:**
- **Active Directory**: Windows domain DNS
- **Kubernetes CoreDNS**: Service discovery in K8s
- **Consul**: Service mesh DNS

**Use Cases:**
- **Load balancing**: Return different IPs based on location (geo-DNS)
- **Failover**: Switch to backup server if primary fails
- **CDN**: Route users to nearest edge server
- **Service discovery**: Microservices finding each other

## DNS Caching Levels

```
1. Browser Cache
   - Chrome, Firefox cache DNS for ~1 minute
   - Fastest (0ms lookup)

2. OS Cache
   - Windows, macOS, Linux cache DNS
   - Fast (~1ms lookup)

3. Router Cache
   - Home router caches queries
   - Fast (~5ms lookup)

4. ISP DNS Resolver Cache
   - Your internet provider's cache
   - Medium (~10-20ms lookup)

5. Root/TLD/Authoritative Servers
   - Full DNS resolution
   - Slow (~100-200ms lookup)
```

**TTL (Time-To-Live)**: Controls cache duration
- Low TTL (60s): Changes propagate quickly, more DNS queries
- High TTL (86400s = 24h): Fewer queries, slower changes

## Common Mistakes / Misconceptions

1. **"DNS is just one server"** ❌
   - It's a distributed hierarchy: Root → TLD → Authoritative

2. **"DNS changes are instant"** ❌
   - Caching means changes take time (TTL duration)
   - Can take hours to propagate globally

3. **"DNS queries are always slow"** ❌
   - First query: 100-200ms
   - Cached queries: <10ms

4. **"One domain = One IP"** ❌
   - Can map to multiple IPs (load balancing)
   - Google.com returns different IPs based on location

5. **"DNS is just for websites"** ❌
   - Also used for email (MX records), services, API endpoints

6. **"8.8.8.8 is always fastest"** ❌
   - Depends on location; ISP DNS might be faster

## Interview Explanation (2-minute answer)

*"DNS is the Domain Name System that translates human-readable domain names like google.com into IP addresses like 142.250.185.46. It's essentially the phone book of the internet.*

*The DNS resolution process works hierarchically. When you type a domain, your browser first checks its cache. If not found, it queries a DNS resolver, usually provided by your ISP or a public service like Google DNS at 8.8.8.8.*

*The resolver then performs recursive queries. It first contacts a root name server, which directs it to the appropriate TLD server—for example, the .com TLD server. The TLD server then points to the authoritative name server for that specific domain, which finally returns the IP address.*

*This entire process happens in milliseconds because of aggressive caching at multiple levels: browser, OS, router, and resolver. Each DNS response includes a TTL value that tells caches how long to store the result.*

*DNS supports various record types. A records map to IPv4 addresses, AAAA to IPv6, MX records specify mail servers, and CNAME creates aliases. For example, www.example.com might be a CNAME pointing to example.com.*

*In production systems, DNS is used for much more than just websites. It enables load balancing by returning different IPs based on user location, supports failover by updating records when servers go down, and provides service discovery in microservices architectures. Services like Amazon Route 53 and Cloudflare provide managed DNS with features like geo-routing and health checks."*

## Top Interview Questions

### 1. How does DNS resolution work step-by-step?
**Answer:**

1. **Browser cache check**: First check if IP is cached
2. **OS cache check**: Check operating system DNS cache
3. **DNS Resolver**: Query recursive resolver (ISP or 8.8.8.8)
4. **Root Server**: Resolver asks root for .com TLD server
5. **TLD Server**: Ask .com server for domain's nameserver
6. **Authoritative Server**: Get actual IP from domain's DNS
7. **Return & Cache**: Return IP to browser, cache result
8. **Connect**: Browser connects to IP address

**Time**: 100-200ms uncached, <10ms cached

### 2. What are the main DNS record types?
**Answer:**

- **A**: Domain → IPv4 (`example.com → 93.184.216.34`)
- **AAAA**: Domain → IPv6
- **CNAME**: Alias (`www → example.com`)
- **MX**: Mail server (`example.com → mail.example.com`)
- **NS**: Nameserver (which DNS server to use)
- **TXT**: Text data (verification, SPF)

### 3. What is DNS caching and TTL?
**Answer:**

**DNS Caching**: Storing DNS query results to avoid repeated lookups.

**Cached at multiple levels:**
- Browser (1 min)
- OS (varies)
- DNS resolver (per TTL)

**TTL (Time-To-Live)**: How long to cache result (in seconds)
- `TTL=300`: Cache for 5 minutes
- `TTL=86400`: Cache for 24 hours

**Trade-off:**
- Low TTL: Fast updates, more DNS queries (load)
- High TTL: Fewer queries, slower propagation

### 4. What's the difference between recursive and iterative DNS queries?
**Answer:**

**Recursive Query:**
- Client asks resolver: "Give me the IP for google.com"
- Resolver does ALL the work (root → TLD → authoritative)
- Returns final answer to client
- Used by: Browser → DNS Resolver

**Iterative Query:**
- Client asks root: "Where's google.com?"
- Root replies: "Ask .com TLD server at X.X.X.X"
- Client then asks TLD, TLD says ask authoritative
- Client makes multiple queries
- Used by: DNS Resolver → Root/TLD/Auth servers

### 5. How does DNS enable load balancing?
**Answer:**

**Multiple A records:**
```
google.com → 142.250.185.46
google.com → 142.250.185.78
google.com → 142.250.185.110
```

**Strategies:**
- **Round-robin**: Return IPs in rotating order
- **Geo-DNS**: Return nearest server IP based on user location
- **Health checks**: Only return healthy server IPs

**Example**: User in India gets Mumbai server IP, user in US gets Virginia server IP.

### 6. What happens if DNS fails?
**Answer:**

**Impact:**
- Can't resolve domain names
- Internet effectively "down" for users
- Cached sites still work until TTL expires

**Mitigation:**
- **Multiple DNS servers**: Primary + Secondary nameservers
- **Anycast routing**: Same IP, multiple physical servers
- **DNS redundancy**: Use multiple DNS providers
- **Long TTL**: Extends cache lifetime during outage

**Famous outage**: Dyn DNS attack (2016) took down Twitter, Netflix, Reddit.

### 7. What is DNS poisoning/cache poisoning?
**Answer:**

**Attack**: Injecting fake DNS records into cache.

**Example:**
```
Attacker tricks DNS resolver:
"bank.com → 203.0.113.50" (attacker's server)

Users think they're visiting bank.com
Actually visiting attacker's fake site
```

**Prevention:**
- **DNSSEC**: Cryptographically signs DNS records
- **Randomize ports**: Harder to spoof responses
- **Use trusted DNS**: Google DNS, Cloudflare

## Cross-Questions Interviewer May Ask

**"Why do we need both root servers and TLD servers?"**
"Separation of concerns and scalability. Root servers handle the top level, directing to TLD servers. TLD servers handle specific domains like .com or .org. This hierarchy distributes the load—root servers don't need to know about every single domain. There are only 13 root server addresses, but they're replicated worldwide using anycast."

**"How does DNS work with CDNs?"**
"CDNs use DNS for geographic routing. When you request netflix.com, the DNS returns the IP of the CDN edge server closest to you. A user in India gets a Mumbai server IP, while a US user gets a Virginia server IP. This is called geo-DNS or latency-based routing. Services like Route 53 and Cloudflare provide this automatically."

**"What happens during DNS propagation?"**
"When you update DNS records, the change doesn't happen instantly globally because of caching. Each resolver caches the old record until its TTL expires. If TTL is 24 hours, full propagation can take 24+ hours. To minimize downtime during migrations, lower the TTL days before the change, make the update, then raise TTL back after propagation."

**"Why use 8.8.8.8 instead of ISP DNS?"**
"Google DNS is often faster and more reliable than ISP resolvers. It has global infrastructure with anycast routing, so you connect to the nearest server. It's also less likely to have filtering or logging. However, your ISP's DNS might be faster if they cache popular sites well and are geographically close."

## Points to Impress the Interviewer

- **"DNS is cached at 4 levels"** — Browser, OS, Router, Resolver
- **"Root servers use anycast"** — 13 logical addresses, hundreds of physical servers
- **"TTL controls propagation speed"** — Lower TTL = faster changes, more queries
- **"Geo-DNS enables CDN routing"** — Different IPs for different locations
- **"DNSSEC prevents cache poisoning"** — Cryptographic signatures
- **"DNS is eventual consistency (AP in CAP)"** — Availability over consistency

**Real-world insight:** "Netflix uses ultra-low TTL (60s) for their DNS because they frequently shift traffic between regions based on load. This allows quick failover but generates massive DNS query volume."

**System design context:** "In microservices, Kubernetes uses CoreDNS for service discovery. Services find each other via DNS names like `user-service.default.svc.cluster.local` instead of hardcoded IPs."

---

> Next HLD topic ready.