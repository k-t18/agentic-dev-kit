---
name: web-to-native
description: Use when migrating a QA-approved web component from packages/ui-web to its React Native twin in packages/ui-native. Only JSX structure and styling change — the props interface (mirrored verbatim as explicit unions), and every @repo/core import (hooks, API, stores, types, tokens) carry over unchanged; @repo/core is never modified. Maps HTML elements to RN primitives, cva variants to keyed StyleSheet, Tailwind classes to rnTokens, and web events/ARIA to onPress/accessibility*. Invoke to port a component to native, migrate ui-web to ui-native, or create a native twin.
license: MIT
metadata:
  author: Frontend
  version: "1.0.0"
  domain: frontend
  triggers: web-to-native, migrate component, port to React Native, ui-native from ui-web, native twin, migration
  role: specialist
  scope: implementation
  output-format: code
  related-skills: rn-component, web-component, design-system-setup
---

# Web-to-Native Migration

Converts a web component (React + Tailwind, `packages/ui-web`) into its native twin
(RN primitives + `StyleSheet`, `packages/ui-native`). **Scope is strictly JSX + styles.**

The native output must follow the **`rn-component`** conventions — this skill is the
*process*; `rn-component` defines the *target*. For styling property mapping, primitives,
and a11y depth, see that skill's references (`rn-style-map.md`, `rn-primitives.md`,
`rn-accessibility.md`). This skill owns only the migration-specific transforms.

## The one rule

> **`@repo/core` does not change. Ever.** Hooks, API, stores, types, tokens — zero
> edits. Only JSX elements and styling are replaced.

## When to Use

- Migrating an approved `packages/ui-web` component to `packages/ui-native`.

**Not for:** authoring a native component from scratch (`rn-component`), building a web
component (`web-component`), or anything in `@repo/core`.

## Pre-migration checklist

Fix the web component first if any fail:
- [ ] Web component is QA-approved (design-qa passed) and merged to `develop`.
- [ ] Uses `onPress` (not `onClick`) and `label` (not bare `children`) for text.
- [ ] Zero hardcoded hex/px — tokens only.
- [ ] Every token it uses has an `rnTokens` equivalent in `rn-styles.ts` (if not, that's
      a `design-system-setup` gap — fix there first).

## Workflow

1. **Read the web component** — its props (`<Name>.types.ts` + the `cva` config), its
   variants, states, and imports (classify each: core = keep, web-only = remove, ui = swap).
2. **Mirror the props verbatim** — same variant names/sizes, re-expressed as **explicit
   unions** (native has no `cva`/`VariantProps`); drop web-only props (`className`, `type`).
   → `references/variants-and-props.md`
3. **Map elements** HTML → RN primitive — **every string must be inside `<Text>`**.
   → `references/element-map.md`
4. **Convert styles** — `cva` variants → **keyed `StyleSheet`** (`styles[intent]`) with
   `rnTokens`; class → property mapping per `rn-component/references/rn-style-map.md`.
   → `references/variants-and-props.md`
5. **Swap imports** — remove `cn`/`cva`/`lucide-react`/`react-router-dom`; keep every
   `@repo/core/*`; add `react-native` primitives. → `references/imports-and-events.md`
6. **Events / nav / a11y** — `onClick`→`onPress`, `onChange`→`onChangeText`, React
   Router→React Navigation, ARIA→`accessibility*`. → `references/imports-and-events.md`
   (a11y detail: `rn-component/references/rn-accessibility.md`)
7. **Loading** — web `Loader2` → `ActivityIndicator`; `ReactNode` icon slots
   (`leftIcon`/`rightIcon`) carry over unchanged (no icon library added).
8. **Output** — `packages/ui-native/src/components/<Name>/<Name>.tsx` + `index.ts`; add
   `export * from './components/<Name>';` to `packages/ui-native/src/index.ts`.
9. **Validate** — `pnpm --filter @repo/ui-native exec tsc --noEmit` clean; run the
   migration checklist.

## Reference Guide

| Topic | Reference | Load when |
| --- | --- | --- |
| HTML element → RN primitive | `references/element-map.md` | Mapping JSX |
| Imports swap · events · navigation · a11y | `references/imports-and-events.md` | Swapping imports / wiring events |
| cva → keyed StyleSheet · props mirroring | `references/variants-and-props.md` | Converting variants / props |
| Style property mapping · primitives · a11y | the **`rn-component`** skill | The native target conventions |

## Migration checklist

**Elements & text** — all HTML replaced with RN primitives · every bare string wrapped in
`<Text>` · `Image` has explicit `width`+`height` · `FlatList` for dynamic lists.
**Styling** — `StyleSheet.create` only · `rnTokens` for every value · no `className` · no
`%` widths · no CSS shorthand (expanded) · shadows via `...rnTokens.shadow.*` spread.
**Imports & logic** — every `@repo/core/*` import unchanged · `cn`/`cva`/`lucide-react`
removed · navigation → React Navigation · `onClick`→`onPress`, `onChange`→`onChangeText`.
**Props** — mirror the web twin verbatim as explicit unions, minus `className`/`type`.
**Output** — correct path under `packages/ui-native/` · exported from the package barrel ·
`@repo/core` untouched.
