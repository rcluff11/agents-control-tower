---
name: architect-design-patterns
description: "[architect only] How to pick an architecture pattern, with ready folder structures for common project types."
user-invocable: false
---

# Choosing an Architecture

Pick the **simplest** pattern that meets the requirements. Write one paragraph in plan.md explaining why.

| Project looks like… | Use | Why |
|---|---|---|
| Static site, portfolio, single page with no saved data | **Component-based static site** | No server needed |
| Browser app that saves data locally (localStorage/IndexedDB) | **MVC in the browser** | Keeps UI, state, and storage separate without a server |
| Web app with accounts or shared data | **REST API + SPA, layered backend** | Clear contract between frontend and backend |
| Console app / game (Java, C#, C++, Python) | **Layered OOP** (UI → game logic → data/models) | Logic is testable without the console |
| Script or data processing tool | **Pipeline** (input → transform → output modules) | Each stage tested on its own |

## Ready structures

**Static / browser-only (HTML/CSS/JS)**
```
src/
├── index.html
├── styles/        # variables.css, base.css, one file per component
├── scripts/
│   ├── models/    # data shapes + validation
│   ├── storage/   # localStorage/IndexedDB access only
│   ├── views/     # DOM rendering only
│   └── main.js    # wires models, storage, views together
└── assets/
```

**REST API + SPA (TypeScript / Express + React or vanilla)**
```
backend/src/
├── routes/  controllers/  services/  repositories/  models/  middleware/
├── config/        # reads env vars
└── server.ts
backend/tests/
frontend/src/
├── components/  pages/  services/  styles/
└── main.tsx
```

**Python (Flask/FastAPI)**
```
app/
├── routes/  services/  repositories/  models/
├── config.py
└── main.py
tests/
```

**OOP console app (Java/C#)**
```
src/
├── ui/            # input/output only
├── game/ or core/ # rules and logic
├── models/        # entities (Player, Item…)
├── data/          # save/load
└── Main.java
test/
```

## Decision checks
- Does anything need to be shared between users or devices? → you need a backend.
- Could this be one file? If it's under ~150 lines total, say so; don't force layers.
- Is the user's class requiring a specific structure or language? That wins over any default here.
