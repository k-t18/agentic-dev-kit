# Combined report

One report per run. It is printed in the conversation when the run stops or finishes,
and the same text goes into the pull request body.

## Order of the report

1. Header — branch, base, issue, spec, piece of work.
2. The stage table.
3. Each stage's findings, with their IDs.
4. Not run.
5. Still to verify by a human.
6. Summary line.

## Template

```
## Ship report — <branch>
Base: <base>  ·  Issue: #<number>  ·  Spec: docs/specs/<flow-name>/spec.md
Work: W-nn <title>  ·  covers <IDs>
Warnings: <from preflight> | none

### Result: CLEAR TO OPEN PR | STOPPED at <stage>

| # | Stage | Result | 🔴 | 🟡 | ⚪ | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| a | Lint | passed | – | – | – | `<script>` |
| a | Type-check | passed | – | – | – | `<script>` |
| a | Tests | not run — no script defined | – | – | – | |
| b | Design QA | passed | 0 | 1 | – | 2 components |
| c | PR check | passed | 0 | 2 | 1 | rerun after fix |
| d | Security check | passed | 0 | 0 | 1 | |
| e | Flow test | not run — declined | – | – | – | AC-03, AC-04 not checked |
| f | Docs sync | passed | – | – | – | 3 files updated |

### Findings

#### Lint · type-check · tests
  (on a failure) <script> exited <code>
    <file>:<line>  <message>
    … <N> more

#### Design QA
  <ComponentName> — PASS
    🟡 check 2  radius: Figma 8px, code 12px
  <ComponentName> — not run — no Figma source recorded

#### PR check
  🟡 P-02  <spec ID or rule>  <one line>  — left open
  🟡 P-03  <…>                            — fixed in <short sha>
  ⚪ P-05  <…>

#### Security check
  ⚪ X-01  <…>

#### Flow test
  AC-03  Pass
  AC-04  Could not test — <reason>

#### Docs sync
  Updated: <files>   | No changes needed

### Not run
  - Tests — no script defined
  - Flow test — declined (AC-03, AC-04 not checked)

### Still to verify by a human
  - <item>

### Summary
Stages: <N> passed · <N> failed · <N> not run   ·   Findings open: 🔴 N · 🟡 N · ⚪ N
```

## Rules

- **Every stage has a row**, including the ones that never started. After a stop they
  read `not run — chain stopped at <stage>`.
- **Findings keep their own IDs** — `P-nn`, `X-nn`, `AC-nn` — and their own severity.
  Do not renumber, merge, reword into a different meaning, or re-rate them.
- **One line per finding.** The stage's full report was already shown; the combined
  report points at it.
- **Say what happened to each finding:** `left open` or `fixed in <short sha>`. Counts
  in the table and the summary are the findings still open.
- **A stage with nothing to say still appears** under Findings with "no findings".
- **"Not run" repeats every `not run` row** with its reason and what went unchecked
  because of it. It is its own section so a reviewer cannot miss it.
- No real credentials, tokens, sign-in details, or personal data in the report. A
  finding that quotes one is shown with the value masked.

## Still to verify by a human

This list is never empty. Build it from the spec and from the run:

| Source | Item to add |
| --- | --- |
| Project mode `web+native` | Native behaviour on real devices for this piece — none of the stages run on a device. Name the section 12 rows for the covered steps. |
| Spec section 8 — data the piece writes offline | Sync and data-loss paths: queued writes replaying after reconnect, a write made twice, the app closed mid-sync, a conflict with a server change. |
| Spec section 11 — sensitive data the piece touches | That the data is not kept, shown, or logged where the spec says it must not be, on a real device and a real session. |
| Every `not run` stage | What that stage would have checked. |
| Every acceptance criterion Could not test or not checked | The `AC-nn` and why. |
| Design QA `not run` for a component | A manual comparison of that component with Figma. |
| 🟡 findings left open | That the reviewer accepts each one. |
| Preflight warnings | The branch is behind the base; the spec was `Draft`; uncommitted files were left out. |
| Always | The stages check the web app on the developer's machine. Behaviour in the deployed environment, and CI, are confirmed on the PR. |

When there is no spec, say that the sections above could not be read and that the
reviewer must decide what needs checking.
