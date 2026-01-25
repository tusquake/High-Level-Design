# API (Application Programming Interface)

## What is this concept?

**API (Application Programming Interface)** is a set of rules and protocols that allows different software applications to communicate with each other.

**Simple definition**: A contract that defines how two systems talk to each other.

**Think of it as**: A restaurant menu - it tells you what you can order (available functions) and what you'll get back (response), without needing to know how the kitchen works.

## Why does this exist? / Problem it solves

- **Abstraction**: Use services without knowing internal implementation
- **Integration**: Connect different systems (mobile app ↔ backend ↔ database)
- **Reusability**: Write once, use everywhere (web, mobile, desktop)
- **Separation of concerns**: Frontend and backend can be developed independently
- **Third-party access**: Allow others to use your service (Google Maps API, Payment APIs)

Without APIs, every app would need to rebuild everything from scratch. No integrations, no third-party services.

## Real-World Analogy

**Waiter in a Restaurant**

```
You (Client) ↔ Waiter (API) ↔ Kitchen (Server)

You don't enter the kitchen directly
You tell waiter what you want (API request)
Waiter gives your order to kitchen
Kitchen prepares food
Waiter brings you food (API response)

You don't know:
- How food is cooked
- Kitchen layout
- Chef's name

You only need:
- Menu (API documentation)
- How to order (request format)
```

## Types of APIs

### 1. REST API (Most Common)

**REST** = **RE**presentational **S**tate **T**ransfer

**Characteristics:**
- Uses HTTP methods (GET, POST, PUT, DELETE)
- Stateless (each request independent)
- Works with JSON/XML
- Resource-based URLs

**Example:**
```
GET    /api/users          → Get all users
GET    /api/users/123      → Get user with ID 123
POST   /api/users          → Create new user
PUT    /api/users/123      → Update user 123
DELETE /api/users/123      → Delete user 123
```

**Pros:**
- Simple and widely used
- Cacheable
- Flexible (any client can use)
- Stateless (scales well)

**Cons:**
- Over-fetching (gets more data than needed)
- Multiple requests for related data
- No built-in real-time support

---

### 2. GraphQL API

**What:** Query language for APIs, request exactly what you need.

**Characteristics:**
- Single endpoint (`/graphql`)
- Client specifies data structure
- Gets exactly what's requested (no over-fetching)
- Strongly typed

**Example:**
```graphql
query {
  user(id: 123) {
    name
    email
    posts {
      title
    }
  }
}
```

**Response:**
```json
{
  "user": {
    "name": "Alice",
    "email": "alice@example.com",
    "posts": [
      {"title": "My First Post"}
    ]
  }
}
```

**Pros:**
- No over-fetching or under-fetching
- Single request for complex data
- Strongly typed schema
- Great for mobile (saves bandwidth)

**Cons:**
- More complex than REST
- Harder to cache
- Learning curve
- Overkill for simple APIs

**When to use:** Complex data relationships, mobile apps, frequent schema changes

---

### 3. SOAP API

**SOAP** = **S**imple **O**bject **A**ccess **P**rotocol

**Characteristics:**
- XML-based protocol
- Strict standards (WSDL)
- Built-in security (WS-Security)
- Supports transactions

**Example:**
```xml
<soap:Envelope>
  <soap:Body>
    <GetUser>
      <UserId>123</UserId>
    </GetUser>
  </soap:Body>
</soap:Envelope>
```

**Pros:**
- Built-in security
- ACID transactions
- Standardized
- Good for enterprise

**Cons:**
- Verbose (XML)
- Complex
- Slower than REST
- Less flexible

**When to use:** Banking, legacy enterprise systems, when strict standards required

---

### 4. gRPC

**gRPC** = **g**oogle **R**emote **P**rocedure **C**all

**Characteristics:**
- Uses Protocol Buffers (binary format)
- HTTP/2 based
- Strongly typed
- Bidirectional streaming

