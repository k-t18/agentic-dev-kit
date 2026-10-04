# What changed

Everything here is read-only. Nothing in this step writes to the repository or to GitHub.

## Allowed commands

| Purpose | Command |
| --- | --- |
| Current branch | `git rev-parse --abbrev-ref HEAD` |
| Does `develop` exist | `git rev-parse --verify --quiet origin/develop` (then local `develop`) |
| Repo default branch | `git symbolic-ref --quiet refs/remotes/origin/HEAD`, or `gh repo view --json defaultBranchRef` |
| Where the branch left the base | `git merge-base <base> HEAD` |
| Changed files | `git diff --name-status <base>...HEAD` |
| The change itself | `git diff <base>...HEAD -- <path>` |
| Commits | `git log --no-merges --format='%h %s' <base>..HEAD` |
| Uncommitted work | `git status --porcelain` |
| A file at a ref | `git show <ref>:<path>` |
| The PR | `gh pr view [<number>] --json number,title,body,state,mergedAt,baseRefName,headRefName,url,files,commits,closingIssuesReferences` |
| The PR's diff | `gh pr diff <number>` |
| The issue | `gh issue view <number> --json number,title,body,labels,state` |

Not allowed: `commit`, `push`, `checkout`, `switch`, `fetch`, `pull`, `merge`, `rebase`,
`stash`, `reset`, `tag`, and any `gh` command that creates, edits, comments, merges, or
closes. If `gh` is missing or not signed in, say so, carry on with git alone, and leave
PR and issue references out rather than guessing them.

## Pick the base branch

1. A PR was given or found → its `baseRefName`.
2. Otherwise `develop`, if it exists.
3. Otherwise the repo default branch.

Say which base was used. If the base ref is only local and may be behind the remote, say
that too — do not fetch.

## Pick the change

| Input | The change is |
| --- | --- |
| Nothing | `<base>...HEAD` on the current branch. If the branch has an open PR, read it as well. |
| A PR number | That PR: its files, commits, body, and closing issues. |
| A flow name or spec path alone | No single change — check the flow's documents against the code as it stands on the current branch. |
| A flow plus a PR | That PR, limited to that flow's documents. |

If the current branch **is** the base branch and no PR or flow was given, there is no
change to read — ask for a PR number or a flow name.

Uncommitted files are listed and counted as work in progress. They never earn `built`.

When a PR's branch is not checked out, read from `gh pr diff` and from
`git show <ref>:<path>` if the ref exists locally. If neither gives the full file, say
that the reading was limited to the diff, and do not report drift for anything that
could not be read in full.

## Find the issue

In order: the PR's closing issues → an issue number in the PR body or the commit messages
(`#<n>`, `closes #<n>`) → a number in the branch name. None found → carry on without one
and say so.

## Find the flow

A change can belong to one flow, several, or none. Evidence, strongest first:

1. The issue number appears in a spec's `Issue` column in section 15, or its epic matches
   a spec's `Epic` header row.
2. The PR body, issue body, or a commit message names a spec path
   (`docs/specs/<flow-name>/`) or a `W-` ID together with a flow.
3. The changed files are the ones an existing `docs/features/<flow-name>.md` lists under
   "Where the code lives".
4. The changed files clearly implement one spec's screens, APIs, or work items — the
   route, slice, or component named in its sections 4, 7 and 15.

One clear match → use it and say how it was matched. Several candidates or only weak
evidence → list them and ask. No match → say the change belongs to no spec; plan only
the changelog and README/`CLAUDE.md` edits, and suggest
`/agentic-dev-kit:spec-builder` if the work is a flow that should have one. A feature doc
is not written for a flow with no spec.

## What to hand to the next steps

- Base branch, and the commit range or PR.
- PR number, state (open / merged), and merge date if merged.
- Issue number and title.
- The flow or flows, with their spec paths.
- Changed files, grouped: `apps/web/app/**` (routes), `apps/native/**` (screens,
  navigation), `packages/core/src/features/**` (slices), stores, `packages/ui-web/**` and
  `packages/ui-native/**` (components), `package.json` files, environment example files,
  lint/build/workspace config, documents.
