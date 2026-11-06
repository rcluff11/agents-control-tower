---
name: reviewer-report-template
description: "[reviewer only] The exact REVIEW.md template, severity definitions, and how to write a finding."
user-invocable: false
---

# REVIEW.md Template

Write `REVIEW.md` at the project root. Overwrite the previous one, but keep its History lines.

```markdown
# Review — <YYYY-MM-DD>

## Verdict
🔴 Not ready / 🟡 Ready with fixes / 🟢 Ready

## Test Results
| Suite | Passed | Failed | Notes |
|---|---|---|---|
| Unit | | | |
| Integration | | | |
| Boundary & hostile input | | | |
| State / concurrency | | | |
| Auth | | | |
| Frontend behavior | | | |
| Load smoke | | | |

## Findings
### 🔴 Critical
- **[C-1] <Title>** — `path/to/file.ts:42`
  - What happens:
  - How to reproduce:
  - Suggested fix:
  - Owner: backend | frontend | architect

### 🟠 Major
### 🟡 Minor
### 💡 Suggestions

## What I tried that held up

## History
- <date>: <verdict>, X critical / Y major
```

## Severity
| Level | Means |
|---|---|
| 🔴 Critical | Crash, data loss/corruption, security hole, a core FR doesn't work |
| 🟠 Major | Wrong behavior a normal user will hit; contract mismatch; missing UI state |
| 🟡 Minor | Rare edge case, naming/structure drift from plan.md, readability |
| 💡 Suggestion | Not a bug; worth doing |

## Verdict rules
- Any 🔴 → **🔴 Not ready**
- No 🔴, any 🟠 → **🟡 Ready with fixes**
- Only 🟡/💡 → **🟢 Ready**

## A good finding
- Has evidence: a test name, a command and its output, or file:line.
- Reproduces in one step someone else can run.
- Names one owner and a concrete fix, so the owner doesn't need to ask a follow-up question.
