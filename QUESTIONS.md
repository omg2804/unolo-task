# QUESTIONS.md

## Technical Questions

---

### 1. If this app had 10,000 employees checking in simultaneously, what would break first? How would you fix it?

The first bottlenecks would be the backend API server and the database.

Currently, the application uses:
- A single Node.js server instance
- SQLite as the database (single-file, limited concurrency)

With 10,000 concurrent check-ins:
- SQLite would become a major bottleneck because it allows limited concurrent writes.
- The Node.js process would also become CPU and I/O bound due to synchronous database operations.
- The server could start timing out or rejecting requests.

To fix this:
- Migrate from SQLite to a production-grade database like PostgreSQL or MySQL.
- Add proper database indexing on frequently queried columns like `employee_id`, `checkin_time`, and `client_id`.
- Run multiple backend instances behind a load balancer.
- Add a queue system (e.g., Redis + BullMQ) to buffer peak write loads.
- Add caching (Redis) for frequently accessed dashboard data.

This would make the system horizontally scalable and able to handle high concurrency.

---

### 2. The current JWT implementation has a security issue. What is it and how would you improve it?

The original implementation was embedding the user's password hash inside the JWT payload. This is a security issue because:
- JWTs are stored on the client side (e.g., in localStorage).
- Anyone can decode a JWT and see its payload.
- Even exposing password hashes increases attack surface.

Improvements:
- Never include sensitive information (password, hash, secrets) in JWT payload.
- Only store minimal identity data in JWT, such as: `userId`, `role`.
- Use short-lived access tokens and optionally implement refresh tokens.
- Store JWT in httpOnly cookies instead of localStorage to reduce XSS risk.
- Add token revocation or versioning for better security control.

---

### 3. How would you implement offline check-in support? (Employee has no internet, checks in, syncs later)

I would implement offline support using a local storage or IndexedDB-based queue.

Approach:
- When the user tries to check in and the network is unavailable:
  - Save the check-in data (client_id, latitude, longitude, time, notes) in IndexedDB or localStorage.
- Show the user a message like: "Check-in saved offline. Will sync when online."
- Add a background sync mechanism:
  - Periodically check network connectivity.
  - When internet is available, send all pending check-ins to the server.
- On the backend:
  - Accept client-provided timestamps and mark these check-ins as "synced later".
  - Handle duplicates using a unique client-generated ID per check-in.

This ensures:
- No data is lost
- User experience is not blocked by network issues
- Data eventually becomes consistent with the server

---
## Theory / Research Questions

---

### 4. Explain the difference between SQL and NoSQL databases. For this Field Force Tracker application, which would you recommend and why?

SQL databases (like PostgreSQL, MySQL) are relational databases that store data in structured tables with fixed schemas and support powerful joins and transactions. They are best suited for applications with well-defined relationships and complex queries.

NoSQL databases (like MongoDB, DynamoDB) are schema-less or flexible-schema databases that store data in formats like documents, key-value pairs, or wide columns. They are good for highly scalable, distributed systems with rapidly changing data structures.

For the Field Force Tracker application, I would recommend a SQL database (e.g., PostgreSQL) because:
- The data is highly relational (users, clients, check-ins, assignments).
- The application requires joins, aggregations, and reporting queries.
- Strong consistency and transactional integrity are important for check-in/check-out records.
- Query patterns are structured and predictable.

SQLite is suitable for development, but in production, PostgreSQL or MySQL would be a better choice.

---

### 5. What is the difference between authentication and authorization? Identify where each is implemented in this codebase.

Authentication is the process of verifying who the user is (e.g., login with email and password).  
Authorization is the process of checking what the user is allowed to do (e.g., manager vs employee access).

In this codebase:
- Authentication is implemented in:
  - `POST /api/auth/login` (verifies credentials and issues JWT)
  - `GET /api/auth/me` (verifies JWT token)
  - `authenticateToken` middleware (checks if user is logged in)

- Authorization is implemented in:
  - `requireManager` middleware (restricts manager-only routes like `/dashboard/stats`)
  - Role checks in routes such as dashboard APIs to ensure only managers can access team data

So:
- Authentication = "Who are you?"
- Authorization = "What are you allowed to do?"

---

### 6. Explain what a race condition is. Can you identify any potential race conditions in this codebase? How would you prevent them?

A race condition occurs when the correctness of a system depends on the timing or order of execution of concurrent operations.

In this codebase, a potential race condition exists in the check-in flow:
- Two check-in requests from the same user could arrive at the same time.
- Both could pass the "no active check-in" check.
- Both could insert a new check-in, resulting in multiple active check-ins.

How to prevent this:
- Use a database transaction with proper isolation.
- Add a database-level constraint (e.g., unique partial index or status constraint).
- Lock the employee row or check-ins row during check-in creation.
- Or enforce logic in the database using a conditional insert.

In production systems, critical business rules like "only one active check-in per employee" should always be enforced at the database level, not only in application logic.

---


