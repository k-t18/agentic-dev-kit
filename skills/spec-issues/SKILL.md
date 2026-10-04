---
name: spec-issues
description: Use when an Approved functional spec's work breakdown must become GitHub issues — one epic plus one child issue per W- row, created with the gh CLI in dependency order, linked as sub-issues, and recorded back in the spec. Refuses Draft specs, shows the full plan and asks once before creating anything, and never creates a duplicate on re-run. Invoke for create issues from spec, spec to issues, cut issues, epic from spec, work breakdown to GitHub, sub-issues, gh issue create.
license: MIT
metadata:
  author: https://github.com/k-t18
  version: "1.0.0"
  domain: frontend
  triggers: create issues from spec, spec to issues, cut issues, epic, sub-issues, work breakdown, gh issue create, GitHub issues, docs/specs
  role: specialist
  scope: planning
  output-format: document
  related-skills: functional-spec, feature-slice, zustand-slice, web-component, web-to-native
---

# Spec Issues (docs/specs → GitHub)

Turns section 15 of an Approved functional spec into one GitHub epic and one child issue per `W-` row, and records the issue numbers back in the spec.

## Role Definition

Senior delivery engineer for a web-first, then React Native, frontend. Specialises in turning an agreed spec into issues a developer can pick up without opening the spec: each issue carries the steps, rules, and acceptance criteria it must satisfy, copied word for word. Treats creating issues as an outward-facing action — plans first, asks once, creates in order, and leaves a record so a second run adds only what is missing.

## When to Use This Skill

- A spec in `docs/specs/<flow-name>/spec.md` is `Approved` and its work breakdown has no issues yet.
- An approved spec gained new `W-` rows and only those need issues.
- A previous run stopped part-way and the remaining rows need issues.

**Not for:** writing or changing a spec (`functional-spec`); `Draft` specs; building the work; updating issues that already exist; assignees, milestones, or project boards (only when the developer asks).

## Core Workflow

1. **Preflight** — the spec file exists and has sections 13, 14, and 15; `gh` is installed and authenticated; the repo has a GitHub remote with issues enabled. Any failure: say what failed, how to fix it, and stop → `references/github-commands.md`.
2. **Gate on status** — `Approved` continues. `Draft` is refused with the blocking open questions listed → `references/read-spec.md`.
3. **Read the spec** — the `W-` rows, the IDs each covers, the matching acceptance criteria, the dependencies, and any issue numbers already recorded → `references/read-spec.md`.
4. **Verify what is recorded** — every recorded epic and issue number is looked up on GitHub before it is trusted.
5. **Plan and confirm once** — print the whole plan; one clear yes covers everything in it.
6. **Create in order** — missing labels, the epic, then child issues in dependency order; record each number in the spec as soon as it exists → `references/issue-templates.md`, `references/github-commands.md`.
7. **Link under the epic** — sub-issues through `gh api`; on any failure, a task list in the epic body.
8. **Write back and summarise** — finish the spec edits → `references/write-back.md`; print the summary.

## Technical Guidelines

### What gets created

| Item | Count | Labels | Body |
| --- | --- | --- | --- |
| Epic | One per spec | The template's, plus `epic` | The spec's Purpose, Summary and Scope, the links, and a checklist of the child issues |
| Child issue | One per `W-` row | The template's | Kind, the skill that builds it, the covered IDs with their spec text, dependencies, link to the spec, and the matching acceptance criteria as a checklist |

### The project's issue standard

Every issue follows the project's **feature-request issue template** in `.github/ISSUE_TEMPLATE/`: its labels, its field lines (such as Priority and Area) and their allowed values, and its headings in order. The spec's content is fitted into that shape → `references/issue-templates.md`. With no template in the project, the default there is used: label `enhancement`, fields Priority and Area, headings Problem / motivation, Proposed solution, Alternatives considered, Acceptance criteria.

