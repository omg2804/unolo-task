## 1. Employee Dashboard Crashed with SQLite Syntax Error

**Location:**
- `backend/routes/dashboard.js`

**Bug:**
The employee dashboard API crashed with a SQLite syntax error.

**Cause:**
The backend was using MySQL-specific SQL functions such as:
- `NOW()`
- `DATE_SUB(NOW(), INTERVAL 7 DAY)`

These functions are not supported by SQLite.

**Fix:**
Replaced MySQL-specific date functions with SQLite-compatible syntax:

- `NOW()` → `datetime('now')`
- `DATE_SUB(NOW(), INTERVAL 7 DAY)` → `datetime('now', '-7 days')`

**Why This Fix Is Correct:**
SQLite uses a different date/time function system. Using SQLite-compatible functions ensures queries execute correctly and the dashboard loads without crashing.

---

## 2. Check-in Insert Failed Due to Schema Mismatch

**Location:**
- `backend/routes/checkin.js`

**Bug:**
Check-in API failed with errors like:
- "no such column: lat"
- "6 values for 7 columns"

**Cause:**
- Backend was using `lat` / `lng` column names, while the database schema uses `latitude` / `longitude`.
- The schema itself was written in MySQL style, while the runtime database is SQLite.
- The INSERT query was not updated after adding `distance_from_client`.

**Fix:**
- Updated queries to use correct column names: `latitude`, `longitude`.
- Updated INSERT query to include `distance_from_client`.
- Fixed SQL syntax to be SQLite-compatible.

**Why This Fix Is Correct:**
Now the SQL matches the actual database schema and all required fields are inserted correctly without runtime errors.

---

## 3. History Page Crashed Due to Calling `.reduce()` on `null`

**Location:**
- `frontend/src/pages/History.jsx`

**Bug:**
History page crashed with: Cannot read properties of null (reading 'reduce')


**Cause:**
The `checkins` state was initialized as `null`, and `.reduce()` was called on it before data was loaded.

**Fix:**
- Initialized `checkins` as an empty array: `useState([])`
- Added a safe fallback: `(checkins || []).reduce(...)`

**Why This Fix Is Correct:**
React components now render safely before data loads and do not crash due to null state.

---

## 4. Check-in API Returned HTTP 200 for Invalid Request

**Location:**
- `backend/routes/checkin.js`

**Bug:**
When `client_id` was missing, the API returned HTTP 200 instead of an error code.

**Cause:**
Incorrect status code was used for invalid input.

**Fix:**
Changed response code from:
```js
res.status(200)

to:
res.status(400)

**Why This Fix Is Correct:**
HTTP 400 correctly represents a client-side validation error.

---

## 5. Dashboard Did Not Update Correctly and Showed Stale Data

**Location:**
- `frontend/src/pages/Dashboard.jsx`

**Bug:**
Dashboard sometimes showed stale or incorrect data after login or user change.

**Cause:**

- useEffect had an empty dependency array while using user.
- Role detection was based on user.id === 1 instead of user.role.

**Fix:**
- Added user to useEffect dependency array.
- Reset loading state before refetching.
- Replaced role check with: user.role === 'manager'.

**Why This Fix Is Correct:**
The dashboard now refreshes correctly when the user changes and no longer relies on hardcoded IDs.