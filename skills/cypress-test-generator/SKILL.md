---
name: cypress-test-generator
description: >-
  Hydrate station, Cypress stack (Tier 2). Turn a validated `.feature` into an
  executable Cypress spec plus a page object, targeting the selectors in the
  app-map. Use when a `.feature` is `@smoke-passed` (feature) or its fix is
  `verified` (bug) and you want it covered in the existing Cypress E2E project.
---

# cypress-test-generator — `.feature` → Cypress spec

You turn a validated scenario into a repeatable Cypress test that runs in CI.

## Input

- A `.feature` at `@smoke-passed` (feature path) or backed by a `verified`
  `bug.md` (bug path).
- `smoke/app-map/<module>/selectors.md` — the page-object contract.

## Output

In the Cypress project `apps/todo-list-e2e/`:

- A page object under `src/support/page-objects/<module>.po.ts` (created or
  extended) using **only** selectors from `selectors.md`.
- A spec `src/e2e/<module>-<slug>.cy.ts` mirroring each `Scenario`.

Then tag the source `.feature` `@hydrated` (feature) or `@regression` (bug).

## Steps (five-step workflow)

1. Read `selectors.md`. Add any missing handles to the page object — never use
   ad-hoc CSS in the spec.
2. Generate/extend the page object methods the scenarios need (add, toggle,
   delete, filter, asserts).
3. Generate the `*.cy.ts` spec, one `it()` per `Scenario`, calling the page
   object. Add a `beforeEach` that resets state (`cy.clearLocalStorage()`).
4. Tag the `.feature`: `@smoke-passed` → `@hydrated`, or add `@regression` on
   the bug path.
5. Confirm it passes locally: `nx run todo-list-e2e:e2e` (or `:e2e-ci`). Stop.

## Next station

`pull-request-publisher`.

## Guardrails

- Selectors live in `selectors.md`; the page object is the only place that knows
  them. If you need a new selector, add it to the component **and** `selectors.md`.
- The generated test's comments should trace back to the `.feature` it came from.
- One `.feature` → one spec. Don't fold unrelated scenarios together.
