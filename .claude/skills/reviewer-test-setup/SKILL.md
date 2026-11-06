---
name: reviewer-test-setup
description: "[reviewer only] How to find or install a test framework per language, where tests go, and the commands to run them."
user-invocable: false
---

# Test Setup by Language

First check what the project already uses (`package.json` scripts, `pyproject.toml`, `pom.xml`, `*.csproj`). Use that. Only install a framework if none exists.

| Language | Framework | Install | Run | Coverage |
|---|---|---|---|---|
| JS/TS (Vite) | Vitest | `npm i -D vitest` | `npx vitest run` | `npx vitest run --coverage` |
| JS/TS (other) | Jest (+ ts-jest) | `npm i -D jest ts-jest @types/jest` | `npx jest` | `npx jest --coverage` |
| Express APIs | Supertest | `npm i -D supertest @types/supertest` | with Jest/Vitest | — |
| Python | pytest | `pip install pytest pytest-cov` | `pytest -q` | `pytest --cov` |
| Java | JUnit 5 | add `junit-jupiter` to Maven/Gradle | `mvn test` / `gradle test` | JaCoCo |
| C# | xUnit | `dotnet new xunit -o Tests` | `dotnet test` | `dotnet test --collect:"XPlat Code Coverage"` |
| C++ | GoogleTest or the course's test harness | — | `ctest` / course command | — |
| Browser UI | Playwright | `npm i -D @playwright/test` | `npx playwright test` | — |

## Where tests go
- Mirror the source tree: `backend/src/services/user-service.ts` → `backend/tests/services/user-service.test.ts`.
- Name tests as behavior: `it('rejects a habit name longer than 100 chars')`.

## Integration test pattern (Express + Supertest)
```ts
import request from 'supertest';
import { app } from '../src/app';
it('POST /api/habits returns 400 for empty name', async () => {
  const res = await request(app).post('/api/habits').send({ name: '' });
  expect(res.status).toBe(400);
  expect(res.body.error.code).toBe('VALIDATION_ERROR');
});
```
Export `app` separately from the `listen()` call so tests don't open a port.

## Isolation
- Use a test database, SQLite in-memory, or mocked repositories. Never the real DB.
- Reset state between tests (`beforeEach`).
- Put temp files under `tests/tmp/` and delete them after.
