---
name: probe
description: Observe a RUNNING system to get runtime evidence instead of inferring from code — authenticated endpoint calls, reading real records, tailing logs, temporary stage traces behind an off-by-default flag. Use when a claim depends on what actually happens (does the handler run, which branch fires, what lands in storage), when debugging a live/local stack, or when another skill needs proof. Read-only by default; anything that costs money, sends real messages or touches production needs explicit approval first.
---

# probe — look at the running thing

Code says what should happen. A probe shows what did.

## 1. Safety rules (non-negotiable)

- **Read-only by default.** GET, SELECT/find, log reads, health checks.
- **Ask first, every time, naming target + count + cost** before anything that:
  spends money (paid APIs, AI calls, SMS), sends a real message to a real person,
  writes to shared or production data, or calls a third party's server.
  A previous "yes" does not carry over to a new run.
- **Never** attack, fuzz, load-test or scan a server you do not own or have not
  been explicitly authorised to test.
- **Never print secrets.** Print lengths, prefixes of ≤4 chars, or "set/empty".
  Pass tokens via env or files, not on a command line that lands in history or
  `ps` output.
- Prefer the local stack. If the target is shared, say so before probing.

## 2. Probe ladder — use the cheapest that can answer

| Level | Tool | Answers |
|---|---|---|
| 1. liveness | health endpoint, port check | is it up, on which port (read the port from the process, not from intent) |
| 2. authenticated call | real request with a real (test) credential | does the HANDLER run and return the right shape — a 401/403 only proves the guard |
| 3. storage | one record read by a narrow filter | did the write land, with which keys and shapes |
| 4. logs | grep the running process's log | which branch fired, which error, in which order |
| 5. stage trace | temporary trace points behind a flag | where exactly a flow stops |

## 3. Temporary trace points

When levels 1–4 cannot locate the problem:

- Guard every trace point with an env flag that is **off by default**; the
  production path must be byte-identical when the flag is unset.
- Emit one structured line per stage: `stage=<name> ok=<bool> key=<non-secret id>`.
- Never log payload bodies, tokens or personal data.
- If the trace bypasses a check or fakes a result, it is a local patch only —
  never commit it (`ground-rules` §5).
- Remove trace points that were only for this investigation, or document the
  flag if they are worth keeping.

## 4. Stability — one probe is not evidence

A process can answer once and die a minute later, or be healthy on the wrong
port. When the claim is "it is running":

```bash
PID0=$(<find the listening pid>)
for i in 1 2 3 4 5; do
  sleep 20
  echo "$((i*20))s pid=$(<pid> || echo DEAD) direct=$(<status code>) proxy=$(<status code>)"
done
```

Up = the same pid on every sample AND every entrance the user will use (direct
port, reverse proxy, tunnel) answering on every sample.

## 5. Report

```
Target: <local|shared> <service> — approved: <yes/n.a.>
Probes: <what was called / read> × <count>
Observed: <status codes, record shapes, log lines (redacted)>
Cost incurred: <none | N paid calls, counter now at M>
Conclusion: <what this proves — and what it does not>
```

## Organisation, debt and scale

- Reusable probes become a small script in the project's tools folder with
  its target and flags as arguments — not one-off commands pasted again each time.
- Trace flags are named, documented config; no hard-coded hosts, ports or ids.
- Remove what was only for this investigation; no leftover debug output.
