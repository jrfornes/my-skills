---
name: e2e-test-generator
description: >-
  Hydrate station, Playwright/BDD stack (Tier 2). Turn a validated `.feature`
  into executable Playwright tests via playwright-bdd: page objects from
  selectors.md, step definitions, then `bddgen && playwright test` green. Use
  when a `.feature` is `@smoke-passed` (feature) or its fix is `verified` (bug)
  and you want it in the BDD E2E project.
---

# e2e-test-generator — `.feature` → Playwright/BDD tests

You compile a validated scenario into runnable Playwright tests. Because
playwright-bdd compiles the **same** `.feature` files the agent explored, the
spec of record and the executable test never drift.

## Input

- A `.feature` at `@smoke-passed` (feature) or backed by a `verified` `bug.md`
  (bug), living under `smoke/flows/...`.
- `smoke/app-map/<module>/selectors.md` — the page-object contract.

## Output

In the BDD project `apps/app-bdd-e2e/`:

- A page object `pages/<module>.page.ts` from `selectors.md`.
- Step definitions `steps/<module>-<slug>.steps.ts` (+ shared `steps/common.steps.ts`).
- Page objects wired into `fixtures.ts` so steps receive them.

The `.feature` files themselves stay in `smoke/flows/`; the BDD config points
`defineBddConfig.featuresRoot`/`features` there.

## Steps (the workflow)

1. **Read `selectors.md`** — it is the source for the page object.
2. **Generate page objects** — one class per module, methods for each action and
   assertion the scenarios need.
3. **Generate step definitions** — implement each `Given/When/Then` using the
   page object (via the `fixtures.ts` fixtures). Reuse existing steps; only add
   what's new.
4. **Tag the `.feature`** `@hydrated` (feature) or `@regression` (bug). Add
   `@smoke` to scenarios that belong in the fast smoke subset.
5. **Confirm green locally:** `bddgen && playwright test` (i.e.
   `nx run app-bdd-e2e:e2e`).
6. **Bug path only:** once the test is green, advance the `bug.md` `status` to
   `regression-added` (per the shared contract at
   [`../references/bug.md`](../references/bug.md)). This is the terminal marker
   `pull-request-publisher` reads to confirm the bug is shippable. Stop.

## Memory-leak profiles

For looping/leak workflows, add a **plain** Playwright spec under `profiles/`
(not BDD) that loops the flow and samples the JS heap via CDP
(`Performance.getMetrics`). Run with `nx run app-bdd-e2e:e2e -- --project=leak`.
This replaces the old Playwright-MCP looping sessions that hung.

## Next station

`pull-request-publisher`.

## Blocked / no progress

If `bddgen` reports undefined steps or the test won't go green, stop **without**
tagging `@hydrated`/`@regression` (and without advancing `bug.md`) — an
un-advanced marker tells the orchestrator hydration didn't complete. Surface the
failing step or assertion rather than reword the scenario to make it pass.

## Guardrails

- Step text comes from the `.feature`; don't reword scenarios to fit code —
  extend the steps instead.
- Selectors only from `selectors.md`; add new ones to the component and that file.
- `bddgen` must pass with **no undefined steps** before you tag `@hydrated`.
- You own the `.feature` hydration tag. The one cross-artifact exception is the
  bug path's terminal `bug.md` `status: regression-added`, which you set only
  after the regression test is green — never any other `bug.md` field.
