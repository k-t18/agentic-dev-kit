# Issue templates

Every issue this skill creates follows the project's **feature-request issue template**.
The spec's content is fitted into that template's fields and headings — the template is
never replaced by a layout of this skill's own.

Write each body to a temporary file and pass it with `--body-file`
(`github-commands.md`). Text in `<angle brackets>` is replaced. Tables are copied from
the spec with their cell text unchanged.

`<spec link>` is the spec's URL on the repository's default branch:
`<repo url>/blob/<default branch>/docs/specs/<flow-name>/spec.md`. Always give the
repo-relative path next to it, so the reference still works if the link does not.

## 1. Find the standard

Read `.github/ISSUE_TEMPLATE/` in the project and pick the feature-request template (the
one whose `name` or `about` describes a feature, addition, or improvement).

| From the template | Used as |
| --- | --- |
| `title:` in the frontmatter | Not used — titles follow the fixed format in section 3 |
| `labels:` | The labels on every issue created |
| `assignees:` | Followed only if it names someone; empty stays empty |
| Bold field lines above the first heading (`**Priority:**`, `**Area:**`, …) and their allowed values | Filled with one of the listed values, never a new one |
| Headings, in order | The sections of the body — same text, same level, same order, none dropped, none added |
| HTML comments under a heading | Guidance for what goes there; not copied into the issue |

If the project has **no** feature-request template, use the default below and say so in
the plan. If it has a template whose headings differ from the default, follow the
project's and place the content by meaning (the mapping in section 3). If a required
field has no value this skill can fill honestly, ask — do not guess.

Bug-report templates are not used here. Work cut from a spec is new work, not a defect.

## 2. The default standard

Used when the project has no template of its own.

- **Label:** `enhancement`
- **Fields:** `**Priority:** High | Medium | Low` and
  `**Area:** native | web | android | tooling | docs`
- **Headings:** `### Problem / motivation`, `### Proposed solution`,
  `### Alternatives considered`, `### Acceptance criteria` (a checklist)

## 3. Where the spec's content goes

| Template part | Child issue | Epic |
| --- | --- | --- |
| Title | `feat(<flow-name>:W-nn): <piece of work, as written in section 15>` | `feat(<flow-name>:epic): <flow name from the spec's title line>` |
| Labels | The template's labels | The template's labels, plus `epic` |
| Priority | Chosen by the developer in the plan — the spec carries none | Same |
| Area | From the kind: `native port` → the native value; every other kind → the web value | The web value; add the native value if the flow has a `native port` row and the field allows more than one |
| Problem / motivation | Which flow and work item this is, the links, and the spec's Goal | The spec's section 2 Purpose |
| Proposed solution | Kind, the skill that builds it, everything the row covers, dependencies, open questions | The spec's Summary and Scope, and the links |
| Alternatives considered | Fixed sentence — the approved spec set the approach | Same |
| Acceptance criteria | One checkbox per matching `AC-` row | One checkbox per child issue |

Titles use this format in every project, whatever prefix the issue template's `title:`
field suggests: `<flow-name>` is the spec's folder name, `W-nn` the work item's ID, and
`epic` is written in lower case. Nothing is added before `feat(`.

Priority is the one value the spec cannot supply. Ask once in the plan for a priority to
apply to all issues, and let the developer change it per row. Never choose it.

## 4. Child issue

**Title:** `feat(<flow-name>:W-nn): <Piece of work>`
**Labels:** `enhancement`

````markdown
**Priority:** <High | Medium | Low>
**Area:** <web | native>

### Problem / motivation

Work item `W-nn` of the **<flow name>** flow — <Piece of work, as written in section 15>.

- **Epic:** #<epic number>
- **Spec:** [`docs/specs/<flow-name>/spec.md`](<spec link>) — Approved, work item `W-nn`
- **Why the flow exists:** <the Goal line of section 2, copied>

### Proposed solution

**Kind:** <kind>
**Built with:** <the `feature-slice` skill — `/agentic-dev-kit:feature-slice` | the `zustand-slice` skill — `/agentic-dev-kit:zustand-slice` | the `web-component` skill | the `web-to-native` skill | no skill — screen wiring is built by hand>

Build what the spec states for the IDs below. The spec is the source of truth; if it and
this issue disagree, the spec wins.

<One block per ID type present in the row's Covers. Leave out types the row does not cover.>

#### Screens

| ID | Screen | Type | Figma frame | Reached from | Notes |
| --- | --- | --- | --- | --- | --- |
| S-02 | <copied> | <copied> | <copied> | <copied> | <copied> |

#### Flow steps

| ID | Screen | User action | System response | API | Next |
| --- | --- | --- | --- | --- | --- |
| F-01 | <copied> | <copied> | <copied> | <copied> | <copied> |

#### Rules

