---
name: verify-idempotency
description: >-
  Fourth pillar of the verify pipeline. Verifies webhook cryptographic
  signatures, enforces a unique event-ID table for incoming events, requires
  idempotency keys on outbound charges/payments, and tests duplicate events
  both sequentially (2x) and concurrently (2x at the same time).
---

# Verify — Idempotência (Idempotency)

This is **Step 4** of the verification pipeline. Webhooks, retries, and network
timeouts will deliver the same event or charge request more than once. The system
must process each logical event/charge **at most once**.

---

## Step-by-Step Procedure

### 1. Verify Webhook Signatures
For every incoming webhook endpoint (Stripe, Mercado Pago, Clerk, GitHub, custom, etc.):
- Verify the cryptographic signature using the provider's SDK or HMAC with **raw request body** and **constant-time comparison** before parsing or acting on the payload.
- Reject missing or invalid signatures with `400`/`401` immediately.
- Never trust unverified webhook JSON payloads.

### 2. Enforce a Unique Event Table (`event_id` as Primary/Unique Key)
- Create/verify a dedicated table (e.g., `processed_events` / `webhook_events`) with `event_id` (or `(provider, event_id)`) as `PRIMARY KEY` or `UNIQUE` constraint.
- Insert the event record **atomically** (or inside the same DB transaction as the business update):
  - If `INSERT ... ON CONFLICT DO NOTHING` inserts 0 rows (or hits unique violation), return `200 OK` immediately without re-executing side effects.
  - Ensure failed processing does not silently mark an event as succeeded if it needs retry, or uses explicit status tracking cleanly.

### 3. Idempotency Key on Every Outbound Charge / Mutating Call
- Audit all payment charges, refunds, payouts, and critical external mutations:
  - Generate and persist a deterministic or stored `idempotency_key` (e.g., linked to `order_id` + attempt) **before** calling the payment provider.
  - Pass that `idempotency_key` to the provider API on every charge request so network retries never double-charge the customer.

### 4. Test Duplicate Events: Sequential (2x) & Concurrent (2x Simultaneous)
Write integration tests (English code, selective Portuguese comments) for every webhook and charge flow:
1. **Sequential duplicate (`2x`)**: Send the exact same valid event/request twice in a row. Assert no duplicate credits, orders, or charges are created.
2. **Concurrent duplicate (`2x` at the same time)**: Fire the exact same valid event/request **twice simultaneously** (`Promise.all`, `asyncio.gather`, etc.). Assert the DB unique constraint prevents double execution and the final state is identical to processing it once.
3. **Invalid signature**: Send a tampered payload/signature and assert rejection (`400`/`401`) with zero DB changes.

Run the full test suite immediately after edits.
