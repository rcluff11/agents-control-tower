---
name: backend-error-handling
description: "[backend only] Consistent error shape, HTTP status codes, and a central error handler."
user-invocable: false
---

# Error Handling

## One error shape for every failure
Unless plan.md's API Contract says otherwise:
```json
{ "error": { "code": "VALIDATION_ERROR", "message": "Name is required.", "details": [] } }
```

## Status codes
| Code | When | `code` value |
|---|---|---|
| 400 | Bad or missing input | `VALIDATION_ERROR` |
| 401 | Not logged in | `UNAUTHORIZED` |
| 403 | Logged in, not allowed | `FORBIDDEN` |
| 404 | Resource doesn't exist (or isn't theirs) | `NOT_FOUND` |
| 409 | Duplicate / conflicting state | `CONFLICT` |
| 500 | Anything unexpected | `INTERNAL_ERROR` |
Success: 200 read/update, 201 created, 204 deleted (no body).

## Error classes + one handler
```ts
// models/errors.ts
export class AppError extends Error {
  constructor(public status: number, public code: string, message: string, public details: unknown[] = []) { super(message); }
}
export class NotFoundError extends AppError { constructor(m = 'Not found.') { super(404, 'NOT_FOUND', m); } }
export class ConflictError extends AppError { constructor(m: string) { super(409, 'CONFLICT', m); } }

// middleware/error-handler.ts — register LAST
export function errorHandler(err: unknown, req: Request, res: Response, _next: NextFunction) {
  if (err instanceof ZodError) {
    return res.status(400).json({ error: { code: 'VALIDATION_ERROR', message: 'Invalid input.', details: err.issues } });
  }
  if (err instanceof AppError) {
    return res.status(err.status).json({ error: { code: err.code, message: err.message, details: err.details } });
  }
  console.error(err);                       // full detail in server logs only
  res.status(500).json({ error: { code: 'INTERNAL_ERROR', message: 'Something went wrong.' } });
}
```

## Rules
- Services throw `AppError` subclasses; controllers pass errors to `next(err)`.
- Never send stack traces, SQL, or file paths to the client.
- Every `async` path ends in a `try/catch` or a wrapper that forwards to the handler; no unhandled promise rejections.
