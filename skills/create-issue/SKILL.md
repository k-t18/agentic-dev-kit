---
name: create-issue
description: File a GitHub issue with a standard, actionable body — bug reports (repro steps, expected vs. actual, environment), feature requests, and tasks. Use this whenever the user wants to open, file, log, raise, or track an issue, bug, ticket, defect, or feature request, including phrasings like "we should track this", "file a bug for that", "make a ticket", "open an issue for the flaky test", or "note this in GitHub for later" — and also when they hand you a vague one-line complaint and ask you to turn it into something a maintainer can act on. Invoke for issue, bug report, feature request, ticket, gh issue create.
license: MIT
metadata:
  author: https://github.com/Shadab-khan207
  version: "1.0.0"
  domain: workflow
  triggers: issue, bug, bug report, ticket, defect, feature request, file a bug, open an issue, track this, gh issue create
  role: specialist
  scope: workflow
  output-format: markdown
  related-skills: create-pr, design-qa
---

# Filing an issue

An issue is a request for someone else's attention, and its quality decides whether that
attention gets spent well. The test to apply before you submit: **could a maintainer who has
never seen this conversation reproduce or act on it using only what's in the issue?** Almost
every weak issue fails on that one point — it assumes context that lives only in the reporter's
head. Everything below is in service of closing that gap.

## Role Definition

Triage-minded maintainer who turns a vague complaint into an issue someone else can pick up
cold. Classifies the report, checks for duplicates, and writes reproduction steps, acceptance
criteria or a definition of done — whichever the shape needs. Uses the same type prefixes as
this kit's branch strategy (root `CLAUDE.md` §8), so the issue, its `feat/*` or `fix/*` branch
and the eventual PR read as one thing.

## When to Use This Skill

- Filing a bug, feature request, or task/chore in GitHub
- Turning a `design-qa` finding, a failing CI run, or a one-line complaint into a trackable issue
- Splitting a bundled report into separate, cross-linked issues

**Not for:** opening a PR (`create-pr`), or fixing the bug directly when the user asked for a fix.

## Prerequisites

```bash
gh auth status
```

## Step 1 — Classify

Three shapes, three templates. Pick from what's actually being reported, not from how the
user phrased it — "can you make it not crash on empty input" is a bug, not a feature.

- **Bug** — something behaves differently than it should. Needs reproduction.
- **Feature** — a capability that doesn't exist. Needs a problem statement and acceptance criteria.
- **Task / chore** — known work with no open design question. Needs scope and a definition of done.

If one report contains several independent problems, file them separately and cross-link.
A bundled issue can't be assigned, prioritized, or closed cleanly, so it tends to sit.

## Step 2 — Check repo conventions and duplicates

```bash
ls .github/ISSUE_TEMPLATE/ 2>/dev/null
gh issue list --search "<key terms>" --state all --limit 10
gh label list --limit 50
```

Two things come out of this:

**Repo templates win.** If `.github/ISSUE_TEMPLATE/` exists, read the relevant template and
fill in *its* fields. Those headings usually feed triage automation, or encode what maintainers
have learned they always end up having to ask for. The formats below are the fallback for
repos that don't have one.

**Search `--state all`, including closed.** A closed duplicate is often the most useful thing
you can find — it may contain the fix, the workaround, or the reason it was declined. If you
find a live duplicate, tell the user and offer to comment on it with the new information
rather than opening another.

## Step 3 — Title

One specific, searchable line under ~80 characters. Someone scanning fifty titles should be
able to tell whether yours is theirs.

State the symptom *and* its trigger — a title that only names a component ("Login page") or
only names a feeling ("doesn't work") forces everyone to open the issue to learn anything at all.

| Instead of | Write |
|---|---|
| `Login broken` | `Login returns 500 when email contains a plus sign` |
| `Add export` | `Export filtered results to CSV from the reports table` |
| `Flaky test` | `test_sync_retries fails intermittently on CI (~1 in 8 runs)` |

**Lead with the type.** `feat:` · `fix:` · `chore:` · `docs:`, matching the branch prefix and
the eventual PR title, so the issue, the branch and the PR read as one thing. The label still
carries the type for filtering; the prefix is what makes a list of fifty titles scannable
without the labels rendered.

| Instead of | Write |
|---|---|
| `Add a 404 page` | `feat: add a 404 page for unmatched URLs` |
| `Registration errors` | `fix: show the backend’s own registration error text` |

## Step 4 — Body

### Every heading gets its own border — never write `---`

Use `##` for every section, including ones that would naturally be subsections. GitHub draws a
1px underline beneath `##` on its own, so the whole body ends up evenly ruled with nothing
written:

```markdown
## Summary

<body text>

## Problem

<body text>

## 1. The first specific failure

<body text>

## 2. The second

<body text>
```

**Never write `---`.** It is not a border — it renders as an `<hr>`, which GitHub paints as a
~4px grey bar, while a heading underline is 1px. The two can never match, markdown offers no
thin `<hr>`, and GitHub strips inline styles, so any body mixing them looks like a mistake.

`###` renders with no border at all. That is why subsections are promoted to `##` rather than
having a rule drawn under them — number them (`## 1.`, `## 2.`) and the numbering carries the
hierarchy the heading level used to.

The even borders are what let someone scan a long body and stop at the part they need.

### Bug

