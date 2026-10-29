---
name: security-audit
description: Defensive security review and authorised vulnerability testing of systems you own — dependency and runtime advisories, secrets, authentication and authorisation, input handling, injection, SSRF, uploads, webhooks, sessions/cookies/CORS, rate limits, headers, logging of personal data, infrastructure exposure. Use for "check security", "harden", "is this safe?", before a release or a public launch, and after any change to auth, money, uploads or external calls. ALWAYS refreshes current advisories at run time (never relies on memory), tests only local or explicitly authorised targets, and keeps its own checklist up to date.
---

# security-audit — find it before someone else does

Last reviewed: <set when this skill is updated>   ·   Update log: §8

## 1. Scope and authorisation (first, every time)

- Test **only** systems the user owns or has explicitly authorised, and say
  which environment (local / staging / production) before any active test.
- Active tests run against the **local** stack by default. Staging/production
  needs explicit approval per run, with the exact actions listed.
- Never attack, scan or fuzz third-party services, never test denial of
  service, never exfiltrate real data, never keep found credentials.
- A real secret found anywhere is reported (file + rule, never the value) and
  must be rotated — deletion is not enough.

## 2. Refresh current threat data — mandatory, every run

Security knowledge goes stale in weeks. At the start of each audit:

1. List the stack with exact versions: runtimes, frameworks, base images,
   lockfile dependencies, databases, proxies.
2. Fetch **current** advisories for those exact versions — the language's audit
   tool (`npm/pnpm audit`, `pip-audit`, `cargo audit`, `govulncheck`, …),
   an OSV/GitHub advisory query, and the vendors' security-release pages
   (runtime, framework, proxy, database).
3. Record the date and sources checked in the report. If the network is not
   available, say the advisory check was NOT done — never fill it from memory.
4. Anything new and relevant that this checklist does not cover → add it to §4
   and log it in §8 (that is how the skill stays current).

## 3. Passive review (read the code and config)

Trace each entry point → auth → validation → business logic → storage/outbound.

## 4. Checklist

| Area | Check |
|---|---|
| Dependencies / runtime | known-vulnerable versions (from §2), pinned runtime and base image, lockfile committed and frozen in CI, unused dependencies removed |
| Secrets | none in code, history, images, logs or client bundles; loaded from env/secret store; encrypted at rest where stored; boot fails on missing/weak keys |
| Authentication | password hashing (modern KDF), login rate limits per IP and per account, lockout/backoff, session/token expiry and revocation, no tokens in URLs |
| Authorisation | every route guarded by default (deny-by-default), object-level checks (user A cannot read B's record by id), admin-only actions enforced on the server, scoped machine keys |
| Input | schema validation with an allow-list, size limits on bodies/uploads/arrays, safe type coercion, no mass assignment |
| Injection | parameterised queries / no operator injection in document stores, no shell/eval with input, template escaping, path traversal on file names |
| SSRF / outbound | user-supplied URLs restricted (scheme, host allow-list, no private ranges, no redirects to them), timeouts on every outbound call |
| Uploads / media | type detected from content, not extension, size cap, stored outside executable paths, served with safe content-type and no directory listing |
| Webhooks / callbacks | signature verified on the raw body with constant-time compare, replay protection (timestamp/id), never trust source IP alone |
| Sessions / browser | HttpOnly + Secure + SameSite cookies, CORS allow-list (no wildcard with credentials), CSRF protection where cookies authenticate, security headers (CSP, HSTS, frame, content-type options) |
| Abuse | rate limits on expensive and paid endpoints, cost caps on paid third-party calls, pagination bounds |
| Errors / logs | no stack traces or internals to clients, no secrets/personal data in logs, audit trail for privileged actions |
| Infrastructure | only the edge proxy exposes ports, databases/caches not public, admin surfaces behind auth/network rules, TLS everywhere public, origin restricted to the CDN/proxy where used, least-privilege service accounts |
| Supply chain / CI | secret scanning in CI, dependency audit in CI, actions/images pinned, no secrets printed in CI logs |

## 5. Active tests — local / authorised only

For each finding from §3–4 that can be exercised safely, prove it with a minimal
request: unauthenticated and cross-user access to protected routes, oversized
bodies, malformed and injection-shaped input, forged webhook signatures,
private-range URLs for SSRF, path-traversal file names. Record request shape
(redacted) and response. Stop at proof — no persistence, no data extraction.

## 6. Fix and prevent

- Fix at the shared layer (one guard, one validator, one outbound client) so the
  whole class is closed, not one instance.
- Add a regression test per finding that fails without the fix
  (`regression-guard`), and a CI check where possible (audit, secret scan).
- Security settings are configuration with safe defaults — secure when unset.
- Changes to auth, cookies, CORS or stored data are announced first
  (`ground-rules`).

## 7. Report

```
Scope: <env> · authorised: <by whom/when> · advisories checked: <date, sources | NOT done — why>
Findings (most severe first):
  [severity] <title> — <file:line | config> — <impact> — <proof> — <fix>
Fixed: <…>   Regression tests: <…>   Residual risk / accepted: <…>
Next review due: <date or trigger>
```

Severity: critical (remote takeover/data exposure), high, medium, low, info.

## 8. Keeping this skill current

- Re-run §2 at every audit; add any new relevant class of issue to §4.
- After each audit, append: `<date> — <what was added/changed in the checklist> — <source>`.
- Review this file at least monthly or on any major runtime/framework release;
  update "Last reviewed".

Update log:
- (empty — first entry on first audit)
