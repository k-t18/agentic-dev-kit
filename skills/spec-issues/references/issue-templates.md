# Issue templates

Write each body to a temporary file and pass it with `--body-file`
(`github-commands.md`). Text in `<angle brackets>` is replaced. Tables are copied from
the spec with their cell text unchanged.

`<spec link>` is the spec's URL on the repository's default branch:
`<repo url>/blob/<default branch>/docs/specs/<flow-name>/spec.md`. Always give the
repo-relative path next to it, so the reference still works if the link does not.

## Epic

**Title:** `[<flow-name>] Epic: <flow name from the spec's title line>`
**Label:** `epic`

````markdown
**Spec:** [`docs/specs/<flow-name>/spec.md`](<spec link>) — Approved, last updated <YYYY-MM-DD>
**Figma:** <the header's Figma link>

## Summary

<Section 1 of the spec, copied.>

## Scope

**In scope**

<Section 3 "In scope" list, copied.>

**Out of scope**

<Section 3 "Out of scope" list, copied.>

**Depends on**

<Section 3 "Depends on" list, copied.>

## Child issues

<!-- spec-issues:children:start -->
Being created.
<!-- spec-issues:children:end -->

---
Created from the spec's work breakdown (section 15). The spec is the source of truth;
if it and this issue disagree, the spec wins.
````

The epic is created before its children, so the child list starts as a placeholder.
Once the children exist, replace **only the text between the two markers**:

- Sub-issues linked successfully — a plain list, in spec order:

  ```markdown
  - #13 — W-01: <piece of work> (`feature slice`)
  - #14 — W-03: <piece of work> (`component`)
  ```

- Fallback, sub-issue linking failed — a task list, which GitHub tracks as progress:

  ```markdown
  - [ ] #13 — W-01: <piece of work> (`feature slice`)
  - [ ] #14 — W-03: <piece of work> (`component`)
  ```

On a later run, regenerate the list from all child issues, old and new. Tick a
task-list item (`- [x]`) when that issue is closed. If the markers are no longer in
the epic body (someone edited it), do not rewrite the body — add the new children in a
comment on the epic instead and say so in the summary.

## Child issue

**Title:** `[<flow-name>] W-nn: <Piece of work, as written in section 15>`
**Label:** the kind's label (`SKILL.md` → Kind → label and skill)

````markdown
**Epic:** #<epic number>
**Spec:** [`docs/specs/<flow-name>/spec.md`](<spec link>) — work item `W-nn`
**Kind:** <kind>
**Built with:** <the `feature-slice` skill — `/agentic-dev-kit:feature-slice` | the `zustand-slice` skill — `/agentic-dev-kit:zustand-slice` | the `web-component` skill | the `web-to-native` skill | no skill — screen wiring is built by hand>

## What this covers

<One block per ID type present in the row's Covers. Leave out types the row does not cover.>

### Screens

| ID | Screen | Type | Figma frame | Reached from | Notes |
| --- | --- | --- | --- | --- | --- |
| S-02 | <copied> | <copied> | <copied> | <copied> | <copied> |

### Flow steps

| ID | Screen | User action | System response | API | Next |
| --- | --- | --- | --- | --- | --- |
| F-01 | <copied> | <copied> | <copied> | <copied> | <copied> |

### Rules

| ID | Applies at | Condition | Then | Source of the fact |
| --- | --- | --- | --- | --- |
| R-01 | <copied> | <copied> | <copied> | <copied> |
| R-01 | <copied> | <copied> | <copied> | <copied> |

### APIs

| ID | What it does | Status | Method and path | Used by | Owner |
| --- | --- | --- | --- | --- | --- |
| API-01 | <copied> | <copied> | <copied> | <copied> | <copied> |

Request and response samples: section 7 of the spec.

### Validation and errors

| ID | Where | Trigger | What the user sees | What happens next |
| --- | --- | --- | --- | --- |
| E-01 | <copied> | <copied> | <copied> | <copied> |

### Data and offline

| Data | Lives in | Read offline? | Written offline? | Notes |
| --- | --- | --- | --- | --- |
| <copied> | <copied> | <copied> | <copied> | <copied> |

## Acceptance criteria

| ID | Criterion | Covers |
| --- | --- | --- |
| AC-01 | <copied> | <copied> |

<If no criterion matches: "None reference these IDs directly. See the screen-wiring issues of the epic for the criteria this work is verified through.">

## Depends on

- #<n> — W-01: <piece of work>
- #<n> — W-03: <piece of work>

<If none: "Nothing.">

## Open questions

- **Q-05** (open, does not block approval) — <question text>. Answered by: <who answers>.
- **Q-02** (answered) — <question text>. Answer: <answer text>.
- **API-03** is `<pending | unknown>` — owner: <owner>.

<Leave the whole section out if the row's Depends on names no Q- or API- ID.>

---
Created from `W-nn` in the spec's work breakdown (section 15). The spec is the source of
truth; if it and this issue disagree, the spec wins.
````

## Rules for filling the templates

- **Copy, do not rewrite.** Cell text from the spec goes in unchanged. No summarising,
  no added detail, no corrected wording.
- **All outcomes of a rule.** A covered `R-` ID brings every row with that ID.
- **Dependencies are issue references.** `#<n>` for each `W-` dependency — GitHub
  renders it as a link with the issue's state. This is why children are created in
  dependency order.
- **An open `Q-` never blocks creation here.** The spec is Approved, so the question is
  non-blocking; the issue is created and names the question, its text, and its owner.
- **Leave out empty blocks**, except Acceptance criteria and Depends on, which always
  appear and say so when empty.
- **No assignees, milestones, or projects** in the create command unless asked.
- **Escape nothing by hand.** The body goes through a file, so tables, backticks, and
  `|` inside copied cells need no shell quoting. A literal `|` inside a cell keeps the
  `\|` it has in the spec.
