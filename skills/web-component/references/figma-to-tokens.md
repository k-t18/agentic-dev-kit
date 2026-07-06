# Figma → tokens

The Figma MCP is connected (file ID in `.claude/settings.json`). Pulling the node is
**mandatory** before writing any style — never guess design values.

## Which MCP tool to call

| Tool | Returns | Use for |
| --- | --- | --- |
| `get_variable_defs` | Figma **variables** (the design-token definitions) | **Primary** — variables map 1:1 to `@repo/core/tokens`. Resolve values to token names from here first. |
| `get_design_context` | Structured design/code context for the node | Layout, hierarchy, per-layer color/spacing/typography |
| `get_screenshot` | Rendered image of the node | Visual cross-check of the built component |
| `get_metadata` | Node tree / structure | Understanding nesting and which sub-layers exist |

Start with `get_variable_defs` — if the design uses variables, each value already has
a token-shaped name, so the mapping is direct. Fall back to `get_design_context` for
raw values when a layer isn't variable-bound.

## Extraction → mapping

For each visual property on the node, resolve to a **token name** (see
`token-catalog.md`):

| Figma property | Resolve to |
| --- | --- |
| Fill / stroke color | `colors.brand.*` / `colors.neutral.*` / `colors.semantic.*` → `bg-*` / `text-*` / `border-*` |
| Corner radius | `radius.*` → `rounded-*` |
| Padding / gap / item spacing | `spacing.*` (4px grid) → `px-*` / `py-*` / `gap-*` |
| Font size / weight / family / line-height | `typography.*` → `text-*` / `font-*` |
| Effect / drop shadow | `shadow.*` → `shadow-*` |

## The discipline

1. **Prefer the token name, not the value.** A Figma fill of `#3B82F6` that equals
   `colors.brand.primary[500]` becomes `bg-primary-500` — never `bg-[#3B82F6]`.
2. **Snap to the grid.** Spacing should land on the 4px scale (`spacing[4]` = 16px).
   If Figma shows `17px`, confirm intent — usually it's meant to be a token step.
3. **No matching token → flag, don't hardcode.** If a color/size has no token:
   - Report it: *"Figma value `#7C3AED` has no matching token."*
   - Propose adding it to `@repo/core/tokens` (tokens are generated from Figma, so a
     missing token usually means the token file is stale).
   - Do **not** emit a raw value or the nearest-but-wrong token silently.
4. **Cross-check** the finished component against `get_screenshot`.
