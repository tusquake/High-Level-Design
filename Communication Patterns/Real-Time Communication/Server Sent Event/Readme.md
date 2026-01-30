# Server-Sent Events (SSE)

## What is this concept?

**Server-Sent Events (SSE)** is a technology that allows servers to push real-time updates to clients over a single HTTP connection.

**Simple definition**: One-way communication channel from server to client for real-time updates.

**Think of it as**: Live news ticker - server continuously sends updates, client just receives and displays them.

## Why does this exist? / Problem it solves

**Without SSE (Polling):**
```
Client: "Any updates?" → Server: "No"
(wait 5 seconds)
Client: "Any updates?" → Server: "No"
(wait 5 seconds)
Client: "Any updates?" → Server: "Yes, here's data"

Problems:
- Wasteful (constant requests)
- Delayed updates (polling interval)
- High server load
```

**With SSE:**
```
Client: Opens connection → Server: Keeps connection open
Server: "Update 1" → Client receives
Server: "Update 2" → Client receives
Server: "Update 3" → Client receives

Benefits:
- Real-time updates
- Efficient (one connection)
- Lower server load
```

## Real-World Analogy

**Radio Broadcast**

```
Traditional HTTP (Request-Response):
You: "Any news?" → Radio: "Yes, here's news"
(wait)
You: "Any news?" → Radio: "No new news"
(wait)
You: "Any news?" → Radio: "Yes, breaking news"

SSE (Server Push):
You: Turn on radio → Radio: Continuous broadcast
Radio: "News update at 10am..."
Radio: "Traffic update..."
Radio: "Weather update..."
You just listen, no need to keep asking
```

## How it works (High Level)

### Connection Flow

```
1. Client sends HTTP request
   GET /events HTTP/1.1
   Accept: text/event-stream

2. Server responds with headers
   HTTP/1.1 200 OK
   Content-Type: text/event-stream
   Cache-Control: no-cache
   Connection: keep-alive

3. Connection stays open (persistent)

4. Server sends events when ready
   data: {"message": "New order #123"}
   
   data: {"message": "Order #123 shipped"}
   
   data: {"message": "New order #124"}

5. Client receives events in real-time
```

### Event Format

```
data: Simple message

data: {"type": "notification", "text": "New message"}

data: Multiple line
data: message spans
data: several lines

id: 123
event: userJoined
data: {"user": "Alice"}

retry: 10000
```

## Simple Example

### Live Stock Prices

**Traditional Polling:**
```javascript
// Client polls every 5 seconds
setInterval(() => {
    fetch('/api/stock-price')
        .then(res => res.json())
        .then(data => updateUI(data));
}, 5000);

Problems:
- 5 second delay
- Unnecessary requests if no change
- Server hit every 5 seconds × users
```

**SSE:**
```javascript
// Client opens SSE connection
const eventSource = new EventSource('/api/stock-stream');

eventSource.onmessage = (event) => {
    const data = JSON.parse(event.data);
    updateUI(data); // Update immediately
};

// Server pushes when price changes
Server sends:
  data: {"symbol": "AAPL", "price": 150.25}
  data: {"symbol": "AAPL", "price": 150.30}
  data: {"symbol": "AAPL", "price": 150.28}
```

---

### Live Notifications

**Server:**
```
User logs in → Opens SSE connection

Server events:
  data: {"type": "message", "from": "Alice", "text": "Hi!"}
  
  data: {"type": "like", "user": "Bob", "post": 123}
  
  data: {"type": "comment", "user": "Charlie", "post": 123}
```

**Client:**
```javascript
const eventSource = new EventSource('/notifications');

eventSource.addEventListener('message', (e) => {
    const notification = JSON.parse(e.data);
    showNotification(notification);
});
```

---

## SSE vs WebSocket vs Polling

