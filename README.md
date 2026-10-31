# code-craft

Project-neutral agent skills for disciplined software work — usable in any
repository, any language, any stack. This README is the governing document for
the whole set: the rules here apply to every skill, and every skill must stay
consistent with them.

---

## 1. Purpose

Turn hard-won working rules into a reusable system so that every change is:

- **targeted** — made in the right repo, branch and file;
- **proven** — by measurement, not by a green suite or a confident sentence;
- **safe** — no lost data, no leaked secrets, no surprise costs, nothing done to
  systems that are not ours;
- **clean** — organised, free of new debt, ready to scale, configurable;
- **recoverable** — every step can be undone, and work can resume after any
  interruption.

---

## 2. Hard rules (non-negotiable, every skill, every project)

1. **Confirm before editing.** Repo · branch · files are read from the tools and
   stated before the first edit. Protected branches are never edited directly.
2. **Never guess.** Field names, routes, paths, versions and behaviour come from
   the real code, the real data or the running system — cited.
3. **Stored data is announced first.** Any change to what is persisted (fields,
   shapes, enums, indexes, migrations) is approved before it is made.
4. **Ask before anything costly or outward-facing.** Paid calls, messages to real
   people, shared or production environments, publishing, merging — every time.
5. **Own systems only.** No attacking, scanning, fuzzing or load-testing anything
   not owned or explicitly authorised.
6. **Secrets never travel.** Never printed, logged, committed, published or
   pasted. Report file and rule, never the value. A leaked secret is rotated.
7. **No personal or machine identifiers** in files, commits, notes or reports —
   no names, usernames, hostnames, absolute home paths or account ids.
8. **No simulation or bypass code in version control**, however it is gated.
9. **Report what was measured.** Failed, skipped and unverified steps are named.
10. **Respect the shared machine.** Heavy work runs at low priority inside the
    project's resource cap.

---

## 3. Shared acceptance criteria

A change that works but fails one of these is not done
(full text: `ground-rules` §8).

| Criterion | Means |
|---|---|
| **Organised** | one responsibility per unit, grouped by feature/role, small files, existing patterns first |
| **No new debt** | no duplication, dead code, stray debug output or silent workarounds; known gaps are recorded with owner and fix |
| **Scales** | no process-local shared state, everything bounded, idempotent where retried, scaling assumption stated |
| **Dynamic** | environment/tenant/time-varying values are config or data; variation points are registries, not `if/else` chains; config validated at boot |

---

## 4. Lifecycle — which skill when

```
START      session-recovery        (only if continuing interrupted work)
           kickoff                 new project or feature: ask ≤ 5, decide, write the brief
           ground-rules     confirm target · load project conventions
UNDERSTAND deep-dive  ── probe   evidence from code, data and the running system
DESIGN     living-plan           plan with measurable proof per step
CHANGE     additive-change       new optional things
           bottleneck-check      performance work
           security-audit        security review and hardening
           baseline              capture before, compare after
VERIFY     regression-guard      baseline suites, red-first, mutation
           definition-of-done    build → artifact → restart → observe → stable
DELIVER    ship-safe             lane, tags, explicit commits, push, merge on request
           pre-publish-scan      before anything leaves the machine
CLOSE      living-plan           note + work log
           status-report         what was measured, in plain words
```

| Skill | Trigger |
|---|---|
| `kickoff` | start a new project or feature; goal, scope or acceptance not yet written |
| `ground-rules` | any task that edits code, config, schema or scripts |
| `deep-dive` | analyse, explain, find a root cause, execute a plan step |
| `probe` | the answer depends on what a running system actually does |
| `baseline` | before/after or old/new comparison; behaviour-identical refactor |
| `additive-change` | add an optional field, option, flag or feature without affecting existing behaviour |
| `regression-guard` | prove nothing broke and the tests can fail |
| `bottleneck-check` | slowness, high resource use, before scaling out |
| `security-audit` | security check, hardening, authorised vulnerability testing, pre-release |
| `definition-of-done` | before saying "done"; driving work to completion |
| `ship-safe` | worktrees, tags, commits, pushes, merges |
| `living-plan` | plans, task notes, work-log entries |
| `status-report` | summaries, hand-offs, "what's left?" |
| `session-recovery` | continuing after a dropped or compacted session |
| `pre-publish-scan` | publishing, exporting, sharing code or data |

Small tasks use only what applies: a one-line fix still needs
`ground-rules`, `regression-guard` and `status-report`, not a plan.

---

## 5. Precedence

1. Explicit instruction from the user.
2. The project's conventions (agent-instructions file, conventions file,
   project-scoped skills).
3. These skills.

Project-scoped skills keep authority over their own specifics; these skills
provide the method. When two sources conflict, say so and ask — do not pick one
silently.

---

## 6. Project conventions

The skills never hard-code project facts. Each project provides them:

1. Copy `templates/project-conventions.md` into the project, or merge it into
   the project's agent-instructions file.
2. Fill in branches, record locations, run/test commands, entrances, resource
   cap, paid actions, data rules and organisation limits.
3. Empty entries make the skills ask once and follow the answer.

---

## 7. Layout

```
README.md                      this governing document
PLAN.md                        roadmap, status, content scan
templates/
  project-conventions.md       per-project facts the skills read
  brief.md                     the agreed goal/scope/decisions kickoff writes
skills/<name>/SKILL.md         one skill per folder
```

---

## 8. Install

User-wide (every project):

```bash
SKILLS_DIR=<your agent's user-level skills directory>
mkdir -p "$SKILLS_DIR"
for d in skills/*/; do ln -s "$PWD/$d" "$SKILLS_DIR/$(basename "$d")"; done
```

One project only: link or copy the needed folders into that project's
skills directory. Symlinks keep every project on the latest version.

---

## 9. Authoring rules — adding or changing a skill

- **One job per skill.** If a new rule fits an existing skill, extend it. A new
  skill is justified only for a genuinely new job.
- **No duplication.** Each rule lives in exactly one skill; others link to it by
  name and section.
- **Neutral.** English only. No personal, company, project, product, vendor or
  machine names; no absolute paths; no credentials; no real data in examples.
- **Trigger-first description.** The frontmatter `description` says when to use
  the skill in concrete phrases, and does not overlap another skill's trigger.
- **Shape of a skill:**

  ```
  ---
  name: <kebab-case, matches the folder>
  description: <when to use it, what it guarantees>
  ---
  # <name> — <one-line principle>
  ## 1..n  rules / method / checklist / traps
  ## Report               the output block the skill produces
  ## Organisation, debt and scale   how the shared criteria apply here
  ```

- **Prove it.** Every rule describes how to verify it was followed.
- **Keep it current.** `security-audit` refreshes advisories on every run and
  carries an update log; any skill whose rule turns out wrong is fixed at once,
  with the reason recorded in `PLAN.md`.
- **Scan before saving.** Run the content scan in `PLAN.md` after every change.

---

## 10. Versioning and review

- Changes are small and dated in `PLAN.md`.
- Review the whole set at least monthly: triggers still distinct, no duplicated
  rules, no stale tool advice, security checklist refreshed.
- A skill that is never triggered in practice is merged into another or removed.
