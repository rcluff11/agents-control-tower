---
name: reviewer-attack-inputs
description: "[reviewer only] Catalog of boundary values, hostile inputs, and break-it scenarios to throw at the app."
user-invocable: false
---

# Attack Input Catalog

Use these against every input, endpoint, and form. Only against the local project, never live systems.

## Boundary values
| Type | Try |
|---|---|
| Numbers | `0`, `-1`, `1`, limit−1, limit, limit+1, `2147483647`, `-2147483648`, `1.5` where int expected, `NaN`, `Infinity`, `1e308` |
| Strings | `""`, `" "`, one char, max length, max+1, 10,000 chars, leading/trailing spaces |
| Arrays | `[]`, one item, max items, max+1, duplicates, nested arrays |
| Dates | Feb 29, Dec 31 → Jan 1, timezone edges, far past (`1900-01-01`), far future, invalid (`2026-13-40`) |

## Wrong shape
`null`, `undefined`, missing field, extra unknown field, string where number expected (`"5"`), number where string expected, object where array expected, empty JSON `{}`, malformed JSON `{"a":`, wrong `Content-Type`.

## Hostile strings
```
'; DROP TABLE users;--
' OR '1'='1
<script>alert(1)</script>
<img src=x onerror=alert(1)>
../../etc/passwd
${7*7}  {{7*7}}
%00  \u0000
😀🔥 (emoji)   ́ (combining chars)   مرحبا (RTL text)
```

## State & sequence
- Submit the same form twice fast (double-click).
- Create → delete → update the deleted item.
- Use an ID that never existed; use another user's ID.
- Refresh / go back mid-flow.
- Do step 3 before step 1.

## Concurrency
Fire 20–50 identical requests at once:
```bash
for i in $(seq 1 30); do curl -s -X POST localhost:3000/api/items -H 'Content-Type: application/json' -d '{"name":"dup"}' & done; wait
```
Check for duplicate rows, wrong counts, or 500s.

## Auth
No token, malformed token, expired token, valid token for user A on user B's resource, logged-out session reused.

## Environment
Backend down, slow responses (add delay), offline browser, 375px screen, keyboard only.
