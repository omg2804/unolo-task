Bug: Employee dashboard crashes with SQLite syntax error
Cause: MySQL-specific functions (DATE_SUB, NOW()) used in SQLite database
Fix: Replaced with SQLite-compatible datetime('now', '-7 days')


Bug: Check-in insert failed due to schema mismatch
Cause: Backend used lat/lng but DB schema uses latitude/longitude and schema itself was MySQL-style in SQLite
Fix: Updated queries to use correct column names and SQLite-compatible syntax