**Example:**
```protobuf
service UserService {
  rpc GetUser (UserId) returns (User);
}

message UserId {
  int32 id = 1;
}

message User {
  int32 id = 1;
  string name = 2;
}
```

**Pros:**
- Very fast (binary format)
- Streaming support (real-time)
- Strongly typed
- Efficient (smaller payloads)

**Cons:**
- Not human-readable
- Browser support limited
- Steep learning curve

**When to use:** Microservices communication, real-time streaming, high-performance requirements

---

## API Request/Response

### REST API Example

**Request:**
```http
POST /api/users HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer <token>

{
  "name": "Alice",
  "email": "alice@example.com"
}
```

**Response:**
```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 123,
  "name": "Alice",
  "email": "alice@example.com",
  "created_at": "2024-01-15T10:30:00Z"
}
```

### HTTP Methods (REST)

| Method | Purpose | Example |
|--------|---------|---------|
| **GET** | Retrieve data | Get user list |
| **POST** | Create new resource | Create user |
| **PUT** | Update entire resource | Update user (all fields) |
| **PATCH** | Partial update | Update user email only |
| **DELETE** | Delete resource | Delete user |

### HTTP Status Codes

| Code | Meaning | Use Case |
|------|---------|----------|
| **200** | OK | Request successful |
| **201** | Created | Resource created |
| **204** | No Content | Deleted successfully |
| **400** | Bad Request | Invalid input |
| **401** | Unauthorized | Authentication required |
| **403** | Forbidden | No permission |
| **404** | Not Found | Resource doesn't exist |
| **500** | Internal Server Error | Server crashed |
| **503** | Service Unavailable | Server overloaded |

## API Authentication

### 1. API Key

Simple token in header or URL.

```http
GET /api/users
X-API-Key: abc123xyz789
```

**Pros:** Simple
**Cons:** Not very secure, no expiration

---

### 2. OAuth 2.0

Token-based authentication with scopes.

```http
GET /api/users
Authorization: Bearer eyJhbGc...
```

**Pros:** Secure, standard, token expiration
**Cons:** More complex

**Use:** Google login, Facebook login, third-party access

---

### 3. JWT (JSON Web Token)

Self-contained token with user info.

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJ1c2VyX2lkIjoxMjMsImV4cCI6MTY0MDk5NTIwMH0.
abc123xyz789...
```

**Contains:** User ID, expiration time, signature

**Pros:** Stateless, contains user info
**Cons:** Can't revoke (until expiration)

---

## API Design Best Practices

### 1. RESTful URL Design

**Good:**
```
GET    /api/users           → List users
GET    /api/users/123       → Get specific user
POST   /api/users           → Create user
GET    /api/users/123/posts → Get user's posts
```

**Bad:**
```
GET /api/getAllUsers         ✗ (verb in URL)
GET /api/user?id=123         ✗ (use path, not query)
POST /api/deleteUser         ✗ (use DELETE method)
```

**Rules:**
- Use nouns, not verbs
- Use plural for collections (`/users` not `/user`)
- Use HTTP methods for actions
- Nested resources for relationships

---

### 2. Versioning

Always version your API to avoid breaking changes.

```
/api/v1/users
/api/v2/users
```

Or in header:
```
Accept: application/vnd.myapi.v2+json
```

---

### 3. Pagination

Don't return all records, paginate large results.

```
GET /api/users?page=2&limit=20

