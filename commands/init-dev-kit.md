---
name: init-dev-kit
description: Install the agentic-dev-kit CLAUDE.md guardrails (root + apps/web) into the current project so the plugin's skills run with their invariants. Copies templates from the plugin, never blindly overwrites.
argument-hint: "[--force to overwrite existing CLAUDE.md files]"
---

# /init-dev-kit

**Arguments:** $ARGUMENTS

Installs the **CLAUDE.md guardrails** that the `agentic-dev-kit` skills depend on
(tokens-only, core layering, the never-do list, web-layering boundary, project mode)
into **the current project**. The skills/agents/commands already come from the plugin;
this command lands the always-on invariants that plugins can't ship on their own.

Templates live inside the plugin and are referenced by the **`${CLAUDE_PLUGIN_ROOT}`**
variable:

- Root invariants → `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE.md`
- Web-shell context → `${CLAUDE_PLUGIN_ROOT}/templates/apps/web/CLAUDE.md`

These live under `templates/` in the plugin — kept separate from the plugin repo's own
root `CLAUDE.md` so the shipped guardrails and the plugin's working instructions can
diverge. Always read from `templates/`, never from the plugin's own root `CLAUDE.md`.

## What to do

1. **Confirm you are in the dev's project, not the plugin.** The target is the current
   working directory (the repo the dev is building). Never write into
   `${CLAUDE_PLUGIN_ROOT}`.

2. **Parse `$ARGUMENTS`:** `--force` present → overwrite existing files without asking.
   Absent → the safe path in step 4.

3. **Read the two templates** from `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE.md` and
   `${CLAUDE_PLUGIN_ROOT}/templates/apps/web/CLAUDE.md`. If either is missing, stop and
   report a broken plugin install — do not fabricate content.

4. **Write them into the project, safely:**
   - **Root `CLAUDE.md`** → project root.
   - **`apps/web/CLAUDE.md`** → create `apps/web/` if the project is (or is becoming) a
     monorepo web app; if there is no `apps/web` and no plan for one, ask before creating
     it rather than assuming the layout.
   - **If a target file already exists** and `--force` was not passed: do **not** clobber
     it. Show the dev the diff between their file and the template, and offer to
     **(a) merge** the missing invariants in, **(b) overwrite**, or **(c) skip** — per
     file. Preserve any project-specific rules the dev already wrote.

5. **Set project mode.** `apps/web/CLAUDE.md` has a `MODE:` line (`web-only` |
   `web+native`) that gates the migration boundary. Ask the dev which one applies and set
   it — this is the one value they must choose; leaving it wrong silently enables or
   disables the §5/§6 web-layering guards.

6. **Report** what was written/merged/skipped as a short list, then point the dev at the
   skills that now have their guardrails: `web-component`, `rn-component`, `feature-slice`,
   `design-system-setup`, and `/agentic-dev-kit:design-qa`.

## Notes

- **Idempotent.** Re-running with no `--force` never destroys edits — it re-offers merge
  for anything that has drifted from the template.
- **Scope is CLAUDE.md only.** This does not scaffold `packages/`, tokens, or an app
  skeleton — the `design-system-setup` / `feature-slice` skills own that. Suggest them as
  the next step, don't do their work here.
- The plugin's skills/agents/commands are already active once the plugin is enabled;
  this command exists solely because `CLAUDE.md` memory is loaded from the **project
  tree**, which the global plugin cache is not part of.
