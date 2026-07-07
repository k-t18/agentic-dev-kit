---
name: react-renderer
description: React discipline for this monorepo — effect design (the useEffect checklist), performance (memo/useMemo/useCallback, never index as key, React.lazy), error boundaries, Suspense, and custom (non-data) hooks. Defers component building to web-component, data/hooks/API to feature-slice, and global state to zustand-slice. Use for React patterns, effect/re-render/performance issues, error boundaries, or designing a custom hook.
license: MIT
metadata:
  author: https://github.com/k-t18
  version: "2.1.0"
  domain: frontend
  triggers: React, hooks, useEffect, useMemo, useCallback, memo, re-renders, performance, error boundary, custom hook, Suspense, useOptimistic
  role: specialist
  scope: implementation
  output-format: code
  related-skills: web-component, feature-slice, zustand-slice
---

# React Expert (discipline)

Cross-cutting React know-how: **effects, performance, error boundaries, and custom
non-data hooks.** It does **not** build components or own data/state — those have skills.

- Building a `ui-web` component → `web-component` (native → `rn-component`).
- Hooks / API / server data → `feature-slice`.
- Global store → `zustand-slice`; client state (local + Context) → `references/state-management.md`.

Read root `CLAUDE.md` first. Web is **React 19**, native is **React 18** — see
`references/react-19-features.md` for what's web-only.

## When to Use

- Deciding whether/how to write a `useEffect`; debugging re-renders or performance.
- Designing a **custom (non-data) hook**; adding an **error boundary**; using Suspense.

**Not for:** component anatomy (`web-component`/`rn-component`), data hooks/API
(`feature-slice`), or global state (`zustand-slice`).

## Core Workflow

1. **Prefer no effect** — can the logic be a value derived in render or an event handler?
   Only reach for `useEffect` for external side-effects. → `references/hooks-patterns.md`
2. **Implement** with strict TypeScript, named exports, stable keys, and effect cleanup.
3. **Optimize deliberately** — memoize only across a memoized boundary; split heavy trees
   with `React.lazy`. → `references/performance.md`
4. **Guard** — wrap async/error-prone subtrees in an error boundary; Suspense for async.
5. **Validate** — `tsc --noEmit` clean; no index-as-key; no server fetch in an effect.

## Reference Guide

| Topic | Reference | Load When |
|-------|-----------|-----------|
| Effects, custom hooks, cleanup, useEffect checklist | `references/hooks-patterns.md` | Writing an effect or custom hook |
| memo / useMemo / useCallback / lazy / virtualization | `references/performance.md` | Fixing re-renders or perf |
| Client state: local + Context (Zustand/Query pointers) | `references/state-management.md` | Choosing client state |
| React 19 client features (web-only) | `references/react-19-features.md` | `use()`, `useOptimistic`, ref-as-prop |

## Key Patterns

### Custom hook with cleanup

```tsx
import { useState, useEffect } from 'react';

// web-only (uses window); native would use Dimensions
export function useWindowWidth(): number {
  const [width, setWidth] = useState(() => window.innerWidth);
  useEffect(() => {
    const onResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', onResize);
    return () => window.removeEventListener('resize', onResize); // cleanup
  }, []);
  return width;
}
```

### Error boundary

```tsx
import { Component, type ErrorInfo, type ReactNode } from 'react';

interface Props { fallback: ReactNode; children: ReactNode; }
interface State { hasError: boolean; }

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };
  static getDerivedStateFromError(): State { return { hasError: true }; }
  componentDidCatch(error: Error, info: ErrorInfo) { /* report to a service */ }
  render() { return this.state.hasError ? this.props.fallback : this.props.children; }
}
```

Show a friendly fallback — **never surface a raw error to users.** Pair with graceful
API-error UI (the data hook's `isError` state, per `web-component` states).

**Know its limits:** an error boundary catches errors thrown during **render/lifecycle
only** — it does **not** catch async callbacks, event handlers, or SSR errors. Handle those
where they happen (try/catch, the mutation's `onError`). Keep one shared `ErrorBoundary` in
`packages/ui-web` (mirror it in `ui-native` — RN supports class error boundaries too) rather
than ad-hoc copies, and report from `componentDidCatch` to your error service (Sentry), not
`console`.

### Memoize only across a memoized boundary

```tsx
const Row = memo(function Row({ onSelect }: { onSelect: () => void }) { /* … */ });

// useCallback matters here because Row is memo'd; without it, Row re-renders every parent render
const onSelect = useCallback(() => selectItem(id), [id]);
```

## Constraints

### MUST DO
- Strict TypeScript; **named exports**; stable unique `key`s
- Prefer a derived value / event handler over a `useEffect`; clean up every effect
- Memoize callbacks/objects **only** when passed to a memoized child
- Error boundaries around async/error-prone subtrees; Suspense for async
- In `ui-web` handlers, use `onPress` (not `onClick`) for migration parity
- Place non-data custom hooks by platform: pure logic → `@repo/core`; `window`/DOM hooks →
  `ui-web` (never in `@repo/core`) with a hand-written native twin

### MUST NOT DO
- Mutate state directly; use array index as a `key`
- Fetch server data in a `useEffect` — use `@repo/core` hooks (`feature-slice`)
- Forget effect cleanup; ignore React strict-mode warnings
- Show raw errors to users; skip error boundaries in production
- Use server-action hooks (`useActionState`/`useFormStatus`/inline `'use server'`) in
  `ui-web`/`ui-native` — app shell only (`web+native`); forms there use React Query mutations
- Redefine component/data/state concerns owned by the other skills

## Knowledge Reference

React 19 (web) / 18 (native), effects, custom hooks, memoization, Suspense, error
boundaries, `use()` / `useOptimistic`, React Testing Library,
accessibility (see `web-component`).
