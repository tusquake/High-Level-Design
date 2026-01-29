# WebRTC (Web Real-Time Communication)

**Read Time: 8 minutes**

## What is this concept?

**WebRTC**: Enables peer-to-peer (P2P) real-time communication (video, audio, data) directly between browsers without a server relay.

**Simple definition**: Direct connection between two browsers for video calls, audio calls, or file sharing—no middleman server needed.

**Think of it as**: Two people talking directly instead of passing messages through a messenger. The messenger introduces them, but then they talk face-to-face.

## Why does this exist? / Problem it solves

**Problem: Server Bottleneck**
```
Traditional video call (Zoom-style without P2P):

User A → Upload video → Server → Download → User B
User B → Upload video → Server → Download → User A

Issues:
❌ Server bandwidth cost (2x upload + 2x download)
❌ Latency (2 hops)
❌ Server processing (expensive)
❌ Doesn't scale (1000 users = massive server load)
```

**WebRTC Solution:**
```
User A ←──────── Direct P2P ────────→ User B

Benefits:
✓ No server bandwidth for media
✓ Lower latency (direct path)
✓ Better quality (no re-encoding)
✓ Scales better (server only for signaling)
```

**Problems solved:**
- **Cost**: No server bandwidth for video/audio
- **Latency**: Direct connection = faster
- **Quality**: No server compression
- **Privacy**: Data doesn't pass through server

## Real-World Analogy

**Phone Call System**

```
Old system (Server relay):
Alice calls Bob → Goes through operator → Operator relays everything

WebRTC (Direct):
Alice calls Bob → Operator connects them → They talk directly
Operator no longer involved in conversation

The "operator" (signaling server) only helps establish the connection,
then steps out of the way.
```

---

## How WebRTC Works

### Three Main Components

**1. Signaling (Setup the connection)**
```
Use WebSocket/HTTP to exchange:
- "I want to connect to you"
- "Here's my network info"
- "Here's how to reach me"

Not part of WebRTC spec (you choose: WebSocket, HTTP, etc.)
```

**2. STUN/TURN (Navigate firewalls/NAT)**
```
STUN: Discovers your public IP
TURN: Relay when direct connection impossible

Most connections work with just STUN
~10-20% need TURN (behind strict firewalls)
```

**3. Peer Connection (The actual media transfer)**
```
Direct P2P connection
Send video, audio, or data
Encrypted by default (DTLS)
```

---

### Connection Flow

```
┌─────────┐                                    ┌─────────┐
│ User A  │                                    │ User B  │
└────┬────┘                                    └────┬────┘
     │                                              │
     │ 1. Connect to Signaling Server               │
     ├──────────────────┐      ┌──────────────────┤
     │                  ▼      ▼                   │
     │            ┌──────────────────┐             │
     │            │ Signaling Server │             │
     │            └──────────────────┘             │
     │                                              │
     │ 2. A creates "Offer"                         │
     │    (SDP with media capabilities)             │
     ├─────────────────────────────────────────────▶│
     │                                              │
     │ 3. B creates "Answer"                        │
     │◀─────────────────────────────────────────────┤
     │                                              │
     │ 4. Exchange ICE candidates                   │
     │    (network paths to reach each other)       │
     │◀────────────────────────────────────────────▶│
     │                                              │
     │ 5. Direct P2P connection established         │
     │══════════════════════════════════════════════│
     │         Video/Audio/Data flows               │
     │══════════════════════════════════════════════│
```

**Steps:**
1. **Both connect to signaling server** (WebSocket)
2. **User A creates offer** (SDP describing capabilities)
3. **User B creates answer** (SDP response)
4. **Exchange ICE candidates** (network addresses)
5. **P2P connection established** (media flows directly)

---

## Key Concepts

### 1. SDP (Session Description Protocol)

**What it is:** Text description of media capabilities

**Example SDP Offer:**
```
v=0
o=- 123456 2 IN IP4 127.0.0.1
s=-
t=0 0
m=audio 49170 RTP/AVP 0
m=video 51372 RTP/AVP 99
a=rtpmap:99 H264/90000
```

**Contains:**
- Supported codecs (H.264, VP8 for video)
- Media types (audio, video, data)
- Network info
- Encryption keys

