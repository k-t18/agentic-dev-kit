# Page composition — web pages and their native screen mirror

This is the reference for **shape 2** in the main skill file ("wiring a screen's
container hook") — a whole `use<Screen>.ts` composer, not a single `packages/
ui-native` component. It's the decision procedure for "where does this piece of
screen-level code go" on both platforms, so a native screen doesn't duplicate logic
web's page already proved out.

## The web shape

```
apps/web/app/<route-group>/
  [param]/page.tsx                    # 1. route entry (Server Component)
  components/
    <Route>Master.tsx                 # 2. the screen
    <Route><Section>.tsx              # 2a. screen sub-components (as many as needed)
  hooks/
    use<Route>Context.ts              # 3. web-local hook(s) — router/store-bound only
    use<Route><Concern>.ts            # 3. ...one per web-only concern
    use<Route>.ts                     # 3. composer — threads 3 + 4 into one view-model

packages/core/src/features/<feature>/
  hooks.ts                            # raw React Query hooks
  composers.ts                        # 4. portable container hooks (see below)
  mappers.ts                          # 4. pure business-rule functions
  <feature>.types.ts                  # types for all of the above
```

### 1. `<route-group>/[param]/page.tsx` — Server Component, route entry only

No `'use client'`. No `useState`/`useMemo`/hooks of any kind. Its only job is to
Suspense-wrap and render the screen component (a `Suspense` boundary is required the
moment the screen calls `useSearchParams()`, which it almost always will).

```tsx
import { Suspense } from 'react';
import { ItemDetailsMaster } from '../../components/ItemDetailsMaster';

export default function ItemDetailsPage() {
  return (
    <Suspense fallback={null}>
      <ItemDetailsMaster />
    </Suspense>
  );
}
```

### 2. `<route-group>/components/<Route>Master.tsx` — the screen

`'use client'` lives HERE, not on the page. Calls exactly one top-level composer
hook, destructures its view-model, and renders `@app/ui-web` components —
loading/error/empty/data branches only. No `useState`/`useMemo`/handlers of its own;
if you're reaching for one, it belongs in a hook instead.

### 2a. Screen sub-components

When the screen's JSX grows past one clean read (multiple visually-distinct
sections — a header, a main content grid, a set of side panels, …), split it into
sibling files in the SAME `components/` folder, one per section, each taking its own
slice of the view-model as explicit typed props (never the whole view-model object —
that just hides which fields a section actually uses). The Master file becomes pure
composition of these sections plus any single-shot overlays (a profile drawer, a
modal).

### 3. `<route-group>/hooks/use<Route>*.ts` — web-local hooks, ONLY the irreducibly platform-bound ones

**Do not default a hook here just because it's per-page.** When a page's logic
splits into several focused hooks (data, customization, cart, top-nav, …), test EACH
ONE independently — most of them have no actual web dependency and belong in
`@app/core` instead (see below). A hook earns a spot in `<route-group>/hooks/` only
if it directly touches:

- **Next-router APIs** — `useRouter`, `useSearchParams`, `useParams`,
  `next/navigation` generally, or building `router.push(...)` URL strings. No
  equivalent exists on native (React Navigation's `useRoute`/`useNavigation` read
  something structurally different), so this can never move to `@app/core`.
- **This app's own instantiated stores** (`@/stores` — `useAuthStore`,
  `useScopeStore`, etc.). Native will instantiate its own copies of the same
  factories; importing THIS app's instances into `@app/core` would be wrong even if
  the store *type* is shared.

Composed by a single top-level `use<Route>.ts` in the SAME `hooks/` folder, which
also pulls in the portable hooks from `@app/core` and threads everything into one
flat view-model — the screen component destructures it and does no further
derivation.

### 4. The rest of the container logic — `@app/core`, not the web app

A hook that's just `useState`/`useMemo` composed over `@app/core` React Query hooks,
with **no** `next/navigation`, **no** `@/stores` import, **no** DOM/`window`, is
platform-neutral **even though it's "a page's container hook."** It belongs in
`packages/core/src/features/<feature>/composers.ts` (sibling to that feature's
`hooks.ts`/`mappers.ts`), reusable verbatim by a future native container hook. A
common failure mode: several container hooks get placed under `apps/web` by
pattern-matching "hooks live near the page," when most of them have no actual web
dependency once individually checked. Check each file, not the folder.

Two gotchas that silently make an otherwise-portable hook non-portable:

- **A raw env-var read** (`process.env.NEXT_PUBLIC_*`) inside the hook. Native reads
  `Config.*` (`react-native-config`) instead — accept the resolved value as a
  parameter (e.g. `imageBaseUrl: string`) and let each platform's own thin
  container hook or composer supply it.
- **An import from `@app/ui-web`** (even just a type). Core must never depend on
  ui-web. Define the equivalent shape as a plain interface in the feature's own
  `<feature>.types.ts` instead — it'll still structurally match whatever prop the
  web/native component expects, no import needed.

### Re-audit — don't stop at the first split

Even after the first triage, re-read what's LEFT in `<route-group>/hooks/` line by
line, specifically hunting for conditional/derivation logic buried inside an
otherwise router-bound hook. A hook can legitimately need `useRouter`/`@/stores` for
90% of its body while still hiding a pure, portable rule in the other 10%. Two
concrete patterns to watch for:

- A multi-branch derivation over plain values sitting next to a `useRouter()`/
  `useSearchParams()` call in the same hook. Extract it to `<feature>/mappers.ts`.
- The exact same request-shaping logic copy-pasted more than once **in the same
  file** — a hard signal it should be one function in `@app/core`, not two/three
  inline literals.

### Extraction rules

1. Any derivation/guard/payload-builder that is plain-data-in-plain-data-out (no
   hook, no router, no DOM) belongs in `packages/core/src/features/<feature>/
   mappers.ts`. Use a generic bag type (`Readonly<Record<string, unknown>> | null |
   undefined`) when the function shouldn't hard-depend on another feature's concrete
   type.
2. A helper reusable across ≥2 features/pages with no dependency on one feature's own
   data shape goes in `packages/core/src/utils/<name>.ts`. One specific to a single
   feature's own business rules goes in that feature's own `mappers.ts` — never
   duplicated into the page/hook layer "just this once."
3. Presentational JSX with no data behind it (a full-page loading skeleton, an
   empty-state illustration) is still UI — promote it to a named `@app/ui-web`
   component. `app/**` must never author UI, and neither should a container hook.
4. Small derivations left inside a screen sub-component are lower priority than the
   same thing inside a hook — `packages/ui-web` → `packages/ui-native` is already a
   tracked rewrite (the main `web-to-native` skill), so JSX-level logic gets a
   natural second look when that happens; hook-level logic doesn't get that free
   check.

## The native mirror — `apps/<native>/src/screens/<Screen>/`

Once a screen migrates to native, it follows the **identical** decomposition, with
one seam swapped: Next's router/DOM primitives become React Navigation/RN
primitives, everything else stays exactly as portable as it already was.

```
apps/<native>/src/screens/<Screen>/
  <Screen>Screen.tsx                    # native's Master — pure composition, calls the composer once
  components/
    <Screen><Section>.tsx               # screen sub-components, same split rule as web's 2a
  hooks/
    use<Screen>Nav.ts                   # native-local hook(s) — nav/store-bound only, see below
    use<Screen><Concern>.ts             # ...one per native-only concern
    use<Screen>.ts                      # composer — threads native-local + the SAME @app/core composers as web into one view-model
```

### Which hooks earn a spot in native's `hooks/`

A hook belongs here only if it directly touches:

- **React Navigation's `useNavigation`/`useRoute`** — native's analogue of Next's
  `useRouter`/`useSearchParams`/`useParams`. No portable form exists: route params
  are a structurally different shape from a URL query string, so this can never move
  to `@app/core`.
- **This app's own instantiated stores** (`apps/<native>/src/stores` —
  `useAuthStore`, `useScopeStore`, …). Native instantiates its OWN copies of the same
  store factories web uses — importing native's instances into `@app/core` would be
  wrong even though the store's *type* is shared.

### Everything else is already portable — don't rewrite it, consume it

The native composer (`use<Screen>.ts`) calls the **exact same `@app/core`
composers** web's own `use<Route>.ts` calls. Nothing native-specific needs writing
there; native gets the portable logic for free the moment it calls the same function
web already proved out. Two checks that catch the two ways this goes wrong:

- **About to write new derivation logic directly inside a native container hook?**
  Stop and check whether web's composer has the equivalent yet. If web doesn't have
  it either, the logic belongs in `@app/core`'s `composers.ts`/`mappers.ts` first —
  built once, consumed by both platforms — not written native-only.
- **A `@app/core` composer's return field never appears in the native composer's
  own return object?** That's almost always a missed prop-threading step, not an
  intentional omission. A composer hook can be CALLED (its network requests fire, its
  derivation runs) while its result is silently dropped — this doesn't error, so
  `tsc` won't catch it. Before considering a native screen "done," diff the
  `@app/core` hook's full return type against what the native composer's own
  `return { … }` actually forwards, field by field.

Two gotchas, mirroring the web-side gotchas exactly, just with native's own platform
primitives:

- **A raw env-var read** (`Config.X` from `react-native-config`) inside an
  otherwise-portable hook. `process.env.NEXT_PUBLIC_*` doesn't exist in the RN
  runtime and `Config.*` doesn't exist in Next's — a hook that reads either directly
  can never be portable. Read it in the screen's composer, pass the resolved
  boolean/string down as a parameter.
- **An import from `@app/ui-native`** (even just a type) inside a hook that's
  otherwise portable — same mistake as a `@app/ui-web` import on web. Define the
  equivalent shape as a plain interface in the feature's own `<feature>.types.ts`.

### Worked example shape (illustrative)

For a screen like `ProductCategory`, both platforms' `useProductCategory.ts`
composers call the identical `@app/core` container hooks (`useProductFilterConfig`,
`useCustomiseAction`, `useMoveToForm`, `usePrintActions`, …) — zero business logic
duplicated. Each platform only differs in its own thin native-local/web-local pair
(nav+store-bound state, a right-panel/modal state machine built the same shape on
both sides with each platform's own store/nav primitives underneath) and in the one
genuinely platform-specific leaf at the very end of a pipeline — e.g. web's
`downloadBlob` (a DOM `<a download>`/`window.open` side effect) vs. native's
equivalent using `FileReader.readAsDataURL` + a native share-sheet library. That leaf
has no portable form by design and stays in each platform's own screen/hook layer,
never in `@app/core`.

## MUST NOT DO

- `'use client'` in `page.tsx`.
- A raw `fetch`/`api` call anywhere outside `@app/core`.
- JSX authored inside a container hook (hooks return data + handlers, never markup).
- A Next-only concept (`useRouter`, `next/navigation`, `window`, `document`,
  `@/stores`) imported into `packages/core` or `packages/ui-web`.
- Placing a hook under `apps/web/app/<route-group>/hooks/` by default because it's
  "the page's container hook" — check it against the portability test first.
- Stopping at the first hook split without a re-audit pass.
- A raw `process.env.NEXT_PUBLIC_*` read, or a `@app/ui-web` type import, inside a
  hook that's otherwise portable.
- A one-off full-page skeleton (or any other real JSX) left inline in a page or
  screen file instead of promoted to a named `@app/ui-web` component.
- Duplicating a pure helper that already exists in `packages/core/src/utils/` or
  another feature's `mappers.ts` instead of importing it.
- (native) A raw `Config.*` read, or a `@app/ui-native` type import, inside a hook
  that's otherwise portable.
- (native) Writing new derivation/business logic directly inside a native container
  hook that web's composer doesn't already have.
- (native) Improvising a native composer's shape from scratch instead of mirroring
  the already-proven web composer field-for-field — and shipping a screen without
  diffing the `@app/core` hook's full return type against what the composer
  actually forwards.
