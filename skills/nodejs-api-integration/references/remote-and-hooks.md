# Fetch executors + `data/remote.ts` + repo + hooks

The fetch-based rewrite of a typical axios `http-methods` module, plus how the feature slice
(`data/remote.ts → repo.ts → hooks.ts`) consumes it through the engine seam.

## Fetch executors (the axios rewrite)

Reference implementations of `callGetAPI`/`callPostAPI` use axios; this monorepo bans axios
(CLAUDE.md §6), so implement them on `fetch`. Resolve the path from the registry, prepend `API_BASE_URL`, attach
`Authorization: token <token>`, and normalize errors to a consistent shape.

```ts
// packages/core/src/api/engine/nodeFetch.ts
import { resolveNodePath } from './resolvePath';
import type { NodeApiKey } from '../nodeEndpoints';

const BASE_URL = process.env.NEXT_PUBLIC_API_BASE_URL ?? '';

function authHeaders(token?: string, extra?: Record<string, string>): HeadersInit {
  return { Accept: 'application/json', ...(token ? { Authorization: `token ${token}` } : {}), ...extra };
}

async function toResult<T>(res: Response): Promise<T> {
  const body = (await res.json().catch(() => ({}))) as T;
  if (!res.ok) {
    // Surface the server's error field if present; keep a consistent Error type (never `any`).
    const msg = (body as { error?: string; exception?: string })?.error
      ?? (body as { exception?: string })?.exception
      ?? `Request failed (${res.status})`;
    throw new Error(msg);
  }
  return body;
}

export async function nodeGet<T>(apiName: NodeApiKey, data?: unknown, token?: string): Promise<T> {
  const path = resolveNodePath(apiName);
  const params = data && typeof data === 'object' ? new URLSearchParams(data as Record<string, string>).toString() : '';
  const url = params ? `${BASE_URL}${path}?${params}` : `${BASE_URL}${path}`;
  const res = await fetch(url, { method: 'GET', headers: authHeaders(token) });
  return toResult<T>(res);
}

export async function nodePost<T>(apiName: NodeApiKey, data?: unknown, token?: string): Promise<T> {
  const url = `${BASE_URL}${resolveNodePath(apiName)}`;
  // FormData must NOT get a JSON content-type — let fetch set the multipart boundary.
  const isForm = typeof FormData !== 'undefined' && data instanceof FormData;
  const res = await fetch(url, {
    method: 'POST',
    headers: authHeaders(token, isForm ? undefined : { 'Content-Type': 'application/json' }),
    body: isForm ? (data as FormData) : JSON.stringify(data ?? {}),
  });
  return toResult<T>(res);
}

export async function nodePut<T>(apiName: NodeApiKey, data?: unknown, token?: string): Promise<T> {
  const url = `${BASE_URL}${resolveNodePath(apiName)}`;
  const res = await fetch(url, {
    method: 'PUT',
    headers: authHeaders(token, { 'Content-Type': 'application/json' }),
    body: JSON.stringify(data ?? {}),
  });
  return toResult<T>(res);
}

export async function nodeDelete<T>(apiName: NodeApiKey, data?: unknown, token?: string): Promise<T> {
  const url = `${BASE_URL}${resolveNodePath(apiName)}`;
  const res = await fetch(url, {
    method: 'DELETE',
    headers: authHeaders(token, { 'Content-Type': 'application/json' }),
    body: JSON.stringify(data ?? {}),
  });
  return toResult<T>(res);
}
```

Notes vs a typical axios reference:
- **fetch, not axios** — same behavior (path resolve, base URL, `token <token>` auth, error
  normalization), no new dependency.
- **FormData** (e.g. a product-search hook that posts a `FormData`) is passed through untouched so
  the multipart boundary is set by the runtime — never hand-set `Content-Type` for it.
- Errors throw a real `Error` (never `any`), so React Query's `isError` works and the repo/hook
  need no special-casing.

## `data/remote.ts` — names the key + verb, nothing lower

The Node remote is a sibling of the Frappe `data/remote.ts` — never an edit to it. It calls the
**engine seam**, not a fetch executor directly:

```ts
// packages/core/src/features/product/data/remote.node.ts
import { runApi } from '../../../api/engine/runApi';
import type { Product } from '../product.types';

export const productNodeRemote = {
  getList: (filters: Record<string, string>, token?: string): Promise<Product[]> =>
    runApi<Product[]>('GET', 'get-product-list-api', filters, token),

  updateCart: (payload: Record<string, unknown>, token?: string): Promise<{ msg: string }> =>
    runApi<{ msg: string }>('POST', 'update-cart', payload, token),
};
```

## `repo.ts` — the single local-vs-remote **and** engine decision

```ts
// packages/core/src/features/product/repo.ts
import type { Product } from './product.types';
import { getProducts } from './data/local';            // offline read (getOfflineDb lives here)
import { productNodeRemote } from './data/remote.node'; // Node engine
// import { productRemote } from './data/remote';       // Frappe engine (feature-slice) — untouched

export const productRepo = {
  // repo owns the decision; hooks never see fetch/SQL/engine.
  list: (filters: Record<string, string>, token?: string): Promise<Product[]> =>
    // e.g. offline-first, else Node engine:
    // getConnectivityProvider().isOnline() ? productNodeRemote.getList(filters, token) : getProducts(),
    productNodeRemote.getList(filters, token),
};
```

## `hooks.ts` — React Query glue only

Identical to the Frappe slice's hook shape (`useLocalQuery` + `toLegacyShape`, or
`useApiQuery`/`useApiMutation`); the engine swap happens **below** the hook, so the hook is
unaware of which backend served it:

```ts
// packages/core/src/features/product/hooks.ts
import type { Product } from './product.types';
import { toLegacyShape, type LegacyQueryShape, useLocalQuery } from '@8848digital/catalyst';
import { productRepo } from './repo';

export function useGetProducts(filters: Record<string, string>, token?: string): LegacyQueryShape<Product[]> {
  return toLegacyShape(
    useLocalQuery({
      queryKey: ['products', filters],
      queryFn: () => productRepo.list(filters, token),
    }),
  );
}
```

For a **write** (POST/PUT/DELETE) use `useApiMutation` (online) or `usecases` + `outbox`
(offline) exactly as `feature-slice` prescribes — the engine seam changes only *which backend*
the write hits, not the offline-write machinery.

## Rules

- Executors are called **only** by the Node handler; `data/remote.ts` calls `runApi`; hooks call the
  repo. Never shortcut a layer.
- No axios; `encodeURIComponent` / `URLSearchParams` for query values; feature-local snake_case
  types; never `any` (`unknown` → narrow).
- The Node files (`remote.node.ts`, `nodeEndpoints.ts`, `api/engine/**`) are **new**; the Frappe
  `data/remote.ts` and `api/endpoints.ts` stay untouched.
