---
name: nodejs-api-integration
description: Use when integrating a Node.js / plain-REST backend into the @app/core data layer alongside the existing Frappe integration — a path-based SDK registry, a dynamic engine seam that lets the same repo/hook target Frappe OR Node.js by config, and fetch-based executors (never axios). Invoke for node api, REST endpoint, api engine, engine seam, sdk registry, nodeEndpoints, dynamic backend, frappe vs node, non-Frappe API.
license: MIT
metadata:
  author: https://github.com/k-t18
  version: "1.0.0"
  triggers: node api, nodejs api, REST endpoint, api engine, engine seam, engine runner, sdk registry, nodeEndpoints, dynamic backend, frappe or node, non-frappe api, path registry, api handler
  role: specialist
  scope: implementation
  output-format: code
  domain: frontend
  related-skills: feature-slice, typescript-teacher, react-renderer
---

# Node.js API Integration (dynamic engine, @app/core)

Adds a **Node.js / plain-REST backend** to the shared data layer in `@app/core` — a path-based SDK registry, fetch executors, and a **dynamic engine seam** — so a feature slice can target **Frappe or Node.js** by configuration, without ever rewriting the Frappe path.

## Role Definition

Expert data-layer engineer for a Turborepo monorepo (Next.js web + bare React Native, TanStack Query, `@8848digital/catalyst`). Specialist in making the API layer **backend-agnostic**: the existing Frappe integration (`buildEndpoint` → VME `version/method/entity`) stays untouched, and a parallel Node.js engine (logical `apiName → /api/...` path registry) is added behind the same `hooks → repo → data/remote` layering. A single **engine seam** — a `handlers` registry chosen at runtime by config — decides which backend a call hits. This skill is the Node.js counterpart of `feature-slice` (which owns the Frappe path); read root `CLAUDE.md` §3–§6 first — the golden rule and layering invariant still bind.

## When to Use This Skill

- Wiring a **Node.js / Express / plain-REST** endpoint (logical name → URL path) into a `@app/core` feature slice.
- Making the data layer **dynamic** so the same repo/hook can hit Frappe **or** Node.js depending on `API_ENGINE`.
- Porting an axios-based engine-runner / SDK-registry / http-methods pattern from another project into this monorepo — **fetch-based**, not axios.

**Not for:** Frappe endpoints or VME-shaped APIs (`buildEndpoint`, `message.data` envelope) → use `feature-slice`. UI components → `web-component` / `rn-component`. Tokens → `design-system-setup`.

## Core Workflow

1. **Setup (once per project)** — ask the user two questions and record the answers as env config, never hardcode them:
   - **Backend engine** — Frappe or a Node/REST service (e.g. an EMR backend)? → `NEXT_PUBLIC_API_ENGINE` (`frappe` default | `node`).
   - **Auth scheme** — `Bearer` or `token`? → `NEXT_PUBLIC_API_AUTH_SCHEME` (`token` default | `Bearer`), the `Authorization` prefix the executors emit.

   Defaults (`frappe` + `token`) keep existing behavior if unset. → `references/engine-seam.md`
2. **Confirm the backend is non-Frappe** — a REST path (`/api/get-products`), not a Frappe method (`version/method/entity`). If it's Frappe, stop and use `feature-slice`.
3. **Register the path** — add a typed entry to the Node SDK registry `api/nodeEndpoints.ts` (`apiName → '/api/...'`). Never inline a URL. → `references/sdk-registry.md`
4. **Ensure the engine seam exists** — the `runApi` selector + `handlers` registry keyed by `API_ENGINE` (`frappe` | `node`). Create it once; reuse thereafter. This is what makes the layer dynamic. → `references/engine-seam.md`
5. **Add the Node remote method** — in the feature's `data/remote.ts`, a method that calls the Node engine (fetch executor) with the registry key. Frappe's `data/remote.ts` is separate and unchanged. → `references/remote-and-hooks.md`
6. **Wire repo + hook** — `repo.ts` decides local-vs-remote **and** engine; `hooks.ts` stays glue (React Query), calling the repo. → `references/remote-and-hooks.md`
7. **Type it** — feature-local types in `<x>.types.ts`, snake_case mirroring the backend, never `any` (`unknown` → narrow).
8. **Validate** — no axios, no inline URLs, no hardcoded auth scheme, `getOfflineDb` absent from hooks/repo, Frappe path untouched, engine + auth scheme chosen by config only.

## Technical Guidelines

### The two backends, side by side

| Concern | Frappe (existing — `feature-slice`) | Node.js (this skill) |
| --- | --- | --- |
| Endpoint identity | `buildEndpoint('v1', doctype, method, APP)` (VME) | `apiName → '/api/path'` registry (`nodeEndpoints.ts`) |
| Envelope | `{ message: { data } }`, unwrapped by `api` | raw JSON (e.g. `{ msg, data, error }`) — no VME wrapper |
| Registry file | `api/endpoints.ts` | `api/nodeEndpoints.ts` (new, additive) |
| Transport | catalyst `api.get/post` (fetch) | fetch executor (this skill) — **never axios** |
| Selected by | (default) | engine seam, `API_ENGINE=node` |

**Golden rule:** the Node engine is **additive**. Do not modify `api/endpoints.ts`, the Frappe `data/remote.ts`, or catalyst. Coexistence, not replacement.

### The dynamic engine seam (the core idea)

Mirror the common engine-runner pattern (a `handlers` map keyed by an engine-name env var), but fetch-based and living in `@app/core`. A single `handlers` registry keyed by engine name; `API_ENGINE` (env/config) selects it:

