---
name: smoke-validator
description: >-
  Validate station, feature path (Tier 1). Drive a real browser to exercise a
  new flow against its `.feature` spec — happy path plus key edge cases. Use
  after code exists for a feature and its `.feature` is `@wip`. Confirms the
  behavior and promotes the spec to `@smoke-passed`, updating the app-map as you
  learn the UI. Cheapest path for one-off verification.
---

# smoke-validator — Validate the feature path

You exercise a new or changed flow in a real browser to confirm it behaves as
the `.feature` says, and you record what you learn so the next station doesn't
re-derive it.

## Input

- A `.feature` file tagged `@wip` under `smoke/flows/<module>/...`.
- `smoke/app-map/<module>/` (`navigation.md`, `routes.md`, `selectors.md`).
- **Never** raw issue prose — you work off the compact `.feature` + app-map.

## Browser tool — Cursor browser tools (default)

Smoke is interactive and short-lived, so default to Cursor's native browser
tools — they have no MCP tool-definition overhead and only surface the output
relevant to the current step, which is the cheapest option for one-off
verification.

Drop to the **Playwright CLI** (driven through shell) when a flow must loop many
times — it writes snapshots/screenshots to disk as compact YAML with element
refs instead of streaming the accessibility tree into context. Reach for the
Playwright MCP only where there is no shell access.

Either way, start the app first:

```bash
nx run todo-list:serve &          # app on http://localhost:4200
# then drive the browser, asserting each scenario
```

## Steps

1. Start the app (`nx run todo-list:serve`).
2. For each `Scenario` in the `.feature`, perform the steps in the browser using
   selectors from `selectors.md`. Check the happy path and the obvious edge
   cases (empty input, boundary counts).
3. If a selector or nav detail is wrong or missing, **fix `app-map`** — that is
   the artifact you own. Add durable observations to `smoke/smoke-notes.md`.
4. If every scenario passes, promote the spec: change the tag `@wip` →
   `@smoke-passed`. This is the persisted marker that says "ready to hydrate".
5. Stop.

## Output / marker

- Updated `app-map` + `smoke-notes.md`.
- `.feature` tag advanced `@wip` → `@smoke-passed`.

## Next station

`e2e-test-generator` (Playwright/BDD) and/or `cypress-test-generator`.

## Blocked / no progress

If a scenario fails, stop **without** advancing the tag — leaving the marker
where it is signals "not validated" to the orchestrator. A failing scenario is a
defect, not a smoke pass: surface it (a note in `smoke/smoke-notes.md`) so it can
move to the bug path. If you can't run the flow at all (app won't serve, missing
selectors), say so and stop rather than guessing.

## Guardrails

- You own the `app-map` and the smoke tag, nothing else. Don't touch app code
  here — if a scenario fails, that's a defect: stop and surface it (it may
  belong on the bug path).
