# The test plan

The plan is made from the spec and shown to the developer **before** anything is clicked.

No spec? Sections 1 and 2 are replaced by `without-a-spec.md` (the checks come from a
GitHub issue or from the developer); sections 3 to 5 apply as written.

## 1. Read the spec

| Section | Used for |
| --- | --- |
| Header | `Status`, `Project mode`. A `Draft` spec can be tested — say that it is a draft, and that criteria touching open questions are skipped. |
| 4 Screen inventory | Which screen each step is on, and how it is reached. |
| 5 Flow steps, 6 Branches and rules | How to get to a criterion's Given, and what "next" means. |
| 7 API contracts | Which request to look for in the network, and each API's status. |
| 9 Screen states, 10 Validation and errors | What should be on the page in loading, empty, error, and disabled states, and the exact messages. |
| 11 Sensitive data | What must be masked on screen and must not be copied into the report. |
| 12 Native notes | What goes on the list for a human. |
| 13 Acceptance criteria | The tests. |
| 14 Open questions | Which criteria are not decided yet. |
| 15 Work breakdown | Resolving a `W-` subset. |

## 2. Choose the criteria in scope

- No subset → every row in section 13 that is not marked `removed`.
- `AC-` IDs or a range → those rows. An ID that does not exist is reported, not guessed.
- A `W-` ID → every criterion whose `Covers` shares at least one ID with that work
  item's `Covers`. List which criteria that came to, so the developer can correct it.

## 3. Decide run or skip

Check each criterion in scope against this table, top to bottom. The first row that
applies decides.

| Skip when | Reason to print |
| --- | --- |
| An ID in its `Covers`, or the criterion itself, is the subject of an open `Q-` with no answer | `undecided — Q-nn` |
| It depends on an API whose status is `pending` or `unknown`, and that API is not mocked | `API-nn is pending and not mocked` |
| The APIs are mocked and the Then is about what the real server does | `needs a real backend` |
| The Given is a state the run has no way to reach (see below) and the developer cannot set it up | `cannot reach: <state>` |
| The Then cannot be observed in a browser (an email arriving, a push notification, a device behaviour) | `not observable in the browser` |
| It is about native behaviour only | `native — manual` |
| The screen or step it needs has not been built yet | `not built — <W-nn if known>` |
| It needs a credential typed and the developer will not perform that step | `needs credentials — manual` |
| Its When is a hard-to-undo action and the developer says not to perform it | `not run at developer's request` |

**States the run often cannot reach on its own:** no connection or a slow connection, a
server error on a specific request, an expired session, a second user acting at the same
time, a record in a particular server-side state, a time- or date-dependent condition.
Ask the developer once, in the plan, whether they can set each of these up by hand (for
example with the browser's own developer tools, or by preparing a record). If yes, the
criterion runs with a "developer sets up" step. If no, it is skipped.

Do not skip a criterion because it looks hard, or because it is likely to fail.

## 4. Work out the steps

For each criterion that will run, write the concrete steps from its
"Given … when … then …":

- **Given** — the route, the data, and the prior steps (by `F-` ID) needed to stand in
  that state.
- **When** — the single action.
- **Then** — what will be checked: text or element on the page, the route afterwards,
  the request sent and its status. Take messages word for word from section 10.

Mark any step that **changes data or leaves the app** — see "Stop and ask" in
`executing-criteria.md`. Mark any step the developer performs.

Order the criteria so that one run through the flow serves several of them, and so that
criteria which destroy or use up data come last.

If a criterion cannot be turned into steps because it is vague or bundles several
results, do not reinterpret it. Skip it with `criterion not testable as written` and
list it under questions for the spec.

## 5. Show the plan

```
Test plan — <flow-name>   (spec: Approved, last updated <date>)
App: <url> · Backend: <real test backend | mocked: API-01, API-02> · Signed in as: <role>
Scope: <all criteria | AC-03, AC-04 | W-03 → AC-02, AC-05, AC-06>

Will run (7)
  AC-01  Given <…>, when <…>, then <…>
         Steps: open <route> → <action> → check <result>
  AC-04  …
         Steps: … → [changes data] submit the form → check <result>
  AC-07  …
         Steps: [developer sets up] no connection → <action> → check <result>

Skipped (3)
  AC-05  undecided — Q-02
  AC-08  API-03 is pending and not mocked
  AC-10  native — manual

Needs your answer before running
  - AC-04 submits a real record to the test backend. OK to run?
  - Test data to use for <…>?
```

Wait. The developer can remove criteria, add back skipped ones, or supply data. Run only
what they confirm.
