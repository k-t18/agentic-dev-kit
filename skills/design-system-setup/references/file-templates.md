# Phase 6 — File generation templates

Generate in this exact order. All values below are **generic placeholders** — replace
every value with the real ones extracted from Figma. Never ship these literals.

---

## Step 1 — `/packages/core/src/tokens/index.ts` (master)

Exports all foundation + semantic tokens. Platform-split tokens (shadows) use the
`{ web, native }` structure. Keep the full scale in `palette.<hue>`, expose a single
DEFAULT alias in `brand.*`, and interactive states in `action.*`.

```ts
// /packages/core/src/tokens/index.ts
// Source of truth for all design tokens. Extracted from Figma via Figma MCP.
// DO NOT hardcode values in components — always import from this file.
// DO NOT edit rn-styles.ts directly — it is derived from this file.

export const colors = {
  palette: {
    primary: { 50: '#eff6ff', 100: '#dbeafe', 300: '#93c5fd', 500: '#3b82f6', 600: '#2563eb', 900: '#1e3a8a' },
    neutral: { 50: '#f9fafb', 100: '#f3f4f6', 200: '#e5e7eb', 500: '#6b7280', 900: '#111827', 950: '#030712' },
  },
  brand: {
    primary: '#3b82f6',   // alias of palette.primary[500]
    white: '#ffffff',
    black: '#111827',
  },
  semantic: {
    success: '#22c55e',
    error: '#ef4444',
    warning: '#f59e0b',
    info: '#3b82f6',
  },
  text: { primary: '#111827', secondary: '#6b7280', disabled: '#9ca3af', inverse: '#ffffff' },
  surface: { canvas: '#ffffff', subtle: '#f9fafb', overlay: 'rgba(0,0,0,0.5)' },
  border: { default: '#e5e7eb', strong: '#d1d5db', focus: '#3b82f6' },
  action: {
    primary: { bg: '#3b82f6', hover: '#2563eb', active: '#1d4ed8' },
  },
} as const;

export const typography = {
  family: { sans: 'YourSans', mono: 'YourMono' },
  size: { xs: 12, sm: 14, md: 16, lg: 18, xl: 20, '2xl': 24 },
  weight: { regular: '400', medium: '500', semibold: '600', bold: '700' },
  lineHeight: { tight: 1.25, normal: 1.5, relaxed: 1.75 },
  letterSpacing: { normal: 0 },
} as const;

// Optional: named text styles composed from the primitives above (add if Figma defines them)
export const textStyles = {
  body: {
    fontFamily: typography.family.sans,
    fontSize: typography.size.md,
    fontWeight: typography.weight.regular,
    lineHeight: typography.lineHeight.normal,
  },
} as const;

export const spacing = { 1: 4, 2: 8, 3: 12, 4: 16, 5: 20, 6: 24, 8: 32, 10: 40, 12: 48, 16: 64 } as const;

export const radius = { sm: 4, md: 8, lg: 12, xl: 16, full: 9999 } as const;

export const shadow = {
  sm: { web: '0 1px 2px 0 rgba(0,0,0,0.05)', native: { shadowColor: '#000', shadowOffset: { width: 0, height: 1 }, shadowOpacity: 0.05, shadowRadius: 2, elevation: 1 } },
  md: { web: '0 4px 6px -1px rgba(0,0,0,0.1)', native: { shadowColor: '#000', shadowOffset: { width: 0, height: 4 }, shadowOpacity: 0.1, shadowRadius: 6, elevation: 4 } },
  lg: { web: '0 10px 15px -3px rgba(0,0,0,0.1)', native: { shadowColor: '#000', shadowOffset: { width: 0, height: 10 }, shadowOpacity: 0.1, shadowRadius: 15, elevation: 8 } },
} as const;

// Dark mode tokens: pending design input from Figma
```

---

## Step 2 — `/packages/core/src/tokens/rn-styles.ts` (derived)

StyleSheet-safe only: no CSS shorthand, no `%` widths, no `.web` shadow strings.

