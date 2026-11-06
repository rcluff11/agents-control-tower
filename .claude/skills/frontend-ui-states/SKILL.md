---
name: frontend-ui-states
description: "[frontend only] Patterns for loading, empty, error, and success states on every screen, plus form handling."
user-invocable: false
---

# UI States

Every screen that shows data has **four states**. Build all four, every time.

| State | Show | Never |
|---|---|---|
| Loading | Skeleton or spinner; disable buttons that would resubmit | A blank page |
| Empty | A friendly message + the next action ("No habits yet — add one") | An empty table with headers only |
| Error | What went wrong in plain words + a Retry button | Raw error text, stack traces, `[object Object]` |
| Success | The data | — |

## Vanilla JS pattern
```js
async function renderList(container) {
  container.innerHTML = '<p class="loading" aria-live="polite">Loading…</p>';
  try {
    const items = await itemService.getAll();
    if (items.length === 0) {
      container.innerHTML = '<p class="empty">Nothing here yet.</p>';
      return;
    }
    container.replaceChildren(...items.map(renderItem));
  } catch (err) {
    container.innerHTML = `<p class="error" role="alert">Couldn't load items.</p>`;
    container.append(retryButton(() => renderList(container)));
  }
}
```
Use `textContent` (not `innerHTML`) for anything that came from the user or the API.

## React pattern
```tsx
const [state, setState] = useState<{status: 'loading'|'error'|'ready'; data: Item[]}>({status: 'loading', data: []});
// render: status === 'loading' → <Spinner/>, 'error' → <ErrorMessage onRetry/>, data.length === 0 → <Empty/>, else list
```

## Forms
- Disable the submit button while submitting (prevents double-submit).
- Validate on blur and on submit; show the error next to the field, linked with `aria-describedby`.
- Keep the user's input on error. Never clear the form when the server rejects it.
- Show success clearly (message, redirect, or the new item appearing).