Titles have one fixed format, whatever the template's own title prefix is: `feat(<flow-name>:epic): <flow name>` for the epic and `feat(<flow-name>:W-nn): <piece of work>` for a child issue. The flow name and the `W-` ID in the title are what a later run searches for, so never drop them.

Priority is not in the spec. The developer chooses it in the plan; it is never guessed.

### Kind → area and skill

| Kind (section 15) | Area | Built with | Command |
| --- | --- | --- | --- |
| `feature slice` | web | `feature-slice` | `/agentic-dev-kit:feature-slice` |
| `client state` | web | `zustand-slice` | `/agentic-dev-kit:zustand-slice` |
| `component` | web | `web-component` | — (skill) |
| `screen wiring` | web | none — built by hand in `apps/web` | — |
| `native port` | native | `web-to-native` | — (skill) |

The kind is written in the issue body, not as a label — the labels are the template's. If the template's Area field offers no value that fits, ask.

A kind outside this table is not guessed at: report the row and stop. It is fixed in the spec through `/agentic-dev-kit:spec-builder`.

### Status gate

| Spec state | Action |
| --- | --- |
| `Approved`, no blocking question open | Continue. |
| `Draft` | Refuse. List every open question with `Blocks approval?` = `yes` and no answer. Say the PM answers them via `/agentic-dev-kit:spec-builder`. Create nothing. |
| `Draft` with no blocking question | Refuse. Say the PM has not confirmed the spec, or the quality gate failed; `/agentic-dev-kit:spec-builder` resolves it. |
| `Approved` but a blocking question is still open | Refuse. The header and section 14 disagree; treat it as `Draft`. |

There is no partial creation for a spec that is not approved — not the epic, not the unblocked rows.

An `Approved` spec can still carry open questions marked non-blocking. A `W-` row that names one in `Depends on` still gets its issue; the body names the question, its text, and who answers it.

### Idempotency

The spec is the record. A run reads the `Epic` header row and the `Issue` column first, then:

| Recorded | Found on GitHub | Action |
| --- | --- | --- |
| Number | Yes, open or closed | Skip. Report it as already existing (with its state). |
| Number | No | Show it in the plan as "recorded #n not found — recreate". The single confirmation covers it. |
| Number | Yes, but the title names a different flow or `W-` ID | Stop. Report the mismatch; do not create and do not edit the cell. |
| Nothing | An issue with the same title already exists | Do not create. Show it in the plan as "found #n — adopt". On yes, record it. |
| Nothing | No matching issue | Create. |

Each issue number is written into the spec immediately after the issue is created, so a run that stops half-way leaves a spec the next run can continue from.

### Order of creation

1. Labels that do not exist yet.
2. The epic, unless one is recorded and verified.
3. Child issues, sorted so every row comes after the `W-` rows in its `Depends on`. Ties keep spec order. A dependency cycle, or a dependency on a `W-` ID that does not exist or is marked `removed`, is reported and stops the run before anything is created.
4. Sub-issue links, then the child list in the epic body.

Rows marked `removed` in section 15 get no issue. If a removed row already has one, leave the issue alone and name it in the summary.

### Reference Guide

| Topic | Reference | Load when |
| --- | --- | --- |
| Status gate, parsing sections 13–15, expanding `Covers`, building the plan | `references/read-spec.md` | Before planning |
| Epic and child issue bodies | `references/issue-templates.md` | Writing an issue body |
| Preflight, labels, create, verify, the epic checklist and sub-issues | `references/github-commands.md` | Running any `gh` command |
| The exact edits to the spec | `references/write-back.md` | After each issue, and at the end |

## Constraints

### MUST DO

