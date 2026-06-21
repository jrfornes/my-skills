---
name: pull-request-publisher
description: >-
  Ship station — the final, GATED step. Take a verified code change and turn it
  into a reviewable PR: create an issue-keyed branch, commit, open the pull
  request. Use when the unit is shippable (feature: `.feature` `@hydrated`; bug:
  `bug.md` regression-added + `.feature` `@hydrated/@regression`). The push is
  the one irreversible side effect, so propose and wait for human confirmation.
---

# pull-request-publisher — Branch, commit, open PR (gated)

You turn a verified change into an open PR. This is the only station that
performs an irreversible, state-advancing side effect, so it is **gated**:
propose the branch/commit/push and let a human confirm before you push.

## Preconditions (read the markers — don't trust the conversation)

- **Feature path:** the `.feature` is `@hydrated` and its hydrated tests pass.
- **Bug path:** `bug.md` is at `verified`/`regression-added` and the regression
  `.feature` is `@hydrated`/`@regression`.
- The working tree contains the code change for this issue key only.

## Steps

1. Confirm the unit is shippable by reading the artifacts' markers for the issue
   key. If a marker isn't where it should be, stop and surface the gap.
2. Create an issue-keyed branch: `<ISSUE-KEY>-slug` (e.g. `TODO-2-filter-completed`).
   This name carries the correlation key into git, closing the loop on artifact
   ownership.
3. Stage the relevant change and write a commit message that references the
   issue key and what changed.
4. **Gate:** present the branch name, the diff summary, and the proposed commit.
   Wait for explicit human approval before pushing.
5. On approval: push and open the PR. Link the PR back to the issue key and the
   `.feature` it satisfies.

## Output / marker

- An issue-keyed branch and an open PR (the terminal artifact).

## Guardrails

- Never push without the human gate. "Propose, don't silently commit."
- One issue key per branch/PR. Don't bundle unrelated changes.
- Don't advance any other artifact's marker here — shipping is the last step.
- If the repo's tooling can't open a PR (no remote access), stop at the commit
  and hand the human the exact `git push` + PR command instead of guessing.
