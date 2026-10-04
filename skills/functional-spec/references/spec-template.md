# Spec template

Copy this structure into `docs/specs/<flow-name>/spec.md`. Keep every section and its
number, even when the content is "None". Text in `<angle brackets>` is replaced; the
example rows show the shape only and are deleted.

````markdown
# <Flow name> — functional spec

| | |
| --- | --- |
| Status | Draft |
| Owner | <PM name> |
| Figma | <link to the section or page> |
| Project mode | <web-only \| web+native> |
| Challenged | not yet |
| Last updated | <YYYY-MM-DD> |

## 1. Summary

<Two or three sentences: what this flow lets the user do, start to finish.>

## 2. Purpose

- **User:** <who>
- **Goal:** <what they are trying to get done>
- **Today:** <what happens without this flow>
- **Success:** <the end state that means it worked>

## 3. Scope

**In scope**

- <…>

**Out of scope**

- <…>

**Depends on**

- <another flow, feature, or backend work — or "Nothing">

## 4. Screen inventory

| ID | Screen | Type | Figma frame | Reached from | Notes |
| --- | --- | --- | --- | --- | --- |
| S-01 | <name> | page | <link> | <entry point> | <variants, if any> |
| S-02 | <name> | modal | <link \| no design> | S-01 | |

## 5. Flow steps

| ID | Screen | User action | System response | API | Next |
| --- | --- | --- | --- | --- | --- |
| F-01 | S-01 | <what the user does> | <what the system does> | API-01 | F-02 |
| F-02 | S-02 | <…> | <…> | — | See R-01 |
| F-03 | S-03 | <…> | <…> | — | End: <end state> |

`Next` is a step ID, a rule ID when the path branches, or `End: <state>`.

## 6. Branches and rules

| ID | Applies at | Condition | Then | Source of the fact |
| --- | --- | --- | --- | --- |
| R-01 | F-02 | <condition is true> | Go to F-03 | <API field \| stored data \| user choice> |
| R-01 | F-02 | <condition is false> | <what happens> | <…> |

One row per outcome. A rule is complete only when every outcome has a row.

## 7. API contracts

### API-01 — <what it does>

| | |
| --- | --- |
| Status | <ready \| pending \| unknown> |
| Method and path | <GET /api/method/…> |
| Used by | F-01 |
| Owner | <who builds or confirms it> |

**Request**

```json
{ "<field>": "<type or placeholder value>" }
```

**Response**

```json
{ "message": { "data": { "<field>": "<type or placeholder value>" } } }
```

**Error responses**

| When | Response | Handled by |
| --- | --- | --- |
| <condition> | <status and body> | E-01 |

## 8. Data and offline

| Data | Lives in | Read offline? | Written offline? | Notes |
| --- | --- | --- | --- | --- |
| <thing> | <server \| local DB \| client state> | <yes \| no> | <queued \| blocked \| n/a> | <…> |

**When there is no connection:** <what the user can and cannot do, and what they see.>

**When a queued save is later rejected:** <what happens, or "n/a — no offline writes".>

## 9. Screen states

| Screen | Loading | Empty | Error | Disabled |
| --- | --- | --- | --- | --- |
| S-01 | <what shows> | <what shows \| n/a> | <what shows> | <when, and what shows> |

## 10. Validation and errors

| ID | Where | Trigger | What the user sees | What happens next |
| --- | --- | --- | --- | --- |
| E-01 | F-01 | <invalid input or failure> | <message or behaviour> | <stays, retries, goes to …> |

## 11. Sensitive data

| Data | Kept on device? | Shown masked? | Notes |
| --- | --- | --- | --- |
| <thing> | <no \| yes, until …> | <yes \| no> | <must not be logged, etc.> |

<"None" if the flow handles no personal, identity, payment, or secret data.>

## 12. Native notes

<For web-only projects: "Not applicable — web-only project".>

| Step | What differs on a device | Decision |
| --- | --- | --- |
| F-03 | <camera, file picker, permission, deep link, SDK, backgrounding> | <agreed behaviour \| Q-nn> |

## 13. Acceptance criteria

| ID | Criterion | Covers |
| --- | --- | --- |
| AC-01 | Given <state>, when <action>, then <observable result>. | F-01, R-01 |

Each criterion is one statement that can only pass or fail.

## 14. Open questions

| ID | Question | Raised at | Who answers | Blocks approval? | Answer |
| --- | --- | --- | --- | --- | --- |
| Q-01 | <the unanswered point> | F-02 | <name or role> | yes | — |

Answered questions stay in the table with the answer filled in.

## 15. Work breakdown

| ID | Piece of work | Kind | Covers | Depends on |
| --- | --- | --- | --- | --- |
| W-01 | <short title> | <feature slice \| client state \| component \| screen wiring \| native port> | API-01, F-01 | — |

Proposed only, until `/spec-issues` records issue numbers in this table.

## 16. Change log

| Date | Change | By |
| --- | --- | --- |
| <YYYY-MM-DD> | First draft | <name> |
````

## Column notes

- **Added by other commands** — the spec writer never adds or edits these, and keeps
  them when updating a spec: an `Epic` header row and an `Issue` column in section 15
  (`/spec-issues`); a `Build` column in section 15 (`/docs-sync`); open questions tagged
  `[challenge]`, `[pr-check]`, `[security]` or `[drift]` in section 14, which are
  answered like any other.
- **Challenged** (header): `not yet` until a developer runs `/spec-challenger`, which
  fills in the date, who ran it, and the questions it added. The spec writer never sets
  this row itself.
- **`[challenge]` questions** (section 14): raised by the developer's challenge. They
  are answered like any other open question.
- **Type** (section 4): `page` or `modal`.
- **Source of the fact** (section 6): where the app learns the condition. A rule whose
  source nobody can name is an open question.
- **Status** (section 7): `ready` needs a real request and response sample; `pending`
  means agreed but not built; `unknown` means not yet discussed with the backend.
- **Lives in** (section 8): `server`, `local DB` (the offline database), or
  `client state` (in-memory or persisted store).
- **Covers** (sections 13 and 15): the IDs the row verifies or builds. Every `F-` and
  `R-` ID should appear in at least one acceptance criterion.
- **Placeholders**: sample values in requests and responses are invented stand-ins,
  never real credentials or real personal data.
