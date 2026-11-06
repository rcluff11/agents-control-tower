---
name: frontend-api-client
description: "[frontend only] How to structure the services/ layer that calls the backend API, with error handling."
user-invocable: false
---

# Frontend API Client

All backend calls live in `services/`. Components and pages call services, never `fetch` directly.

## One shared request helper
```ts
// services/api-client.ts
const BASE_URL = import.meta.env?.VITE_API_URL ?? '/api';

export class ApiError extends Error {
  constructor(public status: number, public code: string, message: string) { super(message); }
}

export async function request<T>(path: string, options: RequestInit = {}): Promise<T> {
  let res: Response;
  try {
    res = await fetch(`${BASE_URL}${path}`, {
      headers: { 'Content-Type': 'application/json', ...options.headers },
      ...options,
    });
  } catch {
    throw new ApiError(0, 'NETWORK', "Can't reach the server. Check your connection.");
  }
  if (res.status === 204) return undefined as T;
  const body = await res.json().catch(() => null);
  if (!res.ok) {
    throw new ApiError(res.status, body?.error?.code ?? 'UNKNOWN', body?.error?.message ?? 'Something went wrong.');
  }
  return body as T;
}
```
Match the error shape (`body.error.code`, `body.error.message`) to the **API Contract in plan.md**. If the contract uses a different shape, follow the contract.

## One service per resource
```ts
// services/habit-service.ts
import { request } from './api-client';
import type { Habit, NewHabit } from '../types/habit';

export const habitService = {
  getAll: () => request<Habit[]>('/habits'),
  create: (data: NewHabit) => request<Habit>('/habits', { method: 'POST', body: JSON.stringify(data) }),
  remove: (id: string) => request<void>(`/habits/${encodeURIComponent(id)}`, { method: 'DELETE' }),
};
```

## Rules
- Paths, fields, and types copy the contract **exactly**. Missing endpoint → note a "contract request"; don't invent one.
- `encodeURIComponent` on every ID or value put into a URL.
- No API keys or secrets in frontend code; the backend holds them.
- Plain JS projects: same structure, drop the types.
