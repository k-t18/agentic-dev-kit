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

- `app/**` — the **Next shell**: routing, layouts, `generateMetadata`, prefetch.
  Server Components live here. **Never migrated.** Never author UI here — compose
  from `@repo/ui-web`.
- `@repo/ui-web` — components: `"use client"`, server-free, migratable.
- `@repo/core` — hooks, API client, stores, types, tokens, utils (shared, unchanged).

## Libraries

| Concern       | Tool                                                          |
| ------------- | ------------------------------------------------------------- |
| Icons         | `lucide-react`                                                |
| Class merging | `clsx` + `tailwind-merge`, wrapped as `cn()` in `@/lib/utils` |

---

## React Query

- Hooks come from `@repo/core` **unchanged** — never redefine them in `apps/web`.
- The provider is a **client** component → `app/providers.tsx`.
- The `QueryClient` must be **SSR-safe** (fresh per request on server, singleton in
  browser) → `app/get-query-client.ts`. Wire `<Providers>` in `app/layout.tsx`.
- SSR prefetch + `HydrationBoundary` is allowed **only in the shell** (`app/**`) —
  never inside `ui-web`. _(web-only: use it freely for SSR/SEO.)_

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
in page/component bodies.

## State discipline

Every data-driven view handles all four states — **no exceptions**:
**loading → error → empty → data.** The route's `loading.tsx` covers the
server/Suspense fallback; the client view owns the data states.

## Styling

- **Tailwind is a consumer, never a source** — `tailwind.config.ts` imports token
  values from `@repo/core/tokens`; it never defines them.
- Merge classes with `cn()`.
- Prefer **semantic** token classes (`bg-primary`, `text-text-primary`); scale
  classes (`bg-primary-500`) are also valid. No arbitrary values, no inline styles.

## Fonts & global styles

- `styles/globals.css` (Tailwind directives + resets) is imported **once** in
  `app/layout.tsx`.
- Font families (`styles/fonts.css` or `next/font`) **must match**
  `typography.family` in `@repo/core/tokens`.

## Environment variables

- Client vars: prefix `NEXT_PUBLIC_`, read via `process.env.NEXT_PUBLIC_*`.
- Server-only secrets: unprefixed, used only in `app/**` — never imported into
  `ui-web` or `core`. Never commit `.env`; use `.env.example` + GitHub Secrets.

## Monorepo resolution

`next.config.js` must `transpilePackages: ['@repo/core', '@repo/ui-web']`. If a
`@repo/*` import fails to resolve at build time, check this first.

---

## What lives where

| `apps/web`                             | `@repo/core`           |
| -------------------------------------- | ---------------------- |
| Route pages, layouts, metadata         | Hooks                  |
| `providers.tsx`, `get-query-client.ts` | API client + endpoints |
| `lib/utils.ts` (`cn`)                  | Zustand stores         |
| `globals.css`, `fonts.css`             | Types                  |
| `tailwind.config.ts`, `next.config.js` | Tokens · utils         |

Writing a hook, API call, type, or token? Stop — it belongs in a package. The web
app is a thin shell over `@repo/*`.

## Never do (web)

- ❌ Author UI directly in `app/**` — compose from `@repo/ui-web`
- ❌ Define token values in `tailwind.config.ts`
- ❌ Write hooks or API calls inside page/component files
- ❌ Use relative paths to import from `/packages/*` — always `@repo/*`
- ❌ _(web+native)_ Name a handler `onClick` — use `onPress` so it maps 1:1 to native
- ❌ Use inline styles (`style={{}}`) or Tailwind arbitrary values (`bg-[#...]`)
- ❌ Hardcode colors, spacing, or font sizes
- ❌ Skip loading / error / empty states in a data-driven view

---

_Web-specific. Cross-cutting rules: see root `CLAUDE.md`._
