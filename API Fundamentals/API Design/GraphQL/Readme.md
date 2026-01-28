# GraphQL Design Principles

## What is this concept?

**GraphQL Design Principles**: Best practices for designing GraphQL schemas that are maintainable, performant, and intuitive for clients.

**Simple definition**: Rules for building GraphQL APIs that prevent common mistakes (N+1 queries, over-fetching, unclear naming).

**Think of it as**: Creating a well-organized buffet where customers can pick exactly what they want, but you design it to prevent kitchen chaos.

## Why does this exist? / Problem it solves

**Problem: Poorly Designed GraphQL**
```graphql
# Bad schema
query {
  getUserById(userId: 123) {
    getAllPosts {
      getAllComments {
        getAuthor { ... }
      }
    }
  }
}

Issues:
❌ Redundant naming (get*, getAll*)
❌ No pagination (returns everything)
❌ N+1 query problem
❌ Client can write expensive queries
❌ No error handling strategy
```

**Good GraphQL Design:**
```graphql
query {
  user(id: "123") {
    posts(first: 10, after: "cursor") {
      edges {
        node {
          title
          comments(first: 5) {
            edges {
              node {
                text
                author { name }
              }
            }
          }
        }
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
}

Benefits:
✓ Clean naming
✓ Built-in pagination
✓ Efficient data loading
✓ Query cost control
```

---

## Core Design Principles

### 1. Naming Conventions

**Use clear, consistent names without redundant prefixes:**

**Bad ✗**
```graphql
type Query {
  getUserById(userId: ID!): User
  getAllUsers: [User]
  getPostsByUserId(userId: ID!): [Post]
}
```

**Good ✓**
```graphql
type Query {
  user(id: ID!): User
  users(first: Int): UserConnection
  posts(userId: ID!, first: Int): PostConnection
}
```

**Rules:**
- No `get`, `fetch`, `retrieve` prefixes
- Use singular for single items: `user`, `post`
- Use plural for lists: `users`, `posts`
- Use `Connection` suffix for paginated lists

---

### 2. Schema Design - Types First

**Define clear, reusable types:**

```graphql
# User type
type User {
  id: ID!
  name: String!
  email: String!
  posts(first: Int, after: String): PostConnection
  friends(first: Int): UserConnection
}

# Post type
type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  comments(first: Int): CommentConnection
  createdAt: DateTime!
}

# Comment type
type Comment {
  id: ID!
  text: String!
  author: User!
  post: Post!
  createdAt: DateTime!
}
```

**Key points:**
- Use `!` for non-nullable fields
- Return types, not IDs when possible (prefer `author: User!` over `authorId: ID!`)
- Add relationships directly in types
- Include timestamps

---

### 3. Pagination (Connections Pattern)

**Always paginate lists - never return unbounded arrays:**

**Bad ✗**
```graphql
type Query {
  users: [User]  # Returns ALL users (dangerous)
}
```

