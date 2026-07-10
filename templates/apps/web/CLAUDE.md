# CLAUDE.md — apps/web (Next.js App Router)

> Web-shell context. **Root `CLAUDE.md` is the source of truth** for cross-cutting
> rules (tokens, core layering, TypeScript, the "never do" list). This file adds
> only what's specific to the Next.js web app — rules and pointers, not code.
> Component/hook how-tos live in the `web-component` / `feature-slice` skills.

---

## Project mode — SET THIS FIRST

```
MODE: web+native        # ← `web-only` or `web+native` for THIS project
```

- **`web-only`** — no native target. Use Next.js fully (Server Components, server
  actions, server fetching); `"use client"` only where interactivity needs it. The
  migration boundary does **not** apply.
- **`web+native`** — native twin planned. The migration boundary is **enforced**:
  `packages/ui-web` stays client-side and server-free (root §5).

Rules below assume `web+native`; relaxations for `web-only` are marked inline.

---

## Layer roles

- `app/**` — the **Next shell**: routing, layouts, `generateMetadata`, `providers.tsx`.
  Server Components live here. **Never migrated.** Never author UI here — compose from
  `@app/ui-web`.
- `@app/ui-web` — components: `"use client"`, server-free, migratable. Also home of
  the `cn()` helper (`packages/ui-web/src/lib/utils.ts`).
- `@app/core` — hooks, endpoints registry, types, tokens, utils (shared, unchanged).
- `@8848digital/catalyst` — **external chassis**: the fetch-based `api` client,
  `QueryProvider` + the singleton `queryClient`, auth (`createAuthStore`), and the
  Frappe sync transport (`httpSyncTransport`).
- `@8848digital/offline-kit` — **external engine**: `OfflineDb`, sync, outbox.

## Libraries

| Concern       | Tool                                                                        |
| ------------- | --------------------------------------------------------------------------- |
| Icons         | `lucide-react`                                                              |
| Class merging | `cn()` from `@app/ui-web` (`./lib/utils`) — wraps `clsx` + `tailwind-merge` |

---

## Rendering & data — SSR / SSG / ISR (read before reaching for server rendering)

**Next's rendering features are NOT blocked** — SSR, SSG, ISR, RSC, streaming, server
actions, `generateMetadata`, and route handlers all work. What's constrained is _where
catalyst can be used_: **catalyst is a CLIENT-side data layer, not a server one.** Its
`api` config (`baseUrl`, token, app, navigate) lives in **module-level globals** set at
**client** boot; `queryClient` is a **shared singleton**; the token comes from
`localStorage`; its data hooks are React Query hooks; and `offline-kit` is browser-only.

| Feature                                        | Status | Notes                                                               |
| ---------------------------------------------- | ------ | ------------------------------------------------------------------- |
| SSG / `generateStaticParams`                   | ✅     | Static + public content, no catalyst needed                         |
| ISR / `revalidate`                             | ✅     | Next-level; same data caveats as SSR                                |
| RSC / streaming / server actions / metadata    | ✅     | Shell renders freely                                                |
| Server `fetch()` **direct to Frappe**          | ✅     | For public/SEO data — bypass catalyst                               |
| Catalyst hooks in a Server Component           | ❌     | `useApiQuery`/`useLocalQuery` are React Query → `"use client"` only |
| React Query SSR prefetch / `HydrationBoundary` | ❌     | `queryClient` is a shared singleton → cache leaks across requests   |
| Catalyst `api` called from RSC/SSR             | ❌     | Module-global config is unset on the server + not request-isolated  |
| **Authenticated** SSR/ISR through catalyst     | ❌     | Token is client `localStorage`; no per-request server client exists |

**The working split (both modes):** server-render the shell + public/SEO/marketing
content with plain Next `fetch()` straight to Frappe; hydrate live/authenticated data on
the **client** through catalyst's `api` + `@app/core` hooks. _(web-only:) authenticated
SSR is DIY — you'd add a token cookie **and** a request-scoped server client; catalyst
ships neither today._

## React Query

- Hooks come from `@app/core` **unchanged** — never redefine them in `apps/web`.
- Provider = **`QueryProvider` from `@8848digital/catalyst`**, backed by catalyst's
  **module-level singleton `queryClient`** (`lib/queryClient.ts`). **Never create your
  own `QueryClient`** — write use-cases reach the same singleton via the exported
  `queryClient` for invalidation.