---

### 2. ICE (Interactive Connectivity Establishment)

**What it does:** Finds the best path to connect two peers

**Process:**
```
1. Gather all possible network addresses:
   - Local IP (192.168.1.10)
   - Public IP from STUN (203.0.113.5)
   - TURN relay address (if needed)

2. Exchange candidates with peer

3. Try all paths simultaneously

4. Pick the best one (usually direct)
```

**ICE Candidate Example:**
```
candidate:1 1 UDP 2130706431 192.168.1.10 54321 typ host
candidate:2 1 UDP 1694498815 203.0.113.5 54321 typ srflx
```

---

### 3. STUN (Session Traversal Utilities for NAT)

**What it does:** Discovers your public IP address

**Why needed:**
```
Your device thinks IP is: 192.168.1.10 (private)
Outside world sees: 203.0.113.5 (public NAT IP)

STUN server tells you: "The world sees you as 203.0.113.5"
```

**Free STUN servers:**
```
stun:stun.l.google.com:19302
stun:stun1.l.google.com:19302
```

---

### 4. TURN (Traversal Using Relays around NAT)

**What it does:** Relays traffic when direct P2P fails

**When needed:**
```
Scenario: Both users behind strict firewalls
Direct connection impossible

Solution: TURN server acts as relay
User A → TURN Server → User B

~10-20% of connections need TURN
```

**Cost:** TURN uses server bandwidth (unlike pure P2P)

---

## Code Example

### Basic WebRTC Video Call

```javascript
// Get local video stream
const localStream = await navigator.mediaDevices.getUserMedia({
  video: true,
  audio: true
});

// Display local video
document.getElementById('localVideo').srcObject = localStream;

// Create peer connection
const peerConnection = new RTCPeerConnection({
  iceServers: [
    { urls: 'stun:stun.l.google.com:19302' }
  ]
});

// Add local stream to connection
localStream.getTracks().forEach(track => {
  peerConnection.addTrack(track, localStream);
});

// Handle incoming remote stream
peerConnection.ontrack = (event) => {
  document.getElementById('remoteVideo').srcObject = event.streams[0];
};

// Handle ICE candidates
peerConnection.onicecandidate = (event) => {
  if (event.candidate) {
    // Send candidate to other peer via signaling server
    signalingServer.send({
      type: 'ice-candidate',
      candidate: event.candidate
    });
  }
};

// Create offer (User A)
const offer = await peerConnection.createOffer();
await peerConnection.setLocalDescription(offer);
// Send offer to User B via signaling server
signalingServer.send({ type: 'offer', sdp: offer });

// Receive answer (User A)
signalingServer.on('answer', async (answer) => {
  await peerConnection.setRemoteDescription(answer);
});

// Create answer (User B)
signalingServer.on('offer', async (offer) => {
  await peerConnection.setRemoteDescription(offer);
  const answer = await peerConnection.createAnswer();
  await peerConnection.setLocalDescription(answer);
  signalingServer.send({ type: 'answer', sdp: answer });
});

// Receive ICE candidates
signalingServer.on('ice-candidate', async (candidate) => {
  await peerConnection.addIceCandidate(candidate);
});
```

---

## Data Channels

**Beyond video/audio—send arbitrary data P2P:**

```javascript
// Create data channel
const dataChannel = peerConnection.createDataChannel('chat');

// Send messages
dataChannel.onopen = () => {
  dataChannel.send('Hello!');
};

// Receive messages
dataChannel.onmessage = (event) => {
  console.log('Received:', event.data);
};

// Receive data channel (peer side)
peerConnection.ondatachannel = (event) => {
  const receiveChannel = event.channel;
  receiveChannel.onmessage = (e) => console.log(e.data);
};
```

**Use cases:**
- P2P file sharing
- Gaming (low-latency state sync)
- Collaborative editing
- Screen sharing with annotations

---

## WebRTC vs Other Technologies

### WebRTC vs WebSocket

| Feature | WebRTC | WebSocket |
|---------|--------|-----------|
| **Connection** | P2P (direct) | Client-Server |
| **Use case** | Video, audio, data | Text, data only |
| **Latency** | Lower (direct) | Higher (via server) |
| **Bandwidth** | No server cost | Server bandwidth |
| **Setup** | Complex (signaling) | Simple |

