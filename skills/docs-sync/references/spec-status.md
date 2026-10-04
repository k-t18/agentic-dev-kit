# The spec: build status and drift

The spec is the PM's document. This skill adds to three places in it and changes nothing
else:

| Place | Edit |
| --- | --- |
| Section 15, Work breakdown | The `Build` column |
| Section 14, Open questions | New `[drift]` rows |
| Section 16, Change log, and `Last updated` in the header | One line, one date |

Sections 1–13, every other header row (`Owner`, `Figma`, `Project mode`, `Challenged`,
`Epic`), the `Issue` column, and the sentence under the section 15 table are left exactly
as they are. `Status` is left alone too, with the one exception under "Status" below.

## Build status (section 15)

### The column

The column is named `Build` and is always the **last** column of the section 15 table.
If the spec predates it, add it: the header cell, the separator cell, and a value on
every row. Find the other columns by header name — the table may or may not have an
`Issue` column, and its position is not fixed. Never move, rename, or remove a column.

```markdown
| ID | Piece of work | Kind | Covers | Depends on | Issue | Build |
| --- | --- | --- | --- | --- | --- | --- |
| W-01 | `<domain>` slice: read and create | feature slice | API-01, API-02 | — | #<n> | built — #<pr> |
| W-02 | Multi-step progress store | client state | section 8 row `<data>` | — | #<n> | in progress — #<pr> |
| W-03 | Wire `<screen>` | screen wiring | F-01–F-03, R-01, E-01 | W-01, W-02 | #<n> | not started |
```

### The values

| Value | Means | Reference |
| --- | --- | --- |
| `not started` | No code for this piece was found. | none |
| `in progress` | Some of what the row covers exists, or the work is on a branch or open PR that is not yet agreed as ready to merge, or it is uncommitted. | ` — #<pr>` if a PR exists |
| `built` | Code for everything the row covers was found, and it is merged into the base branch — or it is in the PR being synced and the developer confirms that PR is the finished piece. | ` — #<pr>`, or ` — <short sha>` of the commit on the base branch when there is no PR |
| `—` | The row is marked `removed`. | none |

`built` says the code exists. It does not say the acceptance criteria pass — that is
`/agentic-dev-kit:flow-test`.

### Matching work to rows

Evidence that a change builds a `W-` row, strongest first:

1. The PR closes the issue in that row's `Issue` column.
2. The PR body, issue, or a commit message names the `W-` ID.
3. The changed files are what the row's `Kind` and `Covers` describe:

   | Kind | Look in |
   | --- | --- |
   | `feature slice` | `packages/core/src/features/<x>/` and the endpoints for the row's `API-` IDs |
   | `client state` | the store the row names |
   | `component` | `packages/ui-web/` (the component named in the row) |
   | `screen wiring` | `apps/web/app/**` routes for the row's screens and steps |
   | `native port` | `packages/ui-native/` and `apps/native/src/**` |

Then check the row's `Covers` IDs against the code: an `API-` ID is covered when its
endpoint, remote method and hook exist; an `S-` ID when its route or component exists;
`F-`, `R-` and `E-` IDs when the wiring that performs them can be pointed at. All covered
→ `built` (subject to the merge condition above). Some → `in progress`. If an ID cannot be
confirmed either way, the row is `in progress` and the plan says which ID was not found.

Rows this change does not touch keep their current value. When a flow name was given
with no PR, every row is checked against the code as it stands.

**Never lower a status silently.** If a row says `built` and its code can no longer be
found, do not plan a downgrade — report it to the developer and ask.

## Drift (section 14)

Drift is a place where the code that was built does something different from what the
spec says, or does something the spec does not state. The spec is not corrected to match.
Each difference becomes an open question for the PM.

### What counts

| The spec says | The code does |
| --- | --- |
| A step leads to one screen or step (`F-`) | It leads somewhere else, or the step is missing |
| A rule has a stated outcome (`R-`) | A different outcome, a missing outcome, or an extra condition |
| An API's method, path, request or response (`API-`) | A different endpoint, field, or shape |
| A message or behaviour on failure (`E-`) | A different message, or the failure is not handled |
| What a screen shows while loading, empty, failed, disabled (section 9) | A state is missing or different |
| What is read or written offline (section 8) | Queued where the spec says blocked, or the reverse |
| How sensitive data is kept or masked (section 11) | It is stored, logged, or shown differently |
| Nothing | The code applies a rule, a screen, or a branch the spec does not mention |

Not drift: visual differences from Figma (that is `/agentic-dev-kit:design-qa`); code
quality or layering problems (`/agentic-dev-kit:pr-check`); work that is simply not built
yet (that is build status).

### The bar for raising one

Raise a drift question only when both sides can be quoted: the spec row by ID, and the
code by file and line. If the code could not be read in full, or the difference is a
guess, do not raise it — list it in the plan under "could not verify" instead.

Look only at what this change covers: the `Covers` IDs of the `W-` rows it builds. When a
flow name was given with no PR, look at every row whose status is `built` or
`in progress`.

### The row

```markdown
| Q-07 | [drift] Spec: <what the spec says>. Code: <what the code does> (`<path>:<line>`, #<pr>). Which is correct? | R-02 | <PM> | no | — |
```

- **ID** — the next free `Q-` number. Never renumber or reuse; never fill a gap left by
  a removed row.
- **Question** — starts with `[drift]`, then the three parts in this order: what the
  spec says, what the code does, where (file and line, plus the PR or commit). One
  difference per row.
- **Raised at** — the spec IDs the difference concerns. For behaviour the spec does not
  state, the nearest step or screen ID.
- **Who answers** — the spec's `Owner`.
- **Blocks approval?** — `no` by default. See "Status" below.
- **Answer** — `—`. Never filled in by this skill.

If section 14 ends with a count line, recount it.

### Not raising the same one twice

Before adding a row, read section 14. Skip the difference when a `[drift]` row raised at
the same IDs already describes it — whether it is still open or already answered. An
answered question whose code still differs is not raised again; it is listed in the
feature doc under Known gaps, with the `Q-` ID.

A difference already recorded as a `[challenge]` or plain open question is also skipped.

### Status

This skill never sets `Status` to `Approved`. By default it does not change `Status` at
all: a drift question records that the code departed from the spec, which does not by
itself make the spec's rules unconfirmed.

If the developer says a difference should block — the spec cannot be relied on until the
PM answers — mark that row `yes` and set `Status` to `Draft`, the same way a blocking
`[challenge]` question does. Show this in the plan as its own numbered edit.

### How the PM resolves it

The PM runs `/agentic-dev-kit:spec-builder` on the spec. Either the spec was right — the
answer says so and the code is fixed in a follow-up — or the code was right, and the PM
changes the rule and logs it. Both are the PM's edits, not this skill's.

## Change log and Last updated (section 16, header)

Only when at least one other spec edit is being written:

- One line in section 16: the date, what changed, the developer's name (from
  `git config user.name`; ask if empty).

  ```markdown
  | <YYYY-MM-DD> | Docs sync: W-01 built (#<pr>), W-02 in progress; 1 drift question added (Q-07) | <name> |
  ```

- `Last updated` in the header set to the same date.

A run that changes nothing else in the spec adds no line and leaves the date alone.
