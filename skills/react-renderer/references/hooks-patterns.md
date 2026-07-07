# Hooks Patterns

> **Before writing a `useEffect`, ask:** (1) Is it necessary, or can the logic move to an
> event handler / a value derived during render? (2) Are all deps listed correctly?
> (3) Is cleanup implemented? (4) Should it be extracted into a custom hook? (5) Is it
> causing extra re-renders? Most "sync state to props/state" effects are unnecessary —
> compute during render or in the event handler instead. Effects are for **external**
> side-effects only (subscriptions, DOM, timers, non-React systems). Never fetch server
> data in an effect — use `@repo/core` React Query hooks (`feature-slice`).

## Where a custom hook lives (web+native)

Placement follows the golden rule — platform-agnostic logic in `@repo/core`, platform
APIs kept out of it:

| Hook touches…                                                   | Lives in                        | Native twin                                  |
| --------------------------------------------------------------- | ------------------------------- | -------------------------------------------- |
| Pure logic / timers only (`useDebounce`, `usePrevious`)         | `@repo/core` (shared)           | none — the same file runs on both            |
| `window` / DOM / `localStorage` / `matchMedia`                  | `packages/ui-web` (web-only)    | hand-write a twin (`Dimensions`, AsyncStorage) |
| Server / API data                                               | ❌ not here — `feature-slice`   | shared React Query hook, unchanged           |

> A hook that imports `window`, `document`, `localStorage`, or `matchMedia` can **never**
> live in `@repo/core` (golden rule). If you need it shared, keep the pure part in core and
> push the platform call to the component (`ui-web` / `ui-native`) or a platform interface.

## Custom Hook Pattern (non-data)

There is **no `useApi` here.** Server data is fetched through `@repo/core` React Query
hooks (`useApiQuery` / `useApiMutation`) owned by `feature-slice` — never a hand-rolled
`fetch` + `useState` + `useEffect`. This skill's custom hooks are for **non-data**
concerns only. Canonical example (platform-agnostic → `@repo/core`):

```tsx
// usePrevious — pure logic, belongs in @repo/core
import { useRef, useEffect } from 'react';

export function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();
  useEffect(() => {
    ref.current = value;
  }, [value]);
  return ref.current;
}
```

## useDebounce (platform-agnostic → `@repo/core`)

```tsx
function useDebounce<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debounced;
}

// Usage — debounce the input, then feed the stable value to a React Query hook.
// The effect only debounces; it never fetches (feature-slice owns the query).
function Search() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 300);
  const { data } = useSearchResults(debouncedQuery); // @repo/core hook, keyed by query
  return <SearchResults results={data} />;
}
```

## useLocalStorage (web-only → `packages/ui-web`)

Touches `localStorage`/`window`, so it **cannot** live in `@repo/core`. The native twin
uses AsyncStorage/MMKV. Note: durable app data belongs to `@repo/core` / `@repo/offline-kit`
(`OfflineDb`), not a component-level hook — use this only for view-local UI prefs.

```tsx
function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue;
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue] as const;
}
```

## useMediaQuery (web-only → `packages/ui-web`)

Uses `matchMedia`, so it stays out of `@repo/core`. The native responsive path is
`useWindowDimensions` / `Dimensions` — see `rn-component` / `react-native-expert`.

```tsx
function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(() =>
    typeof window !== 'undefined' && window.matchMedia(query).matches
  );

  useEffect(() => {
    const media = window.matchMedia(query);
    const listener = (e: MediaQueryListEvent) => setMatches(e.matches);

    media.addEventListener('change', listener);
    return () => media.removeEventListener('change', listener);
  }, [query]);

  return matches;
}

// Usage
function Layout() {
  const isMobile = useMediaQuery('(max-width: 768px)');
  return isMobile ? <MobileNav /> : <DesktopNav />;
}
```

## useCallback & useMemo

```tsx
// useCallback: Memoize functions (for child dependencies)
const handleClick = useCallback((id: string) => {
  setSelected(id);
}, []); // Empty deps = stable reference

// useMemo: Memoize expensive calculations
const sortedItems = useMemo(() =>
  [...items].sort((a, b) => a.name.localeCompare(b.name)),
  [items]
);

// When to use:
// - useCallback: When passing to memoized children
// - useMemo: When calculation is expensive AND deps rarely change
```

## Effect Cleanup

```tsx
useEffect(() => {
  const subscription = api.subscribe(handler);

  // Cleanup function
  return () => subscription.unsubscribe();
}, []);

// Async effect pattern
useEffect(() => {
  let cancelled = false;

  async function fetchData() {
    const data = await api.getData();
    if (!cancelled) setData(data);
  }

  fetchData();
  return () => { cancelled = true };
}, []);
```

## Quick Reference

| Hook | Purpose |
|------|---------|
| useState | Component state |
| useEffect | Side effects, subscriptions |
| useCallback | Memoize functions |
| useMemo | Memoize values |
| useRef | Mutable ref, DOM access |
| useContext | Read context |
| useReducer | Complex state logic |

| Custom Hook | Use Case | Placement |
|-------------|----------|-----------|
| useDebounce | Input delay | `@repo/core` (shared) |
| usePrevious | Prior value of a prop/state | `@repo/core` (shared) |
| useLocalStorage | View-local UI prefs | `ui-web` (web-only) |
| useMediaQuery | Responsive logic | `ui-web` (web-only) |

> Server data fetching is **not** a custom hook here — use `@repo/core` React Query hooks
> (`feature-slice`).
