---
name: backend-security
description: "[backend only] Input validation, parameterized queries, secrets, password hashing, and auth checks."
user-invocable: false
---

# Backend Security Essentials

## 1. Validate at the boundary (in controllers)
Check type, presence, length, range, and format for every field. Reject unknown fields.
```ts
import { z } from 'zod';
const NewHabit = z.object({
  name: z.string().trim().min(1).max(100),
  targetPerWeek: z.number().int().min(1).max(7),
}).strict();
const input = NewHabit.parse(req.body);   // throws on bad input → 400
```
Python: Pydantic. Java: Bean Validation (`@NotNull`, `@Size`).

## 2. Parameterized queries only
```ts
// ✅
db.query('SELECT * FROM users WHERE email = ?', [email]);
// ❌ never
db.query(`SELECT * FROM users WHERE email = '${email}'`);
```

## 3. Secrets
- Read from `process.env` (or `os.environ`) in one `config/` file; fail fast if missing.
- `.env` in `.gitignore`. Commit `.env.example` with placeholder values.
- Never log tokens, passwords, or full request bodies with them.

## 4. Passwords
- Hash with bcrypt (cost ≥ 10) or argon2. Never store or return plain text.
- Compare with the library's compare function, not `===`.
- Same error message for "no such user" and "wrong password".

## 5. Auth & ownership
- Every protected route checks the user is logged in (middleware).
- Every query for a user's data filters by **their** user ID: `WHERE id = ? AND user_id = ?`. Never trust an ID from the request alone.

## 6. Other basics
- Limit request body size (`express.json({ limit: '100kb' })`).
- Set CORS to the frontend's origin, not `*`, when cookies/auth are used.
- Rate-limit login and signup.
