# Pull request

Reached only when no stage is `failed`. Stages that are `not run` do not prevent the
PR — they are listed in it.

## Before showing anything

```bash
git status --porcelain                      # anything uncommitted?
git log --oneline origin/<base>..HEAD       # what the PR will contain
```

Uncommitted fixes or docs-sync edits that the developer has not decided on → ask now.
The PR contains committed work only.

## Title

Conventional-commit style, the same as the repo's commit history:
`<type>(<scope>): <summary>`.

| Part | Comes from |
| --- | --- |
| `type` | The branch prefix: `feat/*` → `feat`, `fix/*` → `fix`, `chore/*` → `chore`. For `migration/*`, use the type the repo's history uses for earlier migrations; if there is none, ask. |
| `scope` | The feature folder, component, or flow name the piece of work is about — as the repo's recent commits scope it (`git log --oneline -20 origin/<base>`). |
| `summary` | The `W-nn` row's title, lower case, imperative, no full stop. |

If the repo's history does not use conventional commits, match what it does use.

## Body

If the project has a PR template, fill it in rather than replacing it. Look in:
`.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE.md`,
`pull_request_template.md`, `docs/pull_request_template.md`. Several templates in
`.github/PULL_REQUEST_TEMPLATE/` → ask which.

- Keep the template's headings and order. Put the sections below where they fit; add
  the ones that have no place at the end.
- Tick a checklist box only when a stage of this run proved it (for example "Design QA
  vs Figma" only when design QA passed for every changed component). Leave the rest
  unticked — the developer ticks what they checked themselves.

Without a template, use this:

```
Closes #<issue>

## What this is
<W-nn title> — <one or two sentences on what the branch builds>

## Spec
docs/specs/<flow-name>/spec.md  ·  W-nn
Covers: <F-, R-, API-, E-, S- IDs from the W- row>
Acceptance criteria: <AC- IDs>

## Ship report
<the combined report: stage table, then findings>

## Not run
<the not-run list, each with its reason>

## Still to verify by a human
<the list from the combined report>
```

- `Closes #<issue>` is the first line. No issue → leave the line out and say "No issue
  linked" under "What this is".
- "Not run" and "Still to verify" stay as their own headings even when a template is
  used.
- No real credentials, tokens, sign-in details, app URLs with secrets in them, or
  personal data in the body.

## The one confirmation

Show all of it, then ask once:

```
## Pull request to open
Repo:   <owner>/<repo>
Head:   <branch>  (<N> commits, will be pushed to origin)
Base:   <base>
Title:  <title>

<the full body>

Not run: <one line listing the not-run stages> | none
Push <branch> and open this pull request? (yes / no)
```

- Anything other than a clear yes → do not push, do not create. Edits the developer
  asks for (title, wording, base) are made and the whole thing is shown again.
- The base is never changed to `main` on the skill's own initiative. If the developer
  asks for a base other than the one preflight chose, repeat it back in the
  confirmation.

## Push and create

```bash
git push -u origin <branch>
gh pr create --base <base> --head <branch> --title "<title>" --body-file <file>
```

- Write the body to a temporary file outside the working tree and pass it with
  `--body-file`; remove the file afterwards.
- A plain push only. If it is rejected (the remote branch has commits the local one
  does not), stop and tell the developer. Do not force, and do not pull or rebase for
  them.
- A pre-push hook that fails → stop and report its output. Do not bypass it.
- `gh pr create` fails after a successful push → say that the branch is pushed and the
  PR is not open, show the error, and stop.

### A PR for the branch is already open

Preflight found one. Say so in the confirmation and ask to update it instead:

```bash
git push origin <branch>
gh pr edit <number> --body-file <file>
```

Replace only the sections this skill wrote (Spec, Ship report, Not run, Still to
verify). Keep anything the developer or a reviewer added to the body, and keep the
title unless the developer asks to change it.

## Never

- `gh pr merge`, in any form, including `--auto`.
- `git push --force`, `--force-with-lease`, or `--no-verify`.
- A push to `main`, `develop`, or the default branch.
- Approving, requesting reviewers, adding labels, or changing the issue — unless the
  developer asks for that specific thing.

## After it is open

```
## Shipped — <title>
PR: <url>   (<branch> → <base>)
Closes: #<issue>
Stages: <N> passed · <N> not run
Findings open: 🔴 0 · 🟡 N · ⚪ N
Not run: <list> | none
Still to verify by a human: <N> items — see the PR body
Next: review and merge are done by a person.
```
