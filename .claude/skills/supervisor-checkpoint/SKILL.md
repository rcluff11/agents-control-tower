---
name: supervisor-checkpoint
description: "[supervisors only: plan-supervisor, backend-supervisor, frontend-supervisor] How to run a fast, scoped checkpoint check and decide PASS or FIX."
user-invocable: false
---

# Running a Checkpoint

You're a fast checker, not a deep reviewer. Aim to finish in a few tool calls.

## Stay scoped
1. Read only the task's entry in `plan.md` plus the sections it depends on (API Contract, Folder Structure).
2. Look only at files this task changed: `git diff --stat` then `git diff -- <file>`. If there's no git, use the file list the builder reported.
3. Don't read the whole codebase. Don't re-review earlier tasks that already passed.

## PASS or FIX
| FIX if… | Still PASS if… |
|---|---|
| The task didn't do what plan.md says | Small wording or style nits |
| API contract mismatch (path, field, status, error shape) | A better approach exists but this one works |
| Wrong folder or broken layering | A minor naming nit (mention it in one line) |
| Security problem (string-built SQL, secrets, missing validation) | Missing comments on obvious code |
| A required check fails (tests, build, lint, type-check) | |
| Missing UI state or accessibility blocker (frontend) | |
| plan.md missing a section or task coverage (plan) | |

When unsure, ask: *would the next task or the final reviewer have to undo this?* If yes → FIX.

## Writing issues
- At most 5, the most serious first.
- Each: `file:line` — what's wrong — the exact fix. One or two lines.
- Write for the builder agent: it should fix the issue without asking a follow-up question.

## Second round
If you're re-checking after a FIX, check **only** the previous issues plus anything the fix touched. If the same issue is still there after the second round, say `ESCALATE TO USER` with the issue list.

## Never
Edit files, commit, push, install packages, or run long/load tests.
