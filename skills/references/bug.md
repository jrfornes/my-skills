# `bug.md` artifact contract

The shared, durable artifact for the bug path. It is the handoff between
`defect-reproducer`, `bug-resolver`, and the hydrators — the `status` field is
the program counter that lets a fresh session resume exactly where the last one
stopped. Each station owns the field transitions noted below; no station writes a
field it doesn't own.

## Location

Alongside the flow it belongs to, e.g. `smoke/flows/<module>/<...>/bug.md`. The
`key` front-matter is the correlation key the orchestrator globs on.

## Template

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

## Diagnosis
<root cause — added by bug-resolver at the `diagnosed` step>

## Notes
<selectors used, environment, anything durable>
```

## Status lifecycle and ownership

| `status`           | Set by                                          | Means                                              |
| ------------------ | ----------------------------------------------- | -------------------------------------------------- |
| `reproduced`       | `defect-reproducer`                             | Failure confirmed in a browser; ready to diagnose  |
| `diagnosed`        | `bug-resolver`                                  | Root cause recorded under `## Diagnosis`           |
| `fixed`            | `bug-resolver` (gated: human accepts the patch) | Minimal patch applied                              |
| `verified`         | `bug-resolver`                                  | `verification.command` passes                      |
| `regression-added` | `e2e-test-generator` / `cypress-test-generator` | Regression test green; the unit is shippable       |
| `blocked`          | any station                                     | Can't proceed; carries a note. Orchestrator stops. |
