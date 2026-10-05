---
name: ship
description: Use when a piece of work on a branch is finished and needs to go up for review — runs the project's lint, type-check, and tests, then design QA, PR check, security check, flow test, and docs sync in order, stops at the first failure or blocker, and otherwise opens the pull request with one combined report after a single confirmation. Never merges. Invoke for ship, ship it, open the PR, raise a pull request, ready for review, run all checks, pre-PR checks.
user-invocable: false
license: MIT
metadata:
  author: https://github.com/k-t18
  version: "1.0.0"
  domain: frontend
  triggers: ship, ship it, open the PR, raise a PR, pull request, ready for review, run all checks, pre-PR checks, combined report, gh pr create
  role: orchestrator
  scope: review
  output-format: report
  related-skills: functional-spec, web-component, feature-slice
---

# Ship (checks, combined report, pull request)

Runs every check the kit has on the current work branch, in a fixed order, and opens the pull request only when nothing blocks and the developer says yes.

## Role Definition

Release-minded senior frontend engineer for a web-first, then React Native, monorepo. Specialises in getting a finished piece of work to review with nothing hidden: every check either ran and has a result, or did not run and says why. Runs the other pieces of the kit rather than redoing their work, and treats the pull request as an outward-facing action that needs the developer's explicit yes. Read root `CLAUDE.md` first — §8 names the branches and the PR checklist this skill follows.

## When to Use This Skill

- A piece of work (`W-nn` of a spec, cut into an issue by `/agentic-dev-kit:spec-issues`) is built on its branch and is ready for review.
- A previous run stopped on a failure, the developer fixed it, and the checks need to run again.
- A pull request for the branch is already open and its report needs refreshing after new commits.

**Not for:** merging or releasing; writing the feature; running one check on its own (use that check's command); backend review; deciding whether a finding is acceptable — the developer and the reviewer decide that.

## Core Workflow

1. **Preflight** — work branch, working tree, issue + `W-` ID + spec, `gh`, base branch → `references/preflight.md`. A failed preflight stops the run before any stage.
2. **Run the stages in order** — lint · type-check · tests → design QA → PR check → security check → flow test (ask first) → docs sync → `references/stages.md`.
3. **Record every stage** — `passed`, `failed`, or `not run — <reason>`. Keep each stage's findings with their IDs.
4. **Stop on failure** — a failed lint, type-check, or test, any 🔴 Blocker, or a failed acceptance criterion halts the chain. Report and end; no PR.
5. **Handle fixes** — the fix-offers of PR check and security check apply. After a fix, ask before committing it, then rerun the affected stages.
6. **Build the combined report** → `references/combined-report.md`.
7. **Show the PR and ask once** — base, title, body. On yes, push and create it with `gh` → `references/pull-request.md`.
8. **Print the PR link and the summary.**

## Technical Guidelines

### The stages

| # | Stage | Run by | Stops the chain when |
| --- | --- | --- | --- |
| a | Lint · type-check · tests | The scripts the project defines | Any of them exits non-zero |
| b | Design QA | Agent `agentic-dev-kit:design-qa`, once per changed `packages/ui-web` component | Any 🔴 issue |
| c | PR check | `/agentic-dev-kit:pr-check` | Any 🔴 `P-nn` left unfixed |
| d | Security check | `/agentic-dev-kit:security-check` | Any 🔴 `X-nn` left unfixed |
| e | Flow test | `/agentic-dev-kit:flow-test` — asked for, never assumed | Any `AC-nn` result is Fail |
| f | Docs sync | `/agentic-dev-kit:docs-sync` | Never |

The order is fixed. A later stage does not start until the one before it has a result.

### The three results

| Result | Meaning |
| --- | --- |
| `passed` | The stage ran to the end and nothing in it stops the chain. It may still carry 🟡 and ⚪ findings. |
| `failed` | The stage ran and something in it stops the chain. |
| `not run — <reason>` | The stage did not run, or did not finish. The reason is always written. |

`not run` is never shown as `passed`, and never left blank. Reasons to use, word for word where they fit: `skipped by flag`, `declined`, `not installed`, `no script defined`, `nothing to check`, `no browser tools in this session`, `no Figma access`, `no spec`, `chain stopped at <stage>`.

### What stops the chain and what does not

| Stops | Does not stop |
| --- | --- |
| Lint, type-check, or tests exit non-zero | 🟡 and ⚪ findings from any stage |
| A 🔴 from design QA, PR check, or security check | A stage that is `not run` |
| An acceptance criterion the flow test marks Fail | An acceptance criterion marked Could not test |
| | Docs sync declined or incomplete |

Everything in the right-hand column goes in the report and in the PR body.

### Nothing is committed silently

The pull request contains committed work only. Three moments produce uncommitted changes: the working tree at the start, fixes made from a stage's fix-offer, and the files docs sync writes. At each one, show the changed files, propose a commit message in the repo's style, and commit only on a yes. Hooks always run.

### Reference Guide

| Topic | Reference | Load when |
| --- | --- | --- |
| Branch, working tree, issue and spec, `gh`, base branch | `references/preflight.md` | Starting a run |
| How each stage is run, read, and rerun | `references/stages.md` | Running the stages |
| The one report | `references/combined-report.md` | A run stops, or all stages are done |
| Title, body, confirmation, push, `gh pr create` | `references/pull-request.md` | All stages are clear |

## Constraints

### MUST DO

- Run preflight before any stage, and stop there if it fails.
- Use only scripts the project defines for lint, type-check, and tests; read them from `package.json` and the turbo config.
- Run the stages in the fixed order and give every one a result with its reason.
- Stop at the first failure or 🔴 Blocker, report what stopped the run, and open no PR.
- Ask before running the flow test, before every commit, and once before pushing and creating the PR.
- Keep every finding's own ID (`P-nn`, `X-nn`, `AC-nn`) in the report exactly as its stage gave it.
- Rerun the affected stages after any fix, and use the rerun's results in the report.
- List what a human must still verify: native behaviour on devices, sync and data-loss paths, and everything not run.
- Follow the project's PR template when one exists.

### MUST NOT DO

- Merge, enable auto-merge, or approve the pull request.
- Force-push, rebase a pushed branch, or push to `main`, `develop`, or the default branch.
- Skip hooks (`--no-verify`) or bypass commit signing.
- Commit, stash, or discard the developer's changes without asking.
- Invent a script name, or run an ad hoc substitute for a script the project does not define.
- Report a stage as passed when it did not run, or drop a `not run` item from the report.
- Reuse a stage result from an earlier run, or from before a fix.
- Fix a finding the developer did not choose, or downgrade a finding's severity.
- Open the pull request while any stage is `failed`.
- Tick a checklist box in the PR template that no stage proved.

## Output Templates

When shipping a branch, provide:

1. The preflight summary — branch, base branch, issue, spec path, `W-` ID, the IDs it covers.
2. One line per stage as it finishes — the stage and its result.
3. The combined report (`references/combined-report.md`) — on a stop, as far as the run got, with what stopped it and the next step.
4. Before creating the PR: the base branch, the title, and the full body, with one yes/no question.
5. After creating it: the PR link and a summary — stage results, counts of 🔴 / 🟡 / ⚪, the not-run list, and what a human must still verify.

## Knowledge Reference

pull request, gh CLI, gh pr create, conventional commits, branch strategy, develop, main, feat/fix/chore/migration branches, lint, type-check, tests, Turborepo, pnpm, design QA, Figma, PR check, security check, flow test, acceptance criteria, docs sync, functional spec, W- ID, combined report, blocker, PR template, human verification, native behaviour, offline sync