**When to use:**
- **WebRTC**: Video calls, file sharing, gaming
- **WebSocket**: Chat, notifications, real-time updates

---

### WebRTC vs HTTP/REST

```
HTTP (traditional):
Request video → Server processes → Returns video
- Server bandwidth
- Server processing
- Latency

WebRTC:
Direct P2P stream
- No server bandwidth
- No server processing
- Low latency
```

---

## Common Use Cases

### 1. Video Conferencing
```
Examples: Google Meet, Zoom, Discord
Benefits:
- P2P for 1-on-1 calls (low latency)
- SFU (Selective Forwarding Unit) for group calls
```

### 2. File Sharing
```
Examples: ShareDrop, Snapdrop
Benefits:
- No server storage
- Fast (direct transfer)
- Private (encrypted)
```

### 3. Gaming
```
Examples: Multiplayer browser games
Benefits:
- Low latency (direct connection)
- Fast state synchronization
```

### 4. Screen Sharing
```
Examples: Remote support, collaboration
Benefits:
- Real-time sharing
- Low latency
```

### 5. IoT
```
Examples: Security cameras, smart home
Benefits:
- Direct device connection
- No cloud relay needed
```

---

## Architecture Patterns

### 1. Mesh (P2P for everyone)

```
User A ←→ User B
  ↓   ×   ↓
User C ←→ User D

Pros:
✓ No server bandwidth
✓ Low latency

Cons:
✗ Scales poorly (n² connections)
✗ 10+ users = unusable
```

**Best for:** 2-4 person calls

---

### 2. SFU (Selective Forwarding Unit)

```
User A ───┐
User B ───┼──→ SFU Server ───→ Distribute to all
User C ───┘

Pros:
✓ Scales to 100+ users
✓ Server doesn't decode/encode (just forwards)

Cons:
✗ Some server bandwidth
✗ More complex
```

**Best for:** Group video calls (Zoom, Meet)

---

### 3. MCU (Multipoint Control Unit)

```
User A ───┐
User B ───┼──→ MCU Server (mixes all streams) ──→ Single stream
User C ───┘

Pros:
✓ Low client bandwidth (1 stream received)

Cons:
✗ High server CPU (encoding/mixing)
✗ Quality loss (re-encoding)
```

**Best for:** Large webinars (100+ viewers)

---

## Challenges & Solutions

### 1. Firewall/NAT Traversal

**Problem:**
```
User A behind corporate firewall
Direct connection blocked
```

**Solution:**
```
1. Try STUN (90% success)
2. Fall back to TURN relay (10%)
3. Use ICE to find best path
```

---

### 2. Signaling

**Problem:** WebRTC doesn't define how to exchange SDP/ICE

**Solution:**
```
Use WebSocket, HTTP, or any message protocol:

// WebSocket signaling
const ws = new WebSocket('wss://signal.example.com');
ws.send(JSON.stringify({ type: 'offer', sdp: offer }));
```

---

### 3. Bandwidth Management

**Problem:** Video uses lots of bandwidth

**Solution:**
```javascript
// Adaptive bitrate
const sender = peerConnection.getSenders()[0];
const params = sender.getParameters();
params.encodings[0].maxBitrate = 500000; // 500 kbps
await sender.setParameters(params);
```

---

### 4. Reconnection

**Problem:** Network drops, need to reconnect

**Solution:**
```javascript
peerConnection.onconnectionstatechange = () => {
  if (peerConnection.connectionState === 'failed') {
    // Restart ICE
    peerConnection.restartIce();
  }
};
```

---

## Security

### Built-in Encryption

```
WebRTC uses:
- DTLS (Datagram Transport Layer Security)
- SRTP (Secure Real-time Transport Protocol)

All media encrypted by default
End-to-end encryption for P2P
```

### Risks

**1. Signaling not encrypted by WebRTC:**
```
Solution: Use WSS (WebSocket Secure)
```

**2. IP address exposure:**
```
Problem: ICE reveals your IP
Mitigation: Use VPN or TURN-only mode
```

