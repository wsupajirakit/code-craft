---
name: baseline
description: Capture real samples (inputs + outputs) BEFORE a change and compare the same samples AFTER — or compare two implementations side by side (old vs new, lane vs main line). Use for refactors that must be behaviour-identical, migrations/ports, "must not affect existing behaviour" requests, and any before/after or parity claim. Normalises volatile fields, records provenance, fails loudly when nothing was compared, and recaptures instead of diffing against a stale baseline.
---

# baseline — measure, don't promise

"Nothing else changed" is a claim until two captures say so.

## 1. What a baseline is

A set of **cases**, each `{ id, input, output }`, plus a **provenance header**:

```
captured_at:  <date/time>
commit:       <full hash> (+ dirty: yes/no)
environment:  <runtime version, relevant flags/env — names only, no values>
source:       <where inputs came from — real captures, generated, fixture>
cases:        <count>
```

Store it outside the code being changed (a temp or ignored folder) unless the
project wants committed golden files. Never store secrets or personal data —
replace values, keep structure (field names, nesting, array lengths, types).

## 2. Choosing cases

- Start from **real inputs** (captured requests, stored records, logged
  payloads), not hand-written guesses from documentation.
- Cover every branch the change could touch: happy path, each error path,
  empty/missing fields, the largest realistic input, edge enum values,
  the "feature off / not configured" path.
- Keep real counts. If the real record had 5 items and 2 failures, keep 5 and 2 —
  assertions reason about those numbers.
- Name each case by what it proves, so a diff line reads as a sentence.

## 3. Normalise before comparing

Strip or replace what changes on every run: timestamps, generated ids, random
tokens, durations, absolute paths, unordered collections (sort them). Do it with
an explicit list, printed in the report — a silent normaliser can hide a real
difference.

## 4. Compare

```
for each case id in BEFORE ∪ AFTER:
  missing in AFTER   → FAIL (a case disappeared)
  new in AFTER       → report (not a pass)
  normalised outputs differ → DIFF (show the smallest differing path)
compared == 0        → FAIL (a comparison of nothing is not a pass)
```

Every DIFF is classified: **intended** (cite the requirement) or **regression**
(fix it). An unexplained diff is a regression until explained.

## 5. Parity mode — two implementations

Run the same inputs through both (old service vs new, main line vs lane).
Same normalisation, same report. If one side is unreachable or misconfigured,
the run FAILS — it does not skip. A guard that skips when misconfigured has the
exact defect it exists to catch.

## 6. Staleness

A baseline belongs to its commit. After any merge, rebase or upstream change to
the files under test, recapture BEFORE on the new base. Never diff against an
old capture and call the result "after".

## 7. Report

```
Baseline: <n> cases @ <commit> · normalised: <list>
After:    <n> cases @ <commit>
Result:   <identical | k intended diffs | k regressions | FAIL: nothing compared>
Diffs:    <case id> — <path> — <before> → <after> — <intended: why | regression>
```

## Organisation, debt and scale

- Capture and compare are scripts with inputs as arguments (target, case set,
  normaliser list), reusable for the next change — not hand-run commands.
- Case sets are data files, grouped by feature; adding a case = adding an entry.
- Old captures are deleted or clearly marked stale; no pile of unnamed snapshots.
