---
description: Run the full agent pipeline — plan, build, review, fix — for a project or feature.
argument-hint: <what to build>
---

Build the following using the agent team described in CLAUDE.md:

$ARGUMENTS

1. Use the `architect` agent to write or update `plan.md`, then the `plan-supervisor` agent to check it. On FIX, send the issues back to `architect` and check again. If there are open questions that change the design, ask me before continuing.
2. Use the `backend` agent for the B-* tasks, then the `frontend` agent for the F-* tasks. After each task, run that phase's supervisor (`backend-supervisor` or `frontend-supervisor`) with the task ID:
   - FIX → send the issues back to the same builder and run the supervisor again.
   - After 2 failed rounds on the same task, stop and show me the issues.
   - PASS → move on to the next task.
3. Use the `reviewer` agent to review and test everything and write `REVIEW.md`.
4. Send each 🔴 Critical and 🟠 Major finding to its owner agent to fix (with a supervisor check after each fix), then run `reviewer` again. Stop after 3 rounds or when the verdict is 🟢.
5. Give me a short summary: what was built, the files changed, the final verdict, and anything left open.
6. Stop there for my review. Don't commit, push, or touch GitHub. I'll review the code and tell you if and when to commit or push.
