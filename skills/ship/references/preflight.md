# Preflight

Five checks, in this order. All of them are read-only. If one fails, say which and stop
— no stage runs.

## 1 — `gh` is installed and signed in

```bash
gh --version
gh auth status
```

Either one failing → stop and tell the developer what to do (`install gh`, or run
`gh auth login` themselves). Do not sign in for them.

## 2 — The branch is a work branch

```bash
git rev-parse --abbrev-ref HEAD
gh repo view --json defaultBranchRef --jq .defaultBranchRef.name
```

- On `main`, `develop`, or the repo's default branch → stop. Shipping happens from
  `feat/*`, `fix/*`, `chore/*`, or `migration/*` (root `CLAUDE.md` §8).
- Detached `HEAD` → stop.
- Any other branch name → continue, and note that it does not follow the naming in §8.

## 3 — The base branch

```bash
git fetch origin
git branch -r --list origin/develop
```

- `origin/develop` exists → the base is `develop`.
- Otherwise → the base is the repo's default branch.
- Ask instead of choosing when it is unclear: there is more than one remote, the branch
  was cut from something other than the base (`git merge-base` with the base is far
  behind another long-lived branch), or the repo has neither.

Then check the branch against the base:

```bash
git rev-list --count origin/<base>..HEAD     # commits to ship
git rev-list --count HEAD..origin/<base>     # commits the branch is behind
```

- Zero commits to ship → stop; there is nothing to open a PR for.
- Behind the base → carry a warning into the report. Do not rebase or merge the base in
  — that is the developer's choice to make before running again.

Check whether a PR for this branch is already open:

```bash
gh pr view --json number,url,state,baseRefName
```

An open PR changes the last step from "create" to "update" — see `pull-request.md`.

## 4 — The working tree

```bash
git status --porcelain
```

Clean → continue. Otherwise list the files and ask the developer to choose:

| Choice | What happens |
| --- | --- |
| Commit them | Show the files, propose a message in the repo's style, commit on a yes. Hooks run. |
| Leave them out | Continue. Tell the developer the checks read the working copy, so they see these changes, but the PR will not contain them. Carry that into the report. |
| Stop | End the run. |

Never commit, stash, or discard without being told to. Untracked files that look like
secrets or local config (`.env`, key files) are never proposed for commit — point them
out instead.

## 5 — The issue, the `W-` ID, and the spec

Work it out from the first source that gives an answer, then confirm it in the summary:

1. **The argument** — an issue number, a `W-nn`, or a spec path.
2. **The branch name** — a number in it (`feat/123-short-title`) is the issue.
3. **Ask.**

From an issue number:

```bash
gh issue view <number> --json number,title,body,state,labels
```

Child issues created by `/agentic-dev-kit:spec-issues` link the spec and name their
`W-` ID. Read both from the issue body. Then open the spec and read:

- the header — `Status` and `Project mode`;
- section 15 — the `W-nn` row: its title, `Kind`, `Covers`, `Depends on`;
- section 13 — the acceptance criteria whose `Covers` names any ID this piece covers.
  These are the criteria the flow test checks;
- section 11 (sensitive data), section 8 (data and offline), and section 12 (native
  notes) — the rows that touch the covered IDs. They feed the "still to verify" list.

| Situation | What to do |
| --- | --- |
| Only a `W-nn` was given | The same ID exists in every spec. Find the spec from the branch name or the issue; if neither settles it, ask. |
| Only a spec path was given | Ask which `W-` row this branch builds. |
| The issue names no spec or `W-` ID | Ask. Do not match by title. |
| The issue is closed, or is the epic rather than a child issue | Say so and ask whether it is the right one. |
| The spec `Status` is `Draft` | Carry a warning into the report: the spec was not approved when this was built. Do not stop. |
| The developer says there is no issue | Continue. The PR body has no `Closes` line, and the report says so. |
| The developer says there is no spec | Continue. Stages that need a spec are `not run — no spec`. |

## Preflight summary

Print this before the first stage:

```
## Ship — preflight
Branch:   <branch>  →  base <base>   (<N> commits ahead, <N> behind)
Issue:    #<number> <title>          | none
Spec:     docs/specs/<flow-name>/spec.md  ·  status <Approved | Draft>   | none
Work:     W-nn <title> (<kind>)  ·  covers <IDs>
Criteria: AC-nn, AC-nn               | none cover this piece
Tree:     clean | <N> uncommitted files left out of the PR
Skipped:  <stages skipped by flag>   | none
Warnings: <behind base, spec is Draft, branch name, …>   | none
```