**Good ✓ - Relay-style Cursor Pagination**
```graphql
type Query {
  users(first: Int, after: String): UserConnection!
}

type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int
}

type UserEdge {
  node: User!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

**Usage:**
```graphql
query {
  users(first: 10, after: "cursor123") {
    edges {
      node {
        id
        name
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

**Why Connections?**
- Consistent pagination pattern
- Cursor-based (stable with data changes)
- Includes metadata (totalCount, hasNextPage)

---

### 4. Input Types for Mutations

**Use input types for complex arguments:**

**Bad ✗**
```graphql
type Mutation {
  createUser(
    name: String!
    email: String!
    age: Int
    bio: String
  ): User
}
```

**Good ✓**
```graphql
type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload
}

input CreateUserInput {
  name: String!
  email: String!
  age: Int
  bio: String
}

type CreateUserPayload {
  user: User
  errors: [Error!]
}

type Error {
  field: String
  message: String!
}
```

**Benefits:**
- Easier to extend (add fields)
- Reusable across mutations
- Returns structured errors
- Clear success/failure states

---

### 5. Query Design

**Root Query Organization:**

```graphql
type Query {
  # Single resource by ID
  user(id: ID!): User
  post(id: ID!): Post
  
  # Lists with pagination
  users(first: Int, after: String): UserConnection!
  posts(
    first: Int
    after: String
    authorId: ID
    status: PostStatus
  ): PostConnection!
  
  # Search
  searchUsers(query: String!, first: Int): UserConnection!
  
  # Current user (authenticated)
  me: User
}
```

**Best practices:**
- Group by resource type
- Single items take `id`
- Lists take `first` and `after` for pagination
- Add filters as optional arguments
- Special `me` query for current user

---

### 6. Mutation Design

**Standard mutation pattern:**

```graphql
type Mutation {
  # Create
  createPost(input: CreatePostInput!): CreatePostPayload
  
  # Update
  updatePost(input: UpdatePostInput!): UpdatePostPayload
  
  # Delete
  deletePost(input: DeletePostInput!): DeletePostPayload
  
  # Actions
  publishPost(input: PublishPostInput!): PublishPostPayload
  likePost(input: LikePostInput!): LikePostPayload
}

input CreatePostInput {
  title: String!
  content: String!
}

type CreatePostPayload {
  post: Post
  errors: [Error!]
}

input UpdatePostInput {
  id: ID!
  title: String
  content: String
}

type UpdatePostPayload {
  post: Post
  errors: [Error!]
}
```

**Rules:**
- Use verb naming: `createPost`, `updatePost`, `deletePost`
- All inputs end with `Input`
- All outputs end with `Payload`
- Include `errors` field for validation failures
- Return updated resource in payload

---

### 7. Error Handling

**Three-level error strategy:**

**1. Mutation-level errors (validation):**
```graphql
type CreateUserPayload {
  user: User
  errors: [Error!]  # Field-level errors
}

type Error {
  field: String      # "email"
  message: String!   # "Invalid email format"
  code: String       # "INVALID_EMAIL"
}
```

**2. Field-level nullability:**
```graphql
type User {
  id: ID!          # Always present
  name: String!    # Always present
  email: String    # Nullable (might fail to load)
  bio: String      # Nullable (optional field)
}
```

**3. Top-level errors (system failures):**
```json
{
  "errors": [
    {
      "message": "Database connection failed",
      "extensions": {
        "code": "INTERNAL_SERVER_ERROR"
      }
    }
  ]
}
```

---

### 8. Solving N+1 Query Problem

**Problem:**
```graphql
query {
  posts {          # 1 query
    title
    author {       # N queries (one per post!)
      name
    }
  }
}

If 100 posts → 101 database queries!
```

**Solution 1: DataLoader (Batching)**
```javascript
const DataLoader = require('dataloader');

const userLoader = new DataLoader(async (userIds) => {
  // Batch load all users in single query
  const users = await db.users.findAll({
    where: { id: userIds }
  });
  return userIds.map(id => users.find(u => u.id === id));
});

// Resolver
const resolvers = {
  Post: {
    author: (post) => userLoader.load(post.authorId)
  }
};

// Result: 2 queries total (posts + batched users)
```

**Solution 2: Join in resolver**
```javascript
const resolvers = {
  Query: {
    posts: async () => {
      // Single query with JOIN
      return db.query(`
        SELECT posts.*, users.name as author_name
        FROM posts
        JOIN users ON posts.author_id = users.id
      `);
    }
  }
};
```

---

### 9. Query Complexity & Depth Limiting

**Prevent expensive queries:**

```graphql
# Malicious query
query {
  users {
    friends {
      friends {
        friends {
          friends {
            friends {  # Too deep!
              ...
            }
          }
        }
      }
    }
  }
}
```

**Solution 1: Depth Limiting**
```javascript
const depthLimit = require('graphql-depth-limit');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [depthLimit(5)]  // Max depth: 5
});
```

**Solution 2: Query Cost Analysis**
```javascript
const { createComplexityLimitRule } = require('graphql-validation-complexity');

const server = new ApolloServer({
  validationRules: [
    createComplexityLimitRule(1000, {
      scalarCost: 1,
      objectCost: 5,
      listFactor: 10
    })
  ]
});
```

**Solution 3: Pagination Limits**
```graphql
type Query {
  users(first: Int! @constraint(max: 100)): UserConnection
}
```

---

### 10. Field-Level Authorization

**Check permissions per field:**

```javascript
const resolvers = {
  User: {
    email: (user, args, context) => {
      // Only return email if viewing own profile or admin
      if (context.userId === user.id || context.isAdmin) {
        return user.email;
      }
      return null;  // Hide from others
    },
    
    privateNotes: (user, args, context) => {
      if (context.userId !== user.id) {
        throw new Error('Not authorized');
      }
      return user.privateNotes;
    }
  }
};
```

**Schema with directives:**
```graphql
type User {
  id: ID!
  name: String!
  email: String @auth(requires: OWNER)
  privateNotes: String @auth(requires: OWNER)
  adminField: String @auth(requires: ADMIN)
}
```

---

### 11. Versioning (Schema Evolution)

**GraphQL doesn't version URLs - evolve schema:**

**Add fields (non-breaking):**
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  phoneNumber: String  # New field added
}

# Old clients ignore new fields
# New clients can use them
```

**Deprecate fields (not remove):**
```graphql
type User {
  id: ID!
  name: String!
  username: String! @deprecated(reason: "Use 'name' instead")
}
```

**Breaking changes (last resort):**
```graphql
# Add new field, deprecate old
type Post {
  author: User!
  authorId: ID! @deprecated(reason: "Use 'author { id }' instead")
}
```

---

### 12. Caching Strategy

**Use @cacheControl directive:**
```graphql
type Query {
  user(id: ID!): User @cacheControl(maxAge: 60)
  posts: [Post] @cacheControl(maxAge: 30)
}

type User @cacheControl(maxAge: 300) {
  id: ID!
  name: String!
}
```

**Client-side caching (Apollo):**
```javascript
// Cache by ID
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        user: {
          read(existing, { args, toReference }) {
            return existing || toReference({
              __typename: 'User',
              id: args.id
            });
          }
        }
      }
    }
  }
});
```

---

### 13. Subscription Design

**Real-time updates:**

```graphql
type Subscription {
  postCreated(authorId: ID): Post!
  commentAdded(postId: ID!): Comment!
  userUpdated(userId: ID!): User!
}

