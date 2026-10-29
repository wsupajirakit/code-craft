---
name: additive-change
description: Add something optional to an existing system without changing existing behaviour — a new field/column/key, option, flag, config value, endpoint parameter or switchable feature. Use whenever the request is "add X but don't affect what exists", "optional", "can be turned on/off", or a new field must be threaded through forms, APIs and storage. Covers optional-by-default design, threading the field through every layer that silently drops unknown fields, the full write→read→re-save round trip, proof that the off path is unchanged, and an explicit revert path.
---

# additive-change — new things in, old things untouched

The rule: **when the new thing is not set, the system behaves exactly as before.**
Everything below exists to prove that sentence.

## 1. Design it optional

- Default = absent / empty / off, and that default yields the old behaviour
  byte-for-byte.
- Existing records without the field must read and write correctly, with no
  migration.
- Do not rename, repurpose or re-type anything that exists. Do not change values
  that existing records hold (enums, status codes).
- If any of that is unavoidable, it is no longer additive — announce it first
  (`ground-rules` §2) with before/after, effect on existing records,
  compatibility and revert path.
- A switch that disappears with its flag makes "is it installed?" unanswerable.
  Prefer a switch that is always registered, refuses when off, and makes the
  refusal observable.

## 2. Thread it through every layer

A field can be saved perfectly and still be lost on the way back. Find every layer
by comparison, not by memory:

```bash
# every file that knows a sibling field which already works end to end…
grep -rln "<sibling_field>" <src dirs> | sort > /tmp/sibling.txt
# …versus every file that knows the new one
grep -rln "<new_field>"     <src dirs> | sort > /tmp/new.txt
comm -23 /tmp/sibling.txt /tmp/new.txt     # each line = a probable drop point
```

Run it in every repo the data passes through (client and server).

Layers that drop fields silently:

| Layer | How it drops | Symptom |
|---|---|---|
| request validation with an allow-list | strips unknown properties | request "accepted", nothing saved |
| update handler copying field by field | new field not copied | create works, edit ignores it |
| update allow-lists | field not listed | 200 OK, value unchanged |
| response mapper / serializer | builds a new object from listed fields | API never returns it |
| client normaliser | builds a new object field by field | form opens blank — next save erases the value |
| clone / export / import / copy paths | copy a fixed field list | copies lose the setting |

## 3. Prove the round trip

Through the real entry point (UI or HTTP), not by calling the service directly:

1. write a value → 2. read it back → 3. reopen the form/view → it shows the
value → 4. save again without touching it → the value survives →
5. clear it → it returns to the default behaviour.

Calling a service method directly skips validation, mappers and the client —
exactly where fields disappear. It proves only that storage can hold the value.

## 4. Prove the off path is unchanged

- Capture a `baseline` with the feature unset, on the unchanged code.
- Apply the change; re-run with the feature still unset → **identical**.
- Turn it on → the intended difference appears where it should, and nowhere else.
- Add a test that fails without the new code (red first) and a mutation check
  (`regression-guard`).

## 5. Revert path

Document both:

- **switch off:** which value/flag to clear — and that clearing it restores the
  old behaviour (proven in §4);
- **remove:** which code and which files to delete — and whether data already
  written can be cleaned up or is permanent.

## 6. Checklist

```
[ ] default = old behaviour, existing records need no migration
[ ] sibling-field diff run in every repo — no unexplained drop points
[ ] round trip through the real entry point incl. re-save
[ ] baseline: off path identical before/after
[ ] red-first test + mutation
[ ] revert: switch-off and removal documented
```

## Organisation, debt and scale

- The new option lives in the module that owns the concern, with its own
  small unit if it carries logic — never wedged into an unrelated file.
- Variation goes through config or a registry entry, so the next option of the
  same kind is one more entry, not another branch.
- Multi-instance safe from the start: no per-process state for the new feature.
