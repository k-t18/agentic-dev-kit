---
name: web-to-native
description: Use when migrating a QA-approved web component from packages/ui-web to its React Native twin in packages/ui-native, OR wiring a native screen's container hook (use<Screen>.ts) that mirrors an already-decomposed web page. For a plain presentational component, only JSX structure and styling change — the props interface (mirrored verbatim as explicit unions) carries over, and @repo/core is never modified. For a component with real business logic behind it (a cascading dropdown, a live tax/total recompute), covers the duplicate-locally-vs-narrow-subpath-export decision so a ui-native component never has to import @repo/core's root barrel. For screen-level wiring, references/page-composition.md covers the web page shape, its native mirror, and the portability tests. Also covers splitting a large migration (component build vs. screen-wiring) into separate issues. Maps HTML elements to RN primitives, cva variants to keyed StyleSheet, Tailwind classes to rnTokens, and web events/ARIA to onPress/accessibility*. Invoke to port a component to native, migrate ui-web to ui-native, create a native twin, wire a native screen composer, or reuse core logic in a native component without pulling in the whole @repo/core barrel.
license: MIT
metadata:
  author: https://github.com/k-t18
  version: "1.1.0"
  domain: frontend
  triggers: web-to-native, migrate component, port to React Native, ui-native from ui-web, native twin, migration, reuse core logic, subpath export, narrow export, page composition, screen composer, container hook, use screen hook
  role: specialist
  scope: implementation
  output-format: code
  related-skills: rn-component, web-component, design-system-setup, feature-slice, nextjs-nerd
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

The one narrow, deliberate exception is a new **subpath export** added to
`@repo/core/package.json` — see "Reusing real `@repo/core` logic" below. That's still
zero edits to any *logic* — it's a package-manifest line that makes an already-existing
pure file importable without pulling in the root barrel.

## Two migration shapes — pick the right workflow before you start

This skill's steps 1–9 below are written for the common case: **a single
presentational component** (a `Button`, an `Input`, a `Card`) whose web twin is JSX +
Tailwind + `cva` variants over props, with no real interactive state of its own. Follow
them as written for that case.

Two other shapes come up and need different handling — misreading which one you're in
is the most common way this migration goes over-budget:

1. **A stateful component with real business logic inside it** (a drawer that computes
   cascading dropdown options, recomputes a tax table live as the user types, owns a
   multi-step form). The web twin isn't just JSX over props — it calls a pure
   `@repo/core` function or duplicates real domain logic inline. Steps 1–7 still apply
   for the JSX/styling; **first read "Reusing real `@repo/core` logic" below** to
   decide how that logic itself crosses over, before writing any component code.
2. **Wiring a screen's container hook** (a whole `use<Screen>.ts` composer — the thing
   that calls five `@repo/core` container hooks and returns one view-model for the
   screen to render) — not a `packages/ui-native` component at all. It's a different
   file, in `apps/<native>/src/screens/<Screen>/hooks/`, following the *same*
   decomposition rules as the web page it mirrors — see
   `references/page-composition.md` for the full procedure (web page shape, native
   screen mirror, the portability tests, and a worked example) before improvising the
   shape from scratch. Confusing "port this component" with "wire this screen"
   produces a monolithic PR that tries to do both; keep them as separate pieces of
   work (see "Splitting a large migration into issues" below).

## When to Use

- Migrating an approved `packages/ui-web` component to `packages/ui-native` — the
  common presentational case, or either of the two other shapes above.

**Not for:** authoring a native component from scratch (`rn-component`), building a web
component (`web-component`), or editing `@repo/core`'s actual logic (a new subpath
`export` line is the one narrow exception — see "Reusing real `@repo/core` logic").

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

## Reusing real `@repo/core` logic (shape 1 — a component with real logic inside)

A leaf `packages/ui-native` component may **never** import `@repo/core`'s root barrel
(`@repo/core` / `@repo/core/index`). That barrel typically does `export * from
'./hooks'`, which pulls the whole React-Query/API-engine graph into a component's `tsc`
compilation graph just to resolve one type or function name — this has caused real,
hard-to-diagnose breakage in practice (missing `@types/node`, DOM-only types leaking
into a supposedly platform-agnostic file) that then gets "fixed" by patching
`@repo/core`'s build infra, which is scope creep the component never needed. A screen
or hook in `apps/<native>/src/screens/**` does not have this restriction — the
restriction is about `packages/ui-native` components specifically.

So when a web component's real logic (not just its JSX) needs to cross over, pick one
of two moves — **don't guess; the wrong choice either duplicates real business logic
(drift risk) or breaks the native build (barrel bloat)**:

**Duplicate it locally** when the logic is small (rule of thumb: under ~50 lines) and
low-risk to drift (a cascading-dropdown filter, a "which of these two fields should be
disabled" predicate). Port it verbatim as a plain function inside the native
component's own `.tsx`/`.types.ts`, with a comment naming the `@repo/core` source file
it mirrors. Define any types it needs as a **local mirror** in the component's own
`.types.ts` (same field names/shapes as the real `@repo/core` type) rather than
importing them.

**Add a narrow subpath export** when the logic is substantial and/or correctness-
critical (tax math, a multi-step derivation used by more than one place, anything where
a silent copy-paste drift would produce a wrong number rather than a wrong pixel).
Confirm the source file is itself pure first — no React, no hooks, no API calls, just
functions over plain data (check its own file header comment; this repo's convention is
to say so explicitly). Then add ONE line to `@repo/core/package.json`'s `"exports"`
map, pointing straight at that file (and its types file, if separate):

```json
// packages/core/package.json
"exports": {
  ".": "./src/index.ts",
  "./tokens/rn-styles": "./src/tokens/rn-styles.ts",
  "./features/<feature>/<file>": "./src/features/<feature>/<file>.ts"
}
```

Import it from the native component via that exact subpath — never the root:

```tsx
import { applySlotChange } from '@repo/core/features/invoice-header/mappers';
```

Check whether the native app's `tsconfig.json` already has a wildcard path covering
`@repo/core/*` (many do, for the existing `@repo/core/tokens/rn-styles` import) — if so,
`tsc` resolves the new subpath with zero further config; the `package.json` export is
what the **bundler** (Metro) needs at runtime. Verify both — a subpath that only
resolves for `tsc` and not Metro fails at run time, not build time, and is easy to
miss.

This is a real design decision with a real user-facing tradeoff (does the project want
another `@repo/core` export surface to maintain, vs. two copies of business logic to
keep in sync) — when genuinely unsure which side of the ~50-line line a case falls on,
ask rather than picking silently; both defaults have a real cost.

## Splitting a large migration into issues

If shape 1 or 2 above means the "migration" is actually a component rebuild plus real
wiring work (not just JSX/style conversion), don't ship it as one PR. Split into:

- **A component-build issue** — the `packages/ui-native` component(s) alone, taking
  real, data-driven props (mirroring the web twin's prop shape), validated visually via
  a playground/storybook screen with mock data. No app wiring, no container-hook calls.
- **A wiring issue**, blocked on the first — threading the real `@repo/core` container
  hook(s) into the screen's composer and rendering the now-real component with live
  data.

This mirrors how a Design QA gate separates "is the component built correctly" review
from "is the business logic correct" review — bundling both into one PR makes each
review weaker. If a body of work covers several genuinely **independent** features
(neither shares state with the other), split by feature instead of by layer — e.g.
three unrelated drawers each get their own component+wiring pair, rather than one
"build all three drawers" issue followed by one "wire all three" issue, *unless* their
wiring shares a state machine (e.g. three drawers keyed off the same
mutually-exclusive `activePanel` state) — in that case wiring them together in one
issue is correct, since splitting them would mean building the same state machine
three times.

## Before porting a web layout the native twin already diverged from

Check the native component's own file header / doc comments before assuming the web
twin's current JSX is the thing to copy. If native's version already carries a note
like "deliberately diverges from web's layout as of Figma node X — web hasn't been
updated to match," porting web's *current* markup would silently regress a fix that was
already made once. Pull the Figma node fresh and build against that, not against
whichever of the two platforms happens to be older.

## Extending a local mirror type

A local mirror type (the "duplicate it locally" case above, or an existing one like a
component's own `ProductListItem`-style prop type) sometimes needs a new field once a
new piece of web functionality is ported over — e.g. a `count` field for group/collapse
support, or an `inStock` field for a status-driven border color. Before adding it:
**confirm the field actually exists, under that exact name, on the real `@repo/core`
type** (grep the feature's own `.types.ts`) — don't infer or invent a plausible-looking
field name from the web component's usage alone. Two sibling local-mirror types often
need the *same* new field for two different, independently-scoped pieces of work; that
is not a conflict — whichever piece of work lands first adds the field, since it's
purely additive.

## Reference Guide

| Topic | Reference | Load when |
| --- | --- | --- |
| HTML element → RN primitive | `references/element-map.md` | Mapping JSX |
| Imports swap · events · navigation · a11y | `references/imports-and-events.md` | Swapping imports / wiring events |
| cva → keyed StyleSheet · props mirroring | `references/variants-and-props.md` | Converting variants / props |
| Reusing `@repo/core` logic (duplicate vs. subpath export) | `references/reusing-core-logic.md` | Shape 1 — a component with real logic behind it |
| Web page shape · native screen mirror · portability tests | `references/page-composition.md` | Shape 2 — wiring a screen's container hook |
| Style property mapping · primitives · a11y | the **`rn-component`** skill | The native target conventions |
| iOS build bring-up · CocoaPods · Info.plist parity · signing | `references/ios-build-readiness.md` | The migrated native app needs iOS parity with the working Android build |
| Tablet / large-screen adaptation · reactivity audit · orientation policy | `references/mobile-to-tablet.md` | A migrated component or screen needs to work on tablets, split-view, or multi-window |

## Migration checklist

**Elements & text** — all HTML replaced with RN primitives · every bare string wrapped in
`<Text>` · `Image` has explicit `width`+`height` · `FlatList` for dynamic lists.
**Styling** — `StyleSheet.create` only · `rnTokens` for every value · no `className` · no
`%` widths · no CSS shorthand (expanded) · shadows via `...rnTokens.shadow.*` spread.
**Imports & logic** — every `@repo/core/*` import unchanged (screens/hooks only — never
inside a `packages/ui-native` component, root barrel or otherwise) · `cn`/`cva`/
`lucide-react` removed · navigation → React Navigation · `onClick`→`onPress`,
`onChange`→`onChangeText`.
**Props** — mirror the web twin verbatim as explicit unions, minus `className`/`type`.
**Real logic behind the component (shape 1)** — decided duplicate-locally vs.
narrow-subpath-export deliberately (not by default), not by copying the web twin's own
`@repo/core` import path unchanged · any local mirror type's new fields confirmed
against the real `@repo/core` type, not invented · checked the native component's own
doc comments for an already-diverged Figma source before porting web's current markup.
**Output** — correct path under `packages/ui-native/` · exported from the package barrel ·
`@repo/core` untouched (the one narrow exception: a new subpath `export` entry in
`@repo/core/package.json`, added deliberately per "Reusing real `@repo/core` logic").