# Usage
subscription {
  commentAdded(postId: "123") {
    id
    text
    author {
      name
    }
  }
}
```

**Implementation:**
```javascript
const { PubSub } = require('graphql-subscriptions');
const pubsub = new PubSub();

const resolvers = {
  Mutation: {
    createComment: async (_, { input }) => {
      const comment = await db.createComment(input);
      
      // Publish event
      pubsub.publish('COMMENT_ADDED', {
        commentAdded: comment,
        postId: input.postId
      });
      
      return { comment };
    }
  },
  
  Subscription: {
    commentAdded: {
      subscribe: (_, { postId }) => 
        pubsub.asyncIterator('COMMENT_ADDED')
    }
  }
};
```

---

## Common Anti-Patterns (Avoid)

### ❌ 1. REST-style Naming
```graphql
# Bad
type Query {
  getUserById(id: ID!): User
  getAllPosts: [Post]
}

# Good
type Query {
  user(id: ID!): User
  posts(first: Int): PostConnection
}
```

### ❌ 2. No Pagination
```graphql
# Bad
posts: [Post]  # Returns all posts!

# Good
posts(first: Int, after: String): PostConnection
```

### ❌ 3. Returning IDs Instead of Types
```graphql
# Bad
type Post {
  authorId: ID!  # Client must fetch author separately
}

# Good
type Post {
  author: User!  # Client gets author in one query
}
```

### ❌ 4. No Error Handling
```graphql
# Bad
type Mutation {
  createUser(input: CreateUserInput!): User
}

# Good
type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload
}

type CreateUserPayload {
  user: User
  errors: [Error!]
}
```

### ❌ 5. Ignoring N+1 Problem
```javascript
// Bad - N+1 queries
Post: {
  author: (post) => db.users.findById(post.authorId)
}

