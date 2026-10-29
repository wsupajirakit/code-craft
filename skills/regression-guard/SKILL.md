---
name: regression-guard
description: Prove a change broke nothing and that its tests can actually fail. Use for any fix, refactor, move or feature where "must not affect existing behaviour" matters — before editing (record a test baseline, write characterisation and red-first tests) and after (same suites green, mutation check, blast-radius re-trace). Also defines how to read a suite that is red before you start, how to keep test runs inside a resource cap, and why a green run alone is never proof.
---

# regression-guard — green is the floor, not the finish line

A passing suite proves only that nothing you touched broke a case someone
already thought to write.

## 1. Before editing

1. **Baseline the suites.** Run the relevant suites on the unchanged tree and
   write down the numbers (suites, passed, failed, skipped). A suite that is red
   before you start must be known before you start — otherwise a pre-existing
   failure gets blamed on you, or worse, a new one gets waved off as old.
2. **Know what each red means.** For each pre-existing failure, record why
   (environment, missing external service, intentional guard). Quote which suite
   you ran — never "the whole suite" when you ran one tier.
3. **Characterise what must not move.** If the area has no test, write one that
   freezes current behaviour (output snapshot, request count, rendered markup)
   and passes on the UNCHANGED code. For larger surfaces use `baseline`.
4. **Write the target test red-first.** It must fail on the unchanged tree, for
   the right reason — read the failure message.

Two groups, always separable:
- **TARGET** — the new behaviour: red before, green after.
- **PIN** — what must not change: green before and after.

## 2. After editing

- Same suites, same commands, compare numbers with the baseline.
- **Confirm the tests exercised the change**: the changed file was loaded, the
  test glob includes it, the count of cases in that area is not zero.
- **Typecheck and lint prove shape, not behaviour.** Anything resolved at runtime
  (registries, globs, dynamic imports, ordering) is checked at runtime.
- **Re-read the diff** after every edit, including fixes for failures — those
  are the edits most likely to introduce a second bug.
- **Re-trace the blast radius** each iteration: callers, registries, string-built
  references. A search from three edits ago is stale.
- **Distrust a failure that disappeared** without a clear mechanism; it may have
  moved, not gone.

## 3. Mutation — prove the tests have teeth

For each new rule:

| Mutation | Must go red |
|---|---|
| remove the guard / new branch | TARGET |
| invert the condition | PIN |
| widen a filter, drop a bound | something you can name |

Mutate safely:
- Back up the file (`cp`) and record its hash before mutating.
- Restore from the backup, compare the hash, and re-run the full suite before
  any commit or push. Never restore with a VCS checkout in a shared tree.
- Never let a sub-agent mutate the real working tree.

Record each mutation, which tests caught it, and any that caught nothing — a
mutation that kills nothing means the rule is untested. Say so.

## 4. Behaviour-identical moves (refactor, split, extract)

1. Read the target and grep every reader/writer of the state it touches; a block
   that owns state or effects is coupled — moving it moves behaviour.
2. Mark boundaries by a unique anchor, never by a string that appears twice.
3. Move verbatim; state stays where it is unless covered by interaction tests
   written before the move.
4. After: characterisation green + one mutation in the moved code goes red.
5. One self-contained, revertible commit per move, with checkpoints
   (`ship-safe`).

## 5. Resource cap

Test runs share the machine with the user's work.

- Respect the project's cap (default: keep total CPU under ~75%).
- Run at low priority (`nice -n 19`, or the platform's background QoS) with one
  or two workers.
- Measure during the run (`top`/`ps`) instead of assuming; lower workers if over.
- Quick targeted suites first; the full suite once, at the end.

## 6. Report

```
Baseline (before): <suite> <passed>/<failed>/<skipped> — known reds: <why>
After:             <suite> <passed>/<failed>/<skipped>
TARGET: red→green ✓   PIN: green→green ✓
Mutations: <n> tried — <caught by …> — <uncaught: …>
Not run: <what and why>
```

## Organisation, debt and scale

- New tests sit beside their siblings, named by area and subject; shared
  fixtures live in one place and are imported, never copied into specs.
- A fixture mirroring a real shape is guarded against that shape at run time,
  so it cannot drift silently.
- Fix flaky or environment-dependent tests at the cause, or record them as known
  items — never skip them silently.
