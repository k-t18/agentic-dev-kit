# GitHub commands

Everything goes through the `gh` CLI, run from the repository root. Read the output of
every command before running the next one. `gh` versions differ: when a flag or a JSON
field named here is rejected, run the command's `--help`, adjust, and carry on — do not
guess at a result.

Commands under **Preflight** and **Verify** only read. Everything from **Labels** on
changes the repository and runs only after the developer confirmed the plan.

## Preflight

Run in order. Stop at the first failure and say what to do about it.

| Check | Command | Fails when | Tell the developer |
| --- | --- | --- | --- |
| `gh` installed | `gh --version` | Command not found | Install the GitHub CLI, then re-run. |
| Authenticated | `gh auth status` | Non-zero exit, or no logged-in account | Run `gh auth login` themselves. Never ask for or handle a token. |
| GitHub remote | `git remote -v` then `gh repo view --json nameWithOwner,url,visibility,defaultBranchRef,hasIssuesEnabled` | No remote, the remote is not on GitHub, or `gh repo view` errors | Add or fix the remote. |
| Issues enabled | `hasIssuesEnabled` from the same output | `false` | Enable issues in the repository settings. |

Keep `nameWithOwner`, `url`, `visibility`, and the default branch name — the plan and
the spec links use them. In a fork, `gh` may resolve to the upstream repository; the
plan shows `nameWithOwner` so the developer can catch that before confirming.

Also check, as **warnings only** (shown in the plan, they do not stop the run):

- `git status --porcelain -- <spec path>` shows changes → the spec has uncommitted edits.
- The spec is not on the default branch yet → links in the issues will not resolve
  until it is merged.

## Verify recorded issues

For each recorded number (the epic and every `Issue` cell):

```bash
gh issue view <n> --json number,title,state,url
```

- Returns the issue → it exists. Compare the title: it must start with
  `feat(<flow-name>:W-nn):` for a child, or `feat(<flow-name>:epic):` for the epic. A different flow or ID is a
  mismatch — stop.
- Errors (not found) → the recorded issue no longer exists.

For each row with **no** recorded number, look for an issue left by an earlier,
interrupted run:

```bash
gh issue list --state all --search "\"feat(<flow-name>:W-nn)\" in:title" --json number,title,state,url
```

Search is fuzzy. Accept a result only when its title **starts with** exactly
`feat(<flow-name>:W-nn):` (or `feat(<flow-name>:epic):` for the epic). More than one exact match: stop and ask which one is
real.

## Labels

```bash
gh label list --limit 200 --json name
```

The labels needed are the issue template's (`enhancement` by default) and `epic`. Create
only those the list does not contain. Never pass `--force` — it would overwrite a label
the team already uses.

```bash
gh label create "epic"        --color "5319E7" --description "Parent issue for one flow's spec"
gh label create "enhancement" --color "A2EEEF" --description "New feature or request"
```

No label is created for the kind of work — the kind is written in the issue body.

If a create fails because the label already exists, that is fine — continue.

## Create issues

Write the body to a temporary file (outside the repository, or delete it afterwards —
it must not end up committed), then:

```bash
gh issue create --title "feat(<flow-name>:epic): <flow name>" --label "enhancement" --label "epic" --body-file <body file>
gh issue create --title "feat(<flow-name>:W-01): <piece of work>" --label "enhancement" --body-file <body file>
```

The labels shown are the default standard; use the project's issue template's when it
has one (`issue-templates.md`). The title format is fixed and does not come from the
template. Do not pass `--template` — the body file already
follows the template, and the flag would open an editor.

- On success `gh` prints the new issue's URL; the number is its last path segment.
  Confirm with `gh issue view <n> --json number,title,url` if the output is anything
  other than a single issue URL.
- One issue per command. After each one succeeds, write its number into the spec
  (`write-back.md`) before creating the next.
- Do not pass `--assignee`, `--milestone`, or `--project` unless the developer asked.
- If a create fails: stop. Do not retry blindly — first run the title search above to
  see whether the issue was created anyway. Report what exists and what does not.

Order: the epic first (children reference it), then children sorted by dependency.

## Link children under the epic

Two things are done. The checklist in the epic's Acceptance criteria is **always**
written (below, "The checklist in the epic body"). Sub-issue links are added as well
when the repository supports them.

### Sub-issues, when supported

GitHub has a sub-issues feature with a REST endpoint. It is not available on every
plan, host, or `gh` version, and its request shape has changed over time — treat the
commands below as the expected shape, **check what `gh api` returns, and fall back on
any error.**

The endpoint is expected to take the child's **database id**, not its issue number:

```bash
# 1. The child's database id (a large integer — not the #number)
gh api repos/{owner}/{repo}/issues/<child number> --jq .id

# 2. Attach it under the epic
gh api --method POST repos/{owner}/{repo}/issues/<epic number>/sub_issues -F sub_issue_id=<child id>

# 3. Confirm
gh api repos/{owner}/{repo}/issues/<epic number>/sub_issues --jq '.[].number'
```

`gh api` fills `{owner}` and `{repo}` from the current repository. `-F` sends the id
as a number; `-f` would send a string, which the endpoint may reject.

Try the first child. Treat the path as working only when step 2 returns the issue
without an error **and** step 3 lists the child's number. Then link the rest. A child
already linked under this epic (step 3 lists it) is left as is.

Your `gh` may also offer its own way to set a parent — check `gh issue create --help`
and `gh issue edit --help`. If it does, it is an acceptable substitute for step 2; step
3 still confirms it.

### The checklist in the epic body

**Always run these.** When a sub-issue call returns an error (404, 403, 422, an unknown
endpoint, a feature-not-enabled message), or step 3 does not list the child, the
checklist is the only link — do not retry the sub-issue call with invented parameters,
and say in the summary that sub-issues were not linked.

```bash
gh issue view <epic number> --json body --jq .body > <body file>
# replace the text between the spec-issues:children markers with the task list
gh issue edit <epic number> --body-file <body file>
```

The task list is `- [ ] #<n> — W-nn: <piece of work>` per child (`issue-templates.md`).
GitHub shows each referenced issue's title and state and counts the ticked boxes.

If some children were linked as sub-issues before the failure, leave those links and
still write the task list for **all** children, so the epic body is complete on its own.

### Either way: the child list in the epic body

After linking, update the text between the markers in the epic body with the same
`gh issue view` → edit file → `gh issue edit --body-file` sequence: a plain list when
sub-issues worked, the task list when they did not. Markers gone → comment instead:

```bash
gh issue comment <epic number> --body-file <body file>
```

## What is not done

No assignees, milestones, project boards, issue types, or dependency ("blocked by")
relationships through the API. Dependencies are `#<n>` references in the issue body. If
the developer asks for any of these, add the matching flag or call, show it in the plan,
and check its output the same way.
