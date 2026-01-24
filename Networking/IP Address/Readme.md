# IP Address

## What is this concept?

An **IP (Internet Protocol) Address** is a unique numerical identifier assigned to every device on a network. It's used to locate and communicate with devices across the internet.

**Two versions:**
- **IPv4**: 32-bit address (e.g., `192.168.1.1`) - Most common
- **IPv6**: 128-bit address (e.g., `2001:0db8:85a3::8a2e:0370:7334`) - Future

**Think of it as**: Your home address for the internet - lets others find and send data to you.

## Why does this exist? / Problem it solves

- **Device identification**: Millions of devices need unique addresses
- **Routing**: Routers use IP to forward packets to correct destination
- **Location-independent**: Same device can have different IPs (home vs office)
- **Hierarchical**: Network portion + host portion enables efficient routing

Without IP addresses, devices couldn't communicate across networks. Like a city without street addresses.

## Real-World Analogy

**Phone Number System**

```
IP Address = Phone Number

Country Code   (Network)  → First octets (192.168)
Area Code     (Subnet)   → Third octet (1)
Phone Number  (Host)     → Fourth octet (15)

Complete: +1-212-555-1234 ↔ 192.168.1.15
```

**Key insight**: Just like phone numbers have country/area codes for routing, IP addresses have network/host portions.

## How it works (High Level)

### IPv4 Structure

**Format**: Four octets separated by dots (0-255 each)
```
192.168.1.15

192     .  168     .  1       .  15
Network    Network    Subnet     Host
```

### IP Address Classes (Legacy, but good to know)

| Class | Range | Default Subnet | Use |
|-------|-------|----------------|-----|
| A | 1.0.0.0 - 126.255.255.255 | /8 | Large networks (16M hosts) |
| B | 128.0.0.0 - 191.255.255.255 | /16 | Medium networks (65K hosts) |
| C | 192.0.0.0 - 223.255.255.255 | /24 | Small networks (254 hosts) |

### Public vs Private IP

**Public IP:**
- Unique across entire internet
- Routable on internet
- Assigned by ISP
- Example: `8.8.8.8` (Google DNS)

**Private IP (RFC 1918):**
- Used within local networks
- Not routable on internet
- Reusable in different networks
- Ranges:
  - `10.0.0.0 - 10.255.255.255` (/8)
  - `172.16.0.0 - 172.31.255.255` (/12)
  - `192.168.0.0 - 192.168.255.255` (/16)

### Subnet Mask

Divides IP into **network portion** and **host portion**.

```
IP:           192.168.1.15
Subnet Mask:  255.255.255.0  (/24)

Network:      192.168.1.0
Host:         15
Broadcast:    192.168.1.255
```

**CIDR Notation**: `/24` = First 24 bits are network, last 8 bits are host

### Special IP Addresses

- `127.0.0.1`: Localhost (loopback, your own computer)
- `0.0.0.0`: All interfaces / default route
- `255.255.255.255`: Broadcast (send to all devices)
- `169.254.x.x`: APIPA (auto-assigned when DHCP fails)

## Simple Example

### Home Network Setup

```
Internet (Public IP: 203.0.113.50)
    ↓
Router (NAT Gateway)
    ├─ Public side:  203.0.113.50
    └─ Private side: 192.168.1.1
         ↓
    Home Network (192.168.1.0/24)
         ├─ Laptop:       192.168.1.10
         ├─ Phone:        192.168.1.11
         ├─ Smart TV:     192.168.1.12
         └─ IoT Device:   192.168.1.13
```

**How communication works:**
1. Laptop (192.168.1.10) wants to visit google.com
2. Router NAT translates private IP → public IP
3. Request goes to internet with source: 203.0.113.50
4. Response comes back to 203.0.113.50
5. Router NAT translates back → 192.168.1.10

### Subnet Example

```
Company Network: 192.168.0.0/16
    ├─ Engineering: 192.168.1.0/24  (254 hosts)
    ├─ Sales:       192.168.2.0/24  (254 hosts)
    ├─ HR:          192.168.3.0/24  (254 hosts)
    └─ Guest WiFi:  192.168.100.0/24 (254 hosts)
```

## Code Snippet

