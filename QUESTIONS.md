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

