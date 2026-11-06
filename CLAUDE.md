# Project Workflow — Agent Team

This repo uses seven subagents in `.claude/agents/`: four main agents and three lightweight supervisors. Route work to them instead of doing it all in the main conversation.

| Agent | Owns | Produces |
|---|---|---|
| `architect` | Design, architecture, folder/file structure | `plan.md` |
| `frontend` | Everything the user sees or touches | UI code, F-* tasks |
| `backend` | Everything the user never sees | APIs, logic, data, B-* tasks |
| `reviewer` | Final deep review, testing, trying to break things | Tests in `tests/`, `REVIEW.md` |
| `plan-supervisor` | Quick check of `plan.md` | PASS / FIX |
| `backend-supervisor` | Quick check after each B-* task | PASS / FIX |
| `frontend-supervisor` | Quick check after each F-* task | PASS / FIX |

## Standard flow

1. **New project or big feature → `architect`.** It writes `plan.md` with requirements, structure, API contract, and tasks.
   - Then run **`plan-supervisor`**. On FIX, send the issues back to `architect` and check again.
   - Surface any open questions to the user before building.
2. **Build → `backend` first, then `frontend`** (frontend depends on the API contract). Independent tasks can run in parallel. **After every task:**
   - Run that phase's supervisor (`backend-supervisor` for B-*, `frontend-supervisor` for F-*), giving it the task ID.
   - On **FIX**, send the issues back to the same builder and run the supervisor again. After 2 failed rounds on the same task, stop and show the user the issues.
   - On **PASS**, move to the next task.
3. **Verify → `reviewer`.** It does the full deep review and writes `REVIEW.md`.
4. **Fix loop:** send each 🔴/🟠 finding to its owner agent (with a supervisor check after each fix), then run `reviewer` again. Repeat until the verdict is 🟢 or the user says stop.

Each agent preloads its own skills from `.claude/skills/` (named `<agent>-<topic>`). Don't load those skills in the main session; they belong to the agents.

Supervisors only check and report. They never edit code, and a supervisor PASS doesn't replace the `reviewer` or the user's own review.

## Rules for every agent

- `plan.md` is the source of truth. Changes to the design, API contract, or data model go into plan.md with a Changelog line.
- Respect ownership: frontend doesn't touch backend files and vice versa; the reviewer doesn't fix app code.
- Tiny one-file fixes can be done directly without spinning up an agent.
- Never commit secrets. `.env` stays in `.gitignore`; commit `.env.example` instead.
- **The user decides every commit and push.** No agent runs `git commit`, `git push`, creates branches, opens pull requests, or changes anything on GitHub without the user's explicit approval for that specific action. Agents may stage changes and suggest a commit message, then stop and ask.
- **The user does the final code review.** `REVIEW.md` doesn't replace a human review. When work is done, present a summary of the changes (files touched, `plan.md` updates, `REVIEW.md` verdict) and wait for the user to review and sign off before anything is committed.
