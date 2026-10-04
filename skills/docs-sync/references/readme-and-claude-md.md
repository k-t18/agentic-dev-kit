# README and CLAUDE.md

These are touched **only** when the change altered something they describe. Most runs
plan no edit here, and say so in one line.

Scope: the **project's** files — the root `README.md`, app and package READMEs, the root
`CLAUDE.md`, and nested ones such as `apps/web/CLAUDE.md`. Never the kit's own files or
templates under `${CLAUDE_PLUGIN_ROOT}`; to re-install or merge those, the developer
runs the kit's init command.

## What triggers an edit

| The change did this | Evidence in the diff | Check these documents |
| --- | --- | --- |
| Added, renamed, moved or removed an app or package | A new or removed folder under `apps/` or `packages/`; workspace config | README structure section; `CLAUDE.md` monorepo map |
| Added, renamed or removed a script | `scripts` in a `package.json`; pipeline config | README setup and commands; the feature doc's "Run it locally" |
| Added, renamed or removed an environment variable | The environment example file; a new variable read in code | README setup; `CLAUDE.md` environment section; an environment doc the project keeps, if either file points to one |
| Added or replaced a dependency the documents name | `package.json` dependencies | README stack section; `CLAUDE.md` stack list |
| Changed a convention | Lint, compiler, or build configuration; a new top-level folder pattern; a renamed alias | `CLAUDE.md` — see "Conventions" below |
| Added a flow's feature doc for the first time | The new `docs/features/<flow-name>.md` | README, only if it already has a list of feature docs |

None of these in the diff → no edit to README or `CLAUDE.md`. A change inside existing
folders that follows existing conventions needs neither.

## README

- Correct what is now wrong; add what is now missing. Match the README's existing
  headings, tone, and level of detail.
- Do not restructure it, add new sections it never had, or add badges, tables of
  contents, or marketing text.
- A command written into the README is a script that exists. A variable is a name from
  the example file, with no value.
- If the README has no section where the fact belongs, propose the smallest addition and
  let the developer place it.

## CLAUDE.md

`CLAUDE.md` files are guardrails: Claude Code reads them in every session, so a wrong
edit changes how all later work is done. Treat them more strictly than any other
document.

### What may be edited

| Allowed | Example |
| --- | --- |
| Add a fact that is now true | A new package in the monorepo map; a new variable name in the environment section |
| Correct a fact that is now false | A renamed folder, script, package scope, or branch name |
| Add a pointer | A new row in a skills or docs index the file already keeps |

### What may not be edited on this skill's initiative

- An existing rule, invariant, or never-do entry — not weakened, not removed, not
  reworded, not reordered, not "clarified".
- The project mode line.
- A new rule or convention. A rule is the team's decision, not a finding.

### Conventions

When the diff shows the code now does something an existing rule forbids, or follows a
pattern no rule covers:

1. Do not edit the rule to match the code.
2. Report it to the developer in the plan, under its own heading: the rule (file and
   section), what the code does (file and line), and that it needs a decision.
3. If the developer decides the convention has changed and gives the wording, write
   exactly their wording as a normal `CLAUDE.md` edit, with the diff and its own yes. If
   they do not, write nothing — the difference stays in the summary as unresolved.

Projects set up with this kit state that `CLAUDE.md` is updated first, then the code. A
convention that changed in code without `CLAUDE.md` changing is therefore something to
raise, not something to tidy up.

### How to ask

- Show the **exact diff** — the lines removed and added, with enough context to see
  which section they are in.
- Ask about each `CLAUDE.md` edit by itself. An answer of "all" to the rest of the plan
  does not cover it.
- Edit only the lines in the agreed diff. Do not reflow, reformat, or renumber the
  surrounding text.
- One fact per edit, so the developer can accept one and decline another.

## Not adding it twice

Before planning, search the document for the fact — the package name, script name, or
variable name. If it is already stated, in any wording, plan nothing. A fact stated in
the root file is not repeated in a nested one.
