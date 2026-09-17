---
name: create-pr
description: Open a GitHub pull request with a standard, reviewer-friendly title and body (Summary / What Changed / Design and Approach / Test Plan / Notes). Use this whenever the user wants to open, raise, submit, or update a PR or merge request — including phrasings like "ship this", "put this up for review", "PR this branch", "push and open a PR", or "can you write the PR description" — and also when they ask you to rewrite or fill in the description of a PR that already exists. Covers the pre-flight checks (branch, base, commits, push), the title convention, the body template, and the gh commands. Invoke for pull request, PR, merge request, PR description, open PR, raise PR, gh pr create.
license: MIT
metadata:
  author: https://github.com/Shadab-khan207
  version: "1.0.0"
  domain: workflow
  triggers: pull request, PR, merge request, open a PR, raise a PR, ship this, put this up for review, PR description, gh pr create, gh pr edit
  role: specialist
  scope: workflow
  output-format: markdown
  related-skills: create-issue, feature-slice, web-component, web-to-native
---

# Creating a pull request

A good PR description answers the question a reviewer actually has: *why am I being asked
to read this diff, and what should I look at hardest?* The diff already shows what changed.
Your job is intent, scope, and risk. Everything below exists to serve that.

## Role Definition

Release-minded reviewer's advocate who turns a finished branch into a PR a teammate can
review in one pass. Knows this kit's branch strategy (root `CLAUDE.md` **§8**): topic
branches merge into `develop`, and `main` is production-only behind PR + approval + CI.
Writes intent, scope and risk — never a restated file list — and never claims a check that
wasn't run.

## When to Use This Skill

- Opening a PR for a `feat/*`, `fix/*`, `chore/*` or `migration/*` branch
- Writing or rewriting the description of an existing PR
- Handing off after `web-component`, `web-to-native`, `feature-slice` or `zustand-slice`
  work that is ready for review

**Not for:** filing an issue (`create-issue`), or reviewing someone else's PR.

## Prerequisites

```bash
gh auth status
```

If `gh` is missing or unauthenticated, say so and stop — don't fall back to opening a browser
or printing a compare URL unless the user asks for that.

## Step 1 — Find the base branch and read the real diff

```bash
git branch --show-current
git status --short
gh repo view --json defaultBranchRef -q .defaultBranchRef.name
```

The base is usually the default branch, but not always: stacked branches target their parent,
and some repos merge into `develop` or a release branch. If the current branch obviously
descends from something other than the default, ask which base the user wants.

**In a project following this kit's branch strategy** (root `CLAUDE.md` §8), topic branches
target `develop`, not `main` — `main` receives only release merges from `develop`. Check
that `develop` exists (`git ls-remote --heads origin develop`) and use it as the base unless
the user says this is a release or hotfix PR.

**Opening from a fork:** when `gh repo view --json parent` shows a parent, the PR usually
belongs on the upstream repo. Push to the fork, then create with
`--repo <upstream-owner>/<repo> --head <fork-owner>:<branch>`.

Then read the actual changes, not just the commit subjects:

```bash
git log --oneline <base>..HEAD
git diff <base>...HEAD --stat
git diff <base>...HEAD
```

Note the three dots: `<base>...HEAD` diffs against the merge base, which is what the PR will
show. Two dots will include unrelated changes that landed on the base since you branched.

Reading the full diff matters because a description assembled from commit subjects tends to
restate the file list — which reviewers can already see — and silently omits the reasoning,
which only you have.

## Step 2 — Pre-flight

Work through these before writing anything. Each has a different correct response:

