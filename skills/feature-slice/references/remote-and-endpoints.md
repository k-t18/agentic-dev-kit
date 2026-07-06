# Endpoints + data/remote.ts (HTTP layer, Frappe-shaped)

## Endpoint registry — `packages/core/src/api/endpoints.ts`

Every endpoint is built with `buildEndpoint` and grouped by domain — never inline a URL.

```ts
// buildEndpoint('v1', module, fn) → /api/method/{app}.api.v1.{module}.{fn}
export const endpoints = {
  order: {
    getOrders:   buildEndpoint('v1', 'order', 'get_orders'),
    createOrder: buildEndpoint('v1', 'order', 'create_order'),
  },
  // …other domains (append a key to an existing group; add a group if new)
} as const;
```

## `data/remote.ts` — the HTTP source

An `xRemote` object of methods calling `api.get`/`api.post` with the endpoint. Frappe
returns `{ message: { data: … } }`; the `api` client unwraps the envelope, so methods are
typed to the **inner** data shape.

```ts
import { api } from '../../../api/client';
import { endpoints } from '../../../api/endpoints';
import type { Order, CreateOrderPayload } from '../order.types';

export const orderRemote = {
  // single query param → encodeURIComponent
  getOrders: (tripId: string): Promise<Order[]> =>
    api.get(`${endpoints.order.getOrders}?trip_id=${encodeURIComponent(tripId)}`),

  // multiple params → URLSearchParams
  getOrdersPaged: (params: { trip_id: string; page_no?: number }): Promise<Order[]> => {
    const q = new URLSearchParams();
    q.set('trip_id', params.trip_id);
    q.set('page_no', String(params.page_no ?? 1));
    return api.get(`${endpoints.order.getOrders}?${q.toString()}`);
  },

  // POST → body object
  createOrder: (payload: CreateOrderPayload): Promise<Order> =>
    api.post(endpoints.order.createOrder, payload),
};
```

Rules: `buildEndpoint` only (no inline `/api/method/...`); `encodeURIComponent` /
`URLSearchParams` for query values; snake_case params mirroring the backend; return types
are the inner (post-envelope) shape.
