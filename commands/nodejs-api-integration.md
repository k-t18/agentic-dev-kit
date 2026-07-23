---
name: nodejs-api-integration
description: Integrate a Node.js / plain-REST endpoint into the @repo/core data layer via the dynamic engine seam, without touching the Frappe path. Runs the nodejs-api-integration skill.
argument-hint: "[paste a REST path / apiName+path, or 'seam' to scaffold the engine]"
---

# /nodejs-api-integration

**Arguments:** $ARGUMENTS

Runs the **`nodejs-api-integration`** skill — a path-based SDK registry, a dynamic Frappe⇄Node
engine seam, and **fetch** executors (never axios). Additive only: the existing Frappe path
(`api/endpoints.ts`, Frappe `data/remote.ts`, `@8848digital/catalyst`) is never modified.

## What to do

1. **Parse `$ARGUMENTS`:**
   - A REST path, `apiName → /api/path`, or an endpoint description → **add-endpoint mode**:
     register it in `api/nodeEndpoints.ts`, add the Node `data/remote.ts` method, wire repo + hook.
   - `seam` (or a request to set up the switch) → **scaffold-seam mode**: create
     `api/engine/{types,runApi,nodeHandler,nodeFetch,frappeHandler,resolvePath}.ts` once.
   - Empty → ask for the endpoint (apiName + path + verb) or whether to scaffold the seam.

2. **Confirm the backend is non-Frappe** — a REST path, not VME (`version/method/entity`). If it's
   Frappe, stop and use `/feature-slice` instead.

3. **Generate per the skill** (`references/sdk-registry.md`, `engine-seam.md`,
   `remote-and-hooks.md`):
   - `nodeEndpoints.ts` entry (key → path) — new file on first use, append after.
   - Engine seam (`runApi` + `nodeHandler` + fetch executors) — created once, reused.
   - Feature `data/remote.node.ts` method calling `runApi(method, key, data, token)`.
   - `repo.ts` (local-vs-remote + engine) and `hooks.ts` React Query wiring.

4. **Enforce the constraints** — no axios; every path via the registry (no inline URLs); calls go
   through `runApi` (never a hook/repo calling a fetch executor); `getOfflineDb`/SQL only in
   `data/**`; `@repo/core` stays platform-agnostic; `API_ENGINE` defaults to `frappe`.

5. **Print the summary** of files created/appended (note the Frappe path was left untouched).