// Good - Use DataLoader
Post: {
  author: (post) => userLoader.load(post.authorId)
}
```

---

## Real-World Example: GitHub GraphQL API

**Well-designed schema:**
```graphql
query {
  repository(owner: "facebook", name: "react") {
    stargazers(first: 10) {
      totalCount
      edges {
        node {
          login
          avatarUrl
        }
        cursor
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
    issues(first: 5, states: OPEN) {
      edges {
        node {
          title
          author {
            login
          }
          comments(first: 3) {
            totalCount
          }
        }
      }
    }
  }
}
```

**Features:**
- Connections for pagination
- Nested queries in one request
- Clear naming
- Metadata (totalCount, pageInfo)

---

## Interview Explanation (60-second answer)

*"GraphQL design principles ensure schemas are maintainable and performant. Key principles include clean naming without redundant prefixes—use `user(id: ID!)` not `getUserById`—and always paginate lists using the Connection pattern with edges, nodes, and pageInfo.*

*For mutations, use Input types and Payload types that include both the result and errors for validation feedback. This makes APIs extensible and provides clear success/failure states.*

*The N+1 query problem is critical—when fetching posts and their authors, you need DataLoader to batch requests. Without it, 100 posts trigger 101 database queries instead of 2.*

*Implement query complexity limits to prevent malicious queries with deep nesting. Use depth limiting or cost analysis to protect your server.*

*Field-level authorization is important—return null or throw errors for unauthorized fields rather than filtering at the query level. This way different users see different data from the same query.*

*GraphQL doesn't version URLs like REST. Instead, evolve the schema by adding new fields and deprecating old ones with the @deprecated directive. This allows old and new clients to coexist.*

*Use connections for pagination, DataLoader for N+1 prevention, input/payload patterns for mutations, and always include error handling. These patterns make GraphQL APIs production-ready."*

---

## Top Interview Questions

### 1. How do you prevent N+1 queries in GraphQL?

**Answer:**

**Problem:**
```graphql
query {
  posts {          # 1 query: SELECT * FROM posts
    title
    author {       # N queries: SELECT * FROM users WHERE id = ?
      name         # (one per post!)
    }
  }
}

100 posts = 101 database queries!
```

**Solution: DataLoader**
```javascript
const DataLoader = require('dataloader');

// Create loader
const userLoader = new DataLoader(async (userIds) => {
  console.log('Batch loading users:', userIds);
  // Single query with IN clause
  const users = await db.query(
    'SELECT * FROM users WHERE id IN (?)',
    [userIds]
  );
  // Return in same order as requested
  return userIds.map(id => users.find(u => u.id === id));
});

// Use in resolver
const resolvers = {
  Post: {
    author: (post) => userLoader.load(post.authorId)
  }
};

// Result: 2 queries total
// 1. SELECT * FROM posts
// 2. SELECT * FROM users WHERE id IN (1,2,3,...)
```

**DataLoader features:**
- Batches requests within single tick
- Caches results per request
- Prevents duplicate fetches

### 2. What is the Connections pattern and why use it?

**Answer:**

**Connections pattern:** Standard for pagination in GraphQL

**Structure:**
```graphql
type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int
}

type UserEdge {
  node: User!        # Actual user data
  cursor: String!    # Position in list
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

**Usage:**
```graphql
query {
  users(first: 10, after: "cursor123") {
    edges {
      node { name }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
    totalCount
  }
}
```

**Why use it:**
1. **Cursor-based:** Stable pagination (new items don't shift results)
2. **Metadata:** Know if more pages exist
3. **Consistent:** Same pattern across all lists
4. **Relay-compatible:** Works with Relay framework

**vs Simple arrays:**
```graphql
# Bad - no pagination
users: [User]

# Bad - offset pagination (unstable)
users(limit: 10, offset: 20): [User]

# Good - connections
users(first: 10, after: "cursor"): UserConnection
```

### 3. How do you handle errors in GraphQL?

**Answer:**

**Three levels:**

**1. Field-level (nullable):**
```graphql
type User {
  id: ID!          # Never fails
  name: String!    # Never fails
  email: String    # Nullable - might fail to load
}
```

**2. Mutation-level (validation):**
```graphql
type CreateUserPayload {
  user: User
  errors: [Error!]
}

type Error {
  field: String    # "email"
  message: String! # "Invalid email format"
  code: String     # "INVALID_EMAIL"
}
```

**Client handling:**
```javascript
const result = await createUser({ input: { email: 'invalid' } });

if (result.data.createUser.errors) {
  // Show validation errors
  result.data.createUser.errors.forEach(err => {
    console.log(`${err.field}: ${err.message}`);
  });
} else {
  // Success
  const user = result.data.createUser.user;
}
```

**3. Top-level (system errors):**
```json
{
  "errors": [
    {
      "message": "Database connection failed",
      "extensions": {
        "code": "INTERNAL_SERVER_ERROR"
      }
    }
  ]
}
```

**Best practice:** Use all three levels appropriately

### 4. How do you version GraphQL APIs?

**Answer:**

**GraphQL doesn't version - it evolves:**

**Add fields (non-breaking):**
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  phoneNumber: String  # ✓ New field added
}

# Old queries still work (ignore new field)
# New queries can use it
```

**Deprecate fields:**
```graphql
type User {
  username: String! @deprecated(reason: "Use 'name' field")
  name: String!
}
```

**Breaking changes (avoid, but if needed):**
```graphql
# Step 1: Add new field
type Post {
  author: User!      # New
  authorId: ID!      # Old (keep temporarily)
}

# Step 2: Deprecate old
type Post {
  author: User!
  authorId: ID! @deprecated(reason: "Use 'author { id }'")
}

# Step 3: After migration, remove old field
```

**Why no /v1, /v2?**
- Single endpoint /graphql
- Clients request only what they need
- Old and new clients coexist
- Gradual migration, not breaking changes

### 5. How do you implement authorization in GraphQL?

**Answer:**

**Field-level authorization:**

```javascript
const resolvers = {
  User: {
    email: (user, args, context) => {
      // Show email only to owner or admin
      if (context.userId === user.id || context.isAdmin) {
        return user.email;
      }
      return null;  // Hide from others
    },
    
    salary: (user, args, context) => {
      // Only HR can see salaries
      if (!context.roles.includes('HR')) {
        throw new Error('Not authorized');
      }
      return user.salary;
    }
  },
  
  Query: {
    user: (parent, { id }, context) => {
      // Anyone can query users
      return db.users.findById(id);
    },
    
    adminStats: (parent, args, context) => {
      // Only admins can access
      if (!context.isAdmin) {
        throw new Error('Admin access required');
      }
      return db.getAdminStats();
    }
  }
};
```

**Schema directives:**
```graphql
directive @auth(requires: Role!) on FIELD_DEFINITION

enum Role {
  USER
  ADMIN
  HR
}

type User {
  id: ID!
  name: String!
  email: String @auth(requires: USER)
  salary: Float @auth(requires: HR)
}
```

**Context setup:**
```javascript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => {
    const token = req.headers.authorization;
    const user = verifyToken(token);
    return {
      userId: user.id,
      isAdmin: user.roles.includes('ADMIN'),
      roles: user.roles
    };
  }
});
```

### 6. What are Input and Payload types?

**Answer:**

**Pattern for mutations:**

**Input type:** Arguments for mutation
```graphql
input CreatePostInput {
  title: String!
  content: String!
  tags: [String!]
}
```

**Payload type:** Result including errors
```graphql
type CreatePostPayload {
  post: Post        # Successful result
  errors: [Error!]  # Validation errors
}