```markdown
## Summary

<One or two sentences: what happens, under what condition.>

## Steps to reproduce

1. <Start from a state anyone can reach — a fresh checkout, a specific URL, a named fixture.>
2. <...>
3. <...>

## Expected

<What should happen, and why you believe that — docs, prior behavior, a test, plain reason.>

## Actual

<What happens instead. Exact error text, status code, or wrong value.>

## Environment

- Version / commit: <tag, or the output of `git rev-parse --short HEAD`>
- OS / runtime: <e.g. macOS 15.2, Node 20.11>
- Relevant config: <flags, env vars, feature toggles that differ from defaults>

## Evidence

<Stack trace, log excerpt, or screenshot — trimmed to the lines that matter.>

## Impact

<Who hits this, how often, whether a workaround exists.>

## Suspected cause

<Optional. `file.ts:142` plus a sentence, if you have a real lead. Omit it rather than guess —
a confident wrong diagnosis sends the first responder down the wrong path.>
```

Step 1 is where bug reports most often fail. "Log in and go to settings" isn't reproducible
if the behavior depends on account state only the reporter has. Say which state, and how
someone else gets into it.

### Feature

```markdown
## Problem

<The situation that's painful today, described from the perspective of whoever feels it.
Resist writing the solution here — a problem stated cleanly often turns out to have a cheaper
answer than the one that first came to mind, and stating it in solution terms hides that.>

## Proposed solution

<What you would build, concretely enough to disagree with.>

## Alternatives considered

<What else could solve the problem, and why not that. Include "do nothing" when it's live.>

## Acceptance criteria

- [ ] <Observable, checkable outcome>
- [ ] <...>

## Out of scope

<What this explicitly does not cover — the cheapest way to prevent scope creep later.>
```

### Task / chore

```markdown
## Context

<Why this needs doing, and what happens if it isn't.>

## Scope

- [ ] <Concrete step>
- [ ] <...>

## Done when

<The condition that closes this issue.>

## Links

<Related PRs, issues, docs, dashboards.>
```

## Before you submit: redact

Issues are frequently public and always permanent. Scan anything pasted in — logs, tracebacks,
config, screenshots — for tokens, API keys, internal hostnames, customer names, and personal
data, and replace them with placeholders (`<redacted>`, `user@example.com`). Do this before
the issue exists rather than after: editing an issue doesn't remove the original text from its
edit history, or from the notification emails already sent to watchers.

## Step 5 — Confirm, then create

Filing an issue is outward-facing and notifies watchers, so show the user the rendered title
and body and get an explicit yes before running the command — unless they've already told you
to go ahead without asking.

Use a heredoc so the body's markdown survives intact:

```bash
gh issue create --title "<title>" --body "$(cat <<'BODY'
## Summary
...
BODY
)"
```

Useful flags: `--label <name>` (only labels that already exist — an unknown one fails the
whole command), `--assignee @me`, `--milestone <name>`, and `--repo owner/repo` when filing
against a project other than the current directory's.

Report the returned issue URL to the user as a markdown link.

## Worked example

The user says: *"the CSV export is broken for big reports."*

That sentence is missing everything a maintainer needs, so ask — or go investigate — until you
have the trigger, the threshold, and the actual error. The resulting issue:

**Title:** `CSV export times out for reports over ~50k rows`

```markdown
## Summary

Exporting a report with more than roughly 50,000 rows fails with a gateway timeout. The export
builds the entire file in memory before sending the first byte, so the request exceeds the 30s
proxy limit on large result sets.

## Steps to reproduce

1. Seed the reports table with 60,000 rows: `npm run seed:reports -- --count 60000`
2. Open `/reports` and clear all filters.
3. Click **Export → CSV**.

## Expected

The file downloads, or the export is queued and the user is notified when it's ready.

## Actual

The request hangs for ~30s, then the browser shows a 504. No file is produced and nothing is
surfaced in the UI — the button simply returns to its idle state.

## Environment

- Version / commit: `v2.14.0` (`a3f91c2`)
- OS / runtime: Ubuntu 22.04, Node 20.11
- Relevant config: nginx `proxy_read_timeout 30s` (our deploy default)

## Evidence

    2026-09-12T14:22:31Z ERR upstream timed out (110: Connection timed out)
      while reading response header from upstream,
      request: "GET /api/reports/export?format=csv"

## Impact

Affects any customer with a large dataset — three reported this week. The workaround is to
narrow the date filter and export in chunks, which is manual and easy to get wrong.

## Suspected cause

`src/reports/export.ts:88` collects all rows into an array before serializing. Streaming the
response row by row would put the first bytes well inside the timeout.
```

## Constraints

### MUST DO

- Search existing issues with `--state all` before filing
- Fill the repo's `.github/ISSUE_TEMPLATE/` fields when they exist
- Give bugs reproduction steps that start from a state anyone can reach
- Prefix titles with the same type as the branch (`feat:` · `fix:` · `chore:`)
- Redact tokens, hostnames, customer names and personal data before filing
- Show the title and body and get an explicit yes before `gh issue create`

### MUST NOT DO

- Write `---` in an issue body
- Bundle several independent problems into one issue
- Guess at a suspected cause — omit the section instead
- Pass a `--label` that doesn't exist in `gh label list`
- File against `@8848digital/offline-kit` or `@8848digital/catalyst` from a product repo —
  those are external packages; file in their own repositories with `--repo`

## Output Templates

When filing an issue, provide:

1. The classification (bug / feature / task) and any duplicates found
2. The proposed title and rendered body, for confirmation
3. The issue URL as a markdown link once created

## Knowledge Reference

GitHub CLI, gh issue create, gh issue list, gh label list, issue templates, bug reports, reproduction steps, acceptance criteria, triage, labels, milestones, redaction
