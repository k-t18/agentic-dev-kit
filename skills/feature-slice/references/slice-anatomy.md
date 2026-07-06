# Slice anatomy

A feature is a vertical slice under `packages/core/src/features/<x>/`. One downward
dependency direction — never the reverse.

```
features/order/
├── order.types.ts  feature-local types — API + Local shapes, DTOs, mappers
├── data/
│   ├── remote.ts   HTTP source — orderRemote.*  (api.get/post via endpoints.*)
│   └── local.ts    SQLite source — getOfflineDb() SELECTs  (offline features only)
├── repo.ts         orderRepo — the ONLY place that decides local vs remote
├── hooks.ts        React Query wiring — calls orderRepo, never data/ directly
├── usecases.ts     offline writes — getOfflineDb + insertRow + transaction  (if it writes)
├── outbox.ts       pending queue + sync payload + OutboxAdapter              (if it writes)
└── index.ts        barrel
```

Types live **with the feature** in `order.types.ts` (not the global `types/`); only
genuinely-shared types go in `packages/core/src/types/`.

## Downward dependency

```
hooks.ts ──► repo.ts ──► data/local.ts (SQL)
                    └──► data/remote.ts (HTTP)
usecases.ts / outbox.ts ──► @repo/offline-kit (getOfflineDb, insertRow, OutboxAdapter)
features ──► shared-domain ──► @repo/offline-kit          (never the reverse)
```

- Only `data/**`, `usecases.ts`, `outbox.ts` (and `shared-domain/**`) may call
  `getOfflineDb`. Hooks and repo must delegate downward.
- `repo.ts` returns domain objects; hooks never see SQL or HTTP.

## `index.ts` barrel

Export the public surface — hooks, any offline use-cases pages call directly, the outbox
adapter (registered at app boot), and the repo:

```ts
// features/order/index.ts
export { useGetOrders, useCreateOrder } from './hooks';
export { executeCreateOrder } from './usecases';       // if it has offline writes
export type { CreateOrderResult } from './usecases';
export { orderOutboxAdapter } from './outbox';          // if it has offline writes
export { orderRepo } from './repo';
```

A **read-only online** feature needs only `data/remote.ts` + `repo.ts` + `hooks.ts` +
`index.ts`. Add `data/local.ts` for offline reads, and `usecases.ts` + `outbox.ts` for
offline writes.
