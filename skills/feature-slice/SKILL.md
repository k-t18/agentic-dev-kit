---
name: feature-slice
description: Use when creating a new packages/core/features/<feature> vertical slice, or adding an API endpoint to an existing one, following the core layering invariant. Generates the correct downward layers — hooks → repo → data/local(SQL) + data/remote(HTTP), with offline writes via usecases + outbox — keeping getOfflineDb out of hooks/repo so the ESLint boundary holds. Frappe-shaped API (buildEndpoint, message.data envelope, snake_case). Two modes: scaffold a whole slice, or add one endpoint (type + endpoint + remote + repo + hook) from a pasted curl/JSON/field-list. Invoke for feature slice, add endpoint, curl to api, api hook, repo/outbox scaffolding.
license: MIT
metadata:
  author: https://github.com/k-t18
  version: "1.0.0"
  domain: frontend
  triggers: feature slice, vertical slice, scaffold feature, add endpoint, curl to api, api hook, repo, data/remote, data/local, outbox, usecases, packages/core/features
  role: specialist
  scope: implementation
  output-format: code
  related-skills: design-system-setup
---

# Feature Slice (packages/core/features)

Generates layering-correct code for the shared data/API layer in `@repo/core`: either a
whole new vertical slice, or a single endpoint added to an existing one. Both apps
(web + native) consume the result unchanged.

Read root `CLAUDE.md` §4 first — this skill enforces it.

## The layering invariant (the one rule)

```
hooks.ts → repo.ts → data/local.ts (SQL) + data/remote.ts (HTTP)
                  ↘ usecases.ts / outbox.ts   (writes + sync)
```

- **Hooks are React glue only** — React Query wiring; **no SQL, no HTTP, no `getOfflineDb`.**
- **Data access lives only in a feature's `data/`** — a `SELECT`/`INSERT` only in
  `data/local.ts`; an HTTP call only in `data/remote.ts`. Both hidden behind `repo.ts`.
- **`getOfflineDb` is allowed only** in `data/**`, `usecases.ts`, `outbox.ts` (and
  `shared-domain/**`). Never in a hook or repo — ESLint blocks it (severity error).
- `repo.ts` is the single place that decides **local vs remote**.

## When to Use

- Creating a `packages/core/features/<feature>` slice, or adding an endpoint to one.

**Not for:** UI components (`web-component`/`rn-component`), tokens
(`design-system-setup`), or app-level code.

## Prerequisite (assumed present)

The shared data-layer infra: `@repo/offline-kit` (`getOfflineDb`, `insertRow`, `OutboxAdapter`,
`CreateRecordLogPayload`), `api/client.ts` (`api`), `hooks/useApiQuery.ts` +
`useApiMutation.ts`, `lib/` (`useLocalQuery`, `queryShape`, `invalidateLocalDataAfterWrite`),
`utils/uuid`. This skill *uses* these — it does not build them.

## Types

Types live **with the feature** in `features/<x>/<x>.types.ts` (exported via the slice
barrel); the global `packages/core/src/types/` holds only genuinely-shared types. Mirror the backend:
**snake_case** field names, optional fields marked `?`, never `any` (unknown → `string`).

## Mode A — scaffold a new slice

Create `packages/core/src/features/<x>/`:
- `data/remote.ts` — `xRemote` HTTP methods → `references/remote-and-endpoints.md`
- `data/local.ts` — `getOfflineDb` SQL reads (offline features) → `references/offline-writes.md`
- `repo.ts` — `xRepo`, the local-vs-remote decision → `references/repo-and-hooks.md`
- `hooks.ts` — React Query wiring calling the repo → `references/repo-and-hooks.md`
- `usecases.ts` + `outbox.ts` — offline writes + sync (only if the feature writes)
  → `references/offline-writes.md`
- `index.ts` — barrel (hooks, use-cases, outbox adapter, repo) → `references/slice-anatomy.md`

## Mode B — add an endpoint (the curl case)

Parse the pasted curl / Frappe URL / JSON / field-list → domain, method, params, response.
→ `references/parse-input.md`. Then **append only** (never regenerate existing files):
1. a type in `features/<x>/<x>.types.ts` (or the global `types/` if genuinely shared)
2. an entry in `api/endpoints.ts` via `buildEndpoint`
3. a method on `data/remote.ts` (`xRemote`)
4. a method on `repo.ts` (`xRepo`)
5. a hook in `hooks.ts` + its barrel export

Create the slice folder first if it doesn't exist. `GET` → `useApiQuery`;
`POST`/`PUT`/`PATCH`/`DELETE` → `useApiMutation` (online write) or `usecases` + `outbox`
(offline write).

## Reference Guide

| Topic | Reference | Load when |
| --- | --- | --- |
| Slice folder + layering + barrel | `references/slice-anatomy.md` | Scaffolding a slice |
| Parse a curl/JSON/spec | `references/parse-input.md` | Adding an endpoint (mode B) |
| Endpoints + `data/remote.ts` | `references/remote-and-endpoints.md` | Writing the HTTP layer |
| `repo.ts` + `hooks.ts` | `references/repo-and-hooks.md` | Wiring the repo + hook |
| `usecases.ts` + `outbox.ts` | `references/offline-writes.md` | Offline writes + sync |

## Rules

- ✅ Enforce the layering invariant; `getOfflineDb` only in `data/**`/`usecases`/`outbox`
- ✅ Hooks are glue only (no SQL/HTTP); a hook calls the **repo**, nothing lower
- ✅ Endpoints via `buildEndpoint(...)` — never inline the URL string
- ✅ `encodeURIComponent` every query-param value (or `URLSearchParams` for multiple)
- ✅ Types in `@repo/core/types`, snake_case, optional `?`; named exports + barrels
- ✅ Append to existing files — **never overwrite** existing code
- ❌ No SQL/HTTP in a hook or repo; never import `getOfflineDb` there
- ❌ Never touch `api/client.ts` (infrastructure); never add `axios`
- ❌ No `any` (unknown → `string`); no default exports; no API code in `apps/*`