```ts
// /packages/core/src/tokens/rn-styles.ts
// DERIVED from tokens/index.ts — do not edit manually.
// Regenerate by re-running the design-system-setup skill.

import { colors, typography, spacing, radius, shadow } from './index';

export const rnTokens = {
  colors: {
    primary: colors.brand.primary,
    textPrimary: colors.text.primary,
    textSecondary: colors.text.secondary,
    textInverse: colors.text.inverse,
    surfaceCanvas: colors.surface.canvas,
    borderDefault: colors.border.default,
    borderFocus: colors.border.focus,
    success: colors.semantic.success,
    error: colors.semantic.error,
    actionPrimaryBg: colors.action.primary.bg,
    white: '#ffffff',
    transparent: 'transparent',
  },
  typography: {
    family: typography.family,   // fontFamily — must match linked font PostScript name
    size: typography.size,       // numeric px
    weight: typography.weight,   // string ('400', '700')
    lineHeight: {                // RN uses numeric px, not unitless ratios
      tight: (size: number) => Math.round(size * 1.25),
      normal: (size: number) => Math.round(size * 1.5),
      relaxed: (size: number) => Math.round(size * 1.75),
    },
  },
  spacing,   // numeric — safe for padding/margin/gap
  radius,    // numeric — safe for borderRadius
  shadow: { sm: shadow.sm.native, md: shadow.md.native, lg: shadow.lg.native }, // native objects only
} as const;

export type RnTokens = typeof rnTokens;
```

---

## Step 3 — `/apps/web/tailwind.config.ts` (consumer)

Imports tokens; never defines them. Expose each scaled hue as
`{ DEFAULT: brand.<x>, ...palette.<x> }` so **both** `bg-primary` and `bg-primary-500`
work. Include `app/` in `content` (Next App Router).

```ts
// /apps/web/tailwind.config.ts
import type { Config } from 'tailwindcss';
import { colors, typography, spacing, radius, shadow } from '@repo/core/tokens';

const config: Config = {
  content: ['./app/**/*.{ts,tsx}', './src/**/*.{ts,tsx}', '../../packages/ui-web/src/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: {
        primary: { DEFAULT: colors.brand.primary, ...colors.palette.primary },
        neutral: colors.palette.neutral,
        white: colors.brand.white,
        black: colors.brand.black,
        success: colors.semantic.success,
        error: colors.semantic.error,
        warning: colors.semantic.warning,
        info: colors.semantic.info,
        'text-primary': colors.text.primary,
        'text-secondary': colors.text.secondary,
        'text-disabled': colors.text.disabled,
        'surface-canvas': colors.surface.canvas,
        'surface-subtle': colors.surface.subtle,
        'border-default': colors.border.default,
        'border-focus': colors.border.focus,
      },
      fontFamily: {
        sans: [typography.family.sans, 'sans-serif'],
        mono: [typography.family.mono, 'monospace'],
      },
      fontSize: Object.fromEntries(Object.entries(typography.size).map(([k, v]) => [k, `${v}px`])),
      fontWeight: typography.weight,
      spacing: Object.fromEntries(Object.entries(spacing).map(([k, v]) => [k, `${v}px`])),
      borderRadius: Object.fromEntries(Object.entries(radius).map(([k, v]) => [k, v === 9999 ? '9999px' : `${v}px`])),
      boxShadow: { sm: shadow.sm.web, md: shadow.md.web, lg: shadow.lg.web },
    },
  },
  plugins: [],
};

export default config;
```

---

## Step 4 — Web font loading

Two options — pick one per project:

```ts
// Option A — next/font (recommended for Next.js). e.g. app/layout.tsx
import { Inter } from 'next/font/google';
const sans = Inter({ subsets: ['latin'], variable: '--font-sans' });
// apply sans.variable on <html>; map typography.family.sans to the same family name
```

```css
/* Option B — /apps/web/src/styles/fonts.css, imported once in app/layout.tsx */
@font-face {
  font-family: 'YourSans';
  src: url('/fonts/YourSans-Regular.woff2') format('woff2');
  font-weight: 400;
  font-display: swap;
}
```

Either way, the family name must match `typography.family.sans` in tokens.

---

## Step 5 — Native font setup (document in output)

```md
## Native font setup (required after token extraction)
1. Place font files in /apps/native/assets/fonts/ (one per weight in typography.weight)
2. Add to /apps/native/react-native.config.js:
   module.exports = { assets: ['./assets/fonts/'] };
3. Link: npx react-native-asset
4. Rebuild: npx react-native run-ios / run-android

Note: fontFamily in StyleSheet must match the font's PostScript name (case-sensitive on iOS),
not the filename. Verify via Font Book (macOS).
```

---

## Step 6 — Package barrel

```ts
// /packages/core/src/index.ts — add if not present
export * from './tokens/index';
export * from './tokens/rn-styles';
```
