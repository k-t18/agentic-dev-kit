# repo.ts + hooks.ts

## `repo.ts` — the single local-vs-remote decision

Hooks call the repo and nothing lower. The repo delegates to `data/local` (SQLite) or
`data/remote` (HTTP) and returns domain objects. It **never** calls `getOfflineDb` itself.

```ts
import type { Order } from './order.types';
import { listLocalOrders } from './data/local';
import { orderRemote } from './data/remote';

export const orderRepo = {
  // online read
  listRemote: (tripId: string): Promise<Order[]> => orderRemote.getOrders(tripId),

  // offline read (delegates to data/local, which owns the SQL)
  listLocal: (): Promise<Order[]> => listLocalOrders(),

  // online write
  create: (payload: CreateOrderPayload): Promise<Order> => orderRemote.createOrder(payload),
};
```

For an online/offline flip, the repo is the ONLY file that changes — check connectivity
and choose the source; hooks and screens stay untouched.

## `hooks.ts` — React glue only

React Query wiring: `queryKey`, `enabled`, mapping. No SQL, no HTTP, no `getOfflineDb`.
Reads use `useApiQuery` (remote) or `useLocalQuery` (local); writes use `useApiMutation`.
Return through `queryShape` if the project preserves a legacy return shape.

```ts
import { useQueryClient } from '@tanstack/react-query';
import { useApiQuery } from '../../hooks/useApiQuery';
import { useApiMutation } from '../../hooks/useApiMutation';
import type { Order, CreateOrderPayload } from './order.types';
import { orderRepo } from './repo';

// GET — guard required params with !!
export function useGetOrders(tripId: string, enabled = true) {
  return useApiQuery<Order[]>({
    queryKey: ['orders', tripId],
    queryFn: () => orderRepo.listRemote(tripId),
    enabled: !!tripId && enabled,
  });
}

// POST — invalidate the list on success
export function useCreateOrder() {
  const queryClient = useQueryClient();
  return useApiMutation<Order, CreateOrderPayload>({
    mutationFn: orderRepo.create,
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['orders'] }),
  });
}
```

`enabled`: one required string param → `!!tripId`; two → `!!tripId && !!status`; none →
omit. Every hook exposes `data` / `isLoading` / `isError` — components handle all three.
