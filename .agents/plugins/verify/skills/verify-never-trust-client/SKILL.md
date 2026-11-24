---
name: verify-never-trust-client
description: >-
  Second pillar of the verify pipeline. Audits and fixes client-side trust
  issues: moves business rules to the backend, recalculates totals/permissions
  server-side, detects exposed secrets in public env vars or bundles, enforces
  row-level ownership, explains every database policy in Portuguese, and
  produces a risk-ranked report.
---

# Verify — Nunca Confie no Cliente (Never Trust the Client)

This is **Step 2** of the verification pipeline. Frontend validation is only UX
("educação com o usuário"); the server is the sole source of truth.

---

## Step-by-Step Audit & Remediation

### 1. Recalculate Everything on the Backend
- Scan API routes, controllers, and handlers for any trusted client-sent values that affect state, money, permissions, or rules:
  - Prices, discounts, totals, quantities, currencies, tax/shipping.
  - User IDs, roles, `isAdmin`, ownership flags, or status transitions sent in request bodies/headers.
  - Game scores, cooldowns, rewards, or timestamps computed on the client.
- **Fix**: Ignore client-computed totals/roles; derive user identity from verified server session/token and recalculate all prices, rules, and state transitions on the server.

### 2. Move Business Rules from Frontend to Backend
- Search frontend/client code for domain rules (eligibility checks, state machines, pricing logic, access gates, quota checks) that are **not** enforced on the server.
- **Fix**: Keep lightweight UX hints on the client if helpful, but move/duplicate the authoritative rule enforcement into the backend.

### 3. Audit Exposed Secrets & Public Env Vars
- Scan the codebase and `.env*` / config files for secrets that must **never** reach the client:
  - `SUPABASE_SERVICE_ROLE_KEY`, `service_role`, database connection strings, JWT signing secrets, webhook secrets, private API keys, paid third-party API keys/tokens.
  - Check public-prefixed variables (`NEXT_PUBLIC_*`, `VITE_*`, `REACT_APP_*`, `NUXT_PUBLIC_*`, `PUBLIC_*`, `EXPO_PUBLIC_*`).
  - **Remember**: `.env` does NOT hide public-prefixed variables — they are embedded into the client bundle and visible to anyone.
- **Fix**: Move secret usage exclusively to server-side modules/endpoints. List **every exposed key found** and explicitly instruct the user which keys must be **rotated/revoked immediately**.

### 4. Row-Level Ownership ("Cada usuário, só com as linhas dele")
- Check every query, mutation, and endpoint for IDOR (Insecure Direct Object Reference):
  - Does `GET /items/:id`, `PATCH /items/:id`, `DELETE /items/:id` filter by `user_id = authenticated_user.id` (or equivalent tenant/owner check)?
- Check Database Policies (e.g., Postgres/Supabase RLS, Firestore rules, etc.):
  - Ensure RLS is enabled on user-owned tables and policies restrict `SELECT`, `INSERT`, `UPDATE`, `DELETE` to the authenticated owner (unless explicitly public read).
- **Explain every DB policy in Portuguese**: For each policy found or created, write a clear explanation in Portuguese stating who can read/insert/update/delete and under what condition.

### 5. Risk-Ranked Report & Test Verification
Deliver a structured report in Portuguese classifying every finding by risk:
- **Crítico** (ex: `service_role` no front, IDOR sem auth, preço vindo do cliente)
- **Alto** (ex: regra de negócio só no front, tabela sem RLS)
- **Médio** / **Baixo**

After applying fixes, **run the integration test suite** (and add negative authorization/tampering tests if missing).
