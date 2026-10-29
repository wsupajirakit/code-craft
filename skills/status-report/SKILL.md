---
name: status-report
description: Report results honestly and readably — what changed, how it was proven, what was not verified, what is left, and what needs the user's decision — in the user's language and plain words. Use at the end of any task, when the user asks for a summary ("sum it up", "explain in plain words", "what's left?"), or when handing work to another person or session.
---

# status-report — what was measured, said plainly

## 1. Honesty rules

- Report what was **measured**, not what was **launched**. "Started" ≠ "running";
  "tests pass" ≠ "works".
- Failed → say so, with the specific error and root cause (not the full log).
- Skipped / not run / could not verify → name it and say why.
- Never let a green suite stand in for a check that was not performed.
- "Already built, ready to use" is the claim that needs the most proof — back it
  with a runtime observation or downgrade it to "code exists, untested".
- No overselling, no hype words, no invented numbers.

## 2. Language

- Reply in the language the user writes in (or the one the conventions set).
- Plain words first; technical terms only where they add precision, explained once.
- When the user asks for "plain language", give an everyday example.

## 3. Shape

Short first, detail on demand:

```
<one-line outcome>

What changed:     <bullets, user-visible terms>
Proven by:        <numbers / observations>
Not verified:     <items + why>            (omit if none)
Left / next:      <items>                  (omit if none)
Your decision:    <numbered, each with options + recommendation>   (omit if none)
```

- Link files as `path:line` relative to the repo.
- Numbers beat adjectives: "120 tests, 0 failed" not "all good".
- Decisions are phrased so the user can answer with one word or a number.

## 4. Privacy

Never include credentials, personal account identifiers, machine names, usernames
or absolute home paths in a report — use relative paths and redact values.

## Organisation, debt and scale

- Include a **Debt** line when the work left or found any: what, why, how to fix.
- State the scaling assumption when it matters ("single instance only until X").
