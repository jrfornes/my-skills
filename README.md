# my-skills

The skill library for the **agentic dev & testing workflow** — a sequence of
isolated stations that take an issue all the way to an open PR. Each skill does
one job: it reads a durable artifact, produces a durable artifact, and stops.
The artifact is the handoff, never the conversation.

These skills are consumed by the [`todo-list`](https://github.com/jrfornes/todo-list)
example workspace.

## Install

Skills are distributed via the [`skills`](https://github.com/vercel-labs/skills)
CLI. From a consuming repo:

```bash
# install all workflow skills into .claude/skills/
npx skills add jrfornes/my-skills

# or pick specific ones
npx skills add jrfornes/my-skills --skill spec-author --skill pull-request-publisher

# list what's available
npx skills add jrfornes/my-skills --list
```

The CLI scans `skills/<name>/SKILL.md` in this repo and copies each selected
skill into the consuming project's agent directory (`.claude/skills/` for
Claude Code).

## The stations

| # | Skill | Station | Reads | Writes | Marker |
|---|---|---|---|---|---|
| 0 | [`spec-author`](skills/spec-author/SKILL.md) | Intake | `issues/<KEY>.md` | `.feature` | `@wip` |
| 0f | [`feature-implementer`](skills/feature-implementer/SKILL.md) | Develop (feature) | `.feature` + `app-map` + rules | code change | `.feature` stays `@wip` |
| 1f | [`smoke-validator`](skills/smoke-validator/SKILL.md) | Validate (feature) | `.feature` + `app-map` | updated `app-map`, notes | `@smoke-passed` |
| 1b | [`defect-reproducer`](skills/defect-reproducer/SKILL.md) | Validate (bug) | `.feature` + `app-map` | `bug.md` | `reproduced` |
| 2b | [`bug-resolver`](skills/bug-resolver/SKILL.md) | Develop (bug) | `bug.md` | code patch | `diagnosed→fixed→verified` |
| 3c | [`cypress-test-generator`](skills/cypress-test-generator/SKILL.md) | Hydrate (Cypress) | `.feature` + `selectors.md` | page objects + `*.cy.ts` | `@hydrated` |
| 3p | [`e2e-test-generator`](skills/e2e-test-generator/SKILL.md) | Hydrate (Playwright/BDD) | `.feature` + `selectors.md` | pages + steps | `@hydrated` / `@regression` |
| 4 | [`pull-request-publisher`](skills/pull-request-publisher/SKILL.md) | Ship | verified code | branch + PR | open PR (gated) |

## Principles

- **Context discipline** — each station runs in a fresh session; the artifact
  crosses the boundary, not the chat history.
- **Artifact ownership** — each station owns exactly one artifact's state and
  never writes another's. The unit of work is *derived* from the artifacts
  (by issue key), not stored anywhere.
- **Gates stay human** — only state-advancing side effects (the `pull-request-publisher` push,
  accepting a fix) require confirmation.

## The two paths

```
Feature: spec-author → feature-implementer → smoke-validator → e2e-test-generator / cypress-test-generator → pull-request-publisher
Bug:     spec-author → defect-reproducer → bug-resolver → hydrate(@regression) → pull-request-publisher
```
