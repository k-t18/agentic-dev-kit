# App Router Architecture

## File-Based Routing

```
app/
├── layout.tsx              # Root layout (required)
├── page.tsx               # Home page (/)
├── loading.tsx            # Loading UI
├── error.tsx              # Error boundary
├── not-found.tsx          # 404 page
├── template.tsx           # Re-mounted layout
│
├── (marketing)/           # Route group (no URL segment)
│   ├── layout.tsx
│   ├── about/
│   │   └── page.tsx      # /about
│   └── contact/
│       └── page.tsx      # /contact
│
├── dashboard/
│   ├── layout.tsx        # Shared dashboard layout
│   ├── page.tsx          # /dashboard
│   ├── settings/
│   │   └── page.tsx      # /dashboard/settings
│   └── @analytics/       # Parallel route (slot)
│       └── page.tsx
│
├── blog/
│   ├── [slug]/
│   │   └── page.tsx      # /blog/my-post (dynamic)
│   └── [...slug]/
│       └── page.tsx      # /blog/a/b/c (catch-all)
│
└── api/
    └── users/
        └── route.ts      # API route handler
```

## Root Layout (Required)

```tsx
// app/layout.tsx
import type { Metadata } from 'next'
import { fontClassName } from '@repo/core/tokens' // web font wired by design-system-setup
import './globals.css'

export const metadata: Metadata = {
  title: {
    default: 'My App',
    template: '%s | My App'
  },
  description: 'App shell',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body className={fontClassName}>
        {children}
      </body>
    </html>
  )
}
```

> **Fonts are owned by `design-system-setup`** (tokens/Figma), not chosen ad-hoc here. Don't
> pull an arbitrary `next/font/google` face in the layout — consume the configured font from
> the token pipeline so web and native stay in sync.

## Nested Layouts

```tsx
// app/dashboard/layout.tsx
import { redirect } from 'next/navigation'
import { getServerSession } from '@repo/core/auth' // Frappe session, not NextAuth
import { Sidebar } from '@repo/ui-web'

export default async function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  const session = await getServerSession()

  if (!session) {
    redirect('/login')
  }

  return (
    <div className="flex">
      <Sidebar />
      <main className="flex-1">{children}</main>
    </div>
  )
}
```

> Auth is a Frappe session read through `@repo/core`, not NextAuth. Import UI (`Sidebar`)
> from `@repo/ui-web` — never a local `@/components` path.

## Templates (Re-mount on Navigation)

```tsx
// app/template.tsx
'use client'

import { useEffect } from 'react'

export default function Template({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    // Runs on every navigation
    console.log('Template mounted')
  }, [])

  return <div>{children}</div>
}
```

## Loading States

```tsx
// app/dashboard/loading.tsx
import { Spinner } from '@repo/ui-web'

export default function Loading() {
  return (
    <div className="flex h-screen items-center justify-center">
      <Spinner />
    </div>
  )
}
```

> Render the `ui-web` `Spinner` — never hand-roll `animate-spin … border-b-2` with hardcoded
> sizes. Any layout classes here use tokens only.

## Error Boundaries

```tsx
// app/error.tsx
'use client'

import { useEffect } from 'react'
import * as Sentry from '@sentry/nextjs'
import { ErrorState } from '@repo/ui-web'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    Sentry.captureException(error) // report, don't console.log
  }, [error])

  // Friendly ui-web fallback — never surface the raw error to users.
  return <ErrorState onRetry={reset} />
}
```

## Route Groups

```tsx
// (marketing) and (shop) share the same URL level
app/
├── (marketing)/
│   ├── layout.tsx      # Marketing layout
│   └── about/
│       └── page.tsx    # /about
└── (shop)/
    ├── layout.tsx      # Shop layout
    └── products/
        └── page.tsx    # /products
```

## Parallel Routes

```tsx
// app/dashboard/layout.tsx
export default function Layout({
  children,
  analytics,
  team,
}: {
  children: React.ReactNode
  analytics: React.ReactNode
  team: React.ReactNode
}) {
  return (
    <>
      {children}
      {analytics}
      {team}
    </>
  )
}

// app/dashboard/@analytics/page.tsx
export default function Analytics() {
  return <div>Analytics Dashboard</div>
}
```

## Intercepting Routes

```tsx
// Show modal when navigating from same app
// but show full page on direct navigation

// app/photos/[id]/page.tsx (full page)
export default function PhotoPage({ params }: { params: { id: string } }) {
  return <div>Photo {params.id} - Full Page</div>
}

// app/@modal/(.)photos/[id]/page.tsx (modal)
export default function PhotoModal({ params }: { params: { id: string } }) {
  return <div>Photo {params.id} - Modal</div>
}
```

## Dynamic Routes

```tsx
// app/blog/[slug]/page.tsx
import { apiClient, buildEndpoint } from '@repo/core'

// Next 15: params is async — await it.
export default async function BlogPost({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params
  return <h1>Post: {slug}</h1>
}

// Generate static params at build time — via the Frappe apiClient, not raw fetch
export async function generateStaticParams() {
  const { message } = await apiClient.get(buildEndpoint('blog_post.list'))
  return message.data.map((post: { slug: string }) => ({ slug: post.slug }))
}

// Opt out of static generation
export const dynamic = 'force-dynamic'

// Revalidate every 60 seconds
export const revalidate = 60
```

## Catch-All Routes

```tsx
// app/docs/[...slug]/page.tsx
// Matches: /docs/a, /docs/a/b, /docs/a/b/c
export default function Docs({ params }: { params: { slug: string[] } }) {
  return <div>Docs: {params.slug.join('/')}</div>
}

// Optional catch-all: [[...slug]]
// Also matches: /docs
```

## Route Handlers (API Routes) — rare, and never a data/CRUD layer

**Do not build CRUD route handlers.** Data comes from the Frappe backend through the
`@repo/core` `apiClient` (server components) or React Query hooks (`feature-slice`) — the web
app has no local DB to expose. The only legitimate route handlers are thin infrastructure
endpoints: an **on-demand revalidation webhook**, or a narrow proxy when the browser genuinely
can't reach Frappe directly.

```tsx
// app/api/revalidate/route.ts — on-demand revalidation webhook (the common case)
import { revalidateTag } from 'next/cache'
import { NextRequest } from 'next/server'

export async function POST(request: NextRequest) {
  const secret = request.nextUrl.searchParams.get('secret')
  if (secret !== process.env.REVALIDATE_SECRET) {
    return Response.json({ message: 'Invalid secret' }, { status: 401 })
  }
  const tag = request.nextUrl.searchParams.get('tag')
  if (tag) revalidateTag(tag)
  return Response.json({ revalidated: Boolean(tag) })
}
```

## Metadata API

```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from 'next'

export async function generateMetadata(
  { params }: { params: { slug: string } }
): Promise<Metadata> {
  const post = await fetchPost(params.slug)

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [{ url: post.coverImage }],
    },
  }
}
```

## Quick Reference

| File | Purpose | Use Case |
|------|---------|----------|
| `layout.tsx` | Persistent UI across routes | Shared navigation, auth wrapper |
| `page.tsx` | Route UI | Actual page content |
| `loading.tsx` | Loading fallback | Automatic Suspense boundary |
| `error.tsx` | Error boundary | Handle errors gracefully |
| `template.tsx` | Re-mounted layout | Analytics, animations |
| `not-found.tsx` | 404 page | Custom not found UI |
| `route.ts` | API handler | Backend API endpoints |
