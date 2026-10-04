---
name: security-check
description: Check frontend changes for security problems before they merge (spec sections 8 and 11 as the checklist, plus eleven general frontend checks). Runs the read-only security-checker agent, then offers to fix the findings the developer picks.
argument-hint: "[nothing = this branch | PR number | path | flow name]"
---

# /security-check

**Arguments:** $ARGUMENTS

Runs the read-only **`security-checker`** agent on frontend code — the web app, the
shared core package, the UI packages, and the native app's JS/TS — then, only for the
findings the developer picks, applies the fixes here in the main conversation. The agent
never edits anything.

It is a check, not a guarantee. It does not replace a security review of the backend or
a penetration test; say so when relaying the report.

## What to do

1. **Parse `$ARGUMENTS` into a scope:**
   - Empty → the current branch's changes against the base branch. The base is `develop`
     if that branch exists, otherwise the repository's default branch. If the current
     branch **is** the base, or there are no changes, say so and ask for a PR number or
     a path instead.
   - A number (`123`, `#123`) → that pull request. Read its head and base branch with
     `gh pr view`. If its head is not the branch checked out here, tell the developer the
     check will run on the diff text only (less thorough) and ask whether to continue or
     to check the branch out first. Do not switch branches yourself.
   - A path that exists → every frontend file under it.
   - Anything else → treat it as a flow name. It must match a folder under `docs/specs/`;
     if it does not, list the specs that exist and ask.

2. **Find the spec** (`docs/specs/<flow-name>/spec.md`):
   - A flow name was given → that flow's spec.
   - Otherwise look for one flow that the change clearly belongs to: a spec folder whose
     name matches a changed feature folder or route, a spec named in the branch name or
     the PR description, or a spec file changed in the same diff.
   - More than one candidate → ask the developer which. None → pass `none`; the agent
     runs the general checks only and the report will say the spec checks were not run.
     Never pick a spec on a weak match.

3. **Spawn the agent** (Agent tool, `subagent_type: agentic-dev-kit:security-checker`)
   with the scope, the base branch, and the spec path or `none`. The agent is namespaced
   under the plugin, so use the fully-qualified `agentic-dev-kit:security-checker` — the
   bare name will not resolve once installed. **Relay the report in full**, including
   "A human must verify" and the closing line about what the check does not cover.

4. **Ask the developer which findings to fix.** List them by ID. Default suggestion:
   every 🔴. Let them pick, drop, or dispute any. Do not start editing before they
   answer.

5. **Apply only the agreed fixes**, one finding at a time:
   - Follow the finding's `Fix` line and the project rules in root `CLAUDE.md`
     (layering, injected storage, no new HTTP client). If the fix needs a new dependency
     or a change outside the finding's file, say so before making it.
   - A finding marked `(outside the diff)` is fixed only if the developer named it.
   - **Secrets:** removing a secret from the code does not undo the leak. Remove it,
     then tell the developer plainly that the value must be rotated and, if it was
     pushed, that it remains in the git history. Do not rewrite history.
   - Do not commit or push unless asked.

6. **Spec gaps go to the spec, not into code.** For each finding of type `spec gap` —
   the spec does not say whether something may be stored, shown, or logged — offer to
   add it to the spec's Open questions instead of choosing an answer. If the developer
   agrees, make these edits and nothing else:
   - **Section 14, Open questions:** append one row with the next free `Q-` number.
     Never renumber or reuse an ID. Question = the finding's "Ask" text, prefixed
     `[security]`. `Raised at` = `section 11` or `section 8` (and the spec IDs involved,
     if any). `Who answers` = the spec's owner. `Blocks approval?` = ask the developer;
     suggest `yes` when the code that depends on the answer is in this change.
     `Answer` = `—`. Recount the section's count line if it has one.
   - **Header:** set `Last updated`; set `Status` to `Draft` only if a blocking question
     was added.
   - **Section 16, Change log:** one line — date, "Security check: N questions added",
     the developer's name.

   If there is no spec, give the developer the questions to take to the PM and suggest
   `/agentic-dev-kit:spec-builder` to record them.

7. **Posting to the pull request — only when the developer asks.** Never post on your
   own, and never as part of step 3. When asked:
   - Post one comment on the PR with `gh pr comment`. If no PR number is known, ask.
   - Before posting, re-read the text and remove any secret value, token, or real
     personal data, including partial ones — refer to the file and line only.
   - Keep the verdict, the findings, "A human must verify", and the closing line about
     what the check does not cover. Mark which findings were fixed in this session.

8. **Print the summary** — the verdict, counts by severity, which findings were fixed
   and which files changed, which were left and why, any questions added to the spec
   with their `Q-` IDs, any secret that still needs rotating, and the next step: review
   the edits, then re-run `/agentic-dev-kit:security-check` to confirm.

## Notes

- **Frontend only.** A finding often ends with "the server must enforce this too" —
  that is for the backend team to confirm, not something this command checks.
- Any 🔴 finding means the change must not be merged until it is fixed or the developer
  records why it does not apply.
- A PASS means these checks found nothing. It does not mean the feature is secure.
- The agent **never** edits files; only this command does, and only for findings the
  developer agreed to.
- With no spec, the most useful checks — what may be kept on the device, what must be
  masked, what must never be logged — cannot run. Writing the spec first
  (`/agentic-dev-kit:spec-builder`) makes this check much stronger.
- Fixes are not verified by running anything. After fixing, the developer runs the
  project's own checks and re-runs this command.