type Error {
  field: String
  message: String!
}
```

**Mutation:**
```graphql
type Mutation {
  createPost(input: CreatePostInput!): CreatePostPayload
}
```

**Why use this pattern:**

**1. Extensibility:**
```graphql
# Easy to add fields without breaking clients
input CreatePostInput {
  title: String!
  content: String!
  tags: [String!]
  featuredImage: String  # New field added
}
```

**2. Clear error handling:**
```javascript
// Client code
const result = await createPost({
  input: { title: '', content: 'test' }
});

if (result.data.createPost.errors) {
  // Handle validation errors
  console.log(result.data.createPost.errors);
} else {
  // Success
  const post = result.data.createPost.post;
}
```

**3. Reusability:**
```graphql
input UpdatePostInput {
  id: ID!
  title: String
  content: String
}

# Same error type reused
type UpdatePostPayload {
  post: Post
  errors: [Error!]  # Same Error type
}
```

### 7. How do you limit query complexity?

**Answer:**

**Problem: Expensive queries**
```graphql
query {
  users {
    friends {
      friends {
        friends {
          friends {  # Very deep!
            posts {
              comments {  # Very expensive!
                ...
              }
            }
          }
        }
      }
    }
  }
}
```

**Solution 1: Depth limiting**
```javascript
const depthLimit = require('graphql-depth-limit');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [depthLimit(5)]  // Max 5 levels deep
});
```

**Solution 2: Query cost analysis**
```javascript
const { createComplexityLimitRule } = require('graphql-validation-complexity');

