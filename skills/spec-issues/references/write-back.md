# Write-back to the spec

The spec records which issues exist, so the next run creates only what is missing.
These four edits are the **only** changes this skill makes to a spec file.

| # | Edit | Where | When |
| --- | --- | --- | --- |
| 1 | `Issue` column | Section 15 table | After each child issue is created or adopted |
| 2 | `Epic` row | Header table | As soon as the epic is created or adopted |
| 3 | One change-log line | Section 16 | Once, at the end of a run that created or adopted anything |
| 4 | `Last updated` | Header table | Same time as edit 3 |

A run that created and adopted nothing writes nothing.

## 1. The `Issue` column

Section 15 as written by `/agentic-dev-kit:spec-builder` has five columns. If the
`Issue` column is not there, add it as the **last** column — header cell, separator
cell, and one cell on every row:

```markdown
| ID | Piece of work | Kind | Covers | Depends on | Issue |
| --- | --- | --- | --- | --- | --- |
| W-01 | `<domain>` slice: read and create | feature slice | API-01, API-02 | — | [#13](<issue url>) |
| W-02 | Multi-step progress store | client state | section 8 row `<data>` | — | — |
```

- A row with an issue: `[#<n>](<issue url>)`.
- A row with no issue yet, and every `removed` row without one: `—`.
- Fill one cell at a time, right after that issue exists. Do not wait for the end of
  the run.
- A cell that already holds a verified number is never changed. A cell whose recorded
  issue was not found is replaced only after the replacement issue exists.
- Change no other cell of the row. The `ID`, `Piece of work`, `Kind`, `Covers`, and
  `Depends on` text stays byte for byte.

Leave any sentence under the table as it is (for example the template's note that the
breakdown is a proposal). It is outside the four edits; the `Issue` column is the
record of what has been created.

## 2. The `Epic` row

In the header table, directly **above** `Last updated`. Add the row if the spec
predates it:

```markdown
| Challenged | <unchanged> |
| Epic | [#12](<epic url>) |
| Last updated | <YYYY-MM-DD> |
```

If the row exists with a verified number, leave it. Change no other header row —
`Status`, `Owner`, `Figma`, `Project mode`, and `Challenged` stay as they are.

## 3. The change-log line

Append one row to the section 16 table:

```markdown
| <YYYY-MM-DD> | Issues created: epic #12; W-01 → #13, W-04 → #15 | <name> |
```

- List only what **this run** created or adopted. On a re-run that added children to
  an existing epic: `Issues created: W-06 → #21 (epic #12)`.
- Adopted issues are listed as such: `W-03 → #14 (existing issue recorded)`.
- `<name>` is the person running the command — `git config user.name`, or ask.
- Never edit or reorder earlier change-log rows.

## 4. `Last updated`

Set it to today's date, in the format the header already uses.

## What never changes

- **IDs.** No `W-`, `S-`, `F-`, `R-`, `API-`, `E-`, `AC-`, or `Q-` ID is renumbered,
  reused, removed, or added.
- **`Status`.** Recording issue numbers does not change behaviour, so an `Approved`
  spec stays `Approved`. This skill never sets a status.
- **Sections 1–14.** Not a character.
- **Row wording in section 15**, and the order of its rows.

If the spec's tables are shaped differently from the template (extra columns, a
reordered header), make the smallest edit that adds the `Issue` cell and the `Epic`
row, and say in the summary what was different.

## After writing

- Re-read the edited parts of the file: the section 15 table still has the same number
  of rows, every row has the same number of cells as the header, and the header table
  still has every row it had before.
- Do not commit. Tell the developer the spec was modified and should be committed and
  pushed, so the spec link in each issue shows the recorded numbers.
