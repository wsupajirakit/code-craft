---
name: ground-rules
description: The baseline protocol for ANY task that edits code, config, schema or scripts, in any project and any language. Use BEFORE the first edit (confirm the exact repo, branch and files; look up the project's conventions), WHILE editing (announce stored-data changes before touching them, place new files by role, size inline notes by importance, keep simulation/bypass code out of version control, respect parallel sessions) and AT THE END (record the work, prove it, report it). Skip only for pure read-only work. When there is no repository yet, or the goal/scope is not written down, run kickoff first. It delegates the deep parts to sibling skills — kickoff, deep-dive, additive-change, regression-guard, definition-of-done, living-plan, status-report.
---

# ground-rules — the protocol around every change

Code can be reverted. Stored data, a published secret, or an edit on the wrong
branch often cannot. This skill is the frame every change runs inside; the
sibling skills hold the detail.

```
BEFORE  → kickoff if new project/feature has no brief · 0 confirm where · 1 load conventions · deep-dive if unknown
DURING  → 2 stored data · 3 placement · 4 notes · 5 no bypass code · 6 parallel sessions
AFTER   → 7 record (living-plan) · verify (definition-of-done) · report (status-report)
ALWAYS  → 8 organised · no new debt · scales · dynamic, not hard-coded
```

---

## 0. Confirm where — before any edit

Never assume the target. In a multi-repo, multi-branch, multi-session setup, a
wrong guess silently overwrites someone else's work.

- Read the current branch from the tool (`git branch --show-current`), not from
  memory or from an earlier message.
- State the target plainly before editing: **repo · branch · files**.
- If the user named a branch or project that does not match the checkout, STOP
  and ask. Do not switch, do not pick the "obvious" one.
- Protected branches (main/production and anything the conventions list) are
  never edited directly. Work in an isolated lane — see `ship-safe`.
- Skip the confirmation only when the user has already pinned the repo and branch
  for this exact task in this turn.

---

## 1. Load the project's conventions

Generic skills stay generic. Everything project-specific lives with the project:

1. The agent-instructions file at the repo root.
2. A project conventions file, if present (template: `templates/project-conventions.md`).
3. A project-scoped skill that covers the same area.

Read them before acting. They decide: where the work log lives, how task notes
are named, which branches are protected, the language of in-repo text, the local
run command and URL, the test commands, the resource cap.

**Precedence:** explicit user instruction > project conventions > these skills.
If the conventions are silent on something this skill needs (for example, where
task notes go), ask once, then follow the answer for the rest of the task.

---

## 2. Stored data — announce first, highest stakes

Any change to the **shape of data that persists** is announced and approved
BEFORE editing:

- adding / removing / renaming a field, column or key;
- changing the shape of a value written to storage;
- an index, migration, backfill or seed;
- changing an enum or constant that existing records already hold.

Announce in this form and wait for approval:

```
- field / shape:        <name + collection/table>
- before → after:       <old> → <new>
- existing records:     <migration needed? or absent = legacy, no migration>
- backward compatible:  <can old records still be read and written?>
- revert path:          <how to undo — and whether written data can be undone>
```

Prefer the additive form: a new field defaults to empty/off so every existing
record keeps working with no migration. The full method is `additive-change`.

Every work-log entry for a task that touched data carries a **data effect** line,
even when the answer is "none".

---

## 3. File placement — new files only

Placement is decided by the file's ROLE, never by convenience.

1. Identify the role (what job it does), using the naming suffixes the codebase
   already uses.
2. Find an existing directory that groups that role → place it there.
3. No such directory, and this makes 2+ files of that role → create one and say so.
4. The only file of its role → place it beside its closest collaborator; do not
   create a directory for a single file.
5. Module/package roots hold entry points only. Never dump a new file there.
6. Role genuinely ambiguous → ask one concrete question.

After creating a file, verify it is wired in: imports resolve, it is registered
where registration is needed, and the test/build globs actually include it.
A file that compiles but is never loaded is a silent failure. Report what you
wired and where.

Scope limit: do not move existing files to "fix" placement unless asked.

---

## 4. Inline notes — sized by importance

- **Important logic → detailed note:** money paths, security, external contracts,
  state machines, owner decisions, anything easy to get wrong. Say WHAT it does,
  WHY (the constraint or decision), and HOW to change or revert it.
- **Ordinary code → a one-liner or nothing.** Do not restate the syntax.
- Follow the repo's existing tag convention for marking a change, if it has one.
- A note describes CURRENT behaviour truthfully. If something is deliberately
  left as-is, say it is deliberate; never describe a target the code does not
  yet meet.

---

## 5. Simulation and bypass code never enters version control

Code whose purpose is to **simulate a flow** or **bypass a check** (fake
providers, skipped verification, forced success, test backdoors) is a local
patch only — never committed, never pushed, never in a tracked file, however it
is gated. A flag can be flipped; a file that is not in the repo cannot ship.

- Keep such patches outside the tracked tree, or in an ignored local folder.
- Removing them must leave the local setup working.
- Before any commit that touches money, auth or external-call code, grep the
  staged diff for the project's simulation markers.

---

## 6. Parallel sessions — guardrails

Assume other sessions are working in the same repo.

- Stage explicit paths only. Never `git add -A` / `git add .` in a shared tree.
- Never `stash`, `checkout --`, `reset` or `clean` a tree you do not own.
- Append to shared logs; never rewrite or reorder other entries.
- Before switching branches, back up ignored local files the other branch may
  track — git can delete them on checkout.

---

## 7. Wrap-up — every task that changed something

1. **Record** the task in the project's work log and its task note → `living-plan`.
2. **Prove** it — built, running, observed where it lands → `definition-of-done`.
3. **Report** what was measured, not what was launched → `status-report`.

Never call a task done on the strength of a passing suite alone
(`regression-guard`), and never execute a plan step without re-checking its
claims against the code (`deep-dive`).

---

## 8. Organisation, no debt, scale, dynamic — applies to every change

These four are acceptance criteria, not polish. A change that works but fails
one of them is not done.

**Organised**
- One responsibility per file/module; a new concern gets its own unit from the
  start, never "inline for now".
- Group by feature or role, in folders named after the parent component; no flat
  dumps.
- Keep files small (the project's size limit; a sensible default is ~400 lines).
  A file crossing it is split, not waived — waiver lists only shrink.
- Follow existing patterns before inventing new ones; if a new pattern is
  needed, apply it in one place and say so.

**No new debt**
- No copy-paste of logic — extract and reuse. No dead code, commented-out
  blocks, stray debug output or TODOs without an owner and a reason.
- No silent workaround: if something is knowingly left imperfect, record it as
  a known item with why and how to fix (`living-plan`).
- Leave every file you touch at least as tidy as you found it — within scope.
- Tests, notes and types ship with the change, not "later".

**Scales**
- Assume more than one instance: no process-local state that must be shared
  (counters, locks, caches, files on local disk) without a shared store or a
  documented single-instance assumption.
- Bound everything: pagination, batch sizes, timeouts, cache sizes, retries
  with backoff.
- Make writes and handlers idempotent wherever events can be retried or
  duplicated.

**Dynamic, not hard-coded**
- Values that differ by environment, tenant, customer or over time are
  configuration or data — never literals in code: URLs, limits, ids, names,
  thresholds, texts, feature switches.
- Variation points use a registry/strategy/table so a new case is one new entry,
  not an edit to a central `if/else` chain.
- Config is validated at boot with safe defaults; an invalid value stops startup
  instead of failing later.
