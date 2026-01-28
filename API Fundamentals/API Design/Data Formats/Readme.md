# Data Formats

## What is this concept?

**Data Formats**: Structured ways to represent and exchange data between systems, applications, or services.

**Simple definition**: The "language" systems use to talk to each other (JSON, XML, CSV, Protocol Buffers, etc.).

**Think of it as**: Different ways to write down information:
- **JSON**: Like a labeled filing cabinet (human-readable, key-value pairs)
- **XML**: Like a structured document with tags (verbose but flexible)
- **CSV**: Like a spreadsheet (rows and columns)
- **Protocol Buffers**: Like shorthand notation (compact, fast)

## Why does this exist? / Problem it solves

**Problem: Systems Can't Communicate**
```
Server sends: 010101010110
Client receives: ???

Without agreed format:
❌ Can't parse data
❌ Lost information
❌ Different interpretations
```

**Solution: Standard Data Formats**
```
Server sends JSON: {"name": "Alice", "age": 25}
Client parses: Knows exactly what each field means ✓
```

**Problems solved:**
- **Interoperability**: Different languages/platforms communicate
- **Data exchange**: APIs, databases, file transfers
- **Human readability**: Debug and understand data
- **Efficiency**: Balance between size and speed

## Common Data Formats Comparison

| Format | Human Readable | Size | Speed | Use Case |
|--------|---------------|------|-------|----------|
| **JSON** | ✓ High | Medium | Medium | REST APIs, configs |
| **XML** | ✓ High | Large | Slow | Legacy systems, SOAP |
| **CSV** | ✓ High | Small | Fast | Spreadsheets, exports |
| **Protocol Buffers** | ✗ Low | Very Small | Very Fast | Microservices, gRPC |
| **MessagePack** | ✗ Low | Small | Fast | Binary JSON alternative |
| **YAML** | ✓ Very High | Medium | Slow | Config files, Kubernetes |
| **Avro** | ✗ Low | Small | Fast | Big data, Kafka |

---

## JSON (JavaScript Object Notation)

**Most popular format for REST APIs**

### Structure
```json
{
  "user": {
    "id": 123,
    "name": "Alice",
    "email": "alice@example.com",
    "roles": ["admin", "user"],
    "active": true,
    "metadata": null
  }
}
```

### Data Types
```json
{
  "string": "Hello",
  "number": 42,
  "float": 3.14,
  "boolean": true,
  "null": null,
  "array": [1, 2, 3],
  "object": {"key": "value"}
}
```

### Pros ✓
- Human-readable
- Language-agnostic (every language has JSON parser)
- Native to JavaScript
- Compact (compared to XML)
- Simple syntax

### Cons ✗
- No comments allowed
- No date type (use strings: "2024-01-26T10:00:00Z")
- Larger than binary formats
- No schema validation (need external tools like JSON Schema)

### Use Cases
```
✓ REST APIs
✓ Configuration files
✓ NoSQL databases (MongoDB)
✓ Web applications
✗ Large datasets (use Avro, Parquet)
✗ High-performance microservices (use Protocol Buffers)
```

### Example - API Response
```json
{
  "status": "success",
  "data": {
    "users": [
      {
        "id": 1,
        "name": "Alice",
        "created_at": "2024-01-26T10:00:00Z"
      },
      {
        "id": 2,
        "name": "Bob",
        "created_at": "2024-01-25T09:30:00Z"
      }
    ]
  },
  "pagination": {
    "total": 100,
    "page": 1,
    "per_page": 20
  }
}
```

---

## XML (eXtensible Markup Language)

**Older format, still used in legacy systems**

### Structure
```xml
<?xml version="1.0" encoding="UTF-8"?>
<user>
  <id>123</id>
  <name>Alice</name>
  <email>alice@example.com</email>
  <roles>
    <role>admin</role>
    <role>user</role>
  </roles>
  <active>true</active>
</user>
```

### Pros ✓
- Self-documenting with tags
- Attributes and elements for metadata
- Schema validation (XSD)
- Supports namespaces
- Comment support

### Cons ✗
- Verbose (2-3x larger than JSON)
- Slower to parse
- More complex syntax
- Closing tags redundant

### JSON vs XML
```json
// JSON (62 characters)
{"name":"Alice","age":25}

// XML (89 characters)
<user>
  <name>Alice</name>
  <age>25</age>
</user>
```

