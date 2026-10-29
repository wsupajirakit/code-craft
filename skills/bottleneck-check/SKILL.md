---
name: bottleneck-check
description: Find and remove performance bottlenecks by measurement — slow endpoints or pages, high CPU/memory, slow queries, queue backlogs, large bundles, timeouts under load. Use when something is slow, when asked "where is the bottleneck?", before scaling out, or before/after a performance change. Measure first, locate the single biggest cost, fix it at the right layer, and prove the gain with before/after numbers — on systems you own, inside a resource cap.
---

# bottleneck-check — measure, locate, fix one thing, re-measure

Optimising without a measurement moves cost around and adds debt.

## 1. Rules

- **Own systems only.** Load tests run against local or explicitly approved
  environments — never against third-party services or production without
  written approval.
- **Resource cap.** Respect the project's cap (default: total CPU under ~75%);
  run load generators at low priority and measure the machine during the run.
- **One change at a time**, each with its own before/after number.
- **No premature optimisation.** If the measured cost is not the biggest one,
  leave it.

## 2. Define the symptom as a number

```
Metric:   <p50/p95 latency | throughput | CPU % | memory | query time | bundle KB | backlog>
Where:    <endpoint / page / job / query>
Load:     <requests/s, concurrency, data size>
Target:   <the number that means "fixed">
```

## 3. Locate — top-down

| Layer | Measure with | Typical bottleneck |
|---|---|---|
| Request timeline | server timing, structured logs with durations, tracing | one hop dominating total time |
| Database | query explain plan, slow-query log, index stats | missing/unused index, full scan, N+1, unbounded result, lock contention |
| External calls | per-call duration logs | sequential calls that could be parallel, no timeout, no cache, retry storms |
| CPU | profiler / flame graph | hot loop, sync crypto/compression, heavy serialisation, regex backtracking |
| Memory | heap snapshot, RSS over time | leak (growing caches, listeners), large payloads held in memory |
| Concurrency | queue depth, pool usage, event-loop lag | pool too small, single worker, blocking call on an async runtime |
| Network / payload | response sizes, compression | over-fetching, missing pagination, uncompressed responses |
| Client | bundle analyser, render profiler, network waterfall | large bundle, duplicate requests, re-render storms, polling hidden tabs |

Stop at the first layer that explains most of the time. Write down the evidence
(file:line, plan output, profile frame).

## 4. Fix at the right layer

- Prefer structural fixes: an index, batching, pagination, a bounded cache with
  invalidation, moving work to a queue, parallelising independent calls,
  streaming instead of buffering.
- Caches must have a bound, a TTL or invalidation, and a story for multiple
  instances (a per-process cache is stale on other instances).
- Every limit (pool size, batch size, timeout, TTL) is configuration with a
  sensible default — not a literal buried in code.
- Changing an index, schema or stored shape is a data change — announce first
  (`ground-rules` §2).

## 5. Prove it

- Same load, same data, same machine: before vs after (`baseline`).
- Behaviour unchanged: suites green (`regression-guard`).
- Report the gain in the metric you defined — and any cost (memory, staleness,
  complexity).

## 6. Organisation, debt and scale

- Fixes live in the layer that owns the cost (query in the repository layer,
  caching in a cache module) — never as scattered ad-hoc patches.
- No magic numbers; tunables are named config.
- State the scaling assumption the fix makes (single instance? shared cache?)
  and what breaks first at 10× load.
- Record what was not fixed and why, as a known item — not silent debt.

## 7. Report

```
Symptom: <metric> = <before> at <load>   Target: <x>
Bottleneck: <layer> — <evidence file:line / plan / profile>
Fix: <what, where>
After: <metric> = <after> at <same load>   (Δ <%>)
Cost / trade-off: <…>   Next bottleneck: <…>
```
