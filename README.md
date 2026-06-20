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
npx skills add jrfornes/my-skills --skill gherkin-me --skill ship-it

# list what's available
npx skills add jrfornes/my-skills --list
```

The CLI scans `skills/<name>/SKILL.md` in this repo and copies each selected
skill into the consuming project's agent directory (`.claude/skills/` for
Claude Code).

## The stations

| # | Skill | Station | Reads | Writes | Marker |
|---|---|---|---|---|---|
| 0 | [`gherkin-me`](skills/gherkin-me/SKILL.md) | Intake | `issues/<KEY>.md` | `.feature` | `@wip` |
| 1f | [`browser-smoke`](skills/browser-smoke/SKILL.md) | Validate (feature) | `.feature` + `app-map` | updated `app-map`, notes | `@smoke-passed` |
| 1b | [`browser-repro`](skills/browser-repro/SKILL.md) | Validate (bug) | `.feature` + `app-map` | `bug.md` | `reproduced` |
| 2b | [`bug-resolver`](skills/bug-resolver/SKILL.md) | Develop (bug) | `bug.md` | code patch | `diagnosed→fixed→verified` |
| 3c | [`cypress-hydrate`](skills/cypress-hydrate/SKILL.md) | Hydrate (Cypress) | `.feature` + `selectors.md` | page objects + `*.cy.ts` | `@hydrated` |
| 3p | [`browser-hydrate`](skills/browser-hydrate/SKILL.md) | Hydrate (Playwright/BDD) | `.feature` + `selectors.md` | pages + steps | `@hydrated` / `@regression` |
| 4 | [`ship-it`](skills/ship-it/SKILL.md) | Ship | verified code | branch + PR | open PR (gated) |

## Principles

- **Context discipline** — each station runs in a fresh session; the artifact
  crosses the boundary, not the chat history.
- **Artifact ownership** — each station owns exactly one artifact's state and
  never writes another's. The unit of work is *derived* from the artifacts
  (by issue key), not stored anywhere.
- **Gates stay human** — only state-advancing side effects (the `ship-it` push,
  accepting a fix) require confirmation.

## The two paths

```
Feature: gherkin-me → browser-smoke → browser/cypress-hydrate → ship-it
Bug:     gherkin-me → browser-repro → bug-resolver → hydrate(@regression) → ship-it
```
