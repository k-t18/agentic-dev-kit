# CLAUDE.md — Project Root

> Single source of truth for Claude Code across all sessions and developers.
> This file holds only the **always-on invariants**. Task-specific procedures (building components, migrating web→native, writing API hooks, etc.) live in `.claude/skills/` and load on demand.

---

## 1. Project overview

**Web-first, then React Native.** The full frontend — UI, API integration, state,
business logic — is built on web (ReactJS) first. After QA approval, components
are migrated to React Native. **Only the component layer changes** during
migration; everything in `/packages/core` is shared and never rewritten.

**Stack**

- Web: Next.js (React) + TypeScript + Tailwind CSS
- Native: React Native CLI (bare, no Expo) + TypeScript + StyleSheet
- Shared: React Query (TanStack), Zustand, TypeScript — the **fetch-based `api` client
  from `@8848digital/catalyst`** (no axios)
- Tooling: pnpm · Turborepo · Figma MCP · GitHub Actions

**Project mode** — every project is exactly one of these. Declare it at the top of
`apps/web/CLAUDE.md`; it decides whether the portability rules below apply:

- **`web-only`** — no native target, no `ui-native`. Use Next.js to the fullest:
  Server Components by default, server actions, server-side fetching. `"use client"`
  only where interactivity needs it (standard Next). The §5 web-layering boundary
  and its §6 guards **do not apply.**
- **`web+native`** — a native twin is planned. The §5 web-layering boundary is
  **enforced** so migration stays clean.

---

## 2. Monorepo map (top level)

```
apps/
  web/      Next.js app             — app router, layouts     · has its own CLAUDE.md
  native/   React Native CLI (bare) — src/screens, src/navigation · has its own CLAUDE.md
packages/
  core/         Platform-agnostic shared logic (features, api, state, tokens, types, utils)  → @app/core
  ui-web/       Web component library (React + Tailwind)                                      → @app/ui-web
  ui-native/    Native component library (RN + StyleSheet)                                    → @app/ui-native
```

**Installed engine + chassis — external packages, NOT in `packages/`.** They ship as
versioned deps you install from GitHub Packages; never vendored or edited here:

- **`@8848digital/offline-kit`** — the feature-agnostic **offline engine**: `OfflineDb`,
  sync, outbox, and `getOfflineDb`. Exposes the boot seams the apps wire up.
- **`@8848digital/catalyst`** — the **chassis** on top of offline-kit: the fetch-based
  `api` client, React Query wiring (`QueryProvider`), auth (`createAuthStore`), and the
  Frappe sync transport (`httpSyncTransport`).

---

## 3. The golden rule

> **`/packages/core` is platform-agnostic. It must NEVER import from
> `react-native`, `react-dom`, Tailwind, or any platform-specific library.**

Tempted to add platform-specific code to `core`? Stop. Put it in `ui-web` or
`ui-native` instead.

---

## 4. Core layering invariant (ESLint-enforced, severity `error`)

Each feature is a vertical slice with a single downward dependency direction:

```
hooks.ts → repo.ts → data/local.ts (SQL) + data/remote.ts (HTTP)
                   ↘ usecases.ts / outbox.ts
features → shared-domain → @8848digital/offline-kit   (never the reverse)
```

- **Only the DB layer runs SQL.** `getOfflineDb` (from `@8848digital/offline-kit`) is
  allowed _only_ in `features/*/data/**`, `features/*/usecases.ts`, `features/*/outbox.ts`,
  and `shared-domain/**`. Hooks and repos must delegate downward.
- `axios` is banned package-wide — the API layer is the fetch-based `api` client from
  `@8848digital/catalyst`.

---

## 5. Non-negotiable rules (apply to every edit)

**Tokens** — Never hardcode a color, spacing value, font size, or radius. Always
reference a named token (`colors.brand.primary`, `spacing[4]`). Tokens live in
`packages/core/src/tokens` and are generated from Figma — they are the only source.
_(Building a component? The `web-component` / `rn-component` skills carry the token
tables and usage.)_

**TypeScript**

- Never use `any` — use `unknown` and narrow.
- A feature's types live with its slice (`packages/core/src/features/<x>/<x>.types.ts`);
  the global `packages/core/src/types/` holds **only genuinely-shared** types (used by
  ≥2 features or app-wide). Never redefine a shared type in an app package.
- `interface` for object shapes, `type` for unions/aliases.
- API response types are defined before the hook that uses them.

**Imports** — Always use the workspace alias (`@app/core/hooks`, `@app/ui-web`),
never relative paths across packages.

**Naming** — Components + their files PascalCase (`Button.tsx`); variables, functions,
props, and other files camelCase; constants UPPER_CASE; hooks `use*`, handlers `handle*`.
Folders: component folders PascalCase, feature folders kebab-case, technical dirs lowercase.