### Use Cases
```
✓ SOAP APIs
✓ Configuration files (Maven, Ant)
✓ Document formats (SVG, RSS, XHTML)
✓ Enterprise systems
✗ Modern REST APIs (JSON preferred)
✗ Mobile apps (too verbose)
```

---

## CSV (Comma-Separated Values)

**Simple tabular data format**

### Structure
```csv
id,name,email,age
1,Alice,alice@example.com,25
2,Bob,bob@example.com,30
3,Charlie,charlie@example.com,28
```

### Pros ✓
- Human-readable
- Very small file size
- Opens in Excel/Google Sheets
- Simple to parse
- Universal support

### Cons ✗
- No nested data (flat structure only)
- No data types (everything is string)
- Escaping issues (commas in values)
- No schema
- Can't represent complex objects

### Handling Special Characters
```csv
id,name,description
1,Alice,"Product with, comma"
2,Bob,"Product with ""quotes"""
3,Charlie,"Product with
newline"
```

### Use Cases
```
✓ Data exports (reports, analytics)
✓ Spreadsheet imports
✓ Log files
✓ Bulk data transfers
✗ APIs (use JSON)
✗ Nested data (use JSON, XML)
```

---

## Protocol Buffers (Protobuf)

**Binary format by Google for high-performance systems**

### Schema Definition (.proto file)
```protobuf
syntax = "proto3";

message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
  repeated string roles = 4;
  bool active = 5;
}
```

### Binary Encoding
```
User data:
JSON:     {"id":123,"name":"Alice","email":"alice@example.com"}
Protobuf: \x08{\x12\x05Alice\x1a\x11alice@example.com (binary)

Size comparison:
JSON:     56 bytes
Protobuf: 28 bytes (50% smaller)
```

### Pros ✓
- Very fast serialization/deserialization
- Small payload size (3-10x smaller than JSON)
- Strongly typed with schema
- Backward/forward compatibility
- Code generation for multiple languages

### Cons ✗
- Not human-readable (binary)
- Requires schema file
- More setup complexity
- Debugging harder

### Use Cases
```
✓ gRPC microservices
✓ High-performance systems
✓ Internal APIs (not public)
✓ Large-scale data processing
✗ Public REST APIs (JSON preferred)
✗ Browser-based apps (no native support)
```

---

## YAML (YAML Ain't Markup Language)

**Human-friendly configuration format**

### Structure
```yaml
user:
  id: 123
  name: Alice
  email: alice@example.com
  roles:
    - admin
    - user
  active: true
  metadata: null
```

### Features
```yaml
# Comments allowed
# Indentation matters (like Python)
# No quotes needed for strings
# Multi-line strings

description: |
  This is a multi-line
  string in YAML
  
inline_list: [1, 2, 3]
inline_object: {key: value}
```

### Pros ✓
- Most human-readable
- Comments supported
- Less syntax clutter (no braces, quotes)
- Anchors and aliases for reuse

### Cons ✗
- Indentation errors common
- Slower to parse
- Security issues (arbitrary code execution in some parsers)
- Not suitable for APIs

### Use Cases
```
✓ Configuration files (Docker Compose, Kubernetes)
✓ CI/CD pipelines (GitHub Actions, GitLab CI)
✓ Application configs
✗ APIs (use JSON)
✗ Performance-critical (use Protobuf)
```

### Example - Kubernetes Config
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: web
    image: nginx:latest
    ports:
    - containerPort: 80
```

---

## MessagePack

**Binary JSON alternative**

### Concept
```
JSON in binary format:
- Faster than JSON
- Smaller than JSON
- Schema-less (like JSON)

Example:
JSON:        {"name":"Alice","age":25}
MessagePack: Binary representation (smaller)
```

### Pros ✓
- Faster than JSON (2-5x)
- Smaller than JSON (10-30%)
- No schema required
- Compatible with JSON semantics

### Cons ✗
- Not human-readable
- Less ecosystem support than JSON
- Debugging harder

### Use Cases
```
✓ Real-time applications (chat, gaming)
✓ High-throughput APIs
✓ Cache serialization (Redis)
✗ Public APIs (JSON preferred for compatibility)
```

---

## Avro

**Binary format for big data (Apache project)**

### Schema (JSON format)
```json
{
  "type": "record",
  "name": "User",
  "fields": [
    {"name": "id", "type": "int"},
    {"name": "name", "type": "string"},
    {"name": "email", "type": "string"}
  ]
}
```

### Pros ✓
- Compact binary format
- Schema evolution (add/remove fields)
- Fast serialization
- Used in Hadoop ecosystem

### Cons ✗
- Requires schema
- Not human-readable
- Less language support than Protobuf

### Use Cases
```
✓ Apache Kafka (event streaming)
✓ Hadoop, Spark (big data)
✓ Data lakes
✗ REST APIs
✗ Small-scale applications
```

---

## When to Use Each Format

### JSON
```
Use when:
✓ Building REST APIs
✓ Web/mobile applications
✓ Human readability matters
✓ Interoperability important

