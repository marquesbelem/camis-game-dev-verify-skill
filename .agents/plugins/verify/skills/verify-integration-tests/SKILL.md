---
name: verify-integration-tests
description: >-
  First pillar of the verify pipeline. Identifies critical flows, writes behavior
  sentences in Portuguese for user approval before coding, builds real route +
  test-database integration tests (mocking only external third-party services),
  and provides a single command to run the entire suite.
---

# Verify — Teste de Integração (Integration Tests)

This is **Step 1** of the verification pipeline. Establish a reliable safety net
before modifying security, concurrency, or architecture.

---

## Non-Negotiable Rules

1. **A frase vem antes do teste**: Never write test code before defining the behavior sentence.
2. **Espere o OK do usuário**: Present the list of behavior sentences for all critical flows and **wait for explicit user approval** before writing or generating test files.
3. **Bata na rota real + banco de teste**:
   - Hit the actual HTTP route / RPC handler / CLI entrypoint.
   - Use a real test database (local test DB, isolated schema, ephemeral container, or SQLite test DB matching production engine as closely as possible).
   - **Mock ONLY external third-party boundaries** (payment gateways, email/SMS providers, external webhooks, third-party APIs). Never mock your own database, services, or controllers.
4. **IA mexeu? Roda os testes**: After any AI code change, immediately run the test suite.
5. **1 comando para rodar tudo**: Configure or expose a single command (e.g., `npm test`, `pytest`, `go test ./...`, `dotnet test`, `cargo test`) that runs the full integration suite cleanly.
6. **Proibido sabotar testes**: It is strictly forbidden to delete, skip (`skip`, `xit`, `xdescribe`, `@pytest.mark.skip`), or weaken/relax assertions to make a failing test pass. Fix the code, not the test (unless the requirement itself changed and the user confirmed).

---

## Step-by-Step Procedure

### 1. List Critical Flows
Scan the codebase and list all critical user/system journeys, such as:
- Cadastro / Login / Autenticação (Signup, login, session/token validation)
- Fluxos principais de negócio (Core CRUD, game state transitions, bookings, orders)
- Checkout / Pagamentos (Order creation, price calculation, payment initiation)
- Webhooks / Callbacks externos (Payment confirmation, async event ingestion)

### 2. Write Behavior Sentences & Request Approval
For each critical flow, write clear behavior sentences in **Portuguese** following the pattern:
- *"Quando [contexto/ação], o sistema deve [resultado esperado no banco/resposta] e não deve [efeito colateral indesejado]."*

Present the grouped list to the user and **stop to wait for their OK** before writing the test code.

### 3. Implement Integration Tests (English Code, Portuguese Selective Comments)
Once approved:
- Keep test names, helpers, fixtures, and assertions in **English**.
- Add brief **Portuguese comments** only when explaining non-obvious test setup or invariants.
- Ensure test isolation (clean/reset test DB state between tests or wrap in rolled-back transactions where appropriate, except when testing concurrency/commits).
- Mock only external third-party dependencies at the boundary.

### 4. Configure Single Run Command & Verify
- Ensure `package.json` / `pyproject.toml` / `Makefile` / project runner has **one clear command** to execute all tests.
- Run that single command and verify all tests pass without skips or relaxed assertions.
