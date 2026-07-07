---
name: zustand-slice
description: Use when adding or editing a global client-state store slice in packages/core/src/state (Zustand). Scaffolds a flat-file store (state + actions + selectors colocated) with immutable updates, narrow selector hooks, and cross-platform persist via injected storage (core stays platform-agnostic — the app wires localStorage on web / MMKV on native). Two modes: scaffold a new store, or append a field/action to an existing one. Not for server state (feature-slice), local/Context state (react-renderer), or offline domain data (offline-kit). Invoke for zustand store, global state, store slice, persist, selectors, @repo/core/state.
license: MIT
metadata:
  author: https://github.com/k-t18
  version: "1.0.0"
  domain: frontend
  triggers: zustand, zustand store, global state, store slice, persist, selector, useShallow, @repo/core/state, packages/core/state, client state store
  role: specialist
  scope: implementation
  output-format: code
  related-skills: feature-slice, react-renderer, design-system-setup
---

# Zustand Slice (packages/core/state)

Generates a **global client-state store slice** in `@repo/core/state`. Both apps
(web + native) consume the store unchanged.

Read root `CLAUDE.md` **§3 (golden rule)** and **§5** first — this skill enforces them.

## The one rule

> **Global _client_ state only, and `@repo/core` stays platform-agnostic.**

- **Client state, not server state.** Server/API data is React Query in the data layer
  (`feature-slice`). A store never caches server responses.
- **Platform-agnostic core.** The store lives in `@repo/core`, so it must **never** import
  `localStorage`, `window`, `react-native`, or `react-native-mmkv`. `persist` reads its
  backend through **injected storage** (`getStateStorage()`); each app wires the real
  adapter at startup. → `references/persist-and-storage.md`
- **Read with narrow selectors** — subscribe to a slice of the store, never the whole thing.
  → `references/selectors.md`

## When to Use

- Adding a `packages/core/src/state/<name>Store.ts` slice, or a field/action to one.

**Not for:** server/API state (`feature-slice` — React Query), local component state +
Context (`react-renderer`), design tokens (`design-system-setup`), or durable synced domain
data (`@repo/offline-kit`).

## What belongs in a store (and what doesn't)

| ✅ Global client state | ❌ Not a store |
| --- | --- |
| Cart, cross-screen filters, UI prefs | Server responses / lists (→ `feature-slice`) |
| Wizard / multi-step form progress | Auth **token vault** (→ `@repo/core` auth/secure storage) |
| Selected entity shared across screens | Durable synced records (→ `@repo/offline-kit`) |
| Session UI flags (e.g. `isSidebarOpen`) | State used by one component (→ `useState`, `react-renderer`) |

## Modes

- **Mode A — scaffold a new store:** create `state/<name>Store.ts` (+ barrel export). →
  `references/store-anatomy.md`
- **Mode B — extend a store:** **append** a field + its action (and, if useful, a selector)
  to an existing `state/<name>Store.ts`. Never regenerate the file.

## Store anatomy (flat file)

`state/<name>Store.ts` holds the state interface, actions, the store, and its selector hooks;
`state/index.ts` re-exports them. Persisted stores use `persist` with **injected** storage.

## Reference Guide

| Topic | Reference | Load when |
| --- | --- | --- |
| Store creator, state/actions typing, immutable updates, naming, barrel | `references/store-anatomy.md` | Writing the store |
| Narrow selectors, `useShallow`, selector hooks, re-render pitfalls | `references/selectors.md` | Reading from the store |
| Cross-platform injected persist storage + Next SSR hydration | `references/persist-and-storage.md` | Persisting a store |

## Canonical pattern

```ts
// packages/core/src/state/cartStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { getStateStorage } from './storage'; // injected per-platform (see persist-and-storage.md)
import type { Product } from '../features/products'; // feature-local type, snake_case

interface CartItem {
  product_id: string;
  qty: number;
}

interface CartState {
  items: CartItem[];
  // actions colocated with state; updates are immutable
  addItem: (productId: string) => void;
  removeItem: (productId: string) => void;
  clear: () => void;
}

export const useCartStore = create<CartState>()(
  persist(
    (set) => ({
      items: [],
      addItem: (productId) =>
        set((state) => {
          const existing = state.items.find((i) => i.product_id === productId);
          return existing
            ? { items: state.items.map((i) => (i.product_id === productId ? { ...i, qty: i.qty + 1 } : i)) }
            : { items: [...state.items, { product_id: productId, qty: 1 }] };
        }),
      removeItem: (productId) =>
        set((state) => ({ items: state.items.filter((i) => i.product_id !== productId) })),
      clear: () => set({ items: [] }),
    }),
    {
      name: 'cart',
      storage: createJSONStorage(() => getStateStorage()), // NOT localStorage/MMKV directly
      skipHydration: true, // rehydrate on the client — see persist-and-storage.md
    },
  ),
);

// Named selector hooks — narrow subscriptions, clean call sites
export const useCartItems = () => useCartStore((s) => s.items);
export const useCartCount = () => useCartStore((s) => s.items.reduce((n, i) => n + i.qty, 0));
```

```ts
// packages/core/src/state/index.ts — barrel
export * from './cartStore';
export { configureStateStorage } from './storage';
```

> **Non-persisted store?** Drop the `persist(...)` wrapper entirely:
> `export const useUiStore = create<UiState>()((set) => ({ ... }));`

## Rules

- ✅ Global **client** state only; server data → `feature-slice`, local/Context → `react-renderer`
- ✅ `@repo/core` stays platform-agnostic — persist storage is **injected** (`getStateStorage()`);
  never import `localStorage`/`window`/`react-native`/`react-native-mmkv` in the store
- ✅ Read via **narrow selectors** (`useCartStore((s) => s.items)`); export named selector hooks;
  `useShallow` for multi-field selects
- ✅ Immutable updates in `set`; actions colocated with state
- ✅ `interface` for the state shape, `type` for unions; **no `any`** (unknown → narrow)
- ✅ Named exports + `state/index.ts` barrel; store file `state/<name>Store.ts`, hook `use<Name>Store`
- ✅ Mode B **appends** to an existing store — never regenerate it
- ❌ No server responses / React Query cache in a store; no auth token vault; no offline records
- ❌ No whole-store subscription (`useCartStore()` with no selector) — causes needless re-renders
- ❌ No mutating state in place; no default exports; no `any`