const server = new ApolloServer({
  validationRules: [
    createComplexityLimitRule(1000, {
      scalarCost: 1,        # Each field costs 1
      objectCost: 5,        # Objects cost 5
      listFactor: 10        # Lists multiply cost by 10
    })
  ]
});

// Example costs:
// user { name } = 6 (object + field)
// users(first: 100) { name } = 600 (100 * 6)
```

**Solution 3: Pagination limits**
```graphql
type Query {
  users(first: Int! @constraint(max: 100)): UserConnection
}

# Reject if first > 100
```

**Solution 4: Timeout**
```javascript
const server = new ApolloServer({
  context: () => ({
    timeout: 5000  // 5 second timeout
  })
});
```

### 8. What is the difference between nullable and non-nullable fields?

**Answer:**

**Non-nullable (`!`):** Field always returns value
```graphql
type User {
  id: ID!        # Always present
  name: String!  # Always present
}

# If resolver returns null → entire query fails
```

**Nullable (no `!`):** Field can return null
```graphql
type User {
  id: ID!
  email: String   # Can be null
  bio: String     # Can be null
}

# If resolver returns null → just this field is null
```

**When to use:**

**Non-nullable for:**
- IDs (always exist)
- Required business fields
- Fields that should never fail

**Nullable for:**
- Optional fields (bio, middle name)
- Fields that might fail to load
- Computed fields that can error

**Error propagation:**
```graphql
type User {
  id: ID!
  name: String!   # Non-nullable
  email: String   # Nullable
}

query {
  user(id: "123") {
    id
    name    # If this fails → entire 'user' becomes null
    email   # If this fails → just 'email' is null
  }
}
```

**Best practice:**
```graphql
# Use non-null for guaranteed fields
type User {
  id: ID!
  name: String!
  createdAt: DateTime!
}

# Use nullable for optional or risky fields
type User {
  bio: String           # Optional
  externalApiData: JSON # Might fail
}
```

---

## Points to Impress the Interviewer

**Show best practices:**
- "Always use DataLoader to prevent N+1 queries—it batches requests within a single tick"
- "Connections pattern with edges and pageInfo is industry standard for pagination"
- "Use Input/Payload types for mutations to handle validation errors gracefully"

**Real-world insights:**
- "GitHub's GraphQL API is excellent—they use connections everywhere and deprecate fields instead of versioning"
- "Shopify uses query cost analysis to prevent expensive queries from impacting performance"
- "Apollo Client automatically normalizes and caches data by ID, making subsequent queries instant"

**System design:**
```
"For a social media GraphQL API, I'd design:
- Connections for all lists (posts, comments, followers)
- DataLoader for batching user/post fetches
- Query depth limit of 7 to prevent deep nesting
- Cost analysis with limit of 1000 points
- Field-level auth (email visible to friends only)
- Redis caching with @cacheControl directive
- Subscriptions for real-time notifications"
```

---

## Quick Reference

**Naming:**
```graphql
✓ user(id: ID!)
✗ getUserById(id: ID!)

✓ posts(first: Int): PostConnection
✗ getAllPosts: [Post]
```

**Pagination:**
```graphql
type Query {
  users(first: Int, after: String): UserConnection
}
```

**Mutations:**
```graphql
type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload
}

type CreateUserPayload {
  user: User
  errors: [Error!]
}
```

**N+1 Solution:**
```javascript
const userLoader = new DataLoader(fetchUsers);
author: (post) => userLoader.load(post.authorId)
```

**Authorization:**
```javascript
email: (user, args, context) => {
  if (context.userId !== user.id) return null;
  return user.email;
}
```

---

> GraphQL Design Principles guide complete. Covers schema design, pagination, N+1 prevention, error handling, and production patterns.