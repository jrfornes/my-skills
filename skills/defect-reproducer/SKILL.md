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

## Browser tool — Cursor browser tools (default)

Same as `smoke-validator`: reproduction is interactive and short-lived, so
default to Cursor's native browser tools. `nx run todo-list:serve`, then walk
the repro steps. Drop to the Playwright CLI when the repro needs to loop many
times, and the Playwright MCP only where there is no shell access.

## Steps

1. Start the app and walk the steps-to-reproduce.
2. Confirm the **actual** behavior differs from the **expected** behavior.
3. Write the `bug.md` artifact (contract below) with `status: reproduced` and
   the verification method you'll use to later confirm a fix.
4. Keep the `.feature` as the regression spec; leave its tag `@wip` for now
   (it becomes `@regression` after the fix is verified and hydrated).
5. If reproducing showed the bug needs a capability Cypress can't drive (e.g. a
   second browser tab/window), tag the `.feature` `@playwright-only` and note why
   in `bug.md` `## Notes`. That records "don't push a Cypress test for this" so
   Hydrate routes to `e2e-test-generator` only and `cypress-test-generator`
   no-ops.
6. Stop.

## Output / marker

- `bug.md` with `status: reproduced` (this is the marker the orchestrator reads).
- Location: alongside the flow, e.g. `smoke/flows/<module>/<...>/bug.md`.

## `bug.md` contract

The artifact's template, location, and status lifecycle live in the single
shared contract at [`../references/bug.md`](../references/bug.md). You produce it
with `status: reproduced` and own that initial state; later stations own the
later transitions. Don't redefine the schema here — follow the reference.

## Next station

`bug-resolver`.

## Guardrails

- You own `bug.md`. Do not patch code here — reproducing and diagnosing are
  different stations. If you cannot reproduce, set `status: blocked` with a note
  and stop; don't guess a fix.
