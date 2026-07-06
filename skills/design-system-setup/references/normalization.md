# Phase 3 — Normalization

Convert whatever Figma naming exists into stable engineering naming.

## 3.1 Naming rules

| Rule | Example |
| --- | --- |
| No designer-specific labels | `blue2` → `colors.palette.primary[200]` |
| No ambiguous names | `main` → `colors.palette.primary[500]` |
| Separate palette from semantic | a raw hex is a palette value; `colors.action.primary` is semantic |
| Scale colors numerically | `50, 100, 200 … 900, 950` (extra steps like `250`, `88` are fine when Figma defines them) |
| Scale spacing by base-grid index | `spacing[1]` = 4px, `spacing[4]` = 16px, `spacing[8]` = 32px |
| Use role names for semantic | `text.primary`, `surface.canvas`, `border.default` |

## 3.2 Three-layer token architecture

```
Layer 1 — Foundation (primitives)
  colors.palette.primary[500]
  spacing[4]
  radius.md
  shadow.md

Layer 2 — Semantic (purpose)
  colors.brand.primary        → alias of colors.palette.primary[500]
  colors.text.primary         → colors.palette.neutral[900]
  colors.surface.canvas       → white / neutral[50]
  colors.action.primary.bg    → colors.palette.primary[500]
  colors.action.primary.hover → colors.palette.primary[600]

Layer 3 — Component (optional, only if Figma defines them)
  button.primary.bg.default   → colors.action.primary.bg
  input.border.focus          → colors.palette.primary[300]
```

**Pattern learned from production:** keep the full **scale** in `palette.<hue>`, expose
a single **DEFAULT** alias in `brand.*`, and put interactive states in `action.*`. On
web this lets Tailwind expose both `bg-primary` (DEFAULT) and `bg-primary-500` (scale)
from one entry — see `file-templates.md`.

## 3.3 Confidence levels

Tag every token:

| Level | Meaning |
| --- | --- |
| **Confirmed** | Directly defined in Figma variables |
| **Inferred** | Strongly implied by a visible pattern or scale |
| **Assumed** | Added as a system fallback for completeness |
| **Missing** | Required by the system but absent in Figma |

## 3.4 Handle messy Figma

- Cluster near-identical values → identify the canonical value (document the conflict).
- Isolate one-off exceptions — do **not** promote to global tokens.
- Never silently flatten conflicting values — surface them in the summary.
