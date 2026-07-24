# The dynamic engine seam — Frappe ⇄ Node.js

The seam is what makes the data layer **backend-agnostic**: one entry point, `runApi`, chooses a
handler from a `handlers` registry keyed by `API_ENGINE`. It is the fetch-based, `@repo/core`
port of a common engine-runner pattern (a `handlers` map keyed by an engine-name env var).

## Shared handler contract

```ts
// packages/core/src/api/engine/types.ts
export type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';
export type Engine = 'frappe' | 'node';

/** Every engine handler has the same signature — the seam depends on this, not on a backend. */
export type ApiHandler = <T>(
  method: HttpMethod,
  apiName: string,
  data?: unknown,
  token?: string,
) => Promise<T>;
```

## The selector

```ts
// packages/core/src/api/engine/runApi.ts
import type { ApiHandler, Engine, HttpMethod } from './types';
import { nodeHandler } from './nodeHandler';
import { frappeHandler } from './frappeHandler';

const handlers: Record<Engine, ApiHandler> = {
  frappe: frappeHandler,
  node: nodeHandler,
};

/** Single entry point for the data layer. Engine chosen by config, defaulting to frappe. */
export function runApi<T>(method: HttpMethod, apiName: string, data?: unknown, token?: string): Promise<T> {
  const engine = (process.env.NEXT_PUBLIC_API_ENGINE as Engine) ?? 'frappe';
  const handler = handlers[engine];
  if (!handler) throw new Error(`Unsupported API engine: ${engine}`);
  return handler<T>(method, apiName, data, token);
}
```

## The Node handler (verb dispatch)

A method-dispatch handler — routes by HTTP verb to the fetch executors:

```ts
// packages/core/src/api/engine/nodeHandler.ts
import type { ApiHandler } from './types';
import { nodeGet, nodePost, nodePut, nodeDelete } from './nodeFetch';
import type { NodeApiKey } from '../nodeEndpoints';

export const nodeHandler: ApiHandler = (method, apiName, data, token) => {
  const key = apiName as NodeApiKey;
  switch (method) {
    case 'GET':    return nodeGet(key, data, token);
    case 'POST':   return nodePost(key, data, token);
    case 'PUT':    return nodePut(key, data, token);
    case 'DELETE': return nodeDelete(key, data, token);
    default:       throw new Error(`Unsupported method: ${method}`);
  }
};
```

## The Frappe handler (adapter, not a rewrite)

`frappeHandler` **wraps the existing catalyst path** so the seam has a uniform contract — it does
not reimplement or modify Frappe. Keep it a thin adapter over `api` + `buildEndpoint`:

```ts
// packages/core/src/api/engine/frappeHandler.ts
import { api } from '@8848digital/catalyst';
import type { ApiHandler } from './types';
// Map the logical apiName → the existing endpoints registry however your slice already does it.
// This adapter exists ONLY to give Frappe the same (method, apiName, data, token) shape.
```

If a slice is Frappe-only, it can keep calling `feature-slice`'s `data/remote.ts` directly and skip
the seam entirely. The seam is required only where a slice must be **switchable** between backends.

## Configuration

Two settings are chosen **once, at project setup** — ask the user and record both in `.env` /
`docs/environment.md` alongside `API_BASE_URL`:

- **Backend engine** — `NEXT_PUBLIC_API_ENGINE` (web, via Next) / `API_ENGINE` (native, via
  `react-native-config`) — values `frappe` (default) or `node`. Ask: *"Frappe or a Node/REST
  service (e.g. an EMR backend)?"*
- **Auth scheme** — `NEXT_PUBLIC_API_AUTH_SCHEME` (web) / `API_AUTH_SCHEME` (native) — values
  `token` (default) or `Bearer`. This is the `Authorization` prefix the fetch executors emit
  (`token <token>` for Frappe-style keys, `Bearer <jwt>` for JWT services). Ask: *"Bearer or
  token auth?"* — **never** hardcode it in the executor (see `references/remote-and-hooks.md`).

- **Default to `frappe` + `token`.** An unset engine or scheme must never change existing behavior.
- Engine can be global (one env for the app) or, if you need per-feature routing, pass an explicit
  engine argument down from the repo — but keep the default-frappe fallback.

## Rules

- The seam lives in `@repo/core` and uses only `fetch` + `process.env` — **no** `react-native`,
  `react-dom`, DOM-only globals, or axios (CLAUDE.md §3/§6).
- Handlers share one contract (`ApiHandler`); adding a third backend later = one more registry entry.
- Nothing above `data/remote.ts` (repo, hooks, screens) knows which engine ran — that is the point.
