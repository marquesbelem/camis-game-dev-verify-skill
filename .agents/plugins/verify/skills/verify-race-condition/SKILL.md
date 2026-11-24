---
name: verify-race-condition
description: >-
  Third pillar of the verify pipeline. Finds read-then-write (TOCTOU) patterns,
  explains what happens when two concurrent requests arrive together, fixes
  them at the database level in priority order (unique constraint, atomic
  operation, row lock), and writes concurrent 2-request integration tests.
---

# Verify — Race Condition (Concorrência)

This is **Step 3** of the verification pipeline. Prevent double-spending,
overselling, duplicate registrations, and corrupted counters under concurrent requests.

---

## Step-by-Step Procedure

### 1. Find Every Read-Then-Write Pattern
Scan all backend handlers, services, and jobs for **Read-Modify-Write** / **TOCTOU** (Time-of-Check to Time-of-Use) patterns:
- `SELECT` balance/stock/quota/status -> check condition in app code -> `UPDATE` / `INSERT`.
- Check-if-exists (`SELECT` by email/slug/key) -> `INSERT` without unique constraint handling.
- Non-atomic counters (`count = row.count + 1; save(count)`).
- Double redemption of coupons, rewards, seats, or game actions.

### 2. Explain the 2-Request Interleaving Scenario
For every vulnerable spot found, explain clearly in **Portuguese** what happens when **2 requisições simultâneas** arrive at the exact same time:
- Request A reads `stock = 1` -> passes check.
- Request B reads `stock = 1` -> passes check.
- Request A writes `stock = 0` and succeeds.
- Request B writes `stock = 0` (or `-1`) and ALSO succeeds -> **oversell / double spend**.

### 3. Fix at the Database Level (Strict Preference Order)
Always prefer database-enforced guarantees over in-memory/app locks:

1. **1st Choice — Unique Constraint (`UNIQUE`)**:
   - Use `UNIQUE` constraints/indexes for one-per-user/one-per-key invariants (e.g., `UNIQUE(user_id, coupon_id)`, `UNIQUE(email)`), and handle the unique-violation error cleanly.
2. **2nd Choice — Atomic Operation (`UPDATE ... WHERE` / `ON CONFLICT`)**:
   - Perform check + update in a single atomic SQL/DB statement:
     - `UPDATE items SET stock = stock - 1 WHERE id = $1 AND stock >= 1 RETURNING *`
     - `INSERT ... ON CONFLICT (...) DO NOTHING`
   - Verify affected row count (`0` rows = insufficient stock / already exists).
3. **3rd Choice — Row-Level Lock (`SELECT ... FOR UPDATE`)**:
   - When multiple tables or complex multi-step calculations must be serialized, wrap in a database transaction and lock the target row(s) with `SELECT ... FOR UPDATE` (keeping transactions as short as possible and ordering locks consistently to avoid deadlocks).

### 4. Write Concurrent 2-Request Integration Tests
For each fixed flow:
- Write an integration test (English code, selective Portuguese comments) that fires **2 concurrent requests simultaneously** (e.g., `Promise.all([req1, req2])`, `asyncio.gather(...)`, concurrent threads/goroutines).
- Assert that:
  - Exactly 1 succeeds and 1 fails cleanly (or the final balance/stock/counter reflects both atomically without corruption).
- Run the full test suite immediately after edits.
