# Token catalog — packages/ui-web

The **only** source for colors, spacing, typography, radius, and shadows is
`@repo/core/tokens`. Tailwind classes in `ui-web` map to these tokens (via
`apps/web/tailwind.config.ts`, which *consumes* the tokens — it never defines
values). Reference a token by name; never a raw hex or pixel value.

## Token categories

| Category | Example keys | Notes |
| --- | --- | --- |
| `colors.brand` | `primary`, `secondary` | Main brand colors (scaled `50`→`950`) |
| `colors.neutral` | `50` → `950` | Greys, backgrounds, borders |
| `colors.semantic` | `success`, `error`, `warning`, `info` | State colors |
| `typography.size` | `xs`, `sm`, `md`, `lg`, `xl`, `2xl` | Font sizes |
| `typography.weight` | `regular`, `medium`, `semibold`, `bold` | Font weights |
| `typography.family` | `sans`, `mono` | Font families |
| `typography.lineHeight` | `tight`, `normal`, `relaxed` | Line heights |
| `spacing` | `1` (4px) → `16` (64px) | 4px base grid |
| `radius` | `sm`, `md`, `lg`, `xl`, `full` | Border radii |
| `shadow` | `sm`, `md`, `lg` | Web: CSS shadow |

## Tailwind class mapping

Two valid styles — both map to tokens, neither uses a raw value:

| Intent | Scale class | Semantic class |
| --- | --- | --- |
| Brand background | `bg-primary-500` | `bg-primary` |
| Brand hover | `hover:bg-primary-600` | `hover:bg-primary-hover` |
| Subtle brand tint | `bg-primary-50` | `bg-primary-subtle` |
| Primary text | `text-neutral-900` | `text-text-primary` |
| Default border | `border-neutral-200` | `border-border-default` |
| Success / error | `bg-success-500` / `text-error-500` | `bg-success` / `text-error` |
| Spacing (16px) | `px-4`, `py-4`, `gap-4` | (same) |
| Radius | `rounded-lg` | (same) |
| Font size | `text-md` | (same) |
| Font weight | `font-medium` | (same) |

Semantic classes read better; scale classes are more explicit. Pick one per project
(see `apps/web/CLAUDE.md`) and stay consistent within a component.

## Full class reference

```tsx
// Semantic color classes (map to @repo/core/tokens)
'bg-primary'            // colors.brand.primary (DEFAULT)
'text-text-primary'     // colors.text.primary
'text-text-secondary'   // colors.text.secondary
'text-text-disabled'    // colors.text.disabled
'bg-surface-canvas'     // colors.surface.canvas
'bg-surface-subtle'     // colors.surface.subtle
'border-border-default' // colors.border.default
'border-border-focus'   // colors.border.focus
'text-error' 'text-success' 'text-warning'   // colors.semantic.*

// Spacing (4px grid): p-1=4 · p-2=8 · p-3=12 · p-4=16 · p-6=24 · p-8=32
// Radius: rounded-sm=4 · rounded-md=8 · rounded-lg=12 · rounded-xl=16 · rounded-full
// Type size: text-xs=12 · text-sm=14 · text-md=16 · text-lg=18 · text-xl=20
// Type weight: font-normal · font-medium · font-semibold · font-bold
// Shadow: shadow-sm · shadow-md · shadow-lg   (shadow.<x>.web)
```

## Mapping discipline

```tsx
// ✅ CORRECT — token-mapped classes, merged with cn()
className={cn('bg-primary-500 text-white px-4 py-2 rounded-lg text-md')}

// ❌ WRONG — raw values
className="bg-[#3B82F6] px-[16px] text-[14px]"
style={{ backgroundColor: '#3B82F6' }}
```

**No matching token?** Do not hardcode and do not approximate silently. Stop, report
the gap, and propose adding the token to `@repo/core/tokens` (colors are generated
from Figma — the design likely defines a variable that isn't in the token file yet).
See `figma-to-tokens.md`.