Example: Public API responses
```

### XML
```
Use when:
✓ Legacy system integration
✓ SOAP APIs required
✓ Complex document structure
✓ Schema validation critical

Example: Enterprise SOAP services
```

### CSV
```
Use when:
✓ Tabular data exports
✓ Spreadsheet imports
✓ Simple flat structure
✓ Human editing needed

Example: Analytics reports
```

### Protocol Buffers
```
Use when:
✓ Microservice communication
✓ Performance critical
✓ Internal APIs
✓ Schema evolution needed

Example: gRPC services
```

### YAML
```
Use when:
✓ Configuration files
✓ Human readability critical
✓ Comments needed
✓ Not performance-critical

Example: Docker Compose, Kubernetes
```

### MessagePack
```
Use when:
✓ Binary JSON needed
✓ Performance matters
✓ Schema-less flexibility
✓ Internal systems

Example: Real-time messaging
```

### Avro
```
Use when:
✓ Big data processing
✓ Kafka event streaming
✓ Schema evolution important
✓ Hadoop ecosystem

Example: Data pipelines
```

---

## Performance Comparison

**Serialization benchmark (10,000 objects):**

| Format | Size | Serialize | Deserialize |
|--------|------|-----------|-------------|
| JSON | 1.0 MB | 100 ms | 120 ms |
| XML | 2.5 MB | 250 ms | 300 ms |
| Protobuf | 0.3 MB | 20 ms | 25 ms |
| MessagePack | 0.7 MB | 40 ms | 50 ms |
| Avro | 0.4 MB | 30 ms | 35 ms |

**Takeaway:**
- Binary formats (Protobuf, Avro) = fastest, smallest
- JSON = good balance for most use cases
- XML = slowest, largest (avoid for new systems)

---

## Best Practices

### 1. Choose Format Based on Use Case

```
Public API → JSON
Internal microservices → Protobuf or gRPC
Config files → YAML
Data export → CSV
Big data → Avro
Legacy integration → XML
```

### 2. Version Your Schemas

```protobuf
// Protobuf versioning
message UserV1 {
  int32 id = 1;
  string name = 2;
}

message UserV2 {
  int32 id = 1;
  string name = 2;
  string email = 3;  // New field
}
```

### 3. Validate Data

**JSON Schema:**
```json
{
  "type": "object",
  "properties": {
    "name": {"type": "string"},
    "age": {"type": "number", "minimum": 0}
  },
  "required": ["name"]
}
```

### 4. Handle Null/Missing Values

```json
// Explicit null
{"email": null}

// Missing field (undefined)
{"name": "Alice"}  // no email field

// Empty string
{"email": ""}

// Choose convention and document it
```

### 5. Date/Time Formatting

**Use ISO 8601:**
```json
{
  "created_at": "2024-01-26T10:00:00Z",      // UTC
  "updated_at": "2024-01-26T15:30:00+05:30"  // With timezone
}
```

**Not:**
```json
{
  "created_at": "01/26/2024",  // Ambiguous
  "timestamp": 1706265600      // Unix timestamp (harder to read)
}
```

### 6. Consistent Naming Conventions

**JSON - camelCase or snake_case:**
```json
// camelCase (JavaScript convention)
{"firstName": "Alice", "createdAt": "2024-01-26"}

// snake_case (Python, Ruby convention)
{"first_name": "Alice", "created_at": "2024-01-26"}