- Catalyst ships **no `"use client"` directive**, and `QueryProvider` uses React
  context → you **must** wrap it in your own client boundary. Put it in
  `app/providers.tsx` (`"use client"`) and mount `<Providers>` in `app/layout.tsx`.
  `providers.tsx` also runs the one-time boot wiring (`setApiApp`, `setBaseUrl`,
  `setGetToken`, `setLogout`, `outboundSync.start()`, …) inside `useEffect`.
- **Data is client-side** (see the rendering table above): do NOT SSR-prefetch React
  Query data. Fetch through the `@app/core` hooks on the client.

## Navigation

File-based routing — one folder per route, `page.tsx` per route. No central config.
`next/navigation` hooks are client-only.

| Need             | Use                         |
| ---------------- | --------------------------- |
| Declarative link | `<Link href>` (`next/link`) |
| Programmatic nav | `useRouter().push()`        |
| Current path     | `usePathname()`             |
| Route params     | `useParams()`               |

Auth guards live in **layouts** (`app/(protected)/layout.tsx`) or middleware — never
in page/component bodies. Read the token via `useAuthStore` (in `apps/web/src/stores`,
built on catalyst's `createAuthStore`).

## State discipline

Every data-driven view handles all four states — **no exceptions**:
**loading → error → empty → data.** The route's `loading.tsx` covers the
server/Suspense fallback; the client view owns the data states.

## Styling

- **Tailwind is a consumer, never a source** — `tailwind.config.ts` imports token
  values from `@app/core/tokens`; it never defines them.
- Merge classes with `cn()` (from `@app/ui-web`).
- Prefer **semantic** token classes (`bg-primary`, `text-text-primary`); scale
  classes (`bg-primary-500`) are also valid. No arbitrary values, no inline styles.

## Fonts & global styles

- `app/globals.css` (Tailwind directives + resets) is imported **once** in
  `app/layout.tsx`.
- Font families (via `next/font` or a `fonts.css`) **must match**
  `typography.family` in `@app/core/tokens`.

## Environment variables

- Client vars: prefix `NEXT_PUBLIC_`, read via `process.env.NEXT_PUBLIC_*`
  (e.g. `NEXT_PUBLIC_API_BASE_URL`).
- Server-only secrets: unprefixed, used only in `app/**` — never imported into
  `ui-web` or `core`. Never commit `.env`; use `.env.example` + GitHub Secrets.

## Monorepo resolution

`next.config.js` must `transpilePackages: ['@app/core', '@app/ui-web']` (they export
`./src/index.ts` directly). If an `@app/*` import fails to resolve at build time,
check this first.

---

## What lives where

| `apps/web` (Next shell)                | Packages                                                                        |
| -------------------------------------- | ------------------------------------------------------------------------------- |
| Route pages, layouts, metadata         | Hooks · endpoints registry · types · tokens · utils → `@app/core`               |
| `providers.tsx` (client wrapper)       | Components · `cn()` → `@app/ui-web`                                             |
| `stores/` (wires catalyst factories)   | `api` client · `QueryProvider` · `queryClient` · auth → `@8848digital/catalyst` |
| `globals.css`, fonts                   | Offline engine → `@8848digital/offline-kit`                                     |
| `tailwind.config.ts`, `next.config.js` |                                                                                 |

Writing a hook, API call, type, or token? Stop — it belongs in a package. The web
app is a thin shell over `@app/*` + the `@8848digital/*` chassis/engine.

## Never do (web)

- ❌ Author UI directly in `app/**` — compose from `@app/ui-web`
- ❌ Define token values in `tailwind.config.ts`
- ❌ Instantiate your own `QueryClient` — use catalyst's `QueryProvider` / `queryClient`
- ❌ SSR-prefetch React Query data / use `HydrationBoundary` — catalyst's client is a
  shared singleton; fetch on the client
- ❌ Call catalyst's `api` or hooks from a Server Component — client-only (see rendering table)
- ❌ Write hooks or API calls inside page/component files
- ❌ Use relative paths to import from `/packages/*` — always `@app/*`
- ❌ _(web+native)_ Name a shared component's handler prop `onClick` — call it `onPress`
  so the component API maps 1:1 to its native twin (this is the **prop name**; the DOM
  `<button>` inside still fires `onClick`)
- ❌ Use inline styles (`style={{}}`) or Tailwind arbitrary values (`bg-[#...]`)
- ❌ Hardcode colors, spacing, or font sizes
- ❌ Skip loading / error / empty states in a data-driven view

---

_Web-specific. Cross-cutting rules: see root `CLAUDE.md`._
