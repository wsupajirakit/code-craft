---
name: session-recovery
description: Recover and continue after a dropped, crashed or compacted session — locate the right repo, worktree and branch, inspect the tree, tags and running processes, check for half-applied mutations or edits, read the latest plan/task note, and resume from the last step that is PROVEN done. Use when the user says the session dropped, "continue", "pick up where we left off", or when a summary of earlier work is all you have.
---

# session-recovery — continue from proof, not from memory

A summary of earlier work is a claim (`deep-dive` §4). The tree is the truth.

## 1. Locate

1. Which repo and lane? Use the user's words, then confirm on disk:
   ```bash
   git worktree list
   git -C <worktree> branch --show-current
   git -C <worktree> log --oneline -10 --decorate     # tags show checkpoints
   ```
2. If several lanes could match, list them and ask — do not pick one.

## 2. Inspect the state

```bash
git -C <worktree> status --short          # uncommitted edits — whose?
git -C <worktree> stash list              # do not pop someone else's stash
ls -t <notes dir> | head                  # newest plan / task note
```

- **Half-applied mutation?** Look for backup files and hash notes from a
  mutation run; compare hashes and restore before anything else.
- **Background processes** left by the old session (servers, test runs,
  watchers): find them by port or command, decide keep or stop. Do not print
  their full command lines if they may contain tokens.
- **Partial outputs**: background task logs, temp folders — read the tail to see
  where they stopped.

## 3. Rebuild the checklist

- Read the plan and task note; list each step with its proof.
- For each step marked done, re-check its proof cheaply (the commit exists, the
  test passes, the file contains the change). Mark unproven steps as not done.
- Resume from the first step that is not proven.

## 4. Continue

- Retake a test baseline if the tree moved since the last recorded numbers.
- Keep the task note updated as you go so the next drop costs less.
- Do not re-ask questions the user has already answered; do not redo verified work.

## 5. First message after resuming

Short and factual:

```
Resumed: <repo> · <branch> · last commit <hash> · last tag <tag>
Proven done: <steps>
Continuing with: <next step>
Found: <dirty files / stray processes / restored mutation — or nothing>
```

## Organisation, debt and scale

- Resume into the existing plan and note — never start a parallel file.
- Clean up what the dropped session left behind (temp files, stray processes,
  half-written edits) before continuing, and record what was cleaned.