| ID | Applies at | Condition | Then | Source of the fact |
| --- | --- | --- | --- | --- |
| R-01 | <copied> | <copied> | <copied> | <copied> |
| R-01 | <copied> | <copied> | <copied> | <copied> |

#### APIs

| ID | What it does | Status | Method and path | Used by | Owner |
| --- | --- | --- | --- | --- | --- |
| API-01 | <copied> | <copied> | <copied> | <copied> | <copied> |

Request and response samples: section 7 of the spec.

#### Validation and errors

| ID | Where | Trigger | What the user sees | What happens next |
| --- | --- | --- | --- | --- |
| E-01 | <copied> | <copied> | <copied> | <copied> |

#### Data and offline

| Data | Lives in | Read offline? | Written offline? | Notes |
| --- | --- | --- | --- | --- |
| <copied> | <copied> | <copied> | <copied> | <copied> |

#### Depends on

- #<n> — W-01: <piece of work>
- #<n> — W-03: <piece of work>

<If none: "Nothing.">

#### Open questions

- **Q-05** (open, does not block approval) — <question text>. Answered by: <who answers>.
- **Q-02** (answered) — <question text>. Answer: <answer text>.
- **API-03** is `<pending | unknown>` — owner: <owner>.

<Leave this block out if the row's Depends on names no Q- or API- ID.>

### Alternatives considered

None — the approach is set by the approved spec. To change it, raise it in the spec
(`/agentic-dev-kit:spec-builder`), not here.

### Acceptance criteria

- [ ] **AC-01** — <criterion, copied> _(covers <Covers, copied>)_
- [ ] **AC-04** — <criterion, copied> _(covers <Covers, copied>)_

<If no criterion matches, one line instead:>
- [ ] No acceptance criterion in the spec references these IDs directly — this work is verified through the screen-wiring issues of the epic.
````

## 5. Epic

**Title:** `feat(<flow-name>:epic): <flow name>`
**Labels:** `enhancement`, `epic`

````markdown
**Priority:** <High | Medium | Low>
**Area:** <web | native>

### Problem / motivation

<Section 2 of the spec, copied: User, Goal, Today, Success.>

### Proposed solution

**Spec:** [`docs/specs/<flow-name>/spec.md`](<spec link>) — Approved, last updated <YYYY-MM-DD>
**Figma:** <the header's Figma link>

<Section 1 Summary, copied.>

**In scope**

<Section 3 "In scope" list, copied.>

**Out of scope**

<Section 3 "Out of scope" list, copied.>

**Depends on**

<Section 3 "Depends on" list, copied.>

The spec is the source of truth; if it and this issue disagree, the spec wins.

### Alternatives considered

None — the approach is set by the approved spec.

### Acceptance criteria

Done when every piece of work below is closed.

<!-- spec-issues:children:start -->
- [ ] Child issues are being created.
<!-- spec-issues:children:end -->
````

The epic is created before its children, so the checklist starts as a placeholder. Once
the children exist, replace **only the text between the two markers** with one line per
child, in spec order:

```markdown
- [ ] #13 — W-01: <piece of work> (`feature slice`)
- [ ] #14 — W-03: <piece of work> (`component`)
```

The checklist is always written — it is the template's Acceptance criteria. Linking the
children as sub-issues (`github-commands.md`) is done as well, when the repository
supports it.

On a later run, regenerate the list from all child issues, old and new. Tick an item
(`- [x]`) when that issue is closed. If the markers are no longer in the epic body
(someone edited it), do not rewrite the body — add the new children in a comment on the
epic instead and say so in the summary.

## Rules for filling the templates

- **The template's shape is fixed.** Same field lines, same headings, same order. The
  spec's tables go under `Proposed solution` as sub-blocks one heading level below the
  template's headings; no heading is added at the template's own level.
- **Copy, do not rewrite.** Cell text from the spec goes in unchanged. No summarising,
  no added detail, no corrected wording.
- **All outcomes of a rule.** A covered `R-` ID brings every row with that ID.
- **Acceptance criteria are checkboxes**, one per `AC-` row, with the ID kept so a
  ticked box traces back to the spec.
- **Dependencies are issue references.** `#<n>` for each `W-` dependency — GitHub
  renders it as a link with the issue's state. This is why children are created in
  dependency order.
- **An open `Q-` never blocks creation here.** The spec is Approved, so the question is
  non-blocking; the issue is created and names the question, its text, and its owner.
- **Leave out empty sub-blocks**, except Depends on, which always appears and says so
  when empty. The template's own sections are never left out.
- **No milestones or projects**, and no assignees beyond what the template names, unless
  asked.
- **Escape nothing by hand.** The body goes through a file, so tables, backticks, and
  `|` inside copied cells need no shell quoting. A literal `|` inside a cell keeps the
  `\|` it has in the spec.
