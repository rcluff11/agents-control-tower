---
name: architect-naming-conventions
description: "[architect only] File, folder, and identifier naming rules per language, plus names to reject."
user-invocable: false
---

# Naming Conventions

| Language | Files | Classes/Types | Functions/vars | Constants |
|---|---|---|---|---|
| JavaScript / TypeScript | `kebab-case.ts` | `PascalCase` | `camelCase` | `UPPER_SNAKE` |
| React components | `PascalCase.tsx` | `PascalCase` | `camelCase` | `UPPER_SNAKE` |
| CSS | `kebab-case.css` | — | classes `kebab-case` | vars `--kebab-case` |
| Python | `snake_case.py` | `PascalCase` | `snake_case` | `UPPER_SNAKE` |
| Java | `PascalCase.java` (one public class) | `PascalCase` | `camelCase` | `UPPER_SNAKE` |
| C# | `PascalCase.cs` | `PascalCase` | methods `PascalCase`, locals `camelCase` | `PascalCase` |
| C++ | match the course style guide if one exists; else `snake_case.cpp/.h` | `PascalCase` | `camelCase` | `UPPER_SNAKE` |
| SQL | — | tables `snake_case` (plural or singular, pick one) | columns `snake_case` | — |

## Folders
- Lowercase, plural for collections: `components/`, `services/`, `models/`.
- Named by **what's inside**, never by who or when: no `ryans-stuff/`, `old/`, `v2/`.

## Files
- Name says the one job: `user-service.ts`, `LoginForm.tsx`, `save_game.py`.
- Suffix by layer when it helps: `-service`, `-repository`, `-controller`, `-routes`.
- Tests mirror the file: `user-service.test.ts`, `test_save_game.py`, `PlayerTest.java`.

## Reject on sight
`utils2`, `helpers-new`, `final`, `misc/`, `stuff/`, `temp`, `test1`, `data.js` (data about what?), `manager` with no noun, single letters except loop counters.

**Course rules win:** if a class style guide (e.g. a BYU-I style guide) says otherwise, follow it and note that in plan.md.