```java
// IP Address utilities
class IPAddressDemo {
    
    // Check if IP is private
    public boolean isPrivateIP(String ip) {
        String[] parts = ip.split("\\.");
        int first = Integer.parseInt(parts[0]);
        int second = Integer.parseInt(parts[1]);
        
        // 10.0.0.0/8
        if (first == 10) return true;
        
        // 172.16.0.0/12
        if (first == 172 && second >= 16 && second <= 31) return true;
        
        // 192.168.0.0/16
        if (first == 192 && second == 168) return true;
        
        return false;
    }
    
    // Calculate network address
    public String getNetworkAddress(String ip, String subnetMask) {
        String[] ipParts = ip.split("\\.");
        String[] maskParts = subnetMask.split("\\.");
        
        StringBuilder network = new StringBuilder();
        for (int i = 0; i < 4; i++) {
            int ipOctet = Integer.parseInt(ipParts[i]);
            int maskOctet = Integer.parseInt(maskParts[i]);
            network.append(ipOctet & maskOctet);
            if (i < 3) network.append(".");
        }
        
        return network.toString();
    }
    
    // CIDR to subnet mask
    public String cidrToSubnetMask(int cidr) {
        int mask = 0xffffffff << (32 - cidr);
        return ((mask >> 24) & 0xff) + "." +
               ((mask >> 16) & 0xff) + "." +
               ((mask >> 8) & 0xff) + "." +
               (mask & 0xff);
    }
    
    // Example usage
    public static void main(String[] args) {
        IPAddressDemo demo = new IPAddressDemo();
        
        System.out.println(demo.isPrivateIP("192.168.1.1"));  // true
        System.out.println(demo.isPrivateIP("8.8.8.8"));      // false
        
        String network = demo.getNetworkAddress("192.168.1.15", "255.255.255.0");
        System.out.println(network); // 192.168.1.0
        
        System.out.println(demo.cidrToSubnetMask(24)); // 255.255.255.0
    }
}
```

## Where it is used in real systems

**Public IPs:**
- Web servers (Google, Facebook, Netflix)
- DNS servers (8.8.8.8 - Google DNS)
- Email servers (SMTP, IMAP)
- CDN edge servers
- Cloud instances (AWS EC2 public IP)

**Private IPs:**
- Office computers/laptops
- Home network devices
- Internal microservices (Kubernetes pods)
- Database servers (not exposed to internet)
- Internal load balancers

**IPv6 Adoption:**
- Mobile networks (carrier-grade NAT exhausted IPv4)
- IoT devices (billions of devices need addresses)
- Cloud providers (AWS, Google Cloud support IPv6)

## Common Mistakes / Misconceptions

1. **"Every device has a unique IP on internet"** ❌
   - No. NAT allows multiple devices to share one public IP
   - Home router: 1 public IP, dozens of private IPs behind it

2. **"IP address is permanent for a device"** ❌
   - Dynamic IPs change (DHCP lease expires)
   - Same laptop has different IPs at home vs coffee shop

3. **"192.168.x.x is always safe to use"** ✓
   - Yes, it's a private range, but conflicts can occur if VPN uses same range

4. **"IPv6 will replace IPv4 soon"** ❌
   - Both will coexist for years
   - Dual-stack systems support both

5. **"Subnet mask 255.255.255.0 is always /24"** ✓
   - Correct. /24 means first 24 bits are 1s in binary

## Interview Explanation (2-minute answer)

*"An IP address is a unique identifier for devices on a network, used for routing and communication. There are two versions: IPv4 with 32-bit addresses like 192.168.1.1, and IPv6 with 128-bit addresses for the future.*

*IPv4 addresses have four octets from 0-255. They're divided into network and host portions using a subnet mask. For example, 192.168.1.15 with subnet mask 255.255.255.0 means the first three octets (192.168.1) identify the network, and the last octet (15) identifies the specific host.*

*There are public and private IPs. Public IPs are unique across the internet and assigned by ISPs. Private IPs are used within local networks and defined by RFC 1918: 10.x.x.x, 172.16-31.x.x, and 192.168.x.x ranges. These aren't routable on the internet.*

*NAT enables multiple devices with private IPs to share one public IP. Your home router has one public IP but all your devices have private IPs like 192.168.1.x. When you browse the internet, the router translates your private IP to its public IP.*

*Special addresses include 127.0.0.1 for localhost, 0.0.0.0 for all interfaces, and 255.255.255.255 for broadcast. In system design, we use private IPs for internal microservices and public IPs for user-facing servers."*

## Top Interview Questions

