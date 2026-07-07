---
name: web-component
description: Use when creating or editing a web UI component in packages/ui-web (React + Tailwind). Enforces the Figma-first workflow — pull the Figma node, extract design values, map them to @repo/core tokens — then generate the component as a folder (Name.tsx + Name.types.ts + index.ts) using class-variance-authority (cva) for variants, cn() for class merging, lucide-react for icons, a Readonly props type that extends VariantProps, and default/disabled/loading states, re-exported from the ui-web barrel. Invoke for building a ui-web button, input, card, badge, or any packages/ui-web component.
license: MIT
metadata:
  author: https://github.com/k-t18
  version: "2.0.0"
  domain: frontend
  triggers: ui-web, web component, packages/ui-web, Tailwind component, cva, class-variance-authority, Figma component, design-token component, button, input, card, badge, component library
  role: specialist
  scope: implementation
  output-format: code
  related-skills: design-system-setup, rn-component, web-to-native
---

# Web Component (packages/ui-web)

Builds a single web component in `packages/ui-web` from a Figma design, styled
exclusively with `@repo/core` design tokens. In **`web+native`** projects this is the
migration surface — components stay client-side so they can mirror to `ui-native`. In
**`web-only`** projects, `"use client"` is added only when the component is interactive other wise its a server component.
(Project mode is declared in `apps/web/CLAUDE.md`.)

Read root `CLAUDE.md` and `apps/web/CLAUDE.md` first; this skill never overrides them.
Tokens come from `design-system-setup`; this skill _consumes_ them.

## When to Use

- Building or editing a component in `packages/ui-web`.

**Not for:** route pages/layouts (`apps/web`), hooks/API/stores/types (`@repo/core`,
see `feature-slice`), native components (`rn-component`), or web→native migration
(`web-to-native`).

## Dependencies

The `ui-web` package uses: `class-variance-authority` (variants), `clsx` +
`tailwind-merge` (the `cn()` helper in `packages/ui-web/src/lib/utils.ts`), and
`lucide-react` (icons). Assume these exist; don't reinvent `cn()`.

## Core Workflow (Figma-first — mandatory)

**Never guess design values. Always pull from Figma before writing styles.**

1. **Pull the Figma node** via the Figma MCP. Prefer `get_variable_defs` (Figma
   variables are the token source), plus `get_design_context` / `get_screenshot`.
   → `references/figma-to-tokens.md`
2. **Extract** color, spacing, radius, typography, shadow.
3. **Map each raw value to a token _name_** (`@repo/core/tokens` → Tailwind token
   class). Never a raw hex/px. No matching token → **stop and flag it** (propose a
   token addition); do not hardcode. → `references/token-catalog.md`
4. **Create the component folder** `packages/ui-web/src/components/<Name>/` with three
   files:
   - `<Name>.tsx` — the component. **First line: `// figma: <node-url>`** (the node you
     pulled in step 1 — `design-qa` reads it to re-verify). Then cva variants + `cn()`
     - lucide icons; named export.
   - `<Name>.types.ts` — props `interface` extending `VariantProps<typeof <name>Variants>`
   - `index.ts` — re-exports the component, its variants, and its props type
5. **Define variants with `cva`** (`intent`, `size`, etc.), styled with Tailwind token
   classes; merge at the call site with `cn()`. No inline styles, no arbitrary `[...]`.
6. **Handle every applicable state** — by component type (interactive / input / data /
   async), including error, empty, and the React Query triple. → `references/states.md`.
   Add full a11y (semantic HTML, focus ring, labels/ARIA, contrast, motion). →
   `references/accessibility.md`. Use `lucide-react` (`Loader2`) for spinners.
7. **`"use client"`** first line — always in `web+native`; in `web-only` only if the
   component is interactive (handlers, hooks, state, browser APIs).
8. **Barrel export:** add `export * from './components/<Name>';` to
   `packages/ui-web/src/index.ts`.
9. **Validate:** `pnpm --filter @repo/ui-web exec tsc --noEmit` is clean; grep for
   hardcoded hex/px and arbitrary `[...]` — there must be none.
10. **Verify against Figma:** hand off to the `agentic-dev-kit:design-qa` agent (or run
    `/agentic-dev-kit:design-qa <Name>`) before opening a PR. It re-pulls the node from the `// figma:`
    annotation and runs the five design checks; fix any 🔴 blocking issue it flags.

## Reference Guide

| Topic                                              | Reference                       | Load when                                  |
| -------------------------------------------------- | ------------------------------- | ------------------------------------------ |
| Token names + Tailwind class mapping               | `references/token-catalog.md`   | Mapping a design value / choosing a class  |
| Reading a Figma node, mapping values               | `references/figma-to-tokens.md` | Pulling the node, resolving raw values     |
| States by type + loading/error/empty/async + icons | `references/states.md`          | Deciding which states to build             |
| Semantic HTML, focus, ARIA, contrast, motion       | `references/accessibility.md`   | Wiring a11y                                |
| Migration-aware prop naming + type rules           | `references/migration-props.md` | Designing the props interface (web+native) |

## Canonical pattern

