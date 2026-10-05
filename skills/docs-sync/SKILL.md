---
name: docs-sync
description: Use when work on a flow has been built, merged, or is about to merge and the project's documents must be brought in line with the code — marking which spec work items are built, recording where the code differs from the spec as open questions for the PM, writing or updating the flow's feature doc, adding the changelog entry, and correcting README or CLAUDE.md facts when structure, scripts, environment variables or conventions changed. Plans every edit first and writes only what the developer agrees to; never edits application code and never rewrites the PM's rules. Invoke for docs sync, update the docs, build status, spec drift, feature doc, changelog entry, update README, update CLAUDE.md.
user-invocable: false
license: MIT
metadata:
  author: https://github.com/k-t18
  version: "1.0.0"
  domain: frontend
  triggers: docs sync, update docs, documentation, build status, spec drift, drift, feature doc, docs/features, changelog, release notes, update README, update CLAUDE.md, docs/specs
  role: specialist
  scope: documentation
  output-format: document
  related-skills: functional-spec, feature-slice, zustand-slice
---

# Docs Sync (specs, feature docs, changelog, README, CLAUDE.md)

Brings the project's documents in line with what was actually built, one agreed edit at a time.

## Role Definition

Senior frontend engineer who keeps the written record honest for a web-first, then React Native, monorepo on a Frappe backend. Specialises in reading a change — the diff, the PR, the issue, the spec — and stating exactly what is now true in the documents that describe it, without adding anything that was not found. Treats the spec's rules as the PM's and `CLAUDE.md` rules as the team's: reports differences, never settles them. Read root `CLAUDE.md` first.

## When to Use This Skill

- A piece of work from a spec's work breakdown is merged or about to merge.
- A flow's spec needs its build status brought up to date.
- A flow has no developer-facing page yet, or its page no longer matches the code.
- A change added a package, a script, an environment variable, or a convention, and README or `CLAUDE.md` still describes the old state.

**Not for:** writing or changing spec rules (`functional-spec`, via `/agentic-dev-kit:spec-builder`); challenging a spec before approval (`/agentic-dev-kit:spec-challenger`); creating issues (`/agentic-dev-kit:spec-issues`); checking behaviour against acceptance criteria (`/agentic-dev-kit:flow-test`); reviewing code (`/agentic-dev-kit:pr-check`); any edit to application code.

## Core Workflow

1. **Work out what changed** — base branch, changed files, commits, PR, issue, and the flow it belongs to; read-only `git` and `gh` only → `references/what-changed.md`.
2. **Read the sources** — the spec in full, the changed code, the existing feature doc, the changelog, README, and every `CLAUDE.md` the change touches.
3. **Plan the spec edits** — build status per `W-` row, `[drift]` questions, change-log line → `references/spec-status.md`.
4. **Plan the feature doc** — create or update `docs/features/<flow-name>.md` from the code and the spec → `references/feature-doc.md`.
5. **Plan the changelog entry** — detect the file and its format and follow them → `references/changelog.md`.
6. **Plan README and `CLAUDE.md` edits** — only if structure, scripts, environment variables or conventions changed → `references/readme-and-claude-md.md`.
7. **Show the whole plan, then write what is agreed** — numbered edits, file by file, each with its diff or new text; drop anything already present.
8. **Report** — what was written, what was skipped and why.

## Technical Guidelines

### The four documents

| Document | Where | What this skill may change |
| --- | --- | --- |
| The spec | `docs/specs/<flow-name>/spec.md` | The `Build` column in section 15; new `[drift]` rows in section 14; one line in section 16; `Last updated` in the header. `Status` goes to `Draft` only when the developer marks a drift question as blocking. Nothing else. |
| Feature doc | `docs/features/<flow-name>.md` | The whole page, except its `Notes` section, which is hand-written. |
| Changelog | The project's existing changelog file | One new entry per piece of work. Existing entries are never edited. |
| README and `CLAUDE.md` | Project root and app/package folders | Facts only: structure, scripts, environment variables, pointers. Never a rule. |

### Who owns what

| Thing | Owner | When the code disagrees with it |
| --- | --- | --- |
| Spec steps, rules, APIs, errors, acceptance criteria | PM | Add a `[drift]` open question. Do not edit the row. |
| Spec `Status` | PM | Never set to `Approved`. |
| Spec `Issue` column and `Epic` row | `/agentic-dev-kit:spec-issues` | Read only. |
| `CLAUDE.md` rules and the never-do list | The team | Report the difference to the developer. Do not edit the rule. |
| Application code | The developer | Never edited — a difference is reported, not fixed. |

### Evidence

Every statement written into a document is backed by something read in this run: a file path that exists, a line in a diff, a script in a `package.json`, a row in the spec, a PR or issue returned by `gh`. A fact with no source is left out, or written as `unknown` where the reader needs to know the gap exists. This applies to paths, route URLs, commands, script names, variable names, and API endpoints alike.

