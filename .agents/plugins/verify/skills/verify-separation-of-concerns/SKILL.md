---
name: verify-separation-of-concerns
description: >-
  Fifth pillar of the verify pipeline. Maps mixed or bloated files into clean,
  single-responsibility layers, ensures current behavior is covered by tests
  first, refactors in small incremental steps keeping the app working at every
  step, and runs tests after each step.
---

# Verify — Separação de Responsabilidade (Separation of Concerns)

This is **Step 5** of the verification pipeline. Clean up mixed responsibilities
("God files", routes containing raw SQL + business rules + HTTP formatting, UI
components mixing rendering + fetching + domain logic) without breaking behavior.

---

## Step-by-Step Procedure

### 1. Map the Files ("Mapeia o arquivo e põe cada pedaço na gaveta")
Scan the codebase for files with multiple responsibilities or excessive length/coupling:
- Identify each responsibility inside the file and assign it to its proper layer/drawer:
  - **Transport / HTTP / UI**: Request parsing, validation, response formatting, UI rendering.
  - **Application / Services / Use Cases**: Business rules, orchestration, domain decisions.
  - **Data / Repositories**: Database queries, transactions, external API clients.
  - **Domain / Models / Utils**: Pure types, entities, pure helper functions.
- Keep it simple (KISS): do **not** invent unnecessary enterprise abstractions, frameworks, or extra layers that the project doesn't need.

### 2. Ensure Current Behavior Has Tests First
- Before moving or splitting any code, verify that existing integration/characterization tests cover the current behavior of the target module.
- If any behavior is uncovered, add the missing test first (following `verify-integration-tests`: behavior sentence -> user OK -> test).

### 3. Plan in Small Steps (App Always Standing)
Break the refactoring into small, safe, incremental steps so the application compiles, starts, and passes tests after **every single step**:
- Step A: Extract pure helper / domain functions.
- Step B: Extract repository / data access functions.
- Step C: Extract service / business logic.
- Step D: Keep route/controller/UI thin, delegating to the service.

### 4. Enforce Clean Code & Language Standards
During every step:
- **English** for all file names, classes, functions, variables, and types.
- **Portuguese** only for selective, high-value comments explaining *why* (not *what*).
- Small functions, shallow nesting, early returns, DRY, clear error handling, high cohesion, low coupling.

### 5. Run Tests After Every Step
- **Mandatory**: Execute the single test command after **each** incremental refactoring step.
- Never proceed to the next step with failing tests, and never delete, skip, or relax assertions.
