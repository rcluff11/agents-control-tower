---
name: frontend-accessibility
description: "[frontend only] Accessibility and responsive-layout checklist to apply while building UI."
user-invocable: false
---

# Accessibility & Responsive Checklist

## Structure
- Use real elements: `<button>` for actions, `<a href>` for navigation, `<nav>`, `<main>`, `<header>`, `<label>`. No clickable `<div>`.
- One `<h1>` per page; headings go in order (h1 → h2 → h3).
- `<html lang="en">` and a meaningful `<title>`.

## Forms
- Every input has a `<label for>` (placeholder is not a label).
- Errors use `aria-describedby` and `role="alert"` or `aria-live="polite"`.
- Required fields marked with `required`, not just a red star.

## Keyboard
- Everything works with Tab, Shift+Tab, Enter, Space, Escape.
- Visible focus style: never `outline: none` without a replacement.
- Modals trap focus while open and return it when closed.

## Visuals
- Text contrast at least 4.5:1 (3:1 for large text).
- Don't use color alone to show meaning; add an icon or text.
- Images: meaningful `alt`, or `alt=""` if decorative.
- Respect `prefers-reduced-motion` for animations.

## Responsive
- Mobile-first CSS; check at 375px, 768px, 1280px.
- No horizontal scroll at 375px.
- Touch targets at least 44×44px.
- Use `rem` for font sizes, CSS variables for colors and spacing:
```css
:root { --color-bg: #fff; --color-text: #111; --space-2: 0.5rem; --space-4: 1rem; }
```
