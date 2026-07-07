---
name: nextjs-nerd
description: "Use when working in the Next.js App Router shell (apps/web/app/**) — routing, layouts, templates, metadata/SEO, streaming, loading.tsx/error.tsx boundaries, and (web-only) server components + server actions. The shell renders ui-web components and delegates data to @repo/core React Query hooks / the Frappe apiClient — it never builds UI inline or fetches for a component that has a native twin. Triggers on: Next.js, App Router, RSC, Server Components, Server Actions, generateMetadata, loading.tsx, error.tsx, route handlers, streaming SSR, app shell."
license: MIT
metadata:
  author: https://github.com/k-t18
  version: "2.0.0"
  domain: frontend
  triggers: Next.js, App Router, app shell, Server Components, RSC, Server Actions, generateMetadata, loading.tsx, error.tsx, route handlers, streaming SSR, metadata SEO
  role: specialist
  scope: implementation
  output-format: code
  related-skills: web-component, feature-slice, design-system-setup, react-renderer, typescript-teacher
---

# Next.js Developer (the app shell)

Owns the **Next.js App Router shell** — `apps/web/app/**`: routing, layouts, templates,
metadata/SEO, streaming, and `loading.tsx`/`error.tsx` boundaries. The shell **renders
`ui-web` components** and **delegates data downward**; it does not build UI inline or own
hooks/state/tokens (those have skills).

Read root `CLAUDE.md` and `apps/web/CLAUDE.md` first; this skill never overrides them. The
backend is a remote **Frappe REST API** reached through the fetch-based `apiClient` in
`@repo/core` — there is **no local ORM/DB** (no Prisma, no Postgres) in the web app.

## Project mode decides your posture (read first)

Every project declares its mode at the top of `apps/web/CLAUDE.md`.

- **`web-only`** — use Next to the fullest: Server Components by default, server actions,
  server-side data fetching. `"use client"` only where interactivity needs it. Server-side
  data and mutations go through the Frappe `apiClient`, never a local DB.
- **`web+native`** — the §5 web-layering boundary is **enforced**. The shell owns routing /
  layouts / metadata / data-loading, then renders `ui-web` components. **Any component with a
  native twin must run identically without a server**, so:
  - The shell does **not** fetch data for migratable components — they get data via
    `@repo/core` React Query hooks (`feature-slice`), shared unchanged with native.
  - **Server actions / `useActionState` / `useFormStatus` are banned** from `packages/ui-web`
    — they live only in the shell (and only make sense in `web-only`). Forms in `ui-web` use
    React Query mutations (`useApiMutation`, `feature-slice`).

## When to Use

- Building or editing anything under `apps/web/app/**` — routes, layouts, templates,
  metadata, streaming, loading/error boundaries.

**Not for:** UI components (`web-component` / `rn-component`), data hooks / API / stores /
types (`@repo/core`, see `feature-slice`), design tokens (`design-system-setup`), or React
effect/perf/error-boundary discipline (`react-renderer`).

## Core Workflow

1. **Architecture** — routes, layouts, rendering strategy; confirm the project mode.
2. **Routing** — App Router structure with layouts, templates, `loading.tsx`/`error.tsx`.
3. **Data** — `web-only`: server components fetch via `apiClient`. `web+native`: hand data to
   `ui-web` via `@repo/core` React Query hooks; don't fetch server-side for migratable UI.
4. **Compose** — render `ui-web` components; keep `"use client"` at the leaf boundary.
5. **Validate** — `pnpm --filter web build` (or `turbo build`) succeeds; `tsc --noEmit`
   clean; grep shows no hardcoded hex/px and no local-DB calls; Core Web Vitals stay green.

## Reference Guide

| Topic | Reference | Load When |
|-------|-----------|-----------|
| App Router | `references/app-router.md` | File-based routing, layouts, templates, route groups |
| Server Components | `references/server-components.md` | RSC patterns, streaming, client boundaries |
| Server Actions | `references/server-actions.md` | Form handling / mutations in the shell (web-only) |
| Data Fetching | `references/data-fetching.md` | fetch, caching, ISR, on-demand revalidation |

## Constraints

### MUST DO
- Keep components as Server Components by default; add `"use client"` only at the leaf
  boundary where interactivity is required.
- **Render `ui-web` components** for UI — never hand-build UI with raw `<ul>/<li>` +
  Tailwind in a page/layout. `loading.tsx`/`error.tsx` use `ui-web` (Spinner) + tokens.