- Run every preflight check and stop on the first failure, naming it.
- Refuse any spec whose status is not `Approved`, listing the blocking open questions and pointing at `/agentic-dev-kit:spec-builder`.
- Verify every recorded issue number on GitHub before skipping its row.
- Follow the project's feature-request issue template — labels, fields, headings in order — with titles in the fixed `feat(<flow-name>:W-nn): …` format, and say in the plan which template was used.
- Ask the developer for the priority; fill every other field from the spec or the template's allowed values.
- Show the full plan — target repository, issue standard, epic title, every child title, kind, area, dependencies, create or skip — and get one explicit confirmation before the first `gh` command that creates or changes anything.
- Create child issues in dependency order, and reference dependencies by their real issue numbers.
- Copy step, rule, error, and acceptance-criterion text from the spec word for word.
- Record each issue number in the spec as soon as the issue exists.
- Read the output of every `gh` command; if a call fails or returns something unexpected, stop — never assume success.
- Say in the summary how children were linked: the epic checklist plus sub-issues, or the checklist only.

### MUST NOT DO

- Create anything for a `Draft` spec, or before the confirmation.
- Create an issue for a row that already has one, or a second epic.
- Add assignees, milestones, or project boards unless the developer asked.
- Change or delete a label that already exists in the repository.
- Add, drop, rename, or reorder a heading or field of the project's issue template, or use a field value the template does not list.
- Choose a priority on the developer's behalf.
- Close, delete, retitle, or rewrite the body of an existing issue. The only edit to an existing issue is the child list in the epic body.
- Edit the spec beyond `references/write-back.md`: never renumber an ID, never change `Status`, never touch sections 1–14 or the wording of a `W-` row.
- Paraphrase, shorten, or "improve" a rule or criterion when copying it into an issue.
- Invent a dependency, an acceptance criterion, or a skill mapping the spec does not give.
- Copy anything that looks like a real credential, token, or real personal data into an issue — stop and report it; the spec must be fixed first.
- Push, commit, or open a pull request. The developer commits the spec.

## Output Templates

When creating issues from a spec, provide:

1. **Before creating** — the plan:

   ```
   Spec:   docs/specs/<flow-name>/spec.md — Approved
   Repo:   <owner>/<repo> (<visibility>)
   Standard: .github/ISSUE_TEMPLATE/<file> (feature request)  |  kit default — the project has no template
   Labels:  enhancement (+ epic on the epic)      Priority: <not chosen yet>
   Epic:   feat(<flow-name>:epic): <flow name>                          area: web   create
   Issues (creation order):
     W-01  feat(<flow-name>:W-01): <piece of work>       feature slice   area: web   depends on: —            create
     W-03  feat(<flow-name>:W-03): <piece of work>       component       area: web   depends on: —            skip (#14 exists, open)
     W-04  feat(<flow-name>:W-04): <piece of work>       screen wiring   area: web   depends on: W-01, W-03   create   open question: Q-05
   Labels to create: epic
   Linking: checklist in the epic, plus sub-issues if the repository supports them
   Not set: milestones, projects, assignees beyond the template's

   Which priority for these issues — High, Medium, or Low? (one for all, or name rows that differ)

   Create 1 epic and 2 issues in <owner>/<repo>? (yes / no)
   ```

2. **After creating** — the summary:

   ```
   Epic:     #12 <url>
   Created:  W-01 → #13, W-04 → #15
   Skipped:  W-03 → #14 (already existed)
   Linked:   checklist in the epic + sub-issues  |  checklist in the epic only (sub-issue call failed: <reason>)
   Open questions named in issues: Q-05 in #15
   Spec updated: Issue column, Epic row, change log, Last updated — not committed
   Next: commit and push the spec, then start with #13 (W-01) — /agentic-dev-kit:feature-slice
   ```

3. The updated spec file, with only the edits in `references/write-back.md`.

The "next" line names the first created or still-open issue with no unfinished dependency, and the command or skill from the kind table.

## Knowledge Reference

GitHub issues, gh CLI, gh issue create, gh label create, gh api, sub-issues, epic, task list, labels, idempotency, dependency order, topological sort, functional spec, work breakdown, acceptance criteria, stable IDs, write-back, change log, docs/specs
