# usecases.ts + outbox.ts (offline writes + sync)

For offline-first writes: the use-case writes locally as `sync_status='pending'`; the
outbox later pushes it to the server. `getOfflineDb` is legal here (not in hooks/repo).

## `usecases.ts` — the local write

```ts
import { getOfflineDb, insertRow } from '@repo/offline-kit';
import { generateUuid } from '../../utils/uuid';
import { invalidateLocalDataAfterWrite } from '../../lib/invalidateLocalData';
import type { CreateOrderLocalInput } from './order.types';

export interface CreateOrderResult { orderId: string; }

export async function executeCreateOrder(input: CreateOrderLocalInput): Promise<CreateOrderResult> {
  const orderId = generateUuid();
  const createdAt = new Date().toISOString();
  const db = await getOfflineDb();

  await db.transaction(async (tx) => {
    await insertRow(tx, 'order', {
      id: orderId,
      customer: input.customer,
      total_amount: input.total_amount,
      sync_status: 'pending',   // ← queued for the outbox
      remote_name: null,
      created_at: createdAt,
    });
    for (const line of input.items) {
      await insertRow(tx, 'order_item', { id: generateUuid(), order: orderId, item_code: line.item_code, qty: line.qty });
    }
  });

  invalidateLocalDataAfterWrite();   // refresh mounted local-read views
  return { orderId };
}
```

Rules: one `db.transaction` (all-or-nothing); default `sync_status:'pending'`,
`remote_name:null`; generate local UUIDs; a throw inside the transaction rolls it all back.

## `outbox.ts` — pending queue + payload + adapter

```ts
import { getOfflineDb, type CreateRecordLogPayload, type OutboxAdapter } from '@repo/offline-kit';
import type { PendingOrder, OrderRow, OrderItemRow } from './order.types';

// oldest-first work queue
export async function listPendingOrders(): Promise<PendingOrder[]> {
  const db = await getOfflineDb();
  const orders = await db.all<OrderRow>(`SELECT * FROM "order" WHERE sync_status = 'pending' ORDER BY created_at ASC`);
  const out: PendingOrder[] = [];
  for (const order of orders) {
    const items = await db.all<OrderItemRow>('SELECT * FROM order_item WHERE "order" = ?', [order.id]);
    out.push({ order, items });
  }
  return out;
}

// serialize to the server's create_record_log body (json is STRINGIFIED)
export function buildOrderSyncPayload(pending: PendingOrder): CreateRecordLogPayload {
  const record = {
    customer: pending.order.customer,
    total_amount: pending.order.total_amount,
    items: pending.items.map((l) => ({ item_code: l.item_code, qty: l.qty })),
  };
  return { doctype_name: 'Order', record_id: pending.order.id, json: JSON.stringify(record) };
}

// flip pending → synced after a confirmed push (idempotent)
export async function markOrderSynced(orderId: string, remoteName: string): Promise<void> {
  const db = await getOfflineDb();
  await db.exec(`UPDATE "order" SET sync_status = 'synced', remote_name = ? WHERE id = ?`, [remoteName, orderId]);
}

// registered at app boot; priority orders the drain across adapters
export const orderOutboxAdapter: OutboxAdapter<PendingOrder> = {
  name: 'order',
  priority: 10,
  listPending: listPendingOrders,
  buildPayload: buildOrderSyncPayload,
  markSynced: markOrderSynced,
  idOf: (p) => p.order.id,
  countPending: async () => (await listPendingOrders()).length,
};
```

Notes: `record_id` = local UUID (the server's dedupe key — retries are safe); `json` is a
**stringified** record with a nested child array; confirm the exact Frappe fieldnames with
the backend — this file is the single place that mapping lives.
