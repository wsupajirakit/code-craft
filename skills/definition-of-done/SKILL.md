---
name: definition-of-done
description: The gate before saying a task is finished — and the drive to actually reach it. Use when the user says "finish it", "make it complete", "is it done?", or before any claim that a change works. Requires the change to be built, confirmed inside the shipped artifact, restarted, observed where it lands, and stable over time on every entrance; keeps working through the remaining items instead of stopping half-way; states exactly what is left.
---

# definition-of-done — finished means proven, not launched

"Done" is the most expensive word in a report. It is earned by measurement.

## 1. Keep going until it is done

- Turn the request into an explicit checklist at the start (every case, every
  page, every environment the user named). Work through it to the end.
- Do not stop at the first green, the first working case, or the first partial
  result. Do not stop to ask permission for steps the user has already asked for.
- Stop only for: a decision that is genuinely the user's, a destructive or
  outward-facing action needing approval, a conflict between plan and reality
  (`deep-dive` §4), or a hard blocker — and then say which.
- If the session may drop, keep the checklist and progress in the task note
  (`living-plan`) so `session-recovery` can continue.

## 2. The gate — all of these, in order

| # | Check | Proof |
|---|---|---|
| 1 | Tests | suites green vs baseline, TARGET red→green, mutation caught (`regression-guard`) |
| 2 | Build | a fresh build (clear incremental caches), exit 0 |
| 3 | In the artifact | a symbol or string that only the new code contains is found in the built output (count ≥ 1) |
| 4 | Running the new build | restart; build time and process start time are adjacent |
| 5 | Observed where it lands | the record in storage, the log line, the rendered screen, the delivered message (`probe`) |
| 6 | Stable | same process across several samples over ~1–2 minutes, every entrance (direct, proxy, tunnel) |
| 7 | Client side | UI changes built and served — the browser receives the new bundle (check its hash/string) |
| 8 | Visible | a control behind a condition is in the bundle yet hidden by data — verify the condition with real data |

Skipping a row is allowed only when it does not apply — say which and why.

## 3. Platform traps

- GNU-only tools (`setsid`, `timeout`, `sed -i` without a suffix) behave
  differently or are absent on some systems; check exit status instead of
  assuming a command worked.
- Deleting a build directory that a container or server mounts can break the
  mount; rebuild in place and restart the consumer if needed.
- A backgrounded process can die quietly; re-check it right before every proof run.

## 4. Done statement

Use only what was measured:

```
Done: <what>
Proven by: <tests numbers> · <artifact check> · <observed where> · <stable N s across M entrances>
Not verified: <item — why>
Left: <remaining items, or "none">
```

"Started" and "verified running for N seconds across M entrances" are different
sentences; only the second earns "ready".

## Organisation, debt and scale

- "Done" includes the organisation criteria in `ground-rules` §8: no new
  debt, no hard-coded values, multi-instance assumption stated.
- Leftover temp files, debug flags, stray processes and probe code are cleaned
  up before declaring done — or listed as left.
