---
name: ship-safe
description: Isolated, reversible delivery with git — work in a separate worktree lane, set checkpoint tags before and after, stage explicit paths, write clean commits, push when asked, merge only when explicitly told, and bring lane-only files across by hand. Use for any commit, push, tag, branch, worktree or merge operation, especially in repos shared with other sessions or with protected branches.
---

# ship-safe — every step can be undone

## 1. Never on the shared or protected branch

- Protected branches (main/production, anything the conventions list) are never
  committed to directly, never force-pushed, never reset.
- New work goes in its own lane:

```bash
git worktree add ../<repo>-<lane> -b <lane-branch> <base>
```

- A lane marked standalone in the conventions is never merged anywhere.
- Ignored files do not survive a branch switch if the other branch tracks them;
  back up ignored local notes before any checkout.

## 2. Checkpoints

```bash
git tag pre-<topic>-<yyyymmdd>    # before the first change
git tag post-<topic>-<yyyymmdd>   # after the verified result
```

Push tags with the work when pushing. A bad step is recovered with
`git checkout <tag>` in the lane — no archaeology. Tag names carry no personal
or machine identifiers.

## 3. Commits

- Stage explicit paths only (`git add <path> …`). Never `-A` / `.` — they sweep
  untracked local scripts and notes into history.
- Review `git diff --cached --stat` before every commit.
- One self-contained, revertible change per commit; message says what and why.
- Follow the repository's attribution rules for author, committer and trailers
  exactly as the user or project instructs.
- Before committing: no secrets, no simulation/bypass code, no build output
  (`ground-rules` §5, `pre-publish-scan` for anything public).

## 4. Push and merge

- Push when the user asks to push — the lane branch, plus its tags.
- Merge only when the user explicitly says so, into the branch they named.
- Before merging: rebase/merge the target into the lane, re-run the suites
  there (`regression-guard`), then merge with a merge commit that names the lane.
- After merging: re-run the suites on the target; tag `post-merge-<topic>-<date>`.

## 5. Lane-only files

Notes and plans are often ignored by git and do not travel with a merge.

- Copy them across by hand during the merge.
- If names collide with files already in the target (numbered notes), rename
  the incoming ones with the lane's suffix — never overwrite.

## 6. A lane's red tests may be its environment

A fresh worktree lacks untracked inputs (data folders, local env files,
generated fixtures). "Suite failed to run" there is not a red test; copy the
untracked inputs in, then judge.

## 7. Report

```
Lane: <branch> @ <worktree folder name>
Tags: pre-… → post-… (pushed: yes/no)
Commits: <hash> <subject> …
Pushed: <remote/branch> | Merged: <no | into X at <hash>>
```

## Organisation, debt and scale

- One topic per lane, one change per commit; no "misc fixes" commits.
- Stale lanes, tags and worktrees are listed for cleanup when their work is
  merged or abandoned — not left to accumulate.
- Branch and tag names follow one pattern so they can be listed and pruned by script.
