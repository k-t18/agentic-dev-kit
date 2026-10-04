---
name: spec
description: Write or update the functional spec for one flow (screens, steps, rules, APIs, offline behaviour, acceptance criteria) by interviewing the PM. Runs the functional-spec skill. Saves docs/specs/<flow-name>/spec.md.
argument-hint: "[flow name] [optional figma-url] | [path to an existing spec.md]"
---

# /spec

**Arguments:** $ARGUMENTS

Runs the **`functional-spec`** skill. It interviews first and writes last — the spec is
never drafted from the first message. Writes only under `docs/specs/`; never touches
application code and never creates anything on GitHub.

## What to do

1. **Parse `$ARGUMENTS`:**
   - A path to an existing `docs/specs/<flow-name>/spec.md` → **update mode**. Read it,
     list its open questions, and ask what changed.
   - A flow name (plus an optional Figma URL) → **new-spec mode**. If
     `docs/specs/<flow-name>/spec.md` already exists, switch to update mode instead of
     starting over.
   - A long free-text description of the flow → **new-spec mode**. Treat the text as the
     PM's first answers, not as a finished spec: derive a flow name, confirm it, then
     interview only for what the text leaves unanswered.
   - Empty → ask for the flow name and whether a Figma link exists.

2. **Read context before asking anything** — `functional-spec` → Core Workflow step 1.

3. **Interview in rounds** per `functional-spec` → `references/interview.md`. One topic
   per round. An answer the PM does not have becomes an open question — never a guess.

4. **Draft, show, correct** using `references/spec-template.md`, until the PM confirms
   the draft is accurate.

5. **Run the gate** in `references/quality-gate.md`, then save. Status is `Draft` while
   any blocking open question remains.

6. **Print the summary** — the file path, the status, counts per ID type, and the open
   questions with who must answer each.
