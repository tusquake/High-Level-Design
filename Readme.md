# What is System Design?

When you scroll through Instagram, stream your favorite show on Netflix, or shop on Amazon, you probably don’t pause to think about what’s happening behind the scenes.

But with every tap, click, or refresh, a complex network of interconnected components works seamlessly to deliver a smooth experience.

Behind this seamless experience lies the **art and science of System Design**.

---

## 1. What Is System Design?

At its core, **System Design** is the process of defining how different parts of a software system interact to meet both:

- **Functional requirements** (what the system should do)
- **Non-functional requirements** (how well it should do it)

System design is **not about writing code**—at least not yet.  
It’s about making **high-level architectural decisions** that balance:

- Scalability
- Reliability
- Performance
- Cost

---

### A Real-World Analogy: Designing a City Traffic System 🚦

Imagine you’re responsible for designing the traffic system of a growing city.

You don’t start by placing traffic lights randomly. You start by asking questions:

- How many people live in the city today?
- How many vehicles are expected during peak hours?
- Where are the major entry and exit points?
- What happens during emergencies or road closures?
- How should the system scale as the city grows?

Once the requirements are clear, you design the blueprint:

- Roads and highways
- Traffic signals and roundabouts
- Flyovers and tunnels
- Public transport routes
- Monitoring systems (cameras, sensors)

You also think about:

- **Scalability:** Adding new roads as the city expands
- **Fault tolerance:** What happens if a signal fails?
- **Performance:** Reducing traffic jams and delays
- **Inter-system interaction:** How buses, cars, pedestrians, and emergency vehicles coexist

In the software world, this translates to:

- **Architecture:** Monolith, microservices, or event-driven systems
- **Components/Modules:** Databases, servers, load balancers, caches, message queues, APIs
- **Interfaces:** How components communicate (REST, gRPC, messaging)
- **Data:** How data is stored, accessed, and kept consistent

---

## 2. 10 Big Questions of System Design

On a high level, system design revolves around answering these key questions:

1. **Scalability:** How will the system handle a large number of users or requests?
2. **Latency and Performance:** How do we keep response times low under load?
3. **Communication:** How do components interact with each other?
4. **Data Management:** How should data be stored and retrieved efficiently?
5. **Fault Tolerance and Reliability:** What happens when a component fails?
6. **Security:** How do we protect against unauthorized access and attacks?
7. **Maintainability and Extensibility:** How easy is it to evolve and debug the system?
8. **Cost Efficiency:** How do we balance performance with infrastructure cost?
9. **Observability and Monitoring:** How do we detect and diagnose production issues?
10. **Compliance and Privacy:** Are we meeting legal and regulatory requirements (GDPR, HIPAA)?

---

## 3. Key Components of a System

A typical software system consists of the following components:

### Client / Frontend
- Web browsers or mobile apps
- Displays data, collects user input
- Communicates with backend services

### Server / Backend
- Processes requests
- Executes business logic
- Interacts with databases and other services
- Returns responses to clients

### Database / Storage
- Stores and manages data
- Can be SQL, NoSQL, in-memory caches, or object storage
- Chosen based on access patterns and scale

### Networking Layer
- Load balancers
- APIs
- Communication protocols
- Ensures reliability and performance

### Third-Party Services
- Payments, emails, SMS
- Authentication providers
- Analytics tools
- Cloud-based AI services

---

## 4. The Process of System Design

System design follows a structured, step-by-step approach.

---

### Step 1: Requirements Gathering

Every design starts with understanding **what to build**.

Ask questions like:

- What are the functional requirements?
- What are the non-functional requirements (latency, availability, consistency)?
- Who are the users?
- What scale is expected initially and in the future?
- Any constraints (budget, tech stack, compliance)?

---

### Step 2: Back-of-the-Envelope Estimation

Estimate the system scale using rough numbers:

- Data size
- Requests per second (RPS / QPS)
- Bandwidth requirements
- Number of servers or instances

These estimates guide architectural decisions.

---

### Step 3: High-Level Design (HLD)

Create a bird’s-eye view of the system:

- Major services and components
- Data flow between them
- External dependencies

This is your **architecture blueprint**.

---

### Step 4: Data Model / API Design

Dive into structure and interfaces:

- Choose database types (SQL, NoSQL, time-series)
- Design schemas and relationships
- Define APIs (e.g., `POST /tweet`, `GET /timeline`)

---

### Step 5: Detailed Design / Deep Dive

Zoom into each component:

- Internal logic
- Caching strategies
- Concurrency handling
- Scaling approaches
- Replication and fault tolerance

This is where **non-functional requirements** are fully addressed.

---

### Step 6: Identify Bottlenecks and Trade-offs

No design is perfect.

Ask:

- Where can the system fail?
- What are the single points of failure?
- Can caching or replication help?
- Is eventual consistency acceptable?

Great designs **acknowledge and justify trade-offs**.

---

### Step 7: Review, Explain, and Iterate

- Clearly explain design decisions
- Justify trade-offs
- Incorporate feedback
- Refine weak areas

System design is iterative, not one-shot.

---

## 5. Conclusion

System design is a critical skill for building **scalable, reliable, and maintainable software systems**.

Whether you're designing a small application or a massive distributed platform, strong system design fundamentals help you:

- Make better architectural decisions
- Choose the right technologies
- Optimize performance with confidence

The first step to mastering system design is understanding its **core concepts and building blocks**.

---
