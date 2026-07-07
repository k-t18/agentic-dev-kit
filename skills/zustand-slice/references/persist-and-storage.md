# Persist & Storage (cross-platform)

Persisting a store is the one place platform storage is involved — and `@repo/core` **§3
forbids** importing `localStorage`, `window`, or `react-native-mmkv`. So the store persists
through an **injected** storage adapter: core defines the seam, each app supplies the real
backend at startup.

## 1. Core defines the seam (platform-agnostic)

```ts
// packages/core/src/state/storage.ts
import type { StateStorage } from 'zustand/middleware';

let stateStorage: StateStorage | null = null;

// Called once per app at startup with a platform-specific adapter.
export function configureStateStorage(adapter: StateStorage): void {
  stateStorage = adapter;
}

export function getStateStorage(): StateStorage {
  if (!stateStorage) {
    throw new Error('configureStateStorage() must be called at app startup before a persisted store is used.');
  }
  return stateStorage;
}
```

The store references only `getStateStorage()` — never a concrete backend:

```ts
persist(creator, {
  name: 'cart',
  storage: createJSONStorage(() => getStateStorage()),
  skipHydration: true, // hydrate on the client — see §4
});
```

## 2. Web app wires localStorage (with an SSR guard)

`localStorage` doesn't exist during Next SSR, so guard every access.

```ts
// apps/web — run once at startup (e.g. a "use client" providers component)
import { configureStateStorage } from '@repo/core/state';

configureStateStorage({
  getItem: (name) => (typeof window === 'undefined' ? null : window.localStorage.getItem(name)),
  setItem: (name, value) => { if (typeof window !== 'undefined') window.localStorage.setItem(name, value); },
  removeItem: (name) => { if (typeof window !== 'undefined') window.localStorage.removeItem(name); },
});
```

## 3. Native app wires MMKV

```ts
// apps/native/index.js (or App bootstrap) — run once at startup
import { MMKV } from 'react-native-mmkv';
import { configureStateStorage } from '@repo/core/state';

const mmkv = new MMKV();

configureStateStorage({
  getItem: (name) => mmkv.getString(name) ?? null,
  setItem: (name, value) => mmkv.set(name, value),
  removeItem: (name) => mmkv.delete(name),
});
```

> This is the "MMKV adapter is the mechanism" that `react-native-expert/references/storage-hooks.md`
> points here for — the store itself is owned by this skill; the app only supplies the backend.

## 4. Next.js SSR hydration

A persisted store's value comes from client storage, which the server can't see — reading it
during SSR causes a hydration mismatch. Use `skipHydration: true` and rehydrate on the client.

```tsx
// apps/web — a "use client" component mounted once near the root
'use client';
import { useEffect } from 'react';
import { useCartStore } from '@repo/core/state';

export function StoreHydration() {
  useEffect(() => {
    void useCartStore.persist.rehydrate();
  }, []);
  return null;
}
```

Native has no SSR — `skipHydration` is harmless there (rehydrate on mount the same way, or omit
it for native-only stores).

## What NOT to persist

- **Server data / React Query cache** — it has its own persistence story (`feature-slice`).
- **Secrets / auth tokens** — those belong in `@repo/core` auth / secure storage, not a general
  persisted store keyed in localStorage/MMKV.
- **Ephemeral UI flags** you don't want to survive a reload (`isModalOpen`) — leave the store
  non-persisted, or use `partialize` to persist only the durable fields:

```ts
persist(creator, {
  name: 'cart',
  storage: createJSONStorage(() => getStateStorage()),
  partialize: (s) => ({ items: s.items }), // persist items only, not transient flags
});
```
