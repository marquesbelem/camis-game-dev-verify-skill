---
name: verify
description: >-
  Central technology-agnostic verification orchestrator for AI-generated projects.
  Scans the codebase, enforces clean code and architecture standards, determines
  which engineering pillars apply, and runs the applicable verification skills
  in strict order: integration tests, never trust the client, race conditions,
  idempotency, and separation of concerns.
---

# Verify — Central Engineering & Security Audit

Use this skill to audit, harden, and structure any project (backend, frontend,
fullstack, API, game backend, CLI, or worker) regardless of language or framework.

Communicate with the user in **Portuguese** (matching the project's audit language),
while keeping all **code identifiers in English** and **code comments in Portuguese**
(only where non-obvious, explaining *why* rather than *what*).

---

## Core Code & Architecture Standards (Padrão)

Apply these baseline standards across every phase:

1. **Simplicity First**: Keep what works; avoid unnecessary technologies, abstractions, or dependencies (KISS / YAGNI).
2. **Language Convention**: Code (files, classes, functions, variables, tests) in **English**; selective comments in **Portuguese** only when explaining non-obvious intent or invariants.
3. **Single Responsibility**: Each class, module, and function has one clear responsibility.
4. **Clean Code**:
   - Small, focused functions with shallow nesting and early returns.
   - **DRY** (Don't Repeat Yourself) — extract shared helpers instead of duplicating logic.
   - **KISS** (Keep It Simple, Stupid).
   - Clear, explicit error handling (never swallow errors silently).
   - High cohesion and low coupling.

---

## Step 1: Project Discovery & Stack Profiling

Scan the repository root and key config/source files to build a concise profile:

- **Stack**: Language(s), framework(s), package manager, database/ORM, test runner.
- **Architecture**: Monolith, frontend + backend, serverless/edge functions, workers/queues, CLI, or library.
- **Capabilities present**:
  - HTTP/RPC routes or API handlers?
  - Client/frontend code + env vars?
  - Database / persistent state / Row-Level Security (RLS) policies?
  - Concurrent/shared mutable state (balances, stock, quotas, bookings, counters)?
  - Webhooks, payment charges, or retryable async jobs?
  - Large or mixed-responsibility files ("God files")?

---

## Step 2: Applicability Triage (Central Decision Matrix)

Not every engineering pillar applies to every project. Evaluate each of the 5 pillars
and classify it as **APLICÁVEL** or **NÃO APLICÁVEL** (with a 1-line justification):

| Order | Pillar | Skill | When it applies |
| :--- | :--- | :--- | :--- |
| 1 | **Teste de Integração** | `verify-integration-tests` | Any project with API routes, handlers, DB flows, CLI commands, or critical user journeys. |
| 2 | **Nunca Confie no Cliente** | `verify-never-trust-client` | Any project with client/frontend + backend, auth, user-owned data, RLS policies, or secret keys. |
| 3 | **Race Condition** | `verify-race-condition` | Any project with shared mutable state, read-then-write DB flows, balances, inventory, quotas, or concurrent requests. |
| 4 | **Idempotência** | `verify-idempotency` | Any project receiving webhooks, charging payments, or executing retryable mutating operations. |
| 5 | **Separação de Responsabilidade** | `verify-separation-of-concerns` | Any project with mixed concerns (UI + business rules + DB queries in the same file) or low cohesion. |

Present the triage table and the detected **Padrão** issues to the user before executing
changes, especially because Step 1 (`verify-integration-tests`) requires user approval
of behavior sentences before writing tests.

---

## Step 3: Strict Execution Order

Execute **only** the pillars marked **APLICÁVEL**, strictly in this order:

1. `verify-integration-tests` — Establish the safety net first (behavior sentences -> user OK -> route + test-DB tests -> 1 command).
2. `verify-never-trust-client` — Move business rules to backend, recalculate server-side, audit leaked keys, verify row ownership & explain DB policies in Portuguese, output risk report.
3. `verify-race-condition` — Find read-then-write patterns, explain 2-request failure mode, fix at DB level (unique constraint -> atomic op -> row lock), add 2-concurrent-request tests.
4. `verify-idempotency` — Verify webhook signatures, enforce unique event ID table, add idempotency keys on charges, test duplicate sequential + concurrent events.
5. `verify-separation-of-concerns` — Map files into clean layers, verify existing behavior tests, refactor in small steps keeping the app running, run tests after every step.

> **Mandatory Rule**: Whenever any AI edit is made during any phase, run the test suite
> immediately afterward. Never delete, skip, or weaken test assertions to make tests pass.

---

## Step 4: Final Verification & Summary Report

After completing all applicable pillars:

1. Run the single unified test command and confirm 100% pass rate with zero skipped/weakened tests.
2. Deliver a concise Portuguese summary containing:
   - Pillars executed vs. skipped (and why).
   - Risk findings and fixes applied.
   - Secret keys that must be rotated (if any were exposed).
   - The **1-command** to run all tests.
