---
name: design-qa
description: Verify a built packages/ui-web component against its Figma source (5 checks, flags-only report). Runs the read-only design-qa agent.
argument-hint: "[ComponentName | all] [optional figma-url]"
---

# /design-qa

**Arguments:** $ARGUMENTS

Runs the read-only **`design-qa`** agent to verify a built `packages/ui-web` component
against its Figma source. The agent flags issues only — it never edits files.

## What to do

1. **Parse `$ARGUMENTS`:**
   - `<ComponentName>` → QA that single component.
   - `all` → QA every component under `packages/ui-web/src/components/*`.
   - An optional Figma URL after the name → pass it to the agent as the node override.

2. **Single component** — spawn the design-qa agent (Agent tool,
   `subagent_type: agentic-dev-kit:design-qa`) with the component name (and URL override
   if given). The agent is namespaced under the plugin, so use the fully-qualified
   `agentic-dev-kit:design-qa` — the bare `design-qa` will not resolve once installed. The
   agent resolves the node from the component's `// figma:` annotation (or the override),
   pulls Figma ground truth, reads `<Name>.tsx` + `<Name>.types.ts`, runs the five
   checks, and returns a Pass/Fail report. Relay the report.

3. **`all`** — enumerate the component folders in `packages/ui-web/src/components/`,
   run the `agentic-dev-kit:design-qa` agent for each (sequentially), and present the combined
   audit summary table followed by each component's detail.

## Notes

- A component with **no `// figma:` annotation** and no URL override is reported as
  "no Figma source recorded" — not guessed. (`web-component` stamps this annotation at
  build time.)
- Any 🔴 blocking issue means the component must not be merged to `develop`.
