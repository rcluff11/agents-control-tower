---
name: architect-plan-template
description: "[architect only] The exact plan.md template the architect fills in for every project."
user-invocable: false
---

# plan.md Template

Copy this structure into `plan.md` at the project root. Fill in every section. If a section doesn't apply (no backend, for example), keep the heading and write "N/A — <reason>".

````markdown
# <Project Name> — Plan

## 1. Overview
What it is, who uses it, the core problem it solves. 2–4 sentences.

## 2. Assumptions & Open Questions
- Assumption: ...
- Open question (needs user input): ...

## 3. Requirements
### Functional
- [ ] FR-1: ...
### Non-functional
- [ ] NFR-1: Performance / security / accessibility / browser support ...

## 4. Architecture
- Pattern: <name> — why it fits this project
- Diagram:
  ```mermaid
  flowchart LR
    UI --> API --> Service --> Repository --> DB
  ```
- Tech stack: one line per choice, with the reason.

## 5. Folder & File Structure
```
project-root/
├── <folder>/
│   └── <file>     # one-line purpose
└── plan.md
```
Every file gets a one-line purpose.

## 6. Data Model
| Entity | Field | Type | Rules |
Relationships: ...

## 7. API Contract
| Method | Path | Request body | Success response | Errors |
This table is binding for frontend and backend.

## 8. Task Breakdown
### Backend (owner: backend)
- [ ] B-1: ... (depends on: —) (covers: FR-1)
### Frontend (owner: frontend)
- [ ] F-1: ... (depends on: B-1) (covers: FR-1)
### Review & Testing (owner: reviewer)
- [ ] T-1: ...

## 9. Risks & Edge Cases
Things likely to break; inputs the reviewer should attack.

## 10. Definition of Done
- All FR-* tasks checked off
- Supervisors PASS on every task
- REVIEW.md verdict 🟢
- User has reviewed and signed off

## Changelog
- <date>: Initial plan
````

## Filling it in well

- **Every FR maps to a task.** Use `(covers: FR-n)` on each task so the supervisor can check coverage.
- **Each task is small:** one agent can finish it and a supervisor can check it in one pass. Split anything that touches more than ~5 files.
- **Order by dependency:** data models → repositories → services → routes → UI services → components → pages.
- **Size to the project.** A class assignment might have 5 tasks; that's fine.
