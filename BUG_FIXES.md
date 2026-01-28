Bug: Employee dashboard crashes with SQLite syntax error
Cause: MySQL-specific functions (DATE_SUB, NOW()) used in SQLite database
Fix: Replaced with SQLite-compatible datetime('now', '-7 days')


Bug: Check-in insert failed due to schema mismatch
Cause: Backend used lat/lng but DB schema uses latitude/longitude and schema itself was MySQL-style in SQLite
Fix: Updated queries to use correct column names and SQLite-compatible syntax


Bug: History page crashes due to calling .reduce() on null
Cause: checkins state initialized as null
Fix: Initialize state as empty array [] and add safe reduce

Fixed check-in API returning 200 for missing client_id → now 400

Bug: Dashboard does not update correctly when user changes and sometimes shows stale data
Cause: useEffect had empty dependency array while using user, and role detection was based on id === 1
Fix: Added user as dependency, reset loading state before refetch, and used user.role instead of hardcoded ID