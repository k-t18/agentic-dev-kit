---
name: docs-sync
description: Bring the project's documents in line with what was actually built — build status and drift questions in the flow's spec, the flow's feature doc, the changelog, and (only when structure, scripts, env vars or conventions changed) README and CLAUDE.md. Runs the docs-sync skill. Shows every planned edit first and writes only what the developer agrees to.
argument-hint: "[flow name | path to spec.md] [PR number] — empty = the current branch's changes"
---

# /docs-sync

**Arguments:** $ARGUMENTS

Runs the **`docs-sync`** skill in the main conversation, because every edit is shown and
agreed before it is written. It reads git, GitHub, the spec, and the code; it writes
**documents only** — never application code, never the kit's own templates, and nothing
on GitHub. It does not commit or push.

## What to do

1. **Parse `$ARGUMENTS`:**
   - A path to a `docs/specs/<flow-name>/spec.md`, or a flow name → sync that flow. With
     nothing else given, the flow's documents are checked against the code as it stands
     on the current branch.
   - A PR number (`123` or `#123`) → the change is that PR. May be combined with a flow
     name or spec path.
   - Empty → the change is the current branch against the base branch.
   - A flow name whose spec does not exist → say so, list the specs under `docs/specs/`,
     and ask. Do not sync against a description.

2. **Work out what changed** per `docs-sync` → `references/what-changed.md`: the base
   branch (`develop` if it exists, else the repo default), the changed files and commits,
   the PR, the issue it closes, and which flow or flows it belongs to. Read-only `git`
   and `gh` commands only. If the flow cannot be determined, ask — do not guess.

3. **Read before planning** — the spec in full, the changed code, the existing feature
   doc, the changelog, and the README and `CLAUDE.md` files. Nothing is planned from a
   file that was not read.

4. **Build the plan — write nothing yet.** One numbered edit per change, grouped by
   file, each with its diff or its new text:
   - **Spec** — build status in section 15, `[drift]` questions in section 14, the
     change-log line and `Last updated` → `references/spec-status.md`.
   - **Feature doc** — `docs/features/<flow-name>.md` → `references/feature-doc.md`.
   - **Changelog** — one entry for this piece of work → `references/changelog.md`. If
     the project has no changelog, the plan proposes the format and asks before creating
     the file.
   - **README and `CLAUDE.md`** — only when structure, scripts, environment variables or
     conventions changed → `references/readme-and-claude-md.md`.

   Leave out any edit whose content is already present. If nothing needs changing, say
   "documents are already in line" and stop.

5. **Show the full plan and ask.** The developer may accept all, accept by number, reword
   an edit, or decline. Two kinds of edit are never covered by "all" and need their own
   explicit yes:
   - any edit to a `CLAUDE.md` file — show the exact diff;
   - creating a changelog file that does not exist yet.

6. **Write only the agreed edits**, exactly as shown. If an agreed edit no longer applies
   cleanly because the file changed in the meantime, stop and show the new diff rather
   than forcing it.

7. **Print the summary** — every file written and what changed in it, every edit skipped
   and why (declined, already present, fact not found), the new `Q-` IDs, and the next
   step: commit the documents with the work, and tell the PM about any `[drift]`
   questions, which they resolve by running `/agentic-dev-kit:spec-builder` on the spec.

## Notes

- **The spec's rules belong to the PM.** Where the code differs from the spec, this
  command records a `[drift]` open question. It never rewrites a step, rule, API
  contract, error, or acceptance criterion to match the code, and never sets `Status` to
  `Approved`.
- **`built` means the code is there, not that it was verified.** Checking behaviour
  against the acceptance criteria is `/agentic-dev-kit:flow-test`.
- **Idempotent.** Running it twice on the same change plans nothing the second time: no
  second changelog entry, no repeated question, no duplicated section.
- **Nothing invented.** A path, script, command, or variable that cannot be found in the
  repository is left out or written as `unknown` — never guessed.
- **`CLAUDE.md` files are guardrails.** Facts are added or corrected; an existing rule is
  never weakened, removed, or reworded on this command's own initiative.
- The `Issue` column and `Epic` row that `/agentic-dev-kit:spec-issues` adds to a spec
  are read, never changed.