Pick one, be consistent across entire API
```

---

## Common Pitfalls

### ❌ Mixing Data Formats
```
API endpoint 1 returns JSON
API endpoint 2 returns XML
→ Inconsistent, confusing
```
**Fix:** Standardize on one format (usually JSON)

### ❌ No Schema Validation
```json
// Accepting any data
{"name": 123, "age": "not a number"}
```
**Fix:** Validate with JSON Schema, Protobuf schema, etc.

### ❌ Inefficient Format for Use Case
```
Using JSON for high-frequency microservice calls
→ 5x slower than Protobuf
```
**Fix:** Use binary formats for performance-critical paths

### ❌ Not Handling Special Characters in CSV
```csv
name,description
Alice,Product with, comma  // Breaks parsing
```
**Fix:** Quote fields with special characters
```csv
name,description
Alice,"Product with, comma"
```

---

## Interview Explanation (60-second answer)

*"Data formats define how systems exchange information. The most common is JSON—human-readable, key-value pairs, used in nearly all REST APIs. It's lightweight, language-agnostic, and strikes a balance between readability and size.*

*XML is older and more verbose—2-3x larger than JSON—but still used in legacy systems and SOAP APIs. It supports schema validation and comments but has fallen out of favor for modern applications.*

*For performance-critical systems, binary formats like Protocol Buffers are used. Protobuf is 50-70% smaller than JSON and 5-10x faster to serialize. It requires a schema definition but provides strong typing and backward compatibility. It's the foundation of gRPC.*

*CSV is for tabular data—exports, spreadsheets, bulk transfers. It's simple but can't represent nested structures. YAML is for configuration files—very readable with comments, used in Docker and Kubernetes.*

*The choice depends on your use case: JSON for public APIs and web apps, Protobuf for internal microservices, CSV for data exports, YAML for configs. In interviews, know when to optimize for readability versus performance."*

---

## Top Interview Questions

### 1. What are the main differences between JSON and XML?

**Answer:**

**Size:**
- JSON: Compact
- XML: 2-3x larger (verbose tags)

**Readability:**
- JSON: Cleaner, less clutter
- XML: More verbose with closing tags

**Data types:**
- JSON: String, number, boolean, null, array, object
- XML: Everything is text (needs parsing)

**Comments:**
- JSON: Not supported
- XML: Supported `<!-- comment -->`

**Example:**
```json
// JSON (36 bytes)
{"name":"Alice","age":25}
```
```xml
<!-- XML (68 bytes) -->
<user>
  <name>Alice</name>
  <age>25</age>
</user>
```

**When to use:**
- JSON: Modern REST APIs, web apps
- XML: Legacy systems, SOAP, complex documents

### 2. Why use Protocol Buffers instead of JSON?

**Answer:**

**Performance:**
- 5-10x faster serialization
- 50-70% smaller payload
- Lower CPU usage

**Schema:**
- Strongly typed (compile-time validation)
- Backward/forward compatibility
- Code generation

**Comparison:**
```
User object:
JSON:     56 bytes, 100ms parse time
Protobuf: 28 bytes, 20ms parse time
```

**Trade-offs:**
- Not human-readable (binary)
- Requires .proto schema file
- More complex setup

**When to use:**
- Internal microservices (gRPC)
- High-throughput systems
- Mobile apps (reduce bandwidth)

**When NOT to use:**
- Public REST APIs (JSON better for compatibility)
- Browser-based apps (limited support)

### 3. What is JSON Schema and why use it?

**Answer:**

**JSON Schema**: Defines structure and validation rules for JSON data.

**Example:**
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "name": {"type": "string", "minLength": 1},
    "age": {"type": "integer", "minimum": 0},
    "email": {"type": "string", "format": "email"}
  },
  "required": ["name", "email"]
}
```

**Benefits:**
1. **Validation**: Reject invalid data
2. **Documentation**: Self-documenting API
3. **Code generation**: Auto-generate types
4. **Client validation**: Validate before sending

**Use cases:**
- API request/response validation
- Configuration file validation
- Form validation

### 4. How do you handle dates in JSON?

**Answer:**

**Problem:** JSON has no date type

**Solution: ISO 8601 strings**
```json
{
  "created_at": "2024-01-26T10:00:00Z",      // UTC
  "updated_at": "2024-01-26T15:30:00+05:30"  // With timezone
}
```

**Alternatives:**
```json
// Unix timestamp (milliseconds)
{"created_at": 1706265600000}

// Date only
{"birth_date": "2024-01-26"}
```

**Best practice:**
- Use ISO 8601 strings (human-readable)
- Always include timezone (Z for UTC)
- Be consistent across API

**Parsing:**
```javascript
// JavaScript
const date = new Date("2024-01-26T10:00:00Z");

// Python
from datetime import datetime
date = datetime.fromisoformat("2024-01-26T10:00:00+00:00")
```

### 5. When would you use CSV instead of JSON?

**Answer:**

