# Read the spec

**Do not run any `gh` command that creates or changes something yet.** This step reads
the spec, decides whether the run may continue, and produces the plan.

## 1. Find the pieces

| Needed | Where | If missing |
| --- | --- | --- |
| Status | Header table, `Status` row | Stop — not a spec in the expected format. |
| Flow name | The folder: `docs/specs/<flow-name>/spec.md` | Use the title line; confirm it in the plan. |
| Recorded epic | Header table, `Epic` row | Normal for a first run — no epic yet. |
| Acceptance criteria | `## 13. Acceptance criteria` | Stop — preflight failure. |
| Open questions | `## 14. Open questions` | Stop — preflight failure. |
| Work breakdown | `## 15. Work breakdown`, at least one `W-` row | Stop — preflight failure. Nothing to create. |
| Recorded issues | `Issue` column in section 15 | Normal for a first run — the column is added on write-back. |

Find sections by their number and heading, not by line position.

## 2. Status gate

An open question is **open** when its `Answer` cell is `—` or empty. It is **blocking**
when `Blocks approval?` is `yes`.

- `Status` is `Approved` and no open question is blocking → continue.
- Anything else → refuse, and create nothing:

```
Cannot create issues: docs/specs/<flow-name>/spec.md is Draft.

Blocking open questions:
  Q-02  <question text>   — answered by <who answers>
  Q-07  <question text>   — answered by <who answers>

The PM answers these by running /agentic-dev-kit:spec-builder docs/specs/<flow-name>/spec.md.
Run /agentic-dev-kit:spec-issues again once the spec is Approved.
```

If the spec is `Draft` with no blocking question, say that instead: the PM has not
confirmed the draft or a quality-gate check failed, and `/agentic-dev-kit:spec-builder`
is where that is resolved.

If the header's `Challenged` row says `not yet`, add one line to the plan saying a
developer has not challenged this spec (`/agentic-dev-kit:spec-challenger`). It is a
warning, not a refusal — the spec is approved.

## 3. Parse section 15

One work item per table row. Columns: `ID`, `Piece of work`, `Kind`, `Covers`,
`Depends on`, and `Issue` when present.

- **Removed rows** — a row marked `removed` gets no issue.
- **Kind** — must be one of the five in `SKILL.md`. Anything else: report and stop.
- **Issue cell** — `—` or empty means no issue. Otherwise read the number from
  `#<n>` (the cell may be a link: `[#13](<url>)`).

### Expanding `Covers`

Turn the cell into a flat set of IDs:

| Written as | Means |
| --- | --- |
| `F-01, R-01` | Those two IDs. |
| `F-01–F-03` or `F-01-F-03` | Every ID of that type from the first to the last number, inclusive. Skip numbers that do not exist in the spec. |
| `section 8 row <data>` | The row of the section 8 table whose `Data` cell matches. Not an ID — carried as text. |

An ID in `Covers` that does not exist in the spec is reported in the plan and left out
of the issue body; it does not stop the run.

### Splitting `Depends on`

| Entry | Becomes |
| --- | --- |
| `W-nn` | A dependency on that child issue. Decides creation order. |
| `Q-nn`, still open | An open question named in the issue body (non-blocking, or the spec would not be Approved). |
| `Q-nn`, answered | A line in the body giving the answer. |
| `API-nn` | A line in the body giving that API's status (`pending` / `unknown`). |
| `—` | No dependencies. |

A `W-` dependency that does not exist, is `removed`, or forms a cycle stops the run
before anything is created.

## 4. Collect the text for each work item

For every ID in the expanded `Covers`, copy the spec's row **as written**:

| ID type | Copy from | What to copy |
| --- | --- | --- |
| `S-` | Section 4 | Screen, type, Figma frame link, reached from, notes |
| `F-` | Section 5 | Screen, user action, system response, API, next |
| `R-` | Section 6 | **Every** row with that ID (one per outcome): applies at, condition, then, source |
| `API-` | Section 7 | The heading line, status, method and path, used by, owner. Point at the spec for the request and response samples rather than copying them. |
| `E-` | Section 10 | Where, trigger, what the user sees, what happens next |
| `section 8 row` | Section 8 | The whole row |

Rows marked `removed` in their own section are left out.

### Matching acceptance criteria

Expand each acceptance criterion's `Covers` the same way. A criterion belongs to a work
item when the two expanded sets share **at least one ID**. Copy the criterion's ID,
text, and `Covers` unchanged.

A work item whose `Covers` holds only `S-` or `API-` IDs often matches no criterion,
because criteria cover steps and rules. Say so in the body ("None reference these IDs
directly") — do not widen the match to find some.

## 5. Check what is recorded

For the `Epic` row and every `Issue` cell with a number, look the issue up
(`github-commands.md` → Verify). Then, for every row with **no** recorded number,
search for an existing issue with the same title — a run that created an issue but
stopped before writing it back would otherwise create a duplicate.

Outcome per row: `create`, `skip (#n exists, <state>)`, `adopt (#n found by title)`,
`recreate (recorded #n not found)`, or a mismatch that stops the run. The table in
`SKILL.md` → Idempotency decides which.

## 6. Order and print the plan

Sort the rows to create so each follows its `W-` dependencies; ties keep spec order.
Print the plan in the shape given in `SKILL.md` → Output Templates, including:

- the target repository and its visibility, exactly as `gh` reports it;
- rows that will be skipped, so the developer sees the whole breakdown;
- the issue standard in use — the project's feature-request template file, or the kit
  default when the project has none (`issue-templates.md`);
- each row's area, and the question asking for the priority;
- labels that will be created;
- any warning: spec not committed or not on the default branch (links will not resolve
  until it is pushed), `Challenged` is `not yet`, an unknown ID in `Covers`.

Ask once. Anything other than a clear yes: create nothing, write nothing, say so.

If every row already has a verified issue and the epic exists, there is nothing to
create: print the summary with everything under "skipped" and do not edit the spec.