Response:
{
  "data": [...],
  "pagination": {
    "current_page": 2,
    "total_pages": 10,
    "total_items": 200
  }
}
```

---

### 4. Error Handling

Return meaningful error messages.

```json
{
  "error": {
    "code": "INVALID_EMAIL",
    "message": "Email format is invalid",
    "field": "email"
  }
}
```

---

## Where APIs are used

**Web/Mobile Apps:**
- Frontend calls backend APIs (React → Node.js API)
- Mobile apps call APIs (iOS/Android → REST API)

**Third-Party Integrations:**
- Payment gateways (Stripe, PayPal)
- Maps (Google Maps API)
- Social login (Facebook, Google OAuth)
- SMS (Twilio API)

**Microservices:**
- Service-to-service communication
- Order Service → Payment Service (via API)

**IoT Devices:**
- Smart devices send data via APIs
- Thermostat → Cloud API → Mobile app

**Public APIs:**
- Twitter API (post tweets programmatically)
- GitHub API (access repositories)
- Weather API (get weather data)

---

## API Gateway

**What:** Single entry point for all API requests.

```
Clients → API Gateway → Microservices
             ↓
        - Authentication
        - Rate limiting
        - Routing
        - Load balancing
```

**Benefits:**
- Centralized authentication
- Rate limiting per client
- Request routing
- Protocol translation (REST → gRPC)
- Monitoring and logging

**Examples:** AWS API Gateway, Kong, Apigee, Azure API Management

---

## Interview Explanation (2-minute answer)

*"An API is an interface that allows different software systems to communicate. It defines a contract—what requests you can make, what format to use, and what responses you'll get back.*

*The most common type is REST API, which uses HTTP methods like GET, POST, PUT, DELETE to perform operations on resources identified by URLs. For example, GET /api/users/123 retrieves user 123, POST /api/users creates a new user. REST is simple, stateless, and works with JSON, making it widely adopted.*

*GraphQL is an alternative where clients specify exactly what data they need in a single query, avoiding over-fetching. It's great for mobile apps and complex data relationships but more complex to implement.*

*For internal microservices communication, gRPC is popular because it's fast—it uses binary Protocol Buffers instead of JSON and supports bidirectional streaming. Companies like Netflix and Uber use gRPC for service-to-service calls.*

*Authentication is critical for APIs. Common methods include API keys for simple use cases, OAuth 2.0 for third-party access like 'Login with Google,' and JWT tokens for stateless authentication where the token contains user information and can be verified without database lookups.*

*In production, APIs sit behind an API Gateway which handles authentication, rate limiting, routing, and load balancing. This centralizes cross-cutting concerns. For example, AWS API Gateway or Kong can authenticate requests, enforce rate limits, and route to appropriate backend services.*

*Key design principles include versioning APIs to avoid breaking changes, using proper HTTP status codes, implementing pagination for large datasets, and designing RESTful URLs with nouns not verbs."*

---

## Top Interview Questions

### 1. What is an API and why is it needed?
**Answer:**

API = Set of rules for communication between software systems.

**Why needed:**
- **Integration**: Connect frontend ↔ backend ↔ database
- **Reusability**: One backend API serves web, mobile, desktop
- **Abstraction**: Use services without knowing internals
- **Third-party access**: Let others use your service

**Example:** Weather app calls OpenWeather API for data instead of building own weather tracking system.

### 2. What's the difference between REST and GraphQL?
**Answer:**

**REST:**
- Multiple endpoints (`/users`, `/posts`)
- Fixed response structure
- May over-fetch or under-fetch data
- Simple and widely used

**GraphQL:**
- Single endpoint (`/graphql`)
- Client specifies exact data needed
- No over-fetching
- More complex but flexible

**Example:**
REST: 3 requests to get user, posts, comments
GraphQL: 1 request for all data

**Use REST for:** Simple APIs, public APIs
**Use GraphQL for:** Complex data, mobile apps, frequent changes

### 3. What are HTTP methods and when to use them?
**Answer:**

- **GET**: Retrieve data (read-only, idempotent)
- **POST**: Create new resource (not idempotent)
- **PUT**: Update entire resource (idempotent)
- **PATCH**: Partial update (idempotent)
- **DELETE**: Remove resource (idempotent)

**Idempotent** = Same request multiple times = same result

**Example:**
- GET /users/123 → Always returns same user
- POST /users → Creates new user each time (different IDs)

### 4. What are common API authentication methods?
**Answer:**

**1. API Key:**
```
X-API-Key: abc123
```
Simple but less secure.

**2. OAuth 2.0:**
```
Authorization: Bearer <token>
```
Standard for third-party access (Google login).

**3. JWT (JSON Web Token):**
Self-contained token with user info, expiration.

**Best practice:** Use OAuth 2.0 or JWT for production APIs.

### 5. What are important HTTP status codes?
**Answer:**

**Success:**
- 200 OK - Request successful
- 201 Created - Resource created
- 204 No Content - Successful, no body

**Client Errors:**
- 400 Bad Request - Invalid input
- 401 Unauthorized - Authentication required
- 403 Forbidden - No permission
- 404 Not Found - Resource doesn't exist

**Server Errors:**
- 500 Internal Server Error - Server crashed
- 503 Service Unavailable - Server overloaded

### 6. What is an API Gateway?
**Answer:**

Single entry point for all API requests.

**Functions:**
- Authentication/Authorization
- Rate limiting (prevent abuse)
- Request routing (to correct service)
- Load balancing
- Logging and monitoring
- Protocol translation (REST → gRPC)

**Examples:** AWS API Gateway, Kong, Apigee

**Use case:** Microservices where gateway routes /users → User Service, /orders → Order Service.

### 7. What is REST and what makes an API RESTful?
**Answer:**

**REST principles:**

1. **Stateless**: Each request independent (no session on server)
2. **Resource-based**: URLs represent resources (`/users/123`)
3. **HTTP methods**: GET, POST, PUT, DELETE for operations
4. **Client-server**: Separation of concerns
5. **Cacheable**: Responses can be cached

**RESTful URL examples:**
```
✓ GET /api/users
✓ POST /api/users
✓ GET /api/users/123/posts
✗ GET /api/getUser?id=123 (not RESTful)
```

### 8. How do you version an API?
**Answer:**

**1. URL versioning (most common):**
```
/api/v1/users
/api/v2/users
```

**2. Header versioning:**
```
Accept: application/vnd.myapi.v2+json
```

**3. Query parameter:**
```
/api/users?version=2
```

**Best practice:** URL versioning (clear and simple).

**Why version?** Avoid breaking existing clients when making changes.

### 9. What is the difference between PUT and PATCH?
**Answer:**

**PUT**: Replace entire resource
```json
PUT /api/users/123
{
  "name": "Alice",
  "email": "alice@example.com",
  "age": 30
}
// Must send all fields
```

**PATCH**: Update specific fields
```json
PATCH /api/users/123
{
  "email": "newemail@example.com"
}
// Only send fields to update
```

**Use PUT for:** Complete replacement
**Use PATCH for:** Partial updates (more efficient)

### 10. What is rate limiting and why is it important?
**Answer:**

**Rate Limiting**: Restrict number of API requests per time period.

**Example:**
```
100 requests per minute per user
Exceed → 429 Too Many Requests
```

**Why important:**
- Prevent abuse (DDoS attacks)
- Fair resource allocation
- Protect backend from overload
- Enforce pricing tiers (free: 100/min, paid: 10000/min)

**Implementation:**
- API Gateway (AWS, Kong)
- Redis counters
- Token bucket algorithm

---

## Points to Impress the Interviewer

- **"REST for simplicity, GraphQL for flexibility, gRPC for performance"** — Know when to use each
- **"API Gateway centralizes cross-cutting concerns"** — Authentication, rate limiting, routing
- **"Idempotency is critical for reliability"** — Retry-safe operations
- **"Always version APIs"** — Avoid breaking changes
- **"JWT is stateless, scales well"** — No database lookup needed
- **"Use proper status codes"** — 201 for created, 204 for deleted, 404 for not found

**Real-world insight:** "Netflix API Gateway (Zuul) handles billions of requests daily, routing to 500+ microservices with authentication, rate limiting, and circuit breaking—all without touching individual services."

---