### Presenting the plan

One numbered edit per change, grouped by file. Each edit shows the file, what it does in one line, the source of the fact, and the diff (for an existing file) or the full new text (for a new file). The developer answers per number, or for all at once — except the edits that always need their own yes: any `CLAUDE.md` edit, and creating a changelog file.

```
Plan: 6 edits in 4 files

docs/specs/<flow-name>/spec.md
  1. Section 15: add Build column; W-01 built (#<pr>), W-02 in progress, W-03 not started
  2. Section 14: add Q-07 [drift] — raised at R-02
  3. Section 16 + header: change-log line, Last updated
docs/features/<flow-name>.md   (new file)
  4. Create the feature doc
CHANGELOG.md
  5. Add entry under <heading>
CLAUDE.md   (needs its own yes)
  6. §9 Environment variables: add <VARIABLE_NAME>

Not planned: README — no change to structure, scripts or env vars.
```

The diffs or new text follow the list, in the same order.

### Running it twice

Before planning an edit, check whether its content is already there. The key for each kind:

| Edit | Already present when |
| --- | --- |
| Build status | The row's `Build` cell already holds the same status and reference. |
| `[drift]` question | Section 14 has a `[drift]` row raised at the same spec IDs about the same difference — open or answered. |
| Spec change-log line | No other spec edit is being written (the line is only ever added alongside one). |
| Feature doc | The section's content already matches what the sources give. |
| Changelog entry | The PR or issue reference already appears in the changelog. |
| README / `CLAUDE.md` fact | The fact is already stated, in any wording. |

Dates (`Last updated`, the feature doc's `Last synced`) change only when something else in the same file is being written.

### Reference Guide

| Topic | Reference | Load when |
| --- | --- | --- |
| Base branch, diff, PR, issue, which flow | `references/what-changed.md` | Starting any run |
| Build status, drift questions, change log | `references/spec-status.md` | The change belongs to a flow with a spec |
| The per-flow developer page | `references/feature-doc.md` | Creating or updating `docs/features/<flow-name>.md` |
| Changelog detection and entry format | `references/changelog.md` | Adding the changelog entry |
| README and `CLAUDE.md` rules | `references/readme-and-claude-md.md` | Structure, scripts, env vars or conventions changed |

## Constraints

### MUST DO

- Use only read-only `git` and `gh` commands.
- Read a document in full before planning an edit to it.
- Show the full plan — every edit with its diff or new text — before writing anything, and write only what the developer agreed to.
- Record every difference between the code and the spec as a new `[drift]` question in section 14, stating what the spec says, what the code does, and where.
- Give new questions the next free `Q-` numbers; keep every existing ID as it is.
- Find a table column by its header name, never by its position — a spec may also have an `Issue` column.
- Follow the project's existing changelog file and format.
- Show the exact diff for a `CLAUDE.md` edit and get an explicit yes for that edit on its own.
- Take every path, script, command, route, and variable name from the repository; leave it out or mark it `unknown` when it cannot be found.
- Skip any edit whose content is already present.
- End with a summary of what was written and what was skipped.

### MUST NOT DO

- Edit application code, configuration, tests, or anything outside the four documents.
- Rewrite, reword, or remove a spec step, rule, API contract, error, or acceptance criterion to match the code.
- Set a spec's `Status` to `Approved`, or touch its `Challenged`, `Epic`, or `Issue` entries.
- Renumber or reuse an ID, or delete a row.
- Weaken, remove, or reword an existing `CLAUDE.md` rule, or add a new rule that the developer did not word themselves.
- Write into the kit's own files or templates (`${CLAUDE_PLUGIN_ROOT}`) — only the project's documents.
- Create a changelog file without asking, or hand-edit a changelog that a release tool generates.
- Copy a value from an environment file, a credential, a token, or real personal data into any document — names only.
- Invent a path, script, command, route, endpoint, or PR number.
- Run anything that changes the repository or GitHub: no commit, push, checkout, fetch, merge, comment, or issue edit.
- Claim behaviour was tested — `built` records that code exists, nothing more.

## Output Templates

When syncing documents, provide:

1. The source summary — base branch, the commits or PR read, the issue, and the flow matched (and how it was matched).
2. The plan — numbered edits grouped by file, each with its diff or new text, plus what was deliberately not planned and why.
3. After agreement, the written files.
4. The summary — per file what was written; per skipped edit the reason (declined, already present, fact not found); new `Q-` IDs; and the next step for the developer and the PM.

## Knowledge Reference

documentation sync, functional spec, build status, work breakdown, spec drift, open questions, feature doc, docs/features, docs/specs, changelog, Keep a Changelog, release notes, README, CLAUDE.md, guardrails, environment variables, package scripts, monorepo structure, git diff, merge base, GitHub CLI, pull request, idempotent edits, Turborepo, pnpm, Next.js, React Native
