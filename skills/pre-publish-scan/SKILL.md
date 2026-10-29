---
name: pre-publish-scan
description: Screen content before it leaves the machine — a public or new repository, a shared archive, a portfolio copy, a pasted log or document. Finds credentials, key and env files, personal data, machine and user identifiers, absolute home paths, IPs, internal or banned names and terms, build output and history leaks; reports file names only and never the secret values; proves the scanner by planting decoys. Use before any first push, visibility change to public, export or hand-off of code or data.
---

# pre-publish-scan — nothing private leaves

Publishing is irreversible: a pushed secret is cached, forked and indexed even
after deletion. Rotate any real credential that was ever published — deleting it
is not enough.

## 1. Rules

- **Never print a matched value.** Report file + rule + line number only.
- Scan the exact content that will be published (the staged tree in a fresh
  copy), not the working folder with its ignored files.
- **History counts.** A clean current tree with a secret in an old commit is a
  leak. For a fresh publication, prefer a single new commit from a clean copy.
- Personal data in fixtures is replaced; structure stays.
- A finding is fixed at the source, then the whole scan re-runs.

## 2. Categories

| Category | Look for |
|---|---|
| Secret files | `.env*` (except `*.example`), `*.pem`, `*.key`, `*.p12`, `*.pfx`, `*.jks`, keystores, `id_*`, VPN/tunnel configs, database dumps, `secrets.*` |
| Credential-shaped strings | private-key headers, cloud/API key prefixes, tokens, connection strings with user:password, webhook secrets, bearer headers |
| Personal data | emails, phone numbers, real names, addresses, national/bank ids, avatars/screenshots of real accounts |
| Machine identity | usernames, hostnames, absolute home paths (`/Users/<name>`, `/home/<name>`, `C:\Users\<name>`) |
| Network | public IPs, internal hostnames, real domains of private systems |
| Banned names / terms | company, client, vendor, internal project names; any terms the user excluded — matched case-insensitively, in contents AND file paths, in every language the project uses |
| Build / dependency output | `node_modules`, `dist`, `build`, `vendor`, caches, coverage, `*.tsbuildinfo`, binaries |
| Documentation policy | markdown or docs the user did not allow to publish |

Keep the banned-terms list outside the published tree.

## 3. Method

1. Copy the tree to a temporary folder (excluding VCS and dependency folders),
   initialise a repository, and stage with the real ignore file.
2. Run the file-name rules on `git ls-files`.
3. Run the content rules with `git grep -lI -E -e '<pattern>'` — the `-e`
   stops a pattern that starts with `-` from being read as an option. Make the
   home-path rule case-sensitive so that lowercase routes do not match it.
4. Run a dedicated secret scanner too (the same one CI uses), so local and CI
   agree; mark genuine test fixtures inline and with an ignore entry for history.
5. **Prove the scanner:** plant one decoy per category in the temp copy (a fake
   key, a fake `.env`, a fake home path, a banned term) — each must be caught.
   Delete the decoys.
6. Images and binaries: list them and check each by eye or metadata.

## 4. Gate

Refuse to publish while any category has a finding. A publish script should
embed these checks and exit non-zero with the list of files.

## 5. Report

```
Scanned: <n> files from a clean copy · history: <single new commit | k commits scanned>
Findings: <category — file:line — rule>   (values never shown)
Decoys: <k>/<k> caught
Result: <clean — safe to publish | blocked>
```

## Organisation, debt and scale

- The scan is a reusable script with its rules and banned-term list as inputs,
  shared by every publication — not retyped per project.
- Rules are data (pattern tables), so adding a category is one entry.
- The same rules run locally and in CI, so the two never disagree.