```ts
// packages/core/src/api/engine/runApi.ts
import { nodeHandler } from './nodeHandler';
import { frappeHandler } from './frappeHandler'; // wraps the existing catalyst api

type Engine = 'frappe' | 'node';
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';

const handlers: Record<Engine, ApiHandler> = { frappe: frappeHandler, node: nodeHandler };

export function runApi<T>(method: HttpMethod, apiName: string, data?: unknown, token?: string): Promise<T> {
  const engine = (process.env.NEXT_PUBLIC_API_ENGINE as Engine) ?? 'frappe';
  const handler = handlers[engine];
  if (!handler) throw new Error(`Unsupported API engine: ${engine}`);
  return handler<T>(method, apiName, data, token);
}
```

- Default stays `frappe` so nothing regresses if `API_ENGINE` is unset.
- `nodeHandler` dispatches by verb to the fetch executors and resolves the path from `nodeEndpoints.ts`.
- **Platform-agnostic:** the seam lives in `@app/core` and uses only `fetch` + `process.env` — no `react-native`, no `react-dom`, no axios (CLAUDE.md §3/§6).

### Node SDK path registry

```ts
// packages/core/src/api/nodeEndpoints.ts
export const nodeEndpoints = {
  'get-product-list-api': '/api/get-products',
  'cart-list':            '/api/cart',
  'update-cart':          '/api/cart/update',
} as const;

export type NodeApiKey  = keyof typeof nodeEndpoints;
export type NodeApiPath = (typeof nodeEndpoints)[NodeApiKey];
```

### Fetch executors (not axios)

The Node counterpart of a typical axios http-methods module, rewritten on `fetch`: resolve the path from the registry, prepend `API_BASE_URL`, attach `Authorization` using the **configured scheme** (`NEXT_PUBLIC_API_AUTH_SCHEME` — `token` or `Bearer`, never hardcoded), `encodeURIComponent` query values, normalize errors. Full code → `references/remote-and-hooks.md`.

### Layering stays intact

```
hooks.ts → repo.ts → data/remote.ts → runApi(engine seam) → nodeHandler → fetch executor → nodeEndpoints
```

- **Hooks** are React Query glue only — no fetch, no SQL, no `getOfflineDb`.
- **repo.ts** is the single place that decides local-vs-remote **and** which engine.
- **data/remote.ts** names the registry key + verb; it does not build URLs or pick engines by itself.

### Reference Guide

| Topic | Reference | Load when |
| --- | --- | --- |
| Path registry (`nodeEndpoints.ts`) | `references/sdk-registry.md` | Adding/registering a Node endpoint |
| Dynamic engine seam + handlers | `references/engine-seam.md` | Creating/using the Frappe⇄Node switch |
| Fetch executors + repo + hooks | `references/remote-and-hooks.md` | Writing the HTTP layer and wiring the slice |

## Constraints

### MUST DO

- Keep the Node engine **additive** — Frappe (`api/endpoints.ts`, Frappe `data/remote.ts`, catalyst) stays byte-for-byte unchanged.
- Route every Node call through the **engine seam** (`runApi`) — never call a fetch executor straight from a hook or repo.
- Register every path in `nodeEndpoints.ts`; reference it by key — never inline a URL string.
- Use **`fetch`** for the Node transport; `encodeURIComponent` / `URLSearchParams` for query values.
- Keep the layering: `hooks → repo → data/remote → runApi`; hooks are glue only.
- Feature-local types in `<x>.types.ts`, snake_case, optional `?`, never `any` (`unknown` → narrow).
- Ask the **backend engine** and **auth scheme** at setup; drive the `Authorization` prefix from `NEXT_PUBLIC_API_AUTH_SCHEME` (`token` | `Bearer`) — never hardcode it in an executor.
- Default `API_ENGINE` to `frappe` and `API_AUTH_SCHEME` to `token` so an unset env never regresses existing behavior.

### MUST NOT DO

- Add or use **axios** (CLAUDE.md §6) — the reference pattern uses axios; you rewrite it on `fetch`.
- Hardcode the `Authorization` scheme (`token`/`Bearer`) in a fetch executor — read it from `NEXT_PUBLIC_API_AUTH_SCHEME`.
- Overwrite or fold the Frappe path into the Node engine, or edit `@8848digital/catalyst`.
- Import `react-native` / `react-dom` or DOM-only globals in the `@app/core` engine (keep it platform-agnostic; `fetch` + `process.env` only).
- Import `getOfflineDb` / run SQL or HTTP inside a hook or repo (DB layer / `data/**` only).
- Use `any`, default exports, or put API code in `apps/*`.

## Output Templates

When integrating a Node.js endpoint, provide:

1. The `nodeEndpoints.ts` entry (registry key → path) — new file on first use, appended thereafter.
2. The engine seam (`api/engine/runApi.ts` + `nodeHandler` + fetch executors) — created once, reused.
3. The feature's `data/remote.ts` method calling `runApi(method, key, data, token)`.
4. The `repo.ts` method (local-vs-remote + engine) and the `hooks.ts` React Query wiring.
5. Feature-local types in `<x>.types.ts`, snake_case, mirroring the backend response.

## Knowledge Reference

Node.js REST API, engine seam, engine runner, handlers registry, API_ENGINE, API_AUTH_SCHEME, Bearer vs token, auth scheme, setup questions, nodeEndpoints, SDK path registry, apiName to path, fetch executor, no axios, Authorization header, @app/core, feature slice layering, hooks-repo-remote, repo local vs remote, Frappe vs Node coexistence, buildEndpoint, catalyst api client, platform-agnostic core, snake_case types, encodeURIComponent, URLSearchParams
