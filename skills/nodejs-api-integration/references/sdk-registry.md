# Node SDK path registry — `api/nodeEndpoints.ts`

The Node.js analog of Frappe's `buildEndpoint`. Where Frappe identifies an endpoint by
**VME** (`version/method/entity`), a Node/REST backend identifies it by a **URL path**. Map a
stable logical `apiName` to that path once, in a single registry, and reference it by key
everywhere else — never inline a URL string.

## The registry

```ts
// packages/core/src/api/nodeEndpoints.ts

/**
 * Node.js / REST endpoint registry — logical apiName → server path. The Node
 * counterpart of Frappe's api/endpoints.ts (buildEndpoint). Additive: this file
 * is new; it never replaces the Frappe registry. Paths are relative to
 * API_BASE_URL, prepended by the fetch executors.
 */
export const nodeEndpoints = {
  'get-product-list-api': '/api/get-products',
  'product-detail-api':   '/api/get-product-details',
  'cart-list':            '/api/cart',
  'update-cart':          '/api/cart/update',
  'delete-cart':          '/api/cart/clear',
  'place-order-api':      '/api/orders/create',
  // …append one line per endpoint
} as const;

export type NodeApiKey  = keyof typeof nodeEndpoints;
export type NodeApiPath = (typeof nodeEndpoints)[NodeApiKey];
```

`as const` makes the values literal types; `NodeApiKey` is the union of registry keys, so
`runApi('GET', 'get-product-list-api', …)` is checked at compile time and a typo is a type error.

## Resolving a key to a path

The engine's fetch executors resolve a key through one tiny helper:

```ts
// packages/core/src/api/engine/resolvePath.ts
import { nodeEndpoints, type NodeApiKey } from '../nodeEndpoints';

export function resolveNodePath(apiName: NodeApiKey): string {
  const path = nodeEndpoints[apiName];
  if (!path) throw new Error(`Unknown Node API key: ${apiName}`);
  return path;
}
```

## Rules

- **One registry, keyed by logical name.** Group-by-domain is optional; a flat map is fine and
  keeps keys greppable.
- **Never inline a path** in `data/remote.ts`, a service fn, or a hook — always a registry key.
- **Additive only.** Do not merge Frappe endpoints into this file, and do not touch
  `api/endpoints.ts`.
- **snake_case / kebab keys mirror the backend's own naming** so the registry reads like the
  server's route table.
