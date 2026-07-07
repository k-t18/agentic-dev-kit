# Store Anatomy

A store slice is one flat file, `packages/core/src/state/<name>Store.ts`, re-exported from
`state/index.ts`. It stays platform-agnostic (§3) — no `window`/`localStorage`/`react-native`.

## File shape

```ts
// packages/core/src/state/filtersStore.ts
import { create } from 'zustand';

// 1. State + actions in one interface (actions colocated with the data they change)
interface FiltersState {
  query: string;
  category: string | null;
  setQuery: (query: string) => void;
  setCategory: (category: string | null) => void;
  reset: () => void;
}

// 2. The store
export const useFiltersStore = create<FiltersState>()((set) => ({
  query: '',
  category: null,
  setQuery: (query) => set({ query }),
  setCategory: (category) => set({ category }),
  reset: () => set({ query: '', category: null }),
}));

// 3. Selector hooks (see selectors.md)
export const useFiltersQuery = () => useFiltersStore((s) => s.query);
```

```ts
// packages/core/src/state/index.ts — barrel
export * from './filtersStore';
```

## Typing

- **`interface` for the state shape**, `type` for unions/aliases (CLAUDE.md §5). Never `any` —
  narrow `unknown`.
- Keep field names consistent with the domain. If a slice mirrors backend data, use the
  feature-local type (snake_case) from `features/<x>/<x>.types.ts` — don't redefine it.
- Actions are part of the state interface; type their params explicitly.

## Immutable updates

`set` must return a new object/array — never mutate in place.

```ts
// ❌ mutation — Zustand won't notice, selectors won't re-run
addTag: (tag) => set((s) => { s.tags.push(tag); return s; }),

// ✅ new array
addTag: (tag) => set((s) => ({ tags: [...s.tags, tag] })),
```

> **Deeply nested state?** Prefer flattening the slice. Only if nesting is unavoidable, add the
> `immer` middleware (`create<T>()(immer((set) => ...))`) so `set` can use draft mutation —
> don't hand-roll deep spreads.

## Naming

| Thing | Convention | Example |
| --- | --- | --- |
| File | `state/<name>Store.ts` (camelCase) | `cartStore.ts` |
| Store hook | `use<Name>Store` | `useCartStore` |
| Persist key | short, stable, kebab/lower | `'cart'` |
| Selector hooks | `use<Name><Field>` | `useCartItems` |

## When a flat file is too big

Default to the flat file — stores are usually small. Split into a folder
(`state/<name>/<name>.store.ts` + `.types.ts` + `selectors.ts` + `index.ts`) only when the
slice genuinely outgrows one screen of code. Keep the same barrel contract either way.

## Mode B — extending a store

**Append** a field, its action, and (optionally) a selector to the existing store. Never
rewrite the file or reorder existing members.

```ts
interface CartState {
  items: CartItem[];
  couponCode: string | null;          // ← added field
  addItem: (productId: string) => void;
  applyCoupon: (code: string) => void; // ← added action
  // ...existing members unchanged
}
```