- Import UI from `@repo/ui-web`, data/apiClient/types from `@repo/core` — never `@/…` paths.
- Fetch through the Frappe `apiClient` (server-side, web-only) or `@repo/core` React Query
  hooks (`feature-slice`); use explicit `cache` / `next.revalidate` on any server fetch.
- Use `generateMetadata` (or static `metadata`) for all SEO — never hardcode `<title>`/`<meta>`.
- Optimize every content image with `next/image` (+ `remotePatterns` in `next.config`);
  never a plain `<img>`. Set security headers in `next.config` (HSTS, X-Frame-Options,
  nosniff, Referrer-Policy).
- Add `loading.tsx` and `error.tsx` at every async route segment; `error.tsx` reports to
  Sentry and shows a friendly fallback (never a raw error).
- pnpm + Turborepo for all commands; our env var names (`NEXT_PUBLIC_API_BASE_URL` on web).

### MUST NOT DO
- Import Prisma / query a local DB / spin up Postgres — the backend is Frappe via `apiClient`.
- Convert a component to a Client Component just to fetch — fetch server-side (web-only) or
  use a `@repo/core` hook.
- (`web+native`) Fetch server-side for a component with a native twin, or use server actions /
  `useActionState` / `useFormStatus` in `ui-web`/`ui-native`.
- Hardcode a color / spacing / radius / font (tokens only), or build UI inline in the shell.
- Use `npm`, `@/…` aliases, `next/font` ad-hoc fonts (fonts come from `design-system-setup`),
  or ship without `pnpm --filter web build` passing.

## Code Examples

### Server Component rendering ui-web, data via apiClient (web-only)
```tsx
// app/products/page.tsx
import { Suspense } from 'react'
import { apiClient, buildEndpoint } from '@repo/core'
import { ProductList, Spinner } from '@repo/ui-web'
import type { Product } from '@repo/core/features/products'

async function Products() {
  // Frappe-shaped apiClient; ISR every 60s. No local DB.
  const { message } = await apiClient.get<{ message: { data: Product[] } }>(
    buildEndpoint('product.list'),
    { next: { revalidate: 60 } },
  )
  return <ProductList products={message.data} />
}

export default function Page() {
  return (
    <Suspense fallback={<Spinner />}>
      <Products />
    </Suspense>
  )
}
```

> In **`web+native`**, `ProductList` fetches its own data via a `@repo/core` React Query hook
> (`feature-slice`) so the native twin shares the exact path — the shell just renders it.

### generateMetadata for dynamic SEO
```tsx
// app/products/[id]/page.tsx
import type { Metadata } from 'next'
import { apiClient, buildEndpoint } from '@repo/core'

export async function generateMetadata(
  { params }: { params: Promise<{ id: string }> }, // Next 15: params is async
): Promise<Metadata> {
  const { id } = await params
  const { message } = await apiClient.get(buildEndpoint('product.get', { id }))
  const product = message.data
  return {
    title: product.name,
    description: product.description,
    openGraph: { title: product.name, images: [product.image_url] },
  }
}
```

### Server Action calling apiClient (web-only, shell only)
```tsx
// app/products/actions.ts
'use server'

import { revalidatePath } from 'next/cache'
import { z } from 'zod'
import { apiClient, buildEndpoint } from '@repo/core'

const CreateProduct = z.object({ name: z.string().min(1) })

export async function createProduct(formData: FormData) {
  const parsed = CreateProduct.safeParse({ name: formData.get('name') })
  if (!parsed.success) return { errors: parsed.error.flatten().fieldErrors }

  await apiClient.post(buildEndpoint('product.create'), { name: parsed.data.name })
  revalidatePath('/products')
  return { success: true }
}
```

> Never in `ui-web`/`ui-native`. In `web+native`, a create form is a React Query mutation
> (`useApiMutation`, `feature-slice`), with offline writes via the outbox (`@repo/offline-kit`).

## Output Templates

When implementing shell features, provide:
1. Route organization (`app/**` structure)
2. Layout/page components that render `ui-web` and delegate data
3. Server actions only if `web-only` and mutations are shell-owned
4. Config (`next.config.js`: images/security headers, TypeScript)
5. Brief explanation of the rendering strategy and how it respects the project mode

## Knowledge Reference

Next.js 14+/15, App Router, React Server Components, Server Actions (web-only shell),
Streaming SSR, Partial Prerendering, `next/image`, Metadata API, Route Handlers (thin Frappe
proxy / revalidation only), Edge Runtime, Turbopack. Data → Frappe `apiClient` + React Query
(`feature-slice`); UI → `@repo/ui-web`; tokens → `design-system-setup`; React discipline →
`react-renderer`.