**3. TURN relay sees traffic:**
```
Traffic still encrypted, but TURN can see metadata
Use trusted TURN servers
```

---

## Best Practices

### 1. Use STUN/TURN Servers
```javascript
const config = {
  iceServers: [
    { urls: 'stun:stun.l.google.com:19302' }, // Free STUN
    { 
      urls: 'turn:turn.example.com:3478',
      username: 'user',
      credential: 'pass'
    }
  ]
};
```

### 2. Handle Connection States
```javascript
peerConnection.onconnectionstatechange = () => {
  switch(peerConnection.connectionState) {
    case 'connected':
      console.log('Connected!');
      break;
    case 'disconnected':
    case 'failed':
      // Attempt reconnection
      break;
  }
};
```

### 3. Clean Up Resources
```javascript
// Stop all tracks
localStream.getTracks().forEach(track => track.stop());

// Close peer connection
peerConnection.close();
```

### 4. Adaptive Bitrate
```javascript
// Reduce quality on slow networks
if (networkSlow) {
  sender.setParameters({
    encodings: [{ maxBitrate: 250000 }] // Lower bitrate
  });
}
```

---

## Interview Explanation (60-second answer)

*"WebRTC enables peer-to-peer real-time communication directly between browsers without a server relay. Unlike traditional video calls where everything goes through a server, WebRTC establishes a direct connection—reducing latency, cost, and improving quality.*

*The process involves three components: signaling to exchange connection info using WebSocket or HTTP, STUN servers to discover public IPs and navigate NATs, and the actual peer connection for media transfer.*

*The flow is: both users connect to a signaling server, User A creates an SDP offer describing their capabilities, User B responds with an answer, they exchange ICE candidates to find network paths, and then establish a direct encrypted P2P connection.*

*About 80-90% of connections work with just STUN for NAT traversal, but 10-20% need TURN servers to relay when firewalls block direct connections. TURN uses server bandwidth, but it's still more efficient than relaying all media.*

*For scaling, you use different architectures: mesh for 2-4 users, SFU (Selective Forwarding Unit) for group calls up to 100 users, and MCU for large webinars. Google Meet and Zoom use SFU architecture.*

*WebRTC is used for video conferencing, P2P file sharing, online gaming, and screen sharing. It's encrypted by default using DTLS and SRTP."*

---

## Top Interview Questions

### 1. What is WebRTC and how does it differ from WebSocket?

**Answer:**

**WebRTC:**
- Peer-to-peer (direct connection)
- Video, audio, and data
- Lower latency (no server hop)
- Complex setup (signaling, STUN/TURN)

**WebSocket:**
- Client-server (not P2P)
- Data only (text, binary)
- Higher latency (via server)
- Simple setup

**Example:**
```
WebRTC video call:
User A ←──────── Direct P2P ──────→ User B
(No server bandwidth for video)

WebSocket chat:
User A → Server → User B
(Server relays all messages)
```

**When to use:**
- WebRTC: Video calls, file sharing, gaming
- WebSocket: Chat, notifications, real-time updates

### 2. Explain the WebRTC connection flow

**Answer:**

**Steps:**

**1. Signaling (Setup):**
```
Both users connect to signaling server (WebSocket/HTTP)
Exchange: "I want to connect"
```

**2. Create Offer/Answer (SDP):**
```
User A creates offer (SDP with capabilities)
Sends to User B via signaling server
User B creates answer (SDP response)
Sends back to User A
```

**3. ICE Candidate Exchange:**
```
Both gather network addresses (ICE candidates)
Exchange via signaling server
Try all paths to find best connection
```

**4. Connection Established:**
```
Direct P2P connection created
Media (video/audio) flows directly
Signaling server no longer involved
```

**Code flow:**
```javascript
// User A
const offer = await pc.createOffer();
await pc.setLocalDescription(offer);
sendToB(offer); // Via signaling

// User B
await pc.setRemoteDescription(offer);
const answer = await pc.createAnswer();
await pc.setLocalDescription(answer);
sendToA(answer); // Via signaling

// Both exchange ICE candidates
pc.onicecandidate = (e) => sendCandidate(e.candidate);
```

### 3. What are STUN and TURN servers?

**Answer:**

