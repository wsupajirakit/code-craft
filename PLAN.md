# code-craft — plan

Status: phase 1 drafted   Updated: 2026-10-29

## Goal

One neutral skill set that any project can use, built from rules that proved
necessary in real work, with project specifics moved out into a per-project
conventions file. The existing project-scoped skills stay untouched until the
neutral set is proven.

## Constraints

- English only. No personal, company, project, product or machine names; no
  absolute paths; no credentials — in any file.
- Each skill focused on one job; no content duplicated across skills — link instead.
- Every skill enforces: organised · no new debt · scales · dynamic.

## Phases

| # | Step | Proof of done | Status |
|---|------|---------------|--------|
| 1 | Draft all 14 skills + README + conventions template | files exist; scan below is clean | done |
| 2 | Review: trigger descriptions do not overlap each other or built-in skills (code review, simplify, security review of a diff) | overlap table written, conflicts resolved | todo |
| 3 | Install for one project, run 3 real small tasks (an edit, an additive field, a public-scan dry run) | each task's report shows the skill was used at the right moment | todo |
| 4 | Install user-wide; run a task in a second project with a different language | same | todo |
| 5 | Decide whether project-scoped skills shrink to project facts + links to these | owner decision | waiting |
| 6 | First `security-audit` run; fill its update log and "last reviewed" | log entry exists | todo |

## Waiting on the owner

1. Phase 5: keep the project-scoped skills as they are, or slim them down to
   project facts and point to these skills? Recommendation: decide after phase 3.

## Content scan (run before every change to this repository)

```bash
# banned-term list lives OUTSIDE this repository
grep -rniE -f <path-to-private-banned-terms.txt> . && echo "BLOCKED"
grep -rnE '/Users/[A-Za-z]|/home/[a-z]|[A-Z]:\\\\Users\\\\' . && echo "BLOCKED: absolute path"
grep -rnE 'BEGIN [A-Z ]*PRIVATE KEY|AKIA[0-9A-Z]{16}|gh[pousr]_[A-Za-z0-9]{30,}|sk-[A-Za-z0-9]{20,}' . && echo "BLOCKED: credential"
```

## Origin of the rules

| Skill | Built from |
|---|---|
| ground-rules | confirm-target, stored-data approval, file placement, sized notes, no bypass code in VCS, parallel-session guardrails |
| deep-dive | never guess a field; source ≠ data; a 401 tests the guard; plans are claims |
| probe | stage traces behind flags; authenticated calls; stability sampling |
| baseline | golden / parity comparisons; before/after as a measurement; staleness |
| additive-change | optional-by-default fields; layers that drop fields; full round trip |
| regression-guard | red first; mutation; test trust; behaviour-identical moves; resource cap |
| bottleneck-check | measure-first performance work; bounded caches; multi-instance assumptions |
| security-audit | defensive review checklist; refresh advisories every run |
| definition-of-done | build → artifact → restart → observe → stable |
| ship-safe | worktree lanes; checkpoint tags; explicit staging; merge only when told |
| living-plan | task notes with before/after and dead ends; append-only logs |
| status-report | report what was measured |
| session-recovery | continue from proof after a dropped session |
| pre-publish-scan | screening a codebase before a public release |
