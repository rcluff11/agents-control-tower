---
name: backend-layered-api
description: "[backend only] Route → controller → service → repository skeleton showing what goes in each layer."
user-invocable: false
---

# Layered API Skeleton

Each layer has one job and only talks to the layer below it.

| Layer | Does | Never |
|---|---|---|
| `routes/` | Map method + path → controller | Logic, validation, DB |
| `controllers/` | Parse/validate request, call service, send response | SQL, business rules |
| `services/` | Business rules | `req`/`res`, SQL |
| `repositories/` | All DB/file access | HTTP, business rules |
| `models/` | Types, schemas | Side effects |

## Example (TypeScript + Express)
```ts
// routes/habit-routes.ts
const router = Router();
router.get('/', habitController.list);
router.post('/', habitController.create);
export default router;

// controllers/habit-controller.ts
export const habitController = {
  async create(req: Request, res: Response, next: NextFunction) {
    try {
      const input = parseNewHabit(req.body);          // throws ValidationError on bad input
      const habit = await habitService.create(req.user.id, input);
      res.status(201).json(habit);
    } catch (err) { next(err); }
  },
  // list(...) similar
};

// services/habit-service.ts
export const habitService = {
  async create(userId: string, input: NewHabit) {
    if (await habitRepository.existsByName(userId, input.name)) {
      throw new ConflictError('A habit with that name already exists.');
    }
    return habitRepository.insert(userId, input);
  },
};

// repositories/habit-repository.ts
export const habitRepository = {
  insert: (userId: string, h: NewHabit) =>
    db.query('INSERT INTO habits (user_id, name) VALUES ($1, $2) RETURNING *', [userId, h.name])
      .then(r => r.rows[0]),
  existsByName: (userId: string, name: string) =>
    db.query('SELECT 1 FROM habits WHERE user_id = $1 AND name = $2', [userId, name])
      .then(r => r.rowCount > 0),
};
```
MySQL uses `?` placeholders instead of `$1`. Python/Java: same layers, same rules.

## Unit test each service
Mock the repository; test the happy path and at least one failure (e.g. the ConflictError above).
