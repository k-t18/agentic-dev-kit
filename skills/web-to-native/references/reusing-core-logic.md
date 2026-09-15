# Reusing real `@app/core` logic in a `packages/ui-native` component

Applies when a web component's behavior isn't just JSX-over-props — it calls a pure
`@app/core` function (a cascading-dropdown filter, a live tax/total recompute, a
badge-list builder) or duplicates real domain logic inline. A leaf `packages/ui-native`
component can never import `@app/core`'s root barrel to get at that logic — decide
between the two moves below instead of defaulting to either one.

## Why the root barrel is off-limits for a component

`@app/core`'s root export (`@app/core` / `./index`) typically does
`export * from './hooks'` (or similar), which re-exports every feature's React Query
hooks — and therefore the whole API-engine/network graph — from one barrel file.
Importing **anything** from that barrel, even a single type via `import type`, forces
`tsc` to resolve the entire barrel's export graph to type-check the file. In practice
this has surfaced real breakage: a component importing one type dragged in
Node-only files, which then failed to compile in the RN app's `tsc` config (missing
`@types/node`, a DOM-only type leaking into a supposedly platform-agnostic file). The
failure mode that follows is worse than the original problem: "fixing" it by patching
`@app/core`'s `tsconfig`/`package.json`/build infra to make the barrel resolve — scope
creep into shared infrastructure that the component never actually needed touched.

A screen-level file (`apps/<native>/src/screens/**`) does **not** have this
restriction — it's expected to import hooks, types, and business logic from
`@app/core` freely, same as the web page it mirrors. This restriction is specifically
about components living in `packages/ui-native`.

## Decision: duplicate locally, or add a narrow subpath export

| | Duplicate locally | Narrow subpath export |
|---|---|---|
| When | Small (rule of thumb: under ~50 lines), low drift-risk (a UI-shaping predicate, a cascading option filter) | Substantial and/or correctness-critical (tax/price math, a derivation reused in more than one place) |
| Cost | A second copy that can silently drift from the original if the rule ever changes | One more export surface on `@app/core/package.json` to maintain |
| Where | Inside the component's own `.tsx`, with a comment naming the `@app/core` source file it mirrors | Imported directly via the new subpath, no copy at all |
| Types | Local mirror `interface`/`type` in the component's own `.types.ts` (same shape, not imported) | Import the real type via the same or a sibling subpath — no mirror needed |

Precedent from this pattern in practice: a ~30-line Metal→Purity/Tone cascade +
Diamond↔Grade sync algorithm was duplicated locally into a `CustomiseDrawer`-style
component (small, UI-shaping, low risk). A ~200-line GST slot-recompute algorithm
(`applySlotChange`/`computeSlotAmounts`) for an `InvoiceHeaderDrawer`-style component
was instead exposed via a narrow subpath export — real tax math is exactly the kind of
logic where a silent copy-paste drift produces a wrong number, not just a wrong pixel.

When genuinely unsure which side of the line a case falls on, ask rather than picking
silently — this is a real design decision with a real tradeoff, not a mechanical rule.

## Adding a narrow subpath export

1. Confirm the source file is actually pure first — no React, no hooks, no API/network
   calls, just functions over plain data in, plain data out. (This repo's convention is
   to say so explicitly in the file's own header comment — check for it.) If the file
   imports something impure, the export will still compile but silently defeats the
   whole point — you'll have re-created the barrel-bloat problem one file down.
2. Add one line to `@app/core/package.json`'s `"exports"` map, per file (a types file,
   if separate, gets its own line):

   ```json
   // packages/core/package.json
   "exports": {
     ".": "./src/index.ts",
     "./tokens/rn-styles": "./src/tokens/rn-styles.ts",
     "./features/<feature>/<file>": "./src/features/<feature>/<file>.ts",
     "./features/<feature>/<types-file>": "./src/features/<feature>/<types-file>.ts"
   }
   ```

3. Import from the component via that exact subpath — never the root:

   ```tsx
   import { applySlotChange, computeSlotAmounts } from '@app/core/features/invoice-header/mappers';
   import type { InvoiceHeaderSlot } from '@app/core/features/invoice-header/invoiceHeader.types';
   ```

4. **Verify both halves of resolution, not just one:**
   - `tsc` resolution — check the native app's (and `packages/ui-native`'s own)
     `tsconfig.json` for a wildcard path already covering `@app/core/*` (most repos
     that already import `@app/core/tokens/rn-styles` have one). If so, the new
     subpath resolves for free; nothing to add.
   - **Bundler (Metro) resolution** — this is what the `package.json` `"exports"` entry
     is actually for. A subpath that only resolves for `tsc` and not Metro fails at
     *run* time, not build time, and is easy to miss if you only ran `tsc --noEmit`
     before calling it done. Confirm the app actually starts / the screen using the
     component actually mounts, not just that the typecheck passed.

## Anti-patterns

- Importing `@app/core`'s root barrel "just for one type" inside a `packages/ui-native`
  component — even `import type` forces full barrel resolution.
- Duplicating 100+ lines of real business logic locally because adding a subpath export
  felt like "touching core" — a subpath export with zero logic changes is not the same
  risk as editing the logic itself; the "one rule" (`@app/core` never changes) is about
  the *logic*, not about whether a new doorway into an existing file exists.
- Adding the subpath export but never checking Metro can actually resolve it — a green
  `tsc --noEmit` is necessary, not sufficient.
