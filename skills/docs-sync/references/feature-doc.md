# The feature doc

One short page per flow at `docs/features/<flow-name>.md`, using the same kebab-case
flow name as `docs/specs/<flow-name>/`. It is for a developer opening the flow for the
first time: where the code is, what it talks to, how to try it, what is missing.

It does **not** repeat the spec. Behaviour, rules, and acceptance criteria stay in the
spec and are linked.

## Where each fact comes from

| Section | Source | If not found |
| --- | --- | --- |
| What it is | The spec's section 1 and 2, cut to one paragraph | — (a flow with no spec gets no feature doc) |
| Routes | Files under `apps/web/app/**` that render the flow's screens; the URL path is the folder path | Leave the row out |
| Native screens | `apps/native/src/screens/**` and the navigator that registers them (`web+native` only) | Leave the row out |
| Feature slices | `packages/core/src/features/<x>/` folders the flow's routes and components import | Leave the row out |
| Stores | Store files the flow imports | Leave the row out |
| Components | `packages/ui-web` (and `packages/ui-native`) components the flow's screens render | Leave the row out |
| APIs | The endpoint entries, remote methods and hooks in the slices above, matched to the spec's `API-` IDs | List the `API-` ID with `not found in code` |
| Run it locally | `scripts` in the root and app `package.json` files; the project README; variable **names** in the environment example file | Write `unknown` and say what was looked for |
| Known gaps | The spec's open questions with no answer; section 15 rows not `built`; `pending` or `unknown` APIs | Write `None` |

Every path written must exist in the repository at the time of writing. Every command
must be a script that exists in a `package.json`, or a command the README already gives.
Do not compose a command from what the stack usually uses. Do not read real environment
files — only the example file, and only variable names.

## Template

````markdown
# <Flow name>

| | |
| --- | --- |
| Spec | [docs/specs/<flow-name>/spec.md](../specs/<flow-name>/spec.md) — <Draft \| Approved> |
| Last synced | <YYYY-MM-DD> |

## What it is

<One paragraph: who uses the flow and what it lets them do, start to end. For the
steps, rules and acceptance criteria, see the spec.>

## Where the code lives

| What | Path | Notes |
| --- | --- | --- |
| Route | `apps/web/app/<path>/page.tsx` | `/<url-path>` — S-01 |
| Feature slice | `packages/core/src/features/<x>/` | <hooks the flow uses> |
| Store | `<path to the store file>` | <what it holds> |
| Component | `packages/ui-web/src/<ComponentName>/` | S-02 |
| Native screen | `apps/native/src/screens/<ScreenName>.tsx` | S-01 |

## APIs it calls

| Spec ID | Method and path | Endpoint key | Hook | Spec status |
| --- | --- | --- | --- | --- |
| API-01 | <GET /api/method/…> | `<key in api/endpoints.ts>` | `<useHookName>` | <ready \| pending \| unknown> |

## Run it locally

1. <Command — a script found in a package.json, named with the file it came from>
2. <Environment variables the flow needs — names only, from the example file>
3. <Where to go: the route's URL path, and any state needed to reach it>

## Known gaps

**Open questions**

- Q-04 — <question, shortened> (blocking)

**Not built yet**

- W-03 — <piece of work> (not started)

## Notes

<Hand-written. Docs sync never changes this section.>
````

## Rules for writing it

- **Short.** A row per thing, not a paragraph per thing. If a table would have more than
  about fifteen rows, list folders rather than files.
- **Link, do not copy.** Spec IDs (`S-`, `API-`, `Q-`, `W-`) are written as plain IDs
  beside the spec link; their content is not restated.
- **A route's URL path** is read from the folder structure under `apps/web/app/`
  (route groups in parentheses are not part of the URL; dynamic segments are written as
  they appear, e.g. `[id]`). If the routing cannot be read with confidence, give the file
  path only.
- **Only what the flow uses.** A slice or component is listed when the flow's own files
  import it — not because it sits nearby.
- **A section with nothing found keeps its heading** and says `None found` or `unknown`,
  so the reader knows it was looked for.
- **"Run it locally" with nothing found** reads: `unknown — no run script or setup steps
  were found in package.json or the README`. Do not fill it in from memory of how the
  stack is usually started.
- **Known gaps come from the spec as it will be after this run** — including the `[drift]`
  questions and build statuses the same plan adds.

## Updating an existing page

- Read the page first. Compare section by section with what the sources give now, and
  plan a diff only for the sections that differ.
- A path that no longer exists is removed; a new one is added. Show both in the diff.
- `Notes` is never touched. Anything else a person added inside the other sections is
  shown in the diff before it would be changed — if the developer wants it kept, move it
  to `Notes`.
- A page that does not follow this template (written by hand before this skill was used)
  is not restructured. Show what is out of date and offer the specific corrections, or
  offer the template as a replacement — the developer chooses.
- `Last synced` changes only when another part of the page is being written.
- If `docs/features/` does not exist, creating the page creates the folder. Say so in
  the plan.
