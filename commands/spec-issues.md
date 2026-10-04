---
name: spec-issues
description: Turn an Approved functional spec's work breakdown (section 15) into GitHub issues — one epic plus one child issue per W- row — using the gh CLI, then record the issue numbers back in the spec. Runs the spec-issues skill. Refuses Draft specs; never creates duplicates.
argument-hint: "[flow name | path to spec.md]"
---

# /spec-issues

**Arguments:** $ARGUMENTS

Runs the **`spec-issues`** skill on a spec written by `/agentic-dev-kit:spec-builder`.
It creates issues on GitHub and writes their numbers back into the spec — nothing else.
It never touches application code, never changes the spec's behaviour sections, and
never creates anything before the developer has seen the full plan and said yes.

## What to do

1. **Parse `$ARGUMENTS`:**
   - A path to a `spec.md` → use that file.
   - A flow name → use `docs/specs/<flow-name>/spec.md`.
   - Empty → list the specs under `docs/specs/` with their status and ask which one.
   - If the file does not exist, say so and stop. Do not create issues from a description.

2. **Preflight** — `spec-issues` → Core Workflow step 1. The spec has sections 13, 14,
   and 15; `gh` is installed and authenticated; the repo has a GitHub remote with issues
   enabled. If any check fails, say which one and how to fix it, and stop.

3. **Check the status.** Only `Approved` continues. For a `Draft` spec, refuse: list
   the open questions that block approval (ID, question, who answers) and say the PM
   answers them by running `/agentic-dev-kit:spec-builder` on the same file. Create
   nothing — no epic, no partial set.

4. **Read the spec and what it already records** per `spec-issues` →
   `references/read-spec.md`: the `W-` rows, their kinds, what they cover, what they
   depend on, the `Epic` header row and the `Issue` column if present. Verify every
   recorded issue still exists. Rows that already have an issue are skipped.

5. **Show the plan and ask once.** Target repository, the issue standard in use (the
   project's feature-request template in `.github/ISSUE_TEMPLATE/`, or the kit default),
   epic title, then one line per child issue in creation order: title, kind, area,
   dependencies, and whether it will be created or skipped. List the labels that will be
   created. Ask which **priority** to give the issues — the spec does not carry one —
   then ask for **one** confirmation. Without a clear yes, create nothing and write
   nothing.

6. **Create** per `references/github-commands.md` and `references/issue-templates.md`,
   every issue in the template's shape: missing labels → the epic (unless recorded) →
   child issues in dependency order → the checklist of children in the epic, plus
   sub-issue links when the repository supports them.

7. **Write back to the spec** per `references/write-back.md` — the `Issue` column in
   section 15, the `Epic` header row, one change-log line, `Last updated`. Nothing else.

8. **Print the summary** — epic link, issues created, issues skipped because they
   already existed, how the children were linked (sub-issues or task list), any open
   questions named in issue bodies, and the next step.

## Notes

- No assignees, milestones, or project boards unless the developer asks for them.
- Re-running is expected: after the spec gains new `W-` rows and is approved again, the
  command creates only the rows that have no issue yet.
- It does not rewrite the body of an issue that already exists. If the spec changed
  after its issues were created, update those issues by hand.
- The write-back leaves the spec modified on disk. Commit and push it so the links in
  the issues point at a file that shows the issue numbers.
