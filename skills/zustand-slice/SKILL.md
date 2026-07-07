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

Generates a global client-state store slice in `@repo/core/state` that both apps (web + native) consume unchanged.

## Role Definition

Expert Zustand state engineer for a Turborepo monorepo, scaffolding **flat-file** store slices in `@repo/core/state` where state, actions, and selectors are colocated. Enforces the one rule — **global _client_ state only, and `@repo/core` stays platform-agnostic**: server/API data is React Query in the data layer (`feature-slice`), and `persist` reads its backend through **injected storage** (`getStateStorage()`) so the store never imports `localStorage`, `window`, `react-native`, or `react-native-mmkv`. Reads are always narrow selectors, never the whole store. Read root `CLAUDE.md` **§3 (golden rule)** and **§5** first — this skill enforces them.

## When to Use This Skill

- Adding a `packages/core/src/state/<name>Store.ts` slice, or a field/action to one.

**Not for:** server/API state (`feature-slice` — React Query), local component state + Context (`react-renderer`), design tokens (`design-system-setup`), or durable synced domain data (`@repo/offline-kit`).

## Core Workflow

1. **Choose the mode** — scaffold a new `state/<name>Store.ts` (Mode A), or **append** a field + its action (and, if useful, a selector) to an existing store (Mode B, never regenerate the file).
2. **Define the state shape** — an `interface` holding state + colocated actions; immutable updates in `set`. → `references/store-anatomy.md`
3. **Wire persistence (if needed)** — `persist` with **injected** `getStateStorage()`; never `localStorage`/`window`/`react-native`/`react-native-mmkv` directly. → `references/persist-and-storage.md`
4. **Export narrow selector hooks** — subscribe to a slice of the store; `useShallow` for multi-field selects; add the `state/index.ts` barrel export. → `references/selectors.md`
5. **Validate** — no whole-store subscription, no server data in the store, `tsc --noEmit` clean.

## Technical Guidelines

### What belongs in a store (and what doesn't)

| ✅ Global client state | ❌ Not a store |
| --- | --- |
| Cart, cross-screen filters, UI prefs | Server responses / lists (→ `feature-slice`) |
| Wizard / multi-step form progress | Auth **token vault** (→ `@repo/core` auth/secure storage) |
| Selected entity shared across screens | Durable synced records (→ `@repo/offline-kit`) |
| Session UI flags (e.g. `isSidebarOpen`) | State used by one component (→ `useState`, `react-renderer`) |

### Store anatomy (flat file)

`state/<name>Store.ts` holds the state interface, actions, the store, and its selector hooks; `state/index.ts` re-exports them. Persisted stores use `persist` with **injected** storage.

### Reference Guide

| Topic | Reference | Load when |
| --- | --- | --- |
| Store creator, state/actions typing, immutable updates, naming, barrel | `references/store-anatomy.md` | Writing the store |
| Narrow selectors, `useShallow`, selector hooks, re-render pitfalls | `references/selectors.md` | Reading from the store |
| Cross-platform injected persist storage + Next SSR hydration | `references/persist-and-storage.md` | Persisting a store |

### Canonical Pattern

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

## Constraints

### MUST DO

- Store **global client state only**; route server data to `feature-slice` and local/Context to `react-renderer`.
- Keep `@repo/core` platform-agnostic — inject persist storage via `getStateStorage()`; never import `localStorage`/`window`/`react-native`/`react-native-mmkv` in the store.
- Read via **narrow selectors** (`useCartStore((s) => s.items)`); export named selector hooks; use `useShallow` for multi-field selects.
- Make updates immutable in `set`; colocate actions with state.
- Use `interface` for the state shape, `type` for unions; never `any` (unknown → narrow).
- Use named exports + a `state/index.ts` barrel; store file `state/<name>Store.ts`, hook `use<Name>Store`.
- In Mode B, **append** to an existing store — never regenerate it.

### MUST NOT DO

- Store server responses / a React Query cache, an auth token vault, or offline records.
- Subscribe to the whole store (`useCartStore()` with no selector) — it causes needless re-renders.
- Mutate state in place, use default exports, or use `any`.

## Output Templates

When scaffolding or extending a store, provide:

1. `state/<name>Store.ts` — state interface, colocated actions, immutable `set` updates.
2. Injected `persist` storage (if persisted) via `getStateStorage()`.
3. Named narrow selector hooks (`use<Name>Items`, `use<Name>Count`, etc.).
4. The `state/index.ts` barrel export.

## Knowledge Reference

Zustand, create, persist, createJSONStorage, injected storage, getStateStorage, useShallow, narrow selectors, immutable updates, colocated actions, @repo/core/state, packages/core/state, global client state, platform-agnostic, skipHydration, SSR hydration, selector hooks, barrel exports
