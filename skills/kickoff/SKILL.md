---
name: kickoff
description: Turn a request into an agreed brief BEFORE any work starts — for a brand-new project (no repository yet, or an empty one) and for a new feature in an existing project. Use when asked to "start", "set up", "build a new …", "add a feature", or whenever the goal, scope or acceptance is not yet written down. Reads everything it can from the repository first, then asks only what the code cannot answer — at most five questions, once, each with a recommended default — and writes a short brief (plus the project conventions file for a new project). It decides the stack and the architecture shape for a new project; it never writes product code and never writes the plan.
---

# kickoff — ask once, briefly, then agree in writing

Most rework comes from starting on a guess. This skill closes the gap between
what was asked and what will be built, in one short exchange, and leaves a
brief that every later skill can read.

```
1 pick the mode  →  2 read before asking  →  3 ask (≤ 5, once)  →  4 decide
→  5 write the brief  →  6 confirm and hand off
```

---

## 1. Pick the mode

| Situation | Mode |
|---|---|
| No repository, or an empty one | **new project** |
| Existing repository, new capability | **new feature** |
| A fix, a refactor, or the user already gave goal + scope + acceptance | **skip** — restate the brief in three lines and continue with `ground-rules` |

Say which mode you chose in one line. If unsure, ask that as the first question.

---

## 2. Read before asking — never ask what the repository can answer

New feature: read the agent-instructions file, the conventions file
(`templates/project-conventions.md` shape), the dependency manifests, the
entry points and the nearest existing feature that resembles the request.
From them, state in one line: language/framework · data store · how similar
features are structured · test command. Use `deep-dive` if anything is unclear.

New project: check for any existing notes, a target platform the user already
named, and organisation-wide conventions. There is usually little to read; that
is why this mode asks more.

---

## 3. Ask — at most five questions, once, each with a default

Batch them in one message. Every question carries a **recommended answer** so
the user can reply "defaults" and move on. Skip any question already answered.

**New project**
1. What does it do, and for whom? (one sentence each)
2. First release scope — and what is explicitly NOT in it.
3. Stack — recommend one, with a one-line reason tied to the answers above
   and to what the team already runs.
4. Where data lives and where it runs (managed DB? container platform?).
5. Anything that costs money or reaches real people (paid APIs, messages,
   payments), and any hard constraint (deadline, compliance, budget).

**New feature**
1. The outcome, and how we will know it works (acceptance in one or two checks).
2. Does it change stored data? (if yes → `ground-rules` §2 approval later)
3. Does it call anything external, cost money or reach real people?
4. Out of scope for this change.
5. Only if the repository leaves it open: which of the existing patterns to follow.

Do not ask about style, naming or structure the codebase already shows.

---

## 4. Decide

**New project — stack and architecture shape**
- Pick the stack the answers support; prefer what the user already operates
  over the newest option. Name one alternative and why it lost.
- Propose the top-level layout by role (entry points, feature modules, shared
  infrastructure, tests, deploy), sized for the first release, not the fifth.
- Apply the shared criteria from day one (`ground-rules` §8): config validated at
  boot, nothing process-local that must be shared, every external call bounded,
  variation points as registries.
- State the scaling assumption in one line (single instance? stateless?).

**New feature — fit**
- Where it lives in the existing layout (by role, `ground-rules` §3).
- Which existing module or pattern it extends; whether it can be additive and
  off by default (`additive-change`).

---

## 5. Write the brief

Use `templates/brief.md`. Keep it under one screen. For a new project, also
create the project conventions file from `templates/project-conventions.md`,
filling what was decided and leaving the rest empty (the skills will ask).

The brief records decisions and their reasons. It is not a plan: phases, steps
and proof belong to `living-plan`, which starts from this brief.

---

## 6. Confirm and hand off

Show the brief and wait for a yes (or edits). Then:
- `living-plan` writes the plan from the brief;
- `ground-rules` governs the first edit;
- the brief's acceptance checks become the targets of `definition-of-done`.

No product code is written in this skill.

---

## Traps

- **Interrogating.** More than five questions, or a second round on things that
  could have defaulted. Ask once; assume the recommended default for silence.
- **Asking what the code says.** Stack, test command and folder conventions of an
  existing project are read, not asked.
- **Choosing by fashion.** A new stack the team does not run is a cost; say so.
- **Brief turning into a plan.** Steps and estimates go to `living-plan`.
- **Silent scope.** Anything left out must be written under "Out of scope".

## Report

```
kickoff — <new project | new feature | skipped>
read:      <files read, or "nothing to read">
asked:     <n> questions · answered: <n> · defaulted: <n>
decided:   <stack / fit, one line each>
written:   <brief path> [· conventions path]
next:      living-plan → ground-rules
```

## Organisation, debt and scale

The brief is the first place debt can be avoided: every deliberate shortcut is
written under "Known trade-offs" with a reason and the condition for revisiting
it. A decision not written down is re-litigated later.
