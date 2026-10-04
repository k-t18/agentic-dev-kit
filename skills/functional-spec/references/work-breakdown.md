# Work breakdown

Section 15 proposes how the flow could be cut into pieces of work. It is a proposal in
the document — **do not create issues, branches, or files from it.**

## Cut along the project's layers

The build order of this stack decides the pieces. Each piece maps to the skill that
builds it.

| Kind | One piece is | Built with | Comes from |
| --- | --- | --- | --- |
| `feature slice` | The data path for one domain: types, endpoints, repo, hooks, and offline writes | `feature-slice` | Section 7 APIs, section 8 |
| `client state` | One store for state that is not server data (multi-step progress, selections) | `zustand-slice` | Section 8 rows marked `client state` |
| `component` | One reusable UI piece not already in `packages/ui-web` | `web-component` | Section 4 screens |
| `screen wiring` | One screen or a tight group of screens: route, layout, hooks connected, rules and errors applied | — | Sections 5, 6, 9, 10 |
| `native port` | The approved web components and screens for this flow, ported | `web-to-native` | Section 12 (`web+native` only) |

## Rules for cutting

- **Group APIs by domain**, not one piece per endpoint. Endpoints that share a resource
  belong in one feature slice.
- **Check what exists first.** A slice already in `packages/core/src/features/` or a
  component already in `packages/ui-web` is extended, not rebuilt — say so in the title.
- **One piece should be reviewable on its own.** If a screen-wiring piece covers more
  than a handful of steps, split it at a natural boundary in the flow.
- **Order by dependency:** feature slices and client state first, components next,
  screen wiring after both, native port last.
- **Blocked pieces are marked.** A piece that depends on an API with status `pending` or
  `unknown`, or on a blocking open question, names that ID in `Depends on`.
- **Every `API-` ID lands in exactly one feature-slice piece, and every `F-` and `R-`
  ID in exactly one screen-wiring piece.** An ID in none means work is missing; an ID
  in two of the same kind means the cut is unclear.

## What a row looks like

| ID | Piece of work | Kind | Covers | Depends on |
| --- | --- | --- | --- | --- |
| W-01 | `<domain>` slice: read and create | feature slice | API-01, API-02 | — |
| W-02 | Multi-step progress store | client state | section 8 row `<data>` | — |
| W-03 | `<ComponentName>` | component | S-02 | — |
| W-04 | Wire `<screen>` | screen wiring | F-01–F-03, R-01, E-01 | W-01, W-03, Q-02 |
| W-05 | Port flow to native | native port | S-01–S-04 | W-04 |
