# Our type patterns

## Placement — feature-local first

- A feature's own types live **with its slice**: `packages/core/features/<x>/<x>.types.ts`,
  exported through the feature's `index.ts` barrel.
- The **global** `packages/core/src/types/` holds **only genuinely-shared** types — used by
  ≥2 features or app-wide (e.g. `User`, `ApiError`, shared unions).
- Test: one feature uses it → local; shared across features → global. (Organise by domain,
  not by technical type — same reasoning as the slice architecture.)
- Never redefine a shared type in an app package — import it from `@repo/core`.

## Entity contract — one shape for local + remote

A feature's entity has **one** interface, shared by both data sources. The local SQLite
table mirrors the API contract (same field names), so whether the repo reads from the API
or from SQLite it returns the **same** shape. **No divergent `Local*` interface, no
mapper.** Mirror the backend: **snake_case** fields, optional `?` where a field may be
absent, never `any` (unknown → `string`). JSDoc the origin.

```ts
/** `get_orders` row — the shared contract for the API response AND the local table. */
export interface Order {
  name: string;            // PK — same on the server and in the local table
  customer: string;
  total_amount: number;
  status?: string;         // optional — absent on some responses
}
// repo.listRemote() and repo.listLocal() both return Promise<Order[]>.
```

## Request / raw-response / normalized-result triad

Keep the wire shapes separate from the app's normalized shape.

```ts
export interface LoginCredentials { usr: string; pwd: string; }         // request body
export interface LoginResponse { access_token: string; user: AuthUser } // raw API
export interface LoginResult { token: string; user: AuthUser }          // normalized app shape
```

## Discriminated unions for state

Model mutually-exclusive states as a union with a literal discriminant; narrow with a
`switch` (exhaustive `never` check) or a type guard.

```ts
export type SyncStatus = 'pending' | 'synced';

type RequestState =
  | { status: 'loading' }
  | { status: 'success'; data: Order[] }
  | { status: 'error'; error: Error };
```

`interface` for object shapes; `type` for unions/aliases.
