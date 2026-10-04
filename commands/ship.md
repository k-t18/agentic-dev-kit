---
name: ship
description: Run the pre-PR checks in order on the current work branch (lint, type-check, tests, design QA, PR check, security check, flow test, docs sync) and, if nothing blocks, open the pull request with one combined report after a single confirmation. Runs the ship skill. Never merges.
argument-hint: "[issue number | W-nn | path to spec.md] [--skip=design-qa,pr-check,security-check,flow-test,docs-sync]"
---

# /ship

**Arguments:** $ARGUMENTS

Runs the **`ship`** skill in the main conversation. It runs the kit's checks one after
another on the current branch, stops at the first thing that blocks, and — only when
nothing blocks and the developer confirms — pushes the branch and opens the pull request
with `gh`. It never merges. The merge is the human's step.

## What to do

1. **Parse `$ARGUMENTS`:**
   - A number or `#<number>` → the GitHub issue this branch closes.
   - `W-nn` → the piece of work; the spec still has to be found (branch name, issue, or
     ask).
   - A path to a `spec.md` → the spec; the `W-` ID still has to be found (ask).
   - `--skip=<stage>[,<stage>]` → stages to leave out. Allowed: `design-qa`, `pr-check`,
     `security-check`, `flow-test`, `docs-sync`. Each is recorded as
     `not run — skipped by flag`. Lint, type-check, and tests cannot be skipped; an
     unknown stage name is an error — say so and stop.
   - Empty → work the issue out from the branch name, or ask.

2. **Preflight** per `ship` → `references/preflight.md`: work branch, working tree,
   issue + `W-` ID + spec, `gh`, base branch. Print the preflight summary. Anything
   that fails here stops the run before any stage starts.

3. **Run the stages in order** per `references/stages.md`:
   lint · type-check · tests → design QA → PR check → security check → flow test →
   docs sync.
   - Design QA: spawn the agent (Agent tool, `subagent_type: agentic-dev-kit:design-qa`)
     once per changed `packages/ui-web` component. The agent is namespaced under the
     plugin — the bare `design-qa` will not resolve once installed.
   - PR check, security check, flow test, docs sync: run
     `/agentic-dev-kit:pr-check`, `/agentic-dev-kit:security-check`,
     `/agentic-dev-kit:flow-test`, `/agentic-dev-kit:docs-sync` as they are. Their own
     questions and fix-offers still apply.
   - **Ask before the flow test** — it needs the running app and the developer's
     browser. Declined → `not run — declined`.
   - A stage that is not installed or cannot run is `not run` with the reason. Never
     count it as passed.

4. **Stop on failure.** A failed lint, type-check, or test, a 🔴 Blocker from any stage,
   or a failed acceptance criterion halts the chain. Print the combined report as far as
   it got, say what stopped it, and do not open a PR. The developer fixes it and runs
   `/agentic-dev-kit:ship` again.

5. **Build the combined report** per `references/combined-report.md`.

6. **Show the pull request that would be opened** per `references/pull-request.md` —
   base branch, title, body — and ask for **one** confirmation. On yes: push the branch
   and create the PR with `gh`. On no: change nothing outside the working copy.

7. **Print the PR link and the summary.**

## Notes

- Nothing is committed without asking. Fixes and docs-sync edits made during the run
  are shown and committed only when the developer agrees; the PR contains committed
  work only.
- 🟡 and ⚪ findings do not stop the run. They go in the report and the PR body.
- Re-running is the normal way to continue after a stop. Every stage runs again from
  the start — a result from an earlier run is never reused.
- It never merges, enables auto-merge, force-pushes, skips hooks, or pushes to `main`
  or `develop`.