**STUN (Session Traversal Utilities for NAT):**

**Purpose:** Discover public IP address
```
Your device: "My IP is 192.168.1.10"
STUN server: "The world sees you as 203.0.113.5"

Helps navigate NAT (Network Address Translation)
```

**Free STUN:**
```javascript
{ urls: 'stun:stun.l.google.com:19302' }
```

**TURN (Traversal Using Relays around NAT):**

**Purpose:** Relay traffic when direct P2P fails
```
Scenario: Both users behind strict firewalls
Direct connection impossible

Solution:
User A → TURN Server → User B
(Server relays media)
```

**When needed:** ~10-20% of connections

**Cost:** TURN uses server bandwidth (expensive)

**Config:**
```javascript
{
  iceServers: [
    { urls: 'stun:stun.example.com' },
    { 
      urls: 'turn:turn.example.com',
      username: 'user',
      credential: 'pass'
    }
  ]
}
```

**Priority:** Try STUN first (free), fall back to TURN

### 4. What is SDP in WebRTC?

**Answer:**

**SDP (Session Description Protocol):**
Text format describing media capabilities and connection info

**Contains:**
- Supported codecs (H.264, VP8, VP9)
- Media types (audio, video, data)
- Network information
- Encryption keys

**Example:**
```
v=0
o=- 123456 2 IN IP4 127.0.0.1
s=-
t=0 0
m=audio 49170 RTP/AVP 0 8
a=rtpmap:0 PCMU/8000
m=video 51372 RTP/AVP 99
a=rtpmap:99 H264/90000
```

**Usage:**
```javascript
// Create offer
const offer = await peerConnection.createOffer();
// offer.sdp contains the SDP text

// Set local description
await peerConnection.setLocalDescription(offer);

// Send to remote peer (via signaling)
signalingServer.send(offer);

// Remote peer sets it as remote description
await peerConnection.setRemoteDescription(offer);
```

**Why needed:** Both peers must agree on codecs, formats, and capabilities

### 5. How do you scale WebRTC for group calls?

**Answer:**

**Three architectures:**

**1. Mesh (P2P):**
```
Each user connects to every other user

3 users = 6 connections (3×2)
10 users = 90 connections (10×9)

Pros: No server bandwidth
Cons: Doesn't scale (CPU/bandwidth explodes)

Best for: 2-4 users
```

**2. SFU (Selective Forwarding Unit):**
```
Server forwards streams without processing

User A ──┐
User B ──┼─→ SFU ─→ Distribute to all
User C ──┘

Server: Just forwards (no encoding/decoding)
Bandwidth: Linear growth

Pros: Scales to 100+ users
Cons: Some server bandwidth

Best for: Most video calls (Zoom, Meet, Discord)
```

**3. MCU (Multipoint Control Unit):**
```
Server mixes all streams into one

User A ──┐
User B ──┼─→ MCU (mix) ─→ Single stream to all
User C ──┘

Server: Encodes/decodes/mixes
Bandwidth: Low for clients

Pros: Low client bandwidth
Cons: High server CPU, quality loss

Best for: Large webinars (100+ viewers)
```

**Recommendation:** Use SFU for most cases

### 6. What are ICE candidates?

**Answer:**

**ICE (Interactive Connectivity Establishment):**
Finds the best network path between peers

**ICE Candidate:** A possible network address to reach you

**Types:**

**1. Host (local IP):**
```
192.168.1.10:54321
Your local network address
```

**2. Server Reflexive (public IP via STUN):**
```
203.0.113.5:54321
Your public IP seen by the world
```

**3. Relay (TURN server):**
```
turn.example.com:3478
Relay address when direct fails
```

**Example:**
```javascript
peerConnection.onicecandidate = (event) => {
  if (event.candidate) {
    // Send candidate to other peer
    signalingServer.send({
      type: 'ice-candidate',
      candidate: event.candidate
    });
  }
};

// Receive candidate from peer
signalingServer.on('ice-candidate', (candidate) => {
  peerConnection.addIceCandidate(candidate);
});
```

**Process:**
1. Gather all possible addresses
2. Exchange with peer
3. Try all combinations
4. Pick the best working path

### 7. How is WebRTC encrypted?

