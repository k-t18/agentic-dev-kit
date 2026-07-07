# Selectors

How you read from a store decides how often the component re-renders. Always subscribe to the
**narrowest slice** you need.

## Narrow selectors

```ts
// ✅ subscribes only to items — re-renders when items change
const items = useCartStore((s) => s.items);

// ❌ subscribes to the ENTIRE store — re-renders on every field change
const store = useCartStore();
const items = store.items;
```

## Export named selector hooks

Colocate reusable selectors with the store so call sites stay clean and consistent.

```ts
// in cartStore.ts
export const useCartItems = () => useCartStore((s) => s.items);
export const useCartCount = () => useCartStore((s) => s.items.reduce((n, i) => n + i.qty, 0));

// in a component
const items = useCartItems();
```

## Multi-field selects → `useShallow`

Returning a **new object/array** from a selector every render causes an infinite re-render
loop (the reference differs each time). Use `useShallow` to compare fields shallowly.

```ts
import { useShallow } from 'zustand/react/shallow';

// ✅ new object, but re-renders only when query OR category actually change
const { query, category } = useFiltersStore(
  useShallow((s) => ({ query: s.query, category: s.category })),
);

// ❌ without useShallow — new object identity every render → re-render storm
const { query, category } = useFiltersStore((s) => ({ query: s.query, category: s.category }));
```

## Selecting actions

Actions are stable references, so selecting them never causes extra re-renders — select them
directly.

```ts
const addItem = useCartStore((s) => s.addItem);
```

## Outside React

Read/write without a hook via the store's static API (e.g. in a util or an event handler):

```ts
const items = useCartStore.getState().items;
useCartStore.getState().clear();
useCartStore.subscribe((s) => { /* react to changes */ });
```

## Pitfalls

| Pitfall | Fix |
| --- | --- |
| `useCartStore()` with no selector | Pass a selector — subscribe to a slice |
| Selector returns a fresh `{}`/`[]` each render | Wrap in `useShallow` (or select primitives separately) |
| Deriving heavy values in the selector every render | Select raw slice, derive in a `useMemo` (see `react-renderer`) |
| Selecting the whole store to grab one action | Select the action directly (`(s) => s.addItem`) |
