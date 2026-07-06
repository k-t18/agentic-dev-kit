# Phase 2 — Figma extraction via MCP

The Figma MCP is connected (file ID in `.claude/settings.json`). Prefer
`get_variable_defs` — Figma variables are the design-token source and map 1:1 to
`tokens/index.ts`. Use `get_design_context` for per-layer values, `get_screenshot`
for visual reference, `get_metadata` for structure.

## Supported input types

| Input | How to handle |
| --- | --- |
| Figma MCP (connected) | Highest fidelity. Extract variables, styles, modes, effects, text styles. Prefer this. |
| Figma URL | Fetch node metadata via MCP. If MCP unavailable, request export/screenshot. |
| tokens.json / Style Dictionary / Token Studio export | Parse directly, normalize naming, produce dual output. |
| Existing `tokens/index.ts` | Audit against Figma, update drift, regenerate `rn-styles.ts`. |
| Screenshot / image | Extract visually. Mark all values **Inferred**. Never treat as confirmed. |
| Verbal description | Planning only. Never finalize a token system from prose. |

## 2.1 What to extract

### A. Foundation tokens (primitives)

| Category | What to pull |
| --- | --- |
| Color palettes | All color variables: brand, neutral, semantic states (as numeric scales) |
| Typography | Font family, size scale, weight, line height, letter spacing |
| Spacing | All spacing variables — confirm the base grid (commonly 4px) |
| Border radius | All radius values |
| Shadows | All shadow definitions — extract raw, split in Phase 4 |
| Opacity | Any explicit opacity scale |
| Motion | Duration and easing if defined |
| Breakpoints | Any breakpoint/container tokens if defined |

### B. Semantic tokens (purpose-mapped)

| Role | Maps to |
| --- | --- |
| Background / surface | Neutral palette values |
| Text / foreground | Readable text colors |
| Border | Divider / outline colors |
| Primary / secondary action | Brand palette (+ `action.*` for bg/hover/active states) |
| Success / error / warning / info | Semantic state colors |
| Disabled | Muted color + opacity |
| Overlay / scrim | Semi-transparent black/white |
| Focus / ring | Accessibility focus indicator |

### C. Component tokens (only if explicitly defined in Figma)

Promote to component tokens **only** if Figma explicitly defines them — never infer
from visual inspection. Examples if present: button states, input states, card
surface hierarchy, badge variants, toast/alert states.

## 2.2 Interaction states to identify

Where visible in Figma: default, hover, active, focus, disabled, selected, error,
success, loading, pressed.

## 2.3 Theme modes

If Figma defines multiple modes (light/dark/brand), extract each explicitly. Do **not**
fabricate a dark mode that isn't in Figma. If absent, add a comment in `index.ts`:

```ts
// Dark mode tokens: pending design input from Figma
```
