---
name: spec-challenger
description: Challenge a functional spec from the developer's side before it is approved (5 checks against the spec itself, Figma, and the codebase). Runs the read-only spec-challenger agent, then offers to add the doubts to the spec's open questions for the PM.
argument-hint: "[flow name | path to spec.md]"
---

# /spec-challenger

**Arguments:** $ARGUMENTS

Runs the read-only **`spec-challenger`** agent on a spec written by `/spec-builder`, then
— only if the developer agrees — records the doubts in the spec's Open questions so the
PM sees them in the document. Run it **before** the spec is approved and before any
issues are cut from it.

## What to do

1. **Parse `$ARGUMENTS`:**
   - A path to a `spec.md` → challenge that file.
   - A flow name → challenge `docs/specs/<flow-name>/spec.md`.
   - Empty → list the specs under `docs/specs/` with their status and ask which one.
   - If the file does not exist, say so and stop. Do not challenge a description.

2. **Spawn the agent** (Agent tool, `subagent_type: agentic-dev-kit:spec-challenger`)
   with the spec path. The agent is namespaced under the plugin, so use the
   fully-qualified `agentic-dev-kit:spec-challenger` — the bare name will not resolve once
   installed. It reads the spec, the Figma frame list, and the codebase, runs its five
   checks, and returns the report. **Relay the report in full.**

3. **Ask the developer which findings go to the PM.** Default: every 🔴 and 🟡 finding.
   Let them drop any they disagree with or reword a question. ⚪ notes are added only if
   asked for.

4. **If the developer agrees, update the spec — these edits and nothing else:**
   - **Section 14, Open questions:** append one row per chosen finding with the next
     free `Q-` numbers. Never renumber or reuse an ID. The question is the finding's
     "Ask" text, prefixed `[challenge]`. `Raised at` = the spec IDs the finding names.
     `Who answers` = the finding's owner. `Blocks approval?` = `yes` for 🔴, `no`
     otherwise. `Answer` = `—`.
   - If section 14 ends with a count line, recount it.
   - **Header:** set `Status` to `Draft`; set `Challenged` to
     `<YYYY-MM-DD> by <developer> — <N> questions added (Q-nn–Q-nn)` (add the row under
     `Project mode` if the spec predates it); set `Last updated`.
   - **Section 16, Change log:** one line — date, "Developer challenge: N questions
     added", the developer's name.

   Do not edit any other section. The developer's doubts are questions, not changes to
   the PM's rules.

5. **If there are no findings to add**, offer to record only the header row:
   `Challenged | <YYYY-MM-DD> by <developer> — no questions raised`, plus the change-log
   line. Leave `Status` as it is.

6. **If the developer declines**, write nothing.

7. **Print the summary** — how many questions were added and their IDs, the new status,
   the "Ready to build now" rows from the report, and the next step: commit the spec and
   tell the PM, who answers the new questions by running `/agentic-dev-kit:spec-builder`
   on the same file.

## Notes

- The agent **never** edits files; only this command does, and only after the developer
  says yes.
- Questions already open in the spec are not raised again. If the report repeats one,
  drop it rather than adding a duplicate.
- A spec with unanswered `[challenge]` questions marked blocking cannot be `Approved`.
- Re-running after the PM has answered is expected: it challenges the updated spec and
  adds only what is new.
