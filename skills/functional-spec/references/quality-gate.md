# Quality gate

Run every check before every save. A failed check does not stop the save — it stops the
spec from being `Approved`. Report each failure with the ID it concerns.

## Checks

| # | Check | Fails when |
| --- | --- | --- |
| 1 | **Screens are sourced** | A screen has neither a Figma frame link nor `no design` plus an open question. |
| 2 | **Screens are used** | A screen in section 4 appears in no step, or a step names a screen not in section 4. |
| 3 | **Steps lead somewhere** | A step's `Next` is empty, or points at an ID that does not exist. |
| 4 | **The flow can end** | No step ends in `End: <state>`, or the success state named in section 2 is never reached. |
| 5 | **Branches are complete** | A rule has only its positive outcome, or its source is blank. |
| 6 | **APIs are honest** | An API has no status, or is `ready` without a real request and response sample. |
| 7 | **Failures are handled** | An API error response, or an input with a validation rule, has no `E-` row. |
| 8 | **States are covered** | A screen has a blank cell in section 9 (use `n/a` when a state cannot occur). |
| 9 | **Offline is decided** | Section 8 leaves `Read offline?` or `Written offline?` blank for any row. |
| 10 | **Criteria are testable** | An acceptance criterion is vague ("works correctly", "is fast", "looks right"), bundles several results, or has an empty `Covers`. |
| 11 | **Everything is covered** | An `F-` or `R-` ID appears in no acceptance criterion. |
| 12 | **Work is complete** | An `API-` ID is in no feature-slice row, or an `F-` or `R-` ID is in no screen-wiring row, or either is in two rows of the same kind. |
| 13 | **Questions have owners** | An open question has no one to answer it, or no yes/no on blocking. |
| 14 | **Native is addressed** | Mode is `web+native` and section 12 has no rows and no statement that nothing differs. |
| 15 | **Nothing invented** | Any rule, message, API field, or screen in the draft that the PM, Figma, or the codebase did not supply. Move it to Open questions. |
| 16 | **No secrets** | The file contains a real credential, token, key, or real personal data. This one **does** block the save — replace with a placeholder first. |
| 17 | **IDs are stable** | On update: an existing ID was renumbered, reused, or deleted instead of marked `removed`. |

## Deciding the status

- All checks pass, no blocking open question, and the PM confirmed the draft →
  `Approved`.
- Anything else → `Draft`.

**Before asking the PM to confirm:** if the header's `Challenged` row says `not yet`,
say so — a developer has not looked at this spec. The PM may still approve; it is their
call, and the warning is not a failed check.

## Report

```
Gate: 15 of 17 passed — status Draft
  ✗ 5  Branches are complete   R-03 has no outcome for the negative case → Q-04
  ✗ 6  APIs are honest         API-02 is marked ready with no sample
Open questions: 4 (2 blocking: Q-01, Q-04)
```
