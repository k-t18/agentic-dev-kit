# Performance Optimization

## React.memo

```tsx
import { memo } from 'react';

// Memoize component - only re-renders when props change
const ExpensiveList = memo(function ExpensiveList({ items }: { items: Item[] }) {
  return (
    <ul>
      {items.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
});

// Custom comparison function
const UserCard = memo(
  function UserCard({ user }: { user: User }) {
    return <div>{user.name}</div>;
  },
  (prevProps, nextProps) => prevProps.user.id === nextProps.user.id
);
```

## Preventing Re-renders

Referential stability is the point here — **not** styling. Styling is always token
Tailwind classes (never inline `style`, never a hardcoded color); the "new object every
render" trap applies to any object/array/function prop passed to a memoized child.

```tsx
// Problem: new object + new function identity on each render
function Parent({ items }: { items: Item[] }) {
  // ❌ `config` and the handler are recreated every render → break memo on <Child>
  return <Child config={{ pageSize: 20 }} onPress={() => doSomething()} />;
}

// Solution: lift stable data out, memoize the callback
const config = { pageSize: 20 }; // module-scope constant — one identity forever

function Parent({ items }: { items: Item[] }) {
  const handlePress = useCallback(() => doSomething(), []);
  return <Child config={config} onPress={handlePress} />;
}
```

## Code Splitting — heavy in-page subtrees only

**Route-level splitting is Next's job, not yours.** The App Router already code-splits per
route segment; don't hand-roll React Router lazy routes in `apps/web`. Reach for `lazy()`
only to defer a *heavy subtree inside a component* (a chart, an editor, a rarely-opened
panel) that isn't on the first paint.

```tsx
// Inside a ui-web component — defer a heavy client subtree
import { lazy, Suspense } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));

function Dashboard({ showChart, data }: { showChart: boolean; data: ChartData }) {
  return (
    <Suspense fallback={<Loading />}>
      {showChart && <HeavyChart data={data} />}
    </Suspense>
  );
}
```

In the **app shell** (`apps/web/app/**`, `web+native` only) prefer `next/dynamic` — it
composes with SSR (`{ ssr: false }` to skip server render for a client-only widget):

```tsx
import dynamic from 'next/dynamic';
const AdminPanel = dynamic(() => import('@repo/ui-web').then((m) => m.AdminPanel), {
  loading: () => <Loading />,
});
```

> Native has no `lazy()` route story — RN navigation and `FlatList` windowing cover it
> (`react-native-expert`). `lazy()`/`next/dynamic` are web-only.

## Virtualization (web-only)

For long lists (1000+ rows) on **web**, virtualize with `@tanstack/react-virtual`. This is
the one place inline `style` is legitimate — the virtualizer computes absolute pixel
offsets per row, which can't be expressed as static token classes. Keep colors/spacing on
token classes; only the positioning geometry stays inline.

> **Native twin uses `FlatList`/`FlashList`, not this** — RN virtualizes natively; do not
> port `react-virtual` to `ui-native` (`rn-component` / `react-native-expert`).

```tsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }: { items: Item[] }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
  });

  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize() }}>
        {virtualizer.getVirtualItems().map((virtualItem) => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: virtualItem.start,
              height: virtualItem.size,
            }}
          >
            {items[virtualItem.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

## useMemo for Expensive Calculations

```tsx
function Analytics({ data }: { data: DataPoint[] }) {
  // Only recalculate when data changes
  const stats = useMemo(() => ({
    total: data.reduce((sum, d) => sum + d.value, 0),
    average: data.reduce((sum, d) => sum + d.value, 0) / data.length,
    max: Math.max(...data.map(d => d.value)),
  }), [data]);

  return <StatsDisplay stats={stats} />;
}
```

## useTransition for Non-urgent Updates

```tsx
import { useTransition } from 'react';

function Search() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<Item[]>([]);
  const [isPending, startTransition] = useTransition();

  function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    setQuery(e.target.value); // Urgent: update input immediately

    startTransition(() => {
      // Non-urgent: can be interrupted
      setResults(filterItems(e.target.value));
    });
  }

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending ? <Spinner /> : <Results items={results} />}
    </>
  );
}
```

## Quick Reference

| Technique | When to Use |
|-----------|-------------|
| `memo()` | Prevent re-renders from unchanged props |
| `useMemo()` | Cache expensive calculations |
| `useCallback()` | Stable function references |
| `lazy()` | Code split heavy components |
| `useTransition()` | Keep UI responsive during updates |
| Virtualization | Large lists (1000+ items) |

| Anti-pattern | Fix |
|--------------|-----|
| Inline objects | Lift out or useMemo |
| Inline functions | useCallback |
| Large bundle | lazy() + Suspense |
| Long lists | Virtualization |