### 1. What's the difference between IPv4 and IPv6?
**Answer:**

| Feature | IPv4 | IPv6 |
|---------|------|------|
| **Size** | 32-bit | 128-bit |
| **Format** | 192.168.1.1 | 2001:0db8::1 |
| **Addresses** | ~4.3 billion | 340 undecillion |
| **Notation** | Decimal (0-255) | Hexadecimal |
| **Header** | Complex, variable | Simpler, fixed |

**Why IPv6?** IPv4 addresses exhausted, NAT is workaround but not ideal.

### 2. What are private IP ranges?
**Answer:**

Three private IP ranges (RFC 1918):
- **10.0.0.0/8**: 10.0.0.0 - 10.255.255.255 (16M addresses)
- **172.16.0.0/12**: 172.16.0.0 - 172.31.255.255 (1M addresses)
- **192.168.0.0/16**: 192.168.0.0 - 192.168.255.255 (65K addresses)

Used in local networks, not routable on internet.

### 3. What is NAT (Network Address Translation)?
**Answer:**

NAT translates private IPs to public IPs and vice versa.

**Example:**
```
Internal: 192.168.1.10:5000 → External: 203.0.113.50:52341
Router tracks the mapping in NAT table
Response: 203.0.113.50:52341 → 192.168.1.10:5000
```

**Why?** Allows many devices to share one public IP, conserves IPv4 addresses.

### 4. What does /24 mean in 192.168.1.0/24?
**Answer:**

CIDR notation: `/24` means first **24 bits** are the network portion.

```
/24 = 255.255.255.0
- Network: 192.168.1.0
- Usable IPs: 192.168.1.1 - 192.168.1.254 (254 hosts)
- Broadcast: 192.168.1.255

/16 = 255.255.0.0 (65,534 hosts)
/8  = 255.0.0.0 (16M hosts)
```

### 5. What is 127.0.0.1?
**Answer:**

**Localhost / Loopback address**

Points to your own computer. Traffic never leaves the machine.

**Use cases:**
- Testing web servers locally (`http://localhost:8080`)
- Inter-process communication on same machine
- Database running locally

### 6. How does a device get an IP address?
**Answer:**

Two methods:

**1. Static IP (Manual):**
- Manually configured
- Never changes
- Used for servers

**2. Dynamic IP (DHCP):**
- Automatically assigned by DHCP server
- Lease period (e.g., 24 hours)
- Used for laptops, phones

**DHCP Process (DORA):**
1. **D**iscover: Client broadcasts "Need IP"
2. **O**ffer: DHCP server offers available IP
3. **R**equest: Client requests that IP
4. **A**cknowledge: Server confirms assignment

## Cross-Questions Interviewer May Ask

**"Why do we need both MAC and IP addresses?"**
"MAC addresses are physical (Layer 2), permanent, and work locally within a network. IP addresses are logical (Layer 3), can change, and work globally across networks. You need MAC to deliver within your local network and IP to route across the internet."

**"Can two devices have the same IP address?"**
"Not on the same network—causes IP conflict. But different networks can reuse private IPs (every home network uses 192.168.1.x). Public IPs must be globally unique."

**"How do you subnet a network?"**
"Divide a larger network into smaller subnets using subnet masks. For example, 192.168.0.0/16 can be split into 256 /24 subnets: 192.168.0.0/24, 192.168.1.0/24, etc. Helps organize departments, improve security, and reduce broadcast traffic."

**"What happens when IPv4 addresses run out?"**
"They already have! Solutions: NAT (share public IPs), IPv6 adoption (340 undecillion addresses), carrier-grade NAT (ISP-level sharing). Most systems use dual-stack supporting both IPv4 and IPv6."

## Points to Impress the Interviewer

- **"Private IPs save public IPv4 addresses through NAT"** — Understand the scarcity problem
- **"Microservices use private IPs, load balancers use public"** — System design context
- **"127.0.0.1 never leaves the machine"** — Useful for local development
- **"CIDR /24 = 254 usable hosts"** — Know the math (256 - network - broadcast)
- **"IPv6 solves address exhaustion"** — Awareness of modern solutions

**Real-world insight:** "AWS VPC uses private IP ranges (10.0.0.0/16). Your EC2 instances get private IPs for internal communication, and Elastic IPs (public) for internet access. This reduces attack surface."

---

> Next HLD topic ready.