**Components** — Named exports only (never default); always re-exported from the
package `index.ts` barrel. Every component handles default, disabled, and loading
states.

**Web layering (Next.js App Router)** — _Applies to `web+native` projects only._
The web app has two layers with different fates. Keep them separate or migration
breaks:

- `apps/web/app/**` — the **Next shell** (server-side, _never_ migrated). Server
  Components, server actions, route handlers, and server-only APIs (`next/headers`,
  `cookies()`, `async` RSC data fetching) live here and _only_ here. This layer owns
  routing, layouts, metadata, and data-loading, then renders `ui-web` components.
- `packages/ui-web` — the **migration surface** (client-side, mirrored to
  `ui-native`). Every component is a client component (`"use client"`), pure React,
  no server-only APIs. Data arrives via props or `@app/core` React Query hooks —
  never via server-side fetching.

Rule of thumb: **if a component will have a native twin, it must run identically
without a server.** Server-only concerns stay in the app shell.

**Code quality** — SOLID / single-responsibility, KISS, YAGNI. Keep components
focused; if one exceeds ~250 lines, split it or extract a custom hook.

---

## 6. What Claude must never do

- ❌ Hardcode a color, spacing value, font size, or radius
- ❌ Import from `react-native` inside `/packages/core`
- ❌ Import from `react-dom` or use DOM APIs inside `/packages/core`
- ❌ Use the `any` type
- ❌ Use inline styles on web components (`style={{ color: 'red' }}`)
- ❌ Use CSS shorthand properties in React Native `StyleSheet`
- ❌ Remove or modify `watchFolders` in `apps/native/metro.config.js`
- ❌ Create a type/interface in an app package that belongs in `@app/core` (feature slice or global `types/`)
- ❌ Dump a feature's own types into the global `types/` — keep them feature-local (`features/<x>/<x>.types.ts`)
- ❌ Use default exports for components — always named exports
- ❌ Use `%` widths in React Native without the `Dimensions` API
- ❌ Import `getOfflineDb` (run SQL) in a hook or repo — DB layer only (§4 layering invariant)
- ❌ Add `axios` — the API layer is the fetch-based `api` client from `@8848digital/catalyst`
- ❌ Vendor or edit `@8848digital/offline-kit` / `@8848digital/catalyst` — they are installed external packages
- ❌ _(web+native only)_ Use Server Components, server actions, or server-only APIs (`next/headers`, `cookies()`, `async` RSC fetching) inside `packages/ui-web` — that layer stays `"use client"` and migratable
- ❌ _(web+native only)_ Fetch data server-side in a component slated for native — use `@app/core` React Query hooks

---

## 7. Skills index (load on demand)

Procedures live in `.claude/skills/`. The relevant one loads automatically when
its task comes up; this list is orientation:

| Skill                 | Use when                                                                                                            |
| --------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `design-system-setup` | Extracting/updating design tokens from Figma; the `tokens.ts` + `rn-styles.ts` pipeline                             |
| `web-component`       | Building a web component (`packages/ui-web`) — tokens, anatomy, Figma pull                                          |
| `rn-component`        | Building a native component (`packages/ui-native`)                                                                  |
| `web-to-native`       | Migrating a QA-approved web component to React Native                                                               |
| `feature-slice`       | Scaffolding a `packages/core/src/features/<feature>` slice **or** adding an API endpoint (hooks→repo→data + outbox) |
| `zustand-slice`       | Adding a Zustand store slice                                                                                        |
| `native-setup`        | Metro / native build resolution issues                                                                              |

**Agents & commands:** `design-qa` is a **read-only QA agent** (`.claude/agents/`, run
via `/agentic-dev-kit:design-qa <Name>` or `/agentic-dev-kit:design-qa all`) that verifies a built `ui-web` component
against its Figma source — flags only, never edits. `web-component` hands off to it
after a build.

---

## 8. Branch strategy

```
main         production only · protected · PR + 1 approval + CI pass
develop      integration · features merge here first
feat/*       new features        fix/*        bug fixes
chore/*      tooling/config/deps  migration/*  web-to-native migrations
```

PR checklist is enforced via `.github/pull_request_template.md` (Design QA vs
Figma · tokens only · all three states handled · types in the correct location · Metro
verified for native · CI green).

---

## 9. Environment variables

`API_BASE_URL` (the Frappe REST base, fed to `setBaseUrl`) · `FIGMA_TOKEN`
(`.claude/settings.json`, Figma MCP). Web loads via Next
(`process.env.NEXT_PUBLIC_API_BASE_URL`); native via `react-native-config`
(`Config.API_BASE_URL`). Never commit `.env` — use `.env.example`; real values in
GitHub Secrets. Full table: `docs/environment.md`.

---

_Maintained by: Frontend · When conventions change, update this file first, then the code._
