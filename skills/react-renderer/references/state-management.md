# State Management

> **Scope: client state only.** Server/API state = React Query in the data layer →
> `feature-slice`. Global store slices (Zustand) → `zustand-slice`. This reference covers
> **local state + Context**; the Zustand/Query notes below just point you there.

## Local State (useState / useReducer)

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  const increment = () => setCount((prev) => prev + 1); // functional update
  return <button onClick={increment}>{count}</button>;
}
```

Reach for `useReducer` when several values change together or the next state depends on
the previous in non-trivial ways.

## Context for simple app-wide values

Good for theme/auth-style values that rarely change. Not a substitute for a store or
server cache.

```tsx
interface ThemeContextValue { theme: 'light' | 'dark'; toggle: () => void; }
const ThemeContext = createContext<ThemeContextValue | null>(null);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  const toggle = useCallback(() => setTheme((t) => (t === 'light' ? 'dark' : 'light')), []);
  return <ThemeContext.Provider value={{ theme, toggle }}>{children}</ThemeContext.Provider>;
}

export function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme must be inside ThemeProvider');
  return ctx;
}
```

## Global store — Zustand → `zustand-slice`

The canonical store-slice pattern (persist, selectors, `@repo/core/state`) is owned by the
**`zustand-slice`** skill — follow it when adding a store. Read with narrow selectors:

```tsx
const items = useCartStore((s) => s.items);   // subscribe to a slice, not the whole store
```

## Server / API state → `feature-slice`

Server state is **not** client state — it lives in the data layer (`useApiQuery` /
`useApiMutation` → repo). See **`feature-slice`**. **Never** fetch server data with
`useState` + `useEffect`.

## Quick Reference

| Solution | Best for |
|----------|----------|
| `useState` / `useReducer` | Local component state |
| Context | Theme, auth, simple app-wide values |
| Zustand (`zustand-slice`) | Global store slices |
| React Query (`feature-slice`) | Server / API state |
