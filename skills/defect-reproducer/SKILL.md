---
name: defect-reproducer
description: >-
  Validate station, bug path (Tier 1). Reproduce a reported defect in a real
  browser and lock it in as a `bug.md` artifact + a regression `.feature`. Use
  when the issue is a bug (type: bug) and its `.feature` is `@wip`. A confirmed
  reproduction is the gate that lets bug-resolver start from a known-failing
  state.
---

# defect-reproducer — Reproduce and lock a bug

You read the scenario as a description of a failure, reproduce it in a real
browser, and capture the result as a durable `bug.md` so a fresh session can
pick up exactly where you stopped.

## Input

- A `.feature` file tagged `@wip` describing the bug, under `smoke/flows/...`.
- `smoke/app-map/<module>/` for selectors and navigation.
- The source issue's "steps to reproduce" (already distilled into the `.feature`).

## Browser tool — Playwright CLI (not the MCP)

Same as `smoke-validator`: drive Playwright through the shell for token
efficiency. `nx run todo-list:serve`, then walk the repro steps.

## Steps

1. Start the app and walk the steps-to-reproduce.
2. Confirm the **actual** behavior differs from the **expected** behavior.
3. Write the `bug.md` artifact (contract below) with `status: reproduced` and
   the verification method you'll use to later confirm a fix.
4. Keep the `.feature` as the regression spec; leave its tag `@wip` for now
   (it becomes `@regression` after the fix is verified and hydrated).
5. Stop.

## Output / marker

- `bug.md` with `status: reproduced` (this is the marker the orchestrator reads).
- Location: alongside the flow, e.g. `smoke/flows/<module>/<...>/bug.md`.

## `bug.md` contract

```markdown
---
key: TODO-2                      # issue / correlation key — REQUIRED
status: reproduced               # reproduced → diagnosed → fixed → verified → regression-added
verification:
  method: e2e                    # how a fix will be confirmed (e2e | unit | manual)
  command: "nx run app-bdd-e2e:e2e -- --grep @regression"
---

# TODO-2 — <title>

## Expected
<what should happen>

## Actual
<what happens instead — the confirmed reproduction>

## Repro steps
1. ...

## Notes
<selectors used, environment, anything durable>
```

## Next station

`bug-resolver`.

## Guardrails

- You own `bug.md`. Do not patch code here — reproducing and diagnosing are
  different stations. If you cannot reproduce, set `status: blocked` with a note
  and stop; don't guess a fix.