| Situation | What to do |
|---|---|
| On the base branch with local commits | Create a topic branch first (`git switch -c <type>/<short-slug>`, using the kit's prefixes `feat/` · `fix/` · `chore/` · `migration/`), then continue |
| Uncommitted changes present | Show the user what's uncommitted and ask whether it belongs in this PR |
| No commits vs. base | Stop — there's nothing to open a PR for |
| Branch not pushed / behind its remote | `git push -u origin <branch>` — ask first, since pushing is outward-facing |
| A PR already exists for this branch | `gh pr view --json url,state,title` — update that one instead of opening a duplicate |

To update an existing PR's description rather than create a new PR, use `gh pr edit --body-file`.

## Step 3 — Title

Conventional-commit style, imperative mood, no trailing period, roughly 70 characters or less:

```
<type>(<optional scope>): <what this change does>
```

Types: `feat`, `fix`, `refactor`, `perf`, `docs`, `test`, `chore`, `build`, `ci`.

Check the repo's recent history first — if `git log --oneline -20 <base>` shows a different
house convention (ticket prefixes like `PROJ-123:`, or plain sentences), match that instead.
Consistency with the repo beats consistency with this skill.

Good: `fix(auth): reject expired refresh tokens on silent renew`
Weak: `Fixes` / `Update auth.ts` / `feat: various improvements to the authentication flow and some cleanup`

## Step 4 — Body

Use this structure. Delete sections that would be empty rather than filling them with "N/A" —
an empty heading is noise a reviewer has to skip.

### Every heading gets its own border — never write `---`

Use `##` for every section, including ones that would naturally be subsections. GitHub draws a
1px underline beneath `##` on its own, so the whole body ends up evenly ruled with nothing
written:

```markdown
## Summary

<body text>

## What Changed

<body text>

## 1. The first specific point

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

The even borders are what let a reviewer scan a long body and stop at the part they need.

Apply it to the template below.

```markdown
## Summary

<1–3 sentences. Lead with the observable effect — what a user, operator, or caller
experiences differently now — then the reason it was needed. Not a file list.>

## What Changed

- `path/to/file.ext` — <what changed here, and why that was the right place for it>
- `path/to/other.ext` — <...>

## Design and Approach

<How you decided to solve it, and what you weighed. Name the alternative you rejected and the
reason — that's usually the reviewer's first question, and answering it up front turns a round
trip into a nod. Skip this for changes with no real decision in them, like a typo fix or a
version bump.>

## Test Plan

- `<command you ran>` — <result>
- <manual verification you performed, and what you observed>
- <what a reviewer should check themselves, if anything>

## Notes for reviewers

<Trade-offs, deliberate omissions, risky areas, follow-up work. Delete if there's nothing
worth flagging.>

Closes #<issue>
```

Then close the body with the attribution footer your session specifies. For Claude Code that
is, on its own line after a blank line:

```
🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### Three rules that matter more than the template

**The Test Plan must be true.** List only checks you actually ran, with their real outcome.
If the suite wasn't run, write `Test suite not run — <reason>`. If a test fails, say which and
why. A reviewer who trusts this section once and gets burned stops reading it, and an inflated
checklist is worse than no section at all.

**Group What Changed by intent, not by directory.** If the diff touches fifteen files for
three reasons, three grouped bullets read better than fifteen path bullets. The reviewer is
trying to build a mental model, and the grouping *is* the model.

**Design and Approach is for decisions, not narration.** "Added a cache layer, then updated
the callers" is just the diff again in prose. What belongs here is the fork in the road: why
a cache rather than a cheaper query, why this invalidation strategy, what you'd have done with
more time. If you can't name a decision, the change didn't have one — delete the section.

### Linking issues

`Closes #12` / `Fixes #12` auto-close on merge — use these only when the PR fully resolves
the issue. For partial work or related context, write `Refs #12` so the issue stays open.
Cross-repo: `Closes owner/repo#12`.

## Step 5 — Confirm, then create

Opening a PR is outward-facing and notifies reviewers, so show the user the rendered title
and body and get an explicit yes before running the command — unless they've already told
you to go ahead without asking.

Use a heredoc so the body's markdown and backticks survive intact:

```bash
gh pr create --base "<base>" --title "<title>" --body "$(cat <<'EOF'
## Summary
...
EOF
)"
```

Useful flags: `--draft` when the work isn't ready for review, `--reviewer <user>`,
`--label <name>` (verify it exists with `gh label list` — an unknown label fails the command),
`--assignee @me`.

If the repo has `.github/pull_request_template.md`, read it and fill in *its* headings. A
repo's own template encodes team agreements and CI checks; the format above is the fallback
for repos that don't have one.

Projects on this kit ship a template whose checklist is Design QA vs Figma · tokens only ·
all three states handled · types in the correct location · Metro verified for native · CI
green. Tick only the boxes that are actually true for this diff; for the rest, leave them
unticked and say why in the body (e.g. "no native changes — Metro not applicable").

Report the returned PR URL to the user as a markdown link.

## Worked example

Branch `fix/token-refresh`, 3 commits, diff touches the auth client and its tests.

**Title:** `fix(auth): reject expired refresh tokens on silent renew`

**Body:**

```markdown
## Summary

Silent renew accepted refresh tokens past their `exp` claim, so a session could be revived
days after it should have ended. The client now validates expiry before the renew call and
surfaces a `SessionExpired` error instead, which the app already handles by redirecting to login.

## What Changed

- `src/auth/refresh.ts` — validate `exp` with 30s clock skew before issuing the renew request;
  throw `SessionExpired` rather than returning `null`, so callers can't silently ignore it.
- `src/auth/errors.ts` — add `SessionExpired`.
- `src/auth/__tests__/refresh.test.ts` — cases for expired, near-expiry-within-skew, and valid.

## Design and Approach

The check runs client-side, before the request, rather than trusting the server to reject the
token. Both are needed eventually, but the client is where the bug actually bites: a stale token
currently produces a revived session with no signal to the user, and a server-side 401 alone
would still leave the client deciding what to do with it.

Throwing is deliberate over returning `null`. The previous `null` return is exactly how this
slipped through — two of the three call sites treated it as "no refresh needed" and carried on
with the old session. A thrown `SessionExpired` can't be ignored by accident, and the app's
existing error boundary already routes it to login.

Rejected: refreshing proactively on a timer. It hides the problem instead of fixing it, and adds
a background request on every session regardless of need.

## Test Plan

- `npm test -- auth` — 24 passed
- Manually set a token `exp` 1h in the past in dev; renew now redirects to login instead of
  restoring the session.
- Reviewers: worth checking the skew value against any other service you've touched — see below.

## Notes for reviewers

The 30s skew is a guess based on what our other services use — worth a second opinion if
there's a house standard I missed. Refresh tokens issued before this deploy are unaffected;
they'll simply fail the new check at their next renew, which is the intended behavior.

Closes #418
```

## Constraints

### MUST DO

- Read the full `<base>...HEAD` diff before writing the description
- Target `develop` for topic branches in projects on this kit's branch strategy
- Fill the repo's `.github/pull_request_template.md` when it exists
- Keep the Test Plan to checks that were actually run, with their real outcome
- Show the title and body and get an explicit yes before `git push` or `gh pr create`
- Use `##` for every body heading and a heredoc for the body

### MUST NOT DO

- Write `---` in a PR body
- Open a duplicate PR when one already exists for the branch
- Push directly to `main` or `develop`, or open a feature PR straight into `main`
- Tick a template checkbox (Design QA, Metro, CI) that wasn't verified
- Pass a `--label` that doesn't exist in `gh label list`

## Output Templates

When opening a PR, provide:

1. The proposed title and rendered body, for confirmation
2. The base branch chosen, and why if it isn't `develop`
3. The PR URL as a markdown link once created

## Knowledge Reference

GitHub CLI, gh pr create, gh pr edit, gh pr view, conventional commits, merge base, three-dot diff, pull request templates, forks, upstream, branch strategy, develop, Closes/Fixes/Refs keywords, draft PRs
