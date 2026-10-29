---
name: deep-dive
description: Evidence-first investigation before acting — use when asked to "dive", analyse, find a root cause, explain how something works, or execute a step from a plan/note/earlier session. Read the real code AND the real data before claiming anything, cite file:line for every load-bearing claim, separate what was verified from what was inferred, never guess a field/function/path name, and stop to report when reality contradicts the plan. Works for any language or stack.
---

# deep-dive — verify first, believe second

A guess that compiles is the most expensive kind: it passes review and silently
does nothing. This skill turns "I think" into "I checked".

## 1. The three sources, and what each can prove

| Source | Proves | Does NOT prove |
|---|---|---|
| **Code** | what the program INTENDS to do | what the data actually holds |
| **Data** (a real record, a real payload, a real log line) | what exists right now | why it got that way |
| **Runtime** (an authenticated call, an observed side effect) | what actually happens | that it happens on every path |

A claim needs the source that can prove it. Reading `obj.fieldName` in code does
not prove records carry `fieldName` — open one record. An unauthenticated call
that returns 401 tests the guard, not the handler. Use `probe` for runtime evidence.

## 2. Never guess a name

Before writing to, filtering on, or reading any field, key, column, route,
env var or function: find it in the real source of truth.

```bash
# the record decides, not the code — print key names and value SHAPES, never values
<db-client> --eval 'const d = <collection>.findOne(<narrow filter>);
  Object.keys(d).forEach(k => print(k + ": " + typeof d[k]))'
```

Watch for: two similar ids (an internal id vs an external one), renamed fields in
older records, values stored under a nested key, empty strings vs absent keys.

## 3. Method

1. **State the question** in one sentence. What would change your mind?
2. **Map the path:** entry point → every hop → where it lands. Grep callers,
   registries and string-built references (dynamic lookups hide from "find usages").
3. **Read the actual lines**, not summaries. Note file:line as you go.
4. **Check the data** for every field the path depends on.
5. **Check runtime** when the answer depends on configuration, ordering or
   environment.
6. **Write findings** in two lists: **Verified** (with file:line / record /
   command output) and **Inferred** (with why you believe it and how to verify).
7. **Recommend**, with the smallest change that answers the question.

## 4. Plans are claims, not facts

A plan, todo note or earlier session's summary was written at some commit, by
someone who could have been wrong.

- Before each step, re-verify every claim the step relies on: paths, line
  numbers, signatures, "X is the only caller", "Y falls back to Z".
- Re-check at every step boundary — your own earlier steps changed the tree.
- **On conflict: stop and report.** Quote what the plan says, show what the code
  shows (file:line), and wait if the difference changes the design. Quietly
  adapting hides that the source was wrong — and its next claim deserves the
  same doubt.
- A plan verified at commit X is unverified at commit Y after any merge, rebase
  or move touching the cited files.
- A step whose success cannot be measured is not ready; define its proof first.

## 5. Output shape

```
Question: …
Verified:
  - <claim> — <file:line | record shape | command → result>
Inferred (not yet proven):
  - <claim> — <reason> — <how to prove>
Root cause / answer: …
Recommendation: … (blast radius: …)
```

Say "I only read the code" when that is all you did. "Ready" and "works" need
runtime evidence.

## Organisation, debt and scale

- Note structural smells found on the way (duplication, hard-coded values,
  process-local state, oversized files) in a separate **Debt seen** list — report
  them; do not fix outside scope.
- A recommendation that adds a special case to a central chain is a red flag;
  look for the registry/config point instead.
