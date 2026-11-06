# agents-control-tower

A team of [Claude Code](https://claude.com/claude-code) subagents that plan, build, and stress-test my software projects. Each agent has one clear job, its own set of tools, and rules about what it can and can't touch. Together they work like a small dev team: an architect designs it, two engineers build it, a QA engineer tries to break it, and three lightweight **supervisors** check each piece of work as it's finished so mistakes get caught early.

```
idea ──► 🏗️ architect ──► 👁️ plan-supervisor ──► plan.md
                                                    │
         ┌──────────────────────────────────────────┘
         ▼
   ⚙️ backend task ──► 👁️ backend-supervisor ──┐   (FIX → back to backend)
         │ PASS, next task                      │
         ▼                                      │
   🎨 frontend task ──► 👁️ frontend-supervisor ─┘   (FIX → back to frontend)
         │ all tasks PASS
         ▼
   🔍 reviewer ──► REVIEW.md ──► fix findings ──► reviewer again ──► 👤 your review
```

---

## The Agents

### 🏗️ Architect — `architect`

**Role:** System designer and planner. Runs first on any new project or major feature. Turns a rough idea into a structured, buildable design. **Writes no application code.**

| | |
|---|---|
| **Model** | `opus` (deep reasoning for design decisions) |
| **Tools** | `Read`, `Glob`, `Grep`, `Write`, `Edit`, `WebSearch`, `WebFetch` |
| **Output** | `plan.md` at the project root |
| **Triggers on** | "plan", "design", "architect", "structure", starting a new project |

**Abilities**
- Restates the project and lists its assumptions and open questions, so nothing is guessed silently
- Surveys an existing codebase before designing, and respects its conventions
- Picks an architecture pattern (layered, MVC, client–server, REST + SPA, component-based) and says why it fits
- Draws the system as a Mermaid diagram
- Lays out the complete folder and file tree, with a one-line purpose for every file
- Defines the data model (entities, fields, types, relationships, validation)
- Writes the **API contract**, a binding table of endpoints that the frontend and backend must both follow
- Breaks the work into ordered tasks with stable IDs (`B-1`, `F-1`, `T-1`) and their dependencies
- Lists the risks and edge cases the reviewer should attack
- Keeps a changelog in `plan.md` whenever the design changes

**Skills & principles it enforces**
- Separation of concerns: UI, business logic, and data access live in separate layers
- SOLID principles for object-oriented code
- High cohesion and low coupling
- DRY, without premature abstraction
- Naming conventions by language: `kebab-case.ts`, `PascalCase.tsx`, `snake_case.py`, one `PascalCase` class per file in Java/C#
- One file, one job
- Plans sized to the project: a class assignment doesn't get an enterprise spec

---

### 🎨 Frontend — `frontend`

**Role:** Builds everything the user sees, clicks, types into, or waits on.

| | |
|---|---|
| **Model** | `sonnet` (fast, strong at implementation) |
| **Tools** | `Read`, `Glob`, `Grep`, `Write`, `Edit`, `Bash` |
| **Output** | UI code; `F-*` tasks checked off in `plan.md` |
| **Triggers on** | UI, screens, pages, components, styling, forms, interactivity |

**Abilities**
- Builds pages, components, layouts, forms, and client-side state
- Calls the backend **exactly** as the API contract specifies, and never invents endpoints
- Flags missing API pieces as "contract requests" for the backend instead of working around them
- Runs lint, build, and type-check, and fixes what fails before handing off
- Points the reviewer at tricky states and edge-case inputs

**Skills & principles it enforces**
- Small, reusable, one-per-file components. Pages compose components, and API calls live in `services/`
- Every screen handles **loading, empty, error, and success** states
- Client-side validation for good UX, never treated as security
- Accessibility: semantic HTML, labels, alt text, keyboard navigation, visible focus, color contrast
- Responsive from phone width (~375px) up to desktop
- Theming through CSS variables, with no inline style soup
- No secrets or API keys in frontend code

---

### ⚙️ Backend — `backend`

**Role:** Builds everything the user never directly sees: servers, APIs, logic, data, and auth.

| | |
|---|---|
| **Model** | `sonnet` |
| **Tools** | `Read`, `Glob`, `Grep`, `Write`, `Edit`, `Bash` |
| **Output** | Server code, unit tests; `B-*` tasks checked off in `plan.md` |
| **Triggers on** | endpoints, databases, models, schemas, auth, server logic, jobs |

**Abilities**
- Implements the API contract and data model from `plan.md`
- When a contract change is unavoidable, updates `plan.md`, logs it, and tells the frontend
- Writes unit tests for services as it goes (the happy path plus at least one failure path)
- Runs the tests, linter, and type-check before handing off
- Documents how to run the project locally (env vars and setup commands)

**Skills & principles it enforces**
- Strict layering:
  - `routes/` maps URLs to controllers
  - `controllers/` handles requests and responses
  - `services/` holds business rules
  - `repositories/` handles data access
  - `models/` defines data shapes
- Validates all input at the boundary and treats every input as hostile
- Parameterized queries only, with no string-built SQL
- Consistent error shapes and correct HTTP status codes (400, 401/403, 404, 409, 500), with no leaked stack traces
- Secrets come from environment variables. `.env` is gitignored and `.env.example` is committed
- Passwords are hashed (bcrypt/argon2) and never logged

---

### 🔍 Reviewer — `reviewer`

**Role:** QA engineer and adversarial tester. Assumes the code is broken until there's evidence it isn't, then finds out how. **It doesn't fix app code.** It reports exactly what the owning agent needs to fix.

| | |
|---|---|
| **Model** | `opus` (thorough, adversarial reasoning) |
| **Tools** | `Read`, `Glob`, `Grep`, `Write`, `Edit`, `Bash` |
| **Output** | Test suites in `tests/`, plus `REVIEW.md` |
| **Triggers on** | "review", "test", "QA", "security check", "try to break it" |

**Abilities: Phase 1, static review**
- **Plan conformance:** right folders, correct naming, every requirement implemented, API matches the contract field for field
- **Design:** mixed layers, duplicated logic, functions doing too much
- **Correctness:** off-by-one errors, null handling, unhandled exceptions or promises, race conditions
- **Security:** SQL/command/XSS injection, hard-coded secrets, missing auth checks, sensitive data in logs
- **Readability:** unclear names, dead code, undocumented tricky logic

**Abilities: Phase 2, testing it nine ways**
1. **Unit tests:** each function in isolation
2. **Integration tests:** the full request → controller → service → repository flow
3. **Boundary values:** `0`, `-1`, max int, empty strings, 10,000-character strings, values exactly at the limit
4. **Hostile input:** wrong types, `null`, emoji, `'; DROP TABLE users;--`, `<script>` tags, path traversal
5. **State and sequence:** out-of-order actions, double submits, delete-then-update
6. **Concurrency:** the same request fired many times at once
7. **Auth and permissions:** missing or expired tokens, accessing another user's data
8. **Frontend behavior:** UI states, keyboard-only use, 375px width, API down or slow
9. **Load smoke test:** slow endpoints and memory growth

**Abilities: Phase 3, the report**
- Writes `REVIEW.md` with a verdict: 🔴 Not ready / 🟡 Ready with fixes / 🟢 Ready
- Includes a test results table by suite
- Ranks findings by severity: 🔴 Critical, 🟠 Major, 🟡 Minor, 💡 Suggestion. Each finding has a file:line, reproduction steps, a suggested fix, and an **owner** (backend, frontend, or architect)
- Lists what it tried that held up, so you know what's already verified
- Keeps a dated history of verdicts across review rounds

**Skills & principles it follows**
- Every finding needs evidence: a failing test, a command and its output, or a file and line
- Installs a test framework when the project has none: Jest/Vitest (JS/TS), pytest (Python), JUnit (Java), xUnit (C#)
- Never runs destructive commands outside the project, and never touches real databases or live third-party APIs
- Adversarial toward the code, never toward the person who wrote it

---

### 👁️ Supervisors: `plan-supervisor`, `backend-supervisor`, `frontend-supervisor`

**Role:** Lightweight checkpoint reviewers, one per phase of the build. Each one runs right after its agent finishes a piece of work, checks just that piece against `plan.md` and the agent's rules, and returns **PASS** or **FIX**. On FIX, the work goes straight back to the same builder before the next task starts. Small problems never pile up into big ones.

| | |
|---|---|
| **Model** | `haiku` (small, fast, and cheap enough to run after every task) |
| **Tools** | `plan-supervisor`: `Read`, `Glob`, `Grep` · the other two add `Bash` to run checks |
| **Output** | A short PASS / FIX verdict with up to 5 issues, each with a file and line |
| **Runs** | After the architect, and after every B-* and F-* task |

**What each one checks**

| Supervisor | Checks |
|---|---|
| `plan-supervisor` | Every plan.md section is present; every file has a purpose; naming conventions; layers are separated; the API contract is complete; every requirement maps to a task; open questions are surfaced |
| `backend-supervisor` | The task did what plan.md says; the API contract is followed exactly; files are in the right folders; layering (no SQL outside repositories); input validation; parameterized queries; no secrets; error handling; unit tests exist. Runs tests, lint, and type-check |
| `frontend-supervisor` | The task did what plan.md says; API calls match the contract; files are in the right folders; components don't fetch directly; loading/empty/error/success states; accessibility basics; CSS variables; no secrets. Runs build, lint, and type-check |

**Rules they follow**
- **Read-only.** They never edit code, commit, or push. They only report.
- **Scoped to the task.** They check only what the last task changed, which keeps them fast.
- **Short reports.** At most 5 issues, the most serious first. Style nitpicks alone don't cause a FIX.
- **Escalation.** If a task fails its check twice, the build stops and shows you the issues instead of looping.
- **Not a replacement.** A supervisor PASS doesn't replace the full `reviewer` at the end, or your own review.

> **How "supervising" works:** Claude Code subagents can't watch each other live while they run. The supervisors are checkpoints instead: the main session runs one the moment each task finishes. You get the same effect, since nothing moves forward until the last step passes.

---

## Skills

Every agent has its own set of skills in `.claude/skills/`. Each skill is reference material the agent follows: templates, checklists, and code patterns.

**Clear about which agent uses it**
- Every skill folder starts with its owner's name: `architect-*`, `frontend-*`, `backend-*`, `reviewer-*`, `supervisor-*`.
- Every description starts with a tag like `[backend only]`.

**No tokens wasted searching**
- Each agent **preloads** its skills through a `skills:` list in its frontmatter, so the full skill is in its context from the start.
- None of the agents have the `Skill` tool, so they can't go searching through other skills.
- The skills are marked `user-invocable: false`, which keeps them out of your `/` menu.

| Agent | Skill | What it gives the agent |
|---|---|---|
| 🏗️ architect | `architect-plan-template` | The exact plan.md structure and how to fill it in |
| | `architect-design-patterns` | Picking a pattern, plus ready folder structures for static sites, REST + SPA, Python, and OOP console apps |
| | `architect-naming-conventions` | File, folder, and identifier rules per language, and names to reject |
| 🎨 frontend | `frontend-ui-states` | Loading/empty/error/success patterns (vanilla JS and React) and form handling |
| | `frontend-accessibility` | Accessibility and responsive-layout checklist |
| | `frontend-api-client` | The `services/` layer: a shared request helper and one service per resource |
| ⚙️ backend | `backend-layered-api` | Route → controller → service → repository skeleton |
| | `backend-security` | Validation, parameterized queries, secrets, password hashing, ownership checks |
| | `backend-error-handling` | Error shape, status codes, central error handler |
| 🔍 reviewer | `reviewer-attack-inputs` | Boundary values, hostile strings, state/concurrency/auth attacks |
| | `reviewer-test-setup` | Test frameworks and commands for JS/TS, Python, Java, C#, C++, and browser UI |
| | `reviewer-report-template` | REVIEW.md format, severity levels, verdict rules |
| 👁️ all supervisors | `supervisor-checkpoint` | How to keep a check fast and scoped, decide PASS or FIX, and escalate |

**Adding a skill:** create `.claude/skills/<agent>-<topic>/SKILL.md`, start the description with `[<agent> only]`, and add the name to that agent's `skills:` list. Keep skills short, because every line is loaded each time the agent runs.

---

## Slash Commands

| Command | What it does |
|---|---|
| `/build <idea>` | Runs the full pipeline: architect → plan check → backend and frontend tasks, each checked by its supervisor → reviewer → fix loop (up to 3 rounds or until 🟢) → summary, then stops for your review |
| `/review [focus]` | Runs only the reviewer, with an optional focus area. Reports findings without fixing anything |

---

## Repo Layout

```
agents-control-tower/
├── .claude/
│   ├── agents/
│   │   ├── architect.md            # design + plan.md
│   │   ├── frontend.md             # user-facing code
│   │   ├── backend.md              # server, data, logic
│   │   ├── reviewer.md             # final review, test, break, report
│   │   ├── plan-supervisor.md      # quick check of plan.md
│   │   ├── backend-supervisor.md   # quick check after each B-* task
│   │   └── frontend-supervisor.md  # quick check after each F-* task
│   ├── commands/
│   │   ├── build.md        # /build pipeline
│   │   └── review.md       # /review
│   └── skills/             # one folder per skill, named <agent>-<topic>
│       ├── architect-plan-template/SKILL.md
│       ├── architect-design-patterns/SKILL.md
│       ├── architect-naming-conventions/SKILL.md
│       ├── frontend-ui-states/SKILL.md
│       ├── frontend-accessibility/SKILL.md
│       ├── frontend-api-client/SKILL.md
│       ├── backend-layered-api/SKILL.md
│       ├── backend-security/SKILL.md
│       ├── backend-error-handling/SKILL.md
│       ├── reviewer-attack-inputs/SKILL.md
│       ├── reviewer-test-setup/SKILL.md
│       ├── reviewer-report-template/SKILL.md
│       └── supervisor-checkpoint/SKILL.md
├── CLAUDE.md               # workflow + ownership rules all agents follow
└── README.md
```

## Setup

**Per project:** copy `.claude/` and `CLAUDE.md` into the project's root folder.

**Every project, automatically:** install the agents at the user level:
```bash
git clone https://github.com/rcluff11/agents-control-tower
mkdir -p ~/.claude/agents ~/.claude/commands
cp agents-control-tower/.claude/agents/*.md ~/.claude/agents/
cp agents-control-tower/.claude/commands/*.md ~/.claude/commands/
mkdir -p ~/.claude/skills && cp -r agents-control-tower/.claude/skills/* ~/.claude/skills/
```
Project-level agents override user-level ones that have the same name. To confirm they loaded, run `/agents` in Claude Code.

## Usage

```
/build a habit tracker with a REST API and a web UI
/review the login endpoints
```
Or call an agent by name: *"Use the architect agent to plan a gym log app,"* or *"Have the reviewer try to break the signup form."*

## Workflow Rules

- `plan.md` is the single source of truth. Every design change is logged there.
- Ownership is respected. The frontend doesn't touch backend files and vice versa, and the reviewer doesn't fix app code.
- Tiny one-file fixes skip the agents.
- No secrets are ever committed.
- **The user decides on every commit and push.** No agent commits, pushes, creates branches, opens pull requests, or changes anything on GitHub without the user's explicit approval for that specific action. Agents may prepare changes and suggest a commit message, but the user makes the call.
- **The user does a final review of the code.** The reviewer agent's `REVIEW.md` doesn't replace a human review. Before anything is committed, the user reads through the changes (the diff, `plan.md`, and `REVIEW.md`) and signs off.

## Tuning

Each agent's frontmatter sets its `model` and `tools`. Set `model: inherit` to use whatever model your session is running.