```tsx
// packages/ui-web/src/components/Button/Button.tsx
// figma: https://figma.com/design/<FILE>?node-id=<NODE>   ← source node (design-qa reads this)
"use client"; // interactive (onPress) → required in BOTH modes

// ─── External imports ─────────────────────────────
import { cva } from "class-variance-authority";
import { Loader2 } from "lucide-react";

// ─── Internal imports ─────────────────────────────
import { cn } from "../../lib/utils";
import type { ButtonProps } from "./Button.types";

const buttonVariants = cva(
  [
    "inline-flex items-center justify-center rounded-md font-medium select-none",
    "transition-colors duration-150",
    "focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-border-focus focus-visible:ring-offset-2",
    "disabled:pointer-events-none motion-reduce:transition-none",
  ],
  {
    variants: {
      intent: {
        primary:
          "bg-primary-500 text-white hover:bg-primary-600 disabled:bg-border-default disabled:text-border-strong",
        secondary:
          "bg-surface-canvas text-primary-500 border border-primary-500 hover:border-primary-600 hover:text-primary-600 disabled:border-border-strong disabled:text-border-strong",
        ghost:
          "bg-transparent text-primary-500 hover:text-primary-600 disabled:text-border-strong",
      },
      size: {
        sm: "px-3 py-1.5 text-sm",
        md: "px-4 py-2 text-md",
        lg: "px-6 py-3 text-lg",
      },
    },
    defaultVariants: { intent: "primary", size: "md" },
  },
);

export function Button({
  label,
  onPress, // onPress (not onClick) → maps 1:1 to native
  disabled = false,
  loading = false,
  leftIcon,
  intent,
  size,
  type = "button",
  className,
}: Readonly<ButtonProps>) {
  const isDisabled = disabled || loading;
  return (
    <button
      type={type}
      onClick={onPress} // web wires onPress → DOM onClick internally
      disabled={isDisabled}
      aria-busy={loading || undefined}
      className={cn(
        buttonVariants({ intent, size }),
        loading && "cursor-wait",
        className,
      )}
    >
      {loading ? (
        <Loader2 className="mr-2 h-4 w-4 animate-spin" aria-hidden="true" />
      ) : (
        leftIcon && (
          <span className="mr-2" aria-hidden="true">
            {leftIcon}
          </span>
        )
      )}
      <span>{label}</span>
    </button>
  );
}

Button.displayName = "Button";

export { buttonVariants };
```

```ts
// packages/ui-web/src/components/Button/Button.types.ts
import type { VariantProps } from "class-variance-authority";
import type { ReactNode } from "react";
import type { buttonVariants } from "./Button";

export interface ButtonProps extends VariantProps<typeof buttonVariants> {
  label: string;
  onPress?: () => void;
  disabled?: boolean;
  loading?: boolean;
  leftIcon?: ReactNode;
  type?: "button" | "submit" | "reset";
  className?: string;
}
```

```ts
// packages/ui-web/src/components/Button/index.ts
export { Button, buttonVariants } from "./Button";
export type { ButtonProps } from "./Button.types";
```

```ts
// packages/ui-web/src/index.ts  — package barrel
export * from "./components/Button";
```

> Scale classes (`bg-primary-500`) and semantic classes (`bg-surface-canvas`,
> `border-border-default`) are both token-mapped and valid — see
> `apps/web/CLAUDE.md`. Interactive states use the scale (`hover:bg-primary-600`).

## Rules

- ✅ One folder per component: `Name.tsx` + `Name.types.ts` + `index.ts`
- ✅ Stamp `// figma: <node-url>` as the first line of `Name.tsx` (design-qa reads it)
- ✅ Variants via **`cva`**; props type **extends `VariantProps<typeof nameVariants>`**
- ✅ Named export + `Readonly<Props>` + `Name.displayName`; `export { nameVariants }`
- ✅ Group imports (external, then internal); `cn` from `'../../lib/utils'`
- ✅ Icons from `lucide-react` (`Loader2` for loading) — never a glyph char
- ✅ Tokens only — Tailwind token classes; never raw hex/px, never `[...]`
- ✅ Migration-ready prop names: `onPress` (not `onClick`), `label` (not `children`),
  `onChangeText` (not `onChange`); `className` optional. → `references/migration-props.md`
- ✅ Semantic HTML — the right element for the job (`button`/`a`/`input`); never a
  `div` for interactive. → `references/accessibility.md`
- ✅ Handle all applicable states (interactive/input/data/async, incl. error/empty) +
  full a11y. → `references/states.md`, `references/accessibility.md`
- ✅ `"use client"` — always in `web+native`; in `web-only` only when interactive
- ✅ Discriminated unions for multi-state props; shared types from `@repo/core/types`
- ❌ No inline `style={{}}`, no arbitrary Tailwind values, no default exports, no `React.FC`
- ❌ No inline `interface` in the `.tsx` — props live in `Name.types.ts`
- ❌ No `any`, no `as` casts to silence errors
- ❌ No hooks / API / types / tokens defined here — those belong in `@repo/core`
- ❌ Never invent a design value — if no token matches, flag it, don't hardcode