**Answer:**

**Built-in Encryption:**

**1. DTLS (Datagram Transport Layer Security):**
```
Encrypts the connection setup
Key exchange for media encryption
```

**2. SRTP (Secure Real-time Transport Protocol):**
```
Encrypts actual media (video/audio)
End-to-end encryption in P2P mode
```

**Automatic:** No configuration needed
```javascript
// Encryption is automatic
const pc = new RTCPeerConnection();
// All media automatically encrypted
```

**What's NOT encrypted by WebRTC:**
```
Signaling messages (SDP, ICE candidates)
Solution: Use WSS (WebSocket Secure) for signaling
```

**TURN consideration:**
```
TURN server can see encrypted traffic pass through
Can't decrypt, but sees metadata (who, when, size)
Use trusted TURN servers
```

**Verification:**
```javascript
// Check if connection is secure
const stats = await peerConnection.getStats();
// Look for encryption enabled
```

### 8. What are WebRTC data channels and when would you use them?

**Answer:**

**Data Channels:** P2P arbitrary data transfer (not just video/audio)

**Features:**
- Low latency (direct P2P)
- Reliable or unreliable delivery
- Ordered or unordered
- Encrypted by default

**Creation:**
```javascript
const dataChannel = peerConnection.createDataChannel('chat', {
  ordered: true,        // Guarantee order
  maxRetransmits: 3     // Or unreliable mode
});

// Send data
dataChannel.send('Hello!');
dataChannel.send(JSON.stringify({type: 'move', x: 10}));

// Receive
dataChannel.onmessage = (event) => {
  console.log('Received:', event.data);
};
```

**Use cases:**

**1. File sharing (P2P):**
```javascript
// Send file directly
const fileReader = new FileReader();
fileReader.onload = () => {
  dataChannel.send(fileReader.result);
};
fileReader.readAsArrayBuffer(file);
```

**2. Gaming:**
```javascript
// Low-latency game state
dataChannel.send(JSON.stringify({
  player: 'alice',
  position: {x: 100, y: 200},
  action: 'shoot'
}));
```

**3. Collaborative editing:**
```javascript
// Real-time document sync
dataChannel.send(JSON.stringify({
  type: 'edit',
  position: 42,
  text: 'Hello'
}));
```

**Advantages over WebSocket:**
- Lower latency (P2P, no server)
- No server bandwidth cost
- Encrypted by default

---

## Points to Impress the Interviewer

**Show deep understanding:**
- "WebRTC uses DTLS and SRTP for automatic encryption, but signaling must be secured separately with WSS"
- "About 80-90% of connections work with STUN, only 10-20% need TURN relay"
- "SFU architecture scales better than mesh because server just forwards without encoding"

**Real-world insights:**
- "Google Meet uses SFU for group calls—server forwards streams without re-encoding to reduce CPU load"
- "Discord switched from WebRTC mesh to SFU to support larger voice channels efficiently"
- "Services like ShareDrop use data channels for P2P file sharing without server storage"

**System design:**
```
"For a video conferencing app, I'd design:
- WebSocket signaling server (for SDP/ICE exchange)
- STUN server cluster (Google's free STUN or custom)
- TURN server pool (Coturn) for ~15% of connections
- SFU architecture (mediasoup or Janus) for group calls
- HTTPS for signaling security
- Monitor ICE failure rates to optimize TURN usage"
```

---

## Quick Reference

**Connection Flow:**
```
1. Signaling: Exchange SDP offer/answer
2. ICE: Exchange network candidates
3. STUN: Find public IP (90% cases)
4. TURN: Relay if needed (10% cases)
5. P2P: Direct media transfer
```

**Architectures:**
```
2-4 users:   Mesh (P2P)
5-100 users: SFU (forward without processing)
100+ users:  MCU (mix streams)
```

**Key APIs:**
```javascript
getUserMedia()        // Get camera/mic
RTCPeerConnection()   // Main API
createOffer()         // Start connection
createAnswer()        // Respond to offer
addIceCandidate()     // Add network path
createDataChannel()   // P2P data
```

---

> WebRTC guide complete. Covers P2P architecture, signaling, STUN/TURN, scaling patterns, and real-world implementation details.