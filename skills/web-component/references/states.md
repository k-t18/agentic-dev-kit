# Component states

Every component handles all states **applicable to its type** — never skip one.

## States by component type

**Interactive** (Button, Link, Chip): default · hover (`hover:`) · active (`active:`) ·
focus (`focus-visible:ring-*`) · disabled (`disabled:opacity-50 disabled:pointer-events-none`) ·
loading (spinner + disabled).

**Input** (TextInput, Select, Checkbox, Radio): default/empty · focused · filled/has-value ·
disabled · error (error border + message) · success (where applicable).

**Data display** (Card, List, Table): loading skeleton · empty (no data) · error (fetch
failed) · populated.

**Async-driven**: consume and render all three React Query states.

```tsx
const { data, isLoading, isError, error } = useGetSomething();

if (isLoading) return <LoadingSkeleton />;
if (isError)   return <ErrorState message={error.message} />;
if (!data)     return <EmptyState />;
return <PopulatedView data={data} />;
```

## Loading pattern

```tsx
export function Button({ label, loading, disabled, onPress, ...props }: Readonly<ButtonProps>) {
  return (
    <button onClick={onPress} disabled={loading || disabled} aria-busy={loading || undefined}
      className={cn(buttonVariants(props), loading && 'cursor-wait')}>
      {loading && <Loader2 className="h-4 w-4 animate-spin" aria-hidden="true" />}
      <span>{label}</span>
    </button>
  );
}
```

## Input error pattern

```tsx
export function TextInput({ label, error, ...props }: Readonly<TextInputProps>) {
  const inputId = useId();
  const errorId = `${inputId}-error`;
  return (
    <div className="flex flex-col gap-1.5">
      <label htmlFor={inputId} className="text-sm font-medium text-text-primary">{label}</label>
      <input
        id={inputId}
        aria-describedby={error ? errorId : undefined}
        aria-invalid={!!error}
        className={cn(
          'w-full rounded-md border px-3 py-2 text-sm transition-colors',
          'border-border-default bg-surface-canvas text-text-primary placeholder:text-text-disabled',
          'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-border-focus',
          error && 'border-error focus-visible:ring-error',
        )}
        {...props}
      />
      {error && (
        <span id={errorId} role="alert" className="flex items-center gap-1 text-xs text-error">
          <AlertCircle className="h-3 w-3" aria-hidden="true" />
          {error}
        </span>
      )}
    </div>
  );
}
```

## Empty state pattern

```tsx
export function EmptyState({ icon: Icon, title, description, action }: Readonly<EmptyStateProps>) {
  return (
    <div className="flex flex-col items-center justify-center gap-3 rounded-lg border border-dashed border-border-default p-12 text-center">
      {Icon && (
        <div className="rounded-full bg-surface-subtle p-3">
          <Icon className="h-6 w-6 text-text-secondary" aria-hidden="true" />
        </div>
      )}
      <div className="space-y-1">
        <p className="text-sm font-medium text-text-primary">{title}</p>
        {description && <p className="text-sm text-text-secondary">{description}</p>}
      </div>
      {action}
    </div>
  );
}
```

## Icons (lucide-react only)

Size with Tailwind `h-*/w-*`, never `width`/`height` props. Decorative icons get
`aria-hidden="true"`; icon-only buttons get `aria-label`.

| Context | Size |
| --- | --- |
| Inline with text (sm) | `h-3 w-3` |
| Inline with text (default) | `h-4 w-4` |
| Standalone / button | `h-5 w-5` |
| Empty state / hero | `h-6 w-6` |
| Large decorative | `h-8 w-8` |

```tsx
<Loader2 className="h-4 w-4 animate-spin" aria-hidden="true" />        // loading
<CheckCircle className="h-4 w-4 text-success" aria-hidden="true" />     // decorative
<button aria-label="Clear search"><X className="h-4 w-4" aria-hidden="true" /></button>
```
