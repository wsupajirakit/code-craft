# Project conventions

Fill this in per project (or merge it into the project's agent-instructions file).
The neutral skills read these values instead of hard-coding them. Leave a line
empty if it does not apply — the skills will ask.

## Repository

- Protected branches (never edit directly):
- Integration branch for finished work:
- Lanes that must never be merged:
- Worktree naming pattern:
- Checkpoint tag pattern: `pre-<topic>-<yyyymmdd>` / `post-<topic>-<yyyymmdd>`
- Commit message language and attribution rules:

## Records

- Work log file (append-only):
- Task note folder and naming pattern:
- Plan file location:
- Cross-repo partner repositories (if work spans several):
- Language of in-repo text (comments, notes):
- Language of replies to the user:

## Running

- Local start command:
- Local URL(s) / entrances to verify (direct, proxy, tunnel):
- Health endpoint(s):
- Flags that must be set for local runs (for example: paid providers mocked):
- Build command; where the built artifact lands:

## Testing

- Quick suite command:
- Full suite command:
- Known pre-existing failures and why:
- Resource cap (CPU %, workers, priority):

## Data

- Databases / stores and which one local uses:
- Changes that always need approval first:
- Simulation / bypass markers to grep for before commits:

## Paid or outward-facing actions (always ask first)

- Paid APIs and their caps:
- Channels that reach real people:
- Environments that are shared or production:

## Code organisation

- File size limit and waiver list location:
- Folder conventions (feature/role grouping):
- Tag convention for marking changes in comments:
