---
name: living-plan
description: Write and maintain the plan and the per-task record of the work — a plan file with phases and status, one task note per task, an append-only work-log entry with file:line references, before/after numbers, dead ends and plain-language items waiting on the owner. Use when asked to plan, to "write it down", to update the plan or notes, after any task that changed something, and whenever work spans sessions or repos.
---

# living-plan — the record that lets the next session continue

The project conventions decide WHERE these files live and how they are named.
This skill decides WHAT they must contain.

## 1. The plan file

```
# <topic> — plan
Status: <draft | approved | in progress | done>   Updated: <date>   Base: <commit>

## Goal
<what the finished work must do — specifics that decide its shape, in your own words>

## Constraints
<what must not change, protected areas, approvals required>

## Phases
| # | Step | Proof of done | Status |
|---|------|---------------|--------|

## Waiting on the owner
1. <decision> — options: A … / B … — recommendation: … — plain language, no jargon

## Risks / rollback
```

- Every step has a measurable proof; a step without one is not ready.
- Update status as steps complete; never silently rewrite an approved step —
  add a dated "changed because …" line.
- Do not paste chat wording verbatim; state the requirement.

## 2. The task note — one file per task

Append to the existing note for the same task; never overwrite, never start a
second file. Add `## <date> — <this round>` sections.

Must carry:
- **Requirement** — the specifics that decide the work.
- **Root cause** — what the code actually did, with file:line; say plainly if an
  earlier diagnosis was wrong.
- **Before → after numbers** — test counts, baseline results.
- **Mutation results** — what was broken on purpose, what caught it, what didn't.
- **Dead ends** — what was tried and failed, and why. A record of only the
  successful path makes the next session repeat the failures.
- **Left** — deploy state, follow-ups, knowingly undone items.

## 3. The work-log entry — append only

```
## <yyyy-mm-dd> — <one-line title>
- What: …
- Why: …
- Where:
  - path/to/file.ext:LINE — what changed
- Behaviour: <changed vs preserved — say plainly if an output changed>
- Data effect: <shape/field/index touched, or "none">
- Tests: <what ran, numbers, or "none — why">
- Follow-up: <… or "none">
```

file:line is mandatory and re-read after the last edit so it points at final code.

## 4. Work across repos

Each repo gets its own note describing only its half (its refs, its numbers, its
commit). Each note names the other note at the top, states which side is the
source of truth, and the deploy order when the halves depend on each other.

## 5. Hygiene

- No personal names, machine names, absolute paths or credentials in any note.
- Relative dates become absolute dates.
- Notes describe current truth; mark superseded parts instead of deleting the
  argument behind them.

## Organisation, debt and scale

- One plan per topic, one note per task, one work log per repo — no parallel
  copies of the same information.
- Known debt has a home: a "Known items" section with owner, reason and fix,
  so it is visible and shrinks over time.
- Superseded plans are marked superseded and linked to their successor.
