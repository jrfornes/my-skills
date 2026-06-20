---
name: bug-resolver
description: >-
  Develop station, bug path. Drive a reproduced defect to a verified fix:
  diagnose, patch the smallest change, and verify against the reproduction. Use
  when a `bug.md` exists at status `reproduced` (or `diagnosed`). Advances the
  `bug.md` status field, which is what lets a re-run after a context reset resume
  exactly where the last session stopped.
---

# bug-resolver — Diagnose → patch → verify

You take a reproduced bug and drive it to a verified fix. The `bug.md` `status`
field is the program counter: read it, do the next step, advance it, stop.

## Input

- A `bug.md` at `status: reproduced` (or `diagnosed`), carrying the issue key,
  the confirmed actual-vs-expected, and the `verification.method` / `command`.
- The regression `.feature` and `smoke/app-map/<module>/`.

## Status machine (you own this field)

| Current `status` | Do | New `status` |
|---|---|---|
| `reproduced` | Find the root cause; write it under a `## Diagnosis` heading | `diagnosed` |
| `diagnosed` | Apply the **smallest** code change that fixes the root cause | `fixed` (gate: human accepts the fix) |
| `fixed` | Run the verification (`verification.command`); confirm expected behavior | `verified` |

Stop after advancing one step if that step is a gate or the natural session
boundary. Never skip ahead.

## Steps

1. Read `bug.md`. Branch on `status` per the table.
2. **Diagnose:** trace from the symptom to the cause in app code (e.g.
   `apps/todo-list/src/app/...`). Record it in `bug.md` under `## Diagnosis`.
3. **Patch:** make the minimal change. Keep unrelated refactors out.
4. **Verify:** run the `verification.command` from `bug.md`. For this workspace
   that's typically the regression e2e or a unit test. Confirm it now passes.
5. Update `bug.md` `status` and stop.

## Output / marker

- Code patch in the working tree (issue-keyed; not committed — `ship-it` does that).
- `bug.md` `status` advanced to `diagnosed` / `fixed` / `verified`.

## Next station

After `verified` → `browser-hydrate` / `cypress-hydrate` to graduate the
regression test (tag the `.feature` `@regression`), then `ship-it`.

## Guardrails

- Propose, don't silently commit. The `diagnosed → fixed` transition is gated:
  surface the patch for a human to accept.
- Smallest change that makes the reproduction pass. No scope creep.
- You own `bug.md` and the code patch; you do not tag the `.feature` (that's
  hydrate) or push (that's `ship-it`).