| Feature | SSE | WebSocket | Polling |
|---------|-----|-----------|---------|
| **Direction** | Server → Client (one-way) | Both ways (two-way) | Client → Server → Client |
| **Protocol** | HTTP | WebSocket (ws://) | HTTP |
| **Complexity** | Simple | Complex | Very simple |
| **Real-time** | Yes | Yes | No (delayed) |
| **Reconnect** | Auto | Manual | N/A |
| **Browser support** | Good | Excellent | Universal |
| **Use case** | Notifications, feeds | Chat, gaming | Simple updates |

**When to use:**
- **SSE**: Server pushes updates (notifications, news feeds, stock prices)
- **WebSocket**: Two-way communication (chat, multiplayer games)
- **Polling**: Simple, occasional updates (job status check)

---

## SSE Characteristics

### 1. One-Way Communication
```
Server → Client ✓
Client → Server ✗ (use regular HTTP requests)
```

### 2. HTTP-Based
```
Uses standard HTTP/HTTPS
No special protocol needed
Works with existing infrastructure
```

### 3. Auto Reconnect
```
Connection drops → Client auto-reconnects
Built-in retry mechanism
No manual handling needed
```

### 4. Text-Only
```
Sends text data (usually JSON)
data: {"message": "Hello"}

Not binary data
```

### 5. Event IDs
```
Server can send event IDs
Client remembers last ID
Reconnect continues from last event

id: 100
data: Message 1

id: 101  
data: Message 2

(disconnect)
(reconnect with Last-Event-ID: 101)
Server sends from 102 onwards
```

---

## Use Cases

### ✓ Good for SSE

**1. Live Notifications**
```
Social media: New likes, comments, follows
Email: New mail arrived
Chat: New message indicator
```

**2. Live Feeds**
```
News ticker
Sports scores
Stock prices
Weather updates
```

**3. Progress Updates**
```
File upload progress
Job processing status
Report generation progress
```

**4. Live Dashboards**
```
System metrics
Real-time analytics
Server monitoring
```

**5. Collaborative Tools**
```
Google Docs: Show other users' cursors
Live editing indicators
Presence updates
```

---

### ✗ Not Good for SSE

**1. Two-Way Chat**
```
Need: Client sends messages frequently
Better: WebSocket (bidirectional)
```

**2. Binary Data**
```
Need: Send images, files
Better: HTTP file upload or WebSocket
```

**3. Low-Frequency Updates**
```
Need: Check job status every 10 minutes
Better: Simple polling
```

**4. Multiplayer Gaming**
```
Need: Constant two-way, low latency
Better: WebSocket or UDP
```

---

## Advantages

✓ **Simple**: Just HTTP, no special protocol
✓ **Auto-reconnect**: Built-in, no manual handling
✓ **Efficient**: One connection, multiple updates
✓ **Browser support**: Native EventSource API
✓ **Firewall-friendly**: HTTP/HTTPS works everywhere
✓ **Event streaming**: Natural for real-time feeds

---

## Disadvantages

✗ **One-way only**: Can't send from client (need separate requests)
✗ **Text only**: No binary data
✗ **Connection limits**: Browser limits concurrent connections (6 per domain)
✗ **No message acknowledgment**: Can't confirm client received
✗ **HTTP/1.1**: Not as efficient as WebSocket for high-frequency updates

---

## Implementation Pattern

### Server-Side (Node.js)

```javascript
app.get('/events', (req, res) => {
    // Set SSE headers
    res.setHeader('Content-Type', 'text/event-stream');
    res.setHeader('Cache-Control', 'no-cache');
    res.setHeader('Connection', 'keep-alive');
    
    // Send event every 5 seconds
    const intervalId = setInterval(() => {
        const data = {
            time: new Date().toISOString(),
            value: Math.random()
        };
        res.write(`data: ${JSON.stringify(data)}\n\n`);
    }, 5000);
    
    // Cleanup on disconnect
    req.on('close', () => {
        clearInterval(intervalId);
        res.end();
    });
});
```

### Client-Side (JavaScript)

```javascript
const eventSource = new EventSource('/events');

// Handle messages
eventSource.onmessage = (event) => {
    const data = JSON.parse(event.data);
    console.log('Received:', data);
};

// Handle connection open
eventSource.onopen = () => {
    console.log('Connection opened');
};

// Handle errors
eventSource.onerror = (error) => {
    console.error('SSE error:', error);
    // Auto-reconnect happens automatically
};

// Custom event types
eventSource.addEventListener('userJoined', (event) => {
    console.log('User joined:', event.data);
});

// Close connection when done
// eventSource.close();
```

---

## Best Practices

### 1. Send Keep-Alive
```
Send periodic comments to keep connection alive

// Every 30 seconds
: keep-alive
```

### 2. Use Event IDs
```
id: 123
data: Message content

Enables resuming from last event
```

### 3. Handle Reconnection
```javascript
// Client auto-reconnects
// Server should handle Last-Event-ID header
const lastEventId = req.headers['last-event-id'];
// Send events after lastEventId
```

### 4. Set Retry Time
```
retry: 10000
// Client waits 10 seconds before reconnecting
```

### 5. Limit Connections
```
Browser limit: 6 connections per domain
Use subdomains if needed:
  events1.example.com
  events2.example.com
```

### 6. Graceful Shutdown
```javascript
// Server shutdown
req.on('close', () => {
    // Cleanup resources
    clearInterval(intervalId);
    removeFromActiveConnections(clientId);
});
```

---

## Interview Explanation (90-second answer)

*"Server-Sent Events is a technology for pushing real-time updates from server to client over HTTP. Unlike traditional polling where the client repeatedly asks for updates, SSE maintains an open HTTP connection and the server pushes data whenever available.*

*It works by the client sending a request with Accept: text/event-stream, and the server responds with Content-Type: text/event-stream and keeps the connection open. The server then sends events in a simple text format—just 'data:' followed by the message. The client uses the EventSource API in JavaScript to receive these events in real-time.*

*SSE is great for one-way communication like live notifications, stock price updates, news feeds, or progress indicators. For example, in a social media app, when someone likes your post, the server immediately pushes a notification event to your browser without you having to refresh or poll.*

*The key advantage is it's simple—just HTTP, no special protocol—and has auto-reconnect built in. If the connection drops, the browser automatically reconnects. You can also use event IDs so the server knows where to resume from.*

*The main limitation is it's one-way only—server to client. If you need bidirectional communication like in a chat app, WebSocket is better. SSE is also text-only, so it's not suitable for binary data. But for real-time server-to-client updates, it's simpler and more efficient than polling and easier to implement than WebSockets."*

---

## Top Interview Questions

### 1. What is Server-Sent Events?
**Answer:**

One-way communication from server to client over HTTP for real-time updates.

**Key features:**
- Server pushes data to client
- Single persistent connection
- HTTP-based (no special protocol)
- Auto-reconnect built-in

**Example:** Live notifications, stock prices, news feeds

### 2. How is SSE different from WebSocket?
**Answer:**

| Feature | SSE | WebSocket |
|---------|-----|-----------|
| **Direction** | One-way (Server→Client) | Two-way (both) |
| **Protocol** | HTTP | WebSocket (ws://) |
| **Complexity** | Simple | More complex |
| **Use case** | Notifications, feeds | Chat, gaming |

**When to use:**
- SSE: Server pushes updates (notifications)
- WebSocket: Two-way chat, gaming

### 3. How does SSE work?
**Answer:**

**Steps:**
1. Client: `GET /events` with `Accept: text/event-stream`
2. Server: Response with `Content-Type: text/event-stream`
3. Server: Keeps connection open
4. Server: Sends events as `data: message\n\n`
5. Client: Receives events in real-time via `EventSource`

**Auto-reconnect:** If connection drops, browser reconnects automatically

### 4. What are good use cases for SSE?
**Answer:**

**Good:**
- Live notifications (likes, comments)
- News/stock feeds
- Progress updates (file upload)
- Dashboards (metrics)
- Live scores

**Not good:**
- Chat (need two-way → WebSocket)
- Gaming (need low latency → WebSocket)
- Infrequent updates (use polling)

### 5. How do you handle reconnection in SSE?
**Answer:**

**Built-in auto-reconnect:**
```javascript
const eventSource = new EventSource('/events');
// Automatically reconnects on disconnect
```

**Resume from last event:**
```
Server sends:
  id: 100
  data: Message

Client disconnects, reconnects with:
  Last-Event-ID: 100

Server continues from 101
```

**Custom retry:**
```
retry: 10000
// Wait 10 seconds before reconnecting
```

### 6. What are SSE limitations?
**Answer:**

1. **One-way only**: Can't send from client
2. **Text only**: No binary data
3. **Connection limit**: 6 per domain in browser
4. **HTTP/1.1**: Less efficient than WebSocket for high frequency
5. **No ACK**: Can't confirm client received

**Solutions:**
- Two-way: Use WebSocket
- Binary: Use WebSocket or regular HTTP
- Limit: Use subdomains

### 7. How do you implement SSE on server?
**Answer:**

**Key steps:**
1. Set headers: `Content-Type: text/event-stream`
2. Keep connection alive
3. Send events: `data: message\n\n`
4. Handle client disconnect

**Example pattern:**
```javascript
res.setHeader('Content-Type', 'text/event-stream');
res.setHeader('Cache-Control', 'no-cache');

// Send event
res.write(`data: ${JSON.stringify(data)}\n\n`);

// Cleanup on disconnect
req.on('close', () => cleanup());
```

### 8. Can SSE work through firewalls?
**Answer:**

**Yes**, because it uses standard HTTP/HTTPS.

**Advantages:**
- No special ports (80/443)
- Works with corporate firewalls
- No protocol negotiation

Unlike WebSocket which might be blocked (uses ws:// protocol).

### 9. How do you handle SSE connection limits?
**Answer:**

**Problem:** Browsers limit to 6 connections per domain.

**Solutions:**

1. **Subdomains:**
```
events1.example.com
events2.example.com
```

2. **Multiplex:**
```
One connection, multiple event types
event: notifications
event: messages
event: updates
```

3. **Use WebSocket:**
If need many connections, WebSocket better

### 10. SSE vs Long Polling?
**Answer:**

**Long Polling:**
```
Client: Request → Server: Waits → Response
Client: New request → ...
Each response = new connection
```

**SSE:**
```
Client: Request → Server: Keeps open
Server: Send, send, send...
One persistent connection
```

**SSE advantages:**
- More efficient (one connection)
- Built-in reconnect
- Simpler code
- True real-time

**Long polling advantages:**
- Universal compatibility
- Simpler server (no long connections)

---

## Points to Impress the Interviewer

- **"SSE = simple, WebSocket = powerful"** — Know when to use each
- **"Auto-reconnect built-in"** — Big advantage
- **"HTTP-based, firewall-friendly"** — Works everywhere
- **"Event IDs enable resumable streams"** — Advanced feature
- **"One-way limitation by design"** — Understand trade-offs
- **"6 connection limit workaround: subdomains"** — Practical knowledge

**Real-world insight:** "Twitter uses SSE for real-time tweet updates in feeds. When someone you follow tweets, the server pushes the update immediately to your browser without polling. This is more efficient than constantly asking 'any new tweets?'"

**System design context:** "For notification system: Use SSE for pushing notifications to browser (one-way), use regular HTTP POST for user actions (mark as read). Don't need WebSocket since notifications are mostly server→client."

---