**Use CSV when:**
1. **Tabular data**: Rows and columns
2. **Spreadsheet import**: Excel, Google Sheets
3. **Human editing**: Easy to edit in text editor
4. **Size matters**: Smaller than JSON for flat data

**Example - Analytics Export:**
```csv
date,users,revenue
2024-01-01,1000,5000
2024-01-02,1100,5500
2024-01-03,1050,5200
```

**Use JSON when:**
1. **Nested data**: Objects within objects
2. **APIs**: Standard for REST
3. **Data types**: Boolean, null, numbers

**Comparison:**
```csv
// CSV (flat only)
id,name,email
1,Alice,alice@example.com

// JSON (can nest)
{
  "id": 1,
  "name": "Alice",
  "contact": {
    "email": "alice@example.com",
    "phone": "123-456-7890"
  }
}
```

### 6. What are the trade-offs between human-readable and binary formats?

**Answer:**

**Human-Readable (JSON, XML, YAML):**

**Pros:**
- Easy to debug
- Readable in logs
- No special tools needed
- Testable in browser/curl

**Cons:**
- Larger size
- Slower parsing
- More bandwidth

**Binary (Protobuf, MessagePack, Avro):**

**Pros:**
- 50-70% smaller
- 5-10x faster
- Lower bandwidth

**Cons:**
- Not debuggable (need decoder)
- Requires schema
- Harder to test

**Decision matrix:**
```
Public API → Human-readable (JSON)
Internal API → Binary if performance matters (Protobuf)
Config files → Human-readable (YAML)
Big data → Binary (Avro)
```

### 7. How do you version data formats?

**Answer:**

**Protocol Buffers (Built-in versioning):**
```protobuf
message User {
  int32 id = 1;
  string name = 2;
  string email = 3;  // Added in v2
  
  reserved 4;  // Removed field
}

// Field numbers never change
// New fields = optional by default
// Backward/forward compatible
```

**JSON (Manual versioning):**
```json
// Option 1: Version field
{"version": "2.0", "data": {...}}

// Option 2: URL versioning
GET /v1/users  → {"name": "Alice"}
GET /v2/users  → {"name": "Alice", "email": "alice@example.com"}
```

**Best practices:**
1. Never remove required fields
2. Make new fields optional
3. Use default values
4. Document breaking changes

### 8. What is YAML commonly used for?

**Answer:**

**Primary use: Configuration files**

**Examples:**

**Docker Compose:**
```yaml
version: '3'
services:
  web:
    image: nginx
    ports:
      - "80:80"
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: secret
```

**Kubernetes:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: web
    image: nginx:latest
```

**CI/CD (GitHub Actions):**
```yaml
name: Deploy
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: npm install
```

**Why YAML:**
- Most human-readable
- Comments supported
- Less syntax noise than JSON
- Multi-line strings easy

**Not for:**
- APIs (use JSON)
- Performance-critical (use binary)

---

## Points to Impress the Interviewer

**Show practical experience:**
- "JSON for public APIs, Protobuf for internal gRPC microservices"
- "Use ISO 8601 for dates, always include timezone"
- "CSV for data exports but JSON for nested structures"
- "Protobuf is 50-70% smaller and 5-10x faster than JSON"

**Real-world insights:**
- "Google uses Protobuf internally for all microservices—it's the foundation of gRPC"
- "Kafka defaults to Avro for event schemas because it supports schema evolution"
- "AWS APIs use JSON but CloudFormation templates use YAML for readability"

**System design integration:**
```
"In a high-throughput trading system, I'd use:
- Protobuf for inter-service communication (speed)
- JSON for public REST API (compatibility)
- Avro for event streaming to Kafka (schema evolution)
- Redis with MessagePack for caching (fast serialization)"
```

---

## Quick Reference

**Format Selection:**
```
REST API → JSON
gRPC → Protocol Buffers
Config → YAML
Export → CSV
Big Data → Avro
Legacy → XML
Cache → MessagePack
```

**Size Comparison (1000 user objects):**
```
XML:         2.5 MB
JSON:        1.0 MB
MessagePack: 0.7 MB
Avro:        0.4 MB
Protobuf:    0.3 MB
```

**Speed Comparison (serialization):**
```
XML:         250 ms
JSON:        100 ms
MessagePack: 40 ms
Avro:        30 ms
Protobuf:    20 ms
```

---

> Data Formats guide complete. Covers all common formats with practical trade-offs and interview-ready answers.