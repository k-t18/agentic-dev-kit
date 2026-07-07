# React Server Components

> **Boundary (`web+native`):** the shell may fetch server-side only for **shell-owned,
> non-migratable** content. Any component with a native twin gets its data from a `@repo/core`
> React Query hook (`feature-slice`) so web and native share one path — do **not** fetch it
> server-side and pass props. Server-side data here always goes through the Frappe `apiClient`;
> there is no local DB.

## Server Components (Default)

```tsx
// app/page.tsx - Server Component by default
import { apiClient, buildEndpoint } from '@repo/core'
import { UserList } from '@repo/ui-web'

export default async function Page() {
  // Data fetching in a Server Component — via the Frappe apiClient
  const { message } = await apiClient.get(buildEndpoint('user.list'))

  // Render a ui-web component; never hand-build the list inline.
  return <UserList users={message.data} />
}
```

## Benefits of Server Components

- **Zero bundle size** - Server Components don't add JavaScript to client bundle
- **Direct backend access** - Query databases, read files, use secrets
- **Automatic code splitting** - Only Client Components add to bundle
- **Streaming** - Send UI progressively as data loads
- **No client-side waterfalls** - Fetch all data in parallel on server

## Client Components

```tsx
// components/counter.tsx
'use client' // Required directive

import { useState } from 'react'

export function Counter() {
  const [count, setCount] = useState(0)

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  )
}
```

## When to Use Client Components

Use `'use client'` when you need:
- **Interactivity** - onClick, onChange, event handlers
- **State** - useState, useReducer
- **Effects** - useEffect, useLayoutEffect
- **Browser APIs** - localStorage, window, document
- **Custom hooks** - Any hook using client-only features
- **Class components** - Component lifecycle methods

## Composition Pattern

```tsx
// app/page.tsx - Server Component
import { ClientWrapper } from './client-wrapper'
import { apiClient, buildEndpoint } from '@repo/core'

export default async function Page() {
  const { message } = await apiClient.get(buildEndpoint('dashboard.summary'))
  const data = message.data

  return (
    <div>
      {/* Server Component content */}
      <h1>Server Content</h1>

      {/* Pass data to Client Component */}
      <ClientWrapper initialData={data}>
        {/* Server Component as children */}
        <ServerSidebar />
      </ClientWrapper>
    </div>
  )
}

// components/client-wrapper.tsx
'use client'

export function ClientWrapper({
  children,
  initialData,
}: {
  children: React.ReactNode
  initialData: Data
}) {
  const [data, setData] = useState(initialData)

  return (
    <div>
      {/* Client Component UI */}
      <button onClick={() => refresh()}>Refresh</button>
      {/* Server Component children */}
      {children}
    </div>
  )
}
```

## Streaming with Suspense

```tsx
// app/page.tsx
import { Suspense } from 'react'
import { SlowComponent } from './slow-component'
import { FastComponent } from './fast-component'

export default function Page() {
  return (
    <div>
      {/* Renders immediately */}
      <FastComponent />

      {/* Shows fallback while loading */}
      <Suspense fallback={<div>Loading...</div>}>
        <SlowComponent />
      </Suspense>
    </div>
  )
}

// components/slow-component.tsx
async function getData() {
  await new Promise(resolve => setTimeout(resolve, 3000))
  return { data: 'Loaded!' }
}

export async function SlowComponent() {
  const data = await getData()
  return <div>{data.data}</div>
}
```

## Parallel Data Fetching

```tsx
// app/dashboard/page.tsx
import { apiClient, buildEndpoint } from '@repo/core'

export default async function Dashboard() {
  // Fetch in parallel through the apiClient — no client-side waterfall
  const [user, posts] = await Promise.all([
    apiClient.get(buildEndpoint('user.current')),
    apiClient.get(buildEndpoint('post.list')),
  ])

  return (
    <div>
      <UserProfile user={user.message.data} />
      <PostsList posts={posts.message.data} />
    </div>
  )
}
```

## Sequential Data Fetching

```tsx
// app/artist/[id]/page.tsx
import { apiClient, buildEndpoint } from '@repo/core'

export default async function ArtistPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params
  // Sequential: albums depends on artist
  const artist = (await apiClient.get(buildEndpoint('artist.get', { id }))).message.data
  const albums = (await apiClient.get(buildEndpoint('artist.albums', { id: artist.id }))).message.data

  return (
    <div>
      <h1>{artist.name}</h1>
      <Albums albums={albums} />
    </div>
  )
}
```

## Preloading Data

```tsx
// lib/data.ts
import { cache } from 'react'
import { apiClient, buildEndpoint } from '@repo/core'

// React cache() dedupes the apiClient call within one server render
export const getUser = cache(async (id: string) => {
  const { message } = await apiClient.get(buildEndpoint('user.get', { id }))
  return message.data
})

// components/user-profile.tsx
export async function UserProfile({ userId }: { userId: string }) {
  const user = await getUser(userId)
  return <div>{user.name}</div>
}

// app/page.tsx
import { getUser } from './data'
import { UserProfile } from './user-profile'

export default async function Page() {
  // Preload
  getUser('123')

  return (
    <div>
      {/* This will use cached result */}
      <UserProfile userId="123" />
    </div>
  )
}
```

## Server Component Patterns

### Pattern: Layout with Data Fetching

```tsx
// app/dashboard/layout.tsx
import { getServerSession } from '@repo/core/auth'
import { apiClient, buildEndpoint } from '@repo/core'
import { Sidebar } from '@repo/ui-web'

export default async function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  const session = await getServerSession()
  const { message } = await apiClient.get(buildEndpoint('user.get', { id: session.userId }))

  return (
    <div>
      <Sidebar user={message.data} />
      <main>{children}</main>
    </div>
  )
}
```

### Pattern: Conditional Client Components

```tsx
// app/page.tsx
import { ClientComponent } from './client-component'

export default async function Page() {
  const data = await fetchData()

  // Only render Client Component when needed
  if (data.requiresInteractivity) {
    return <ClientComponent data={data} />
  }

  return <div>{data.content}</div>
}
```

### Pattern: Server Component with Client Island

```tsx
// app/blog/[slug]/page.tsx
import { LikeButton } from './like-button'

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug)

  return (
    <article>
      {/* Server-rendered content */}
      <h1>{post.title}</h1>
      {/* dangerouslySetInnerHTML: only with server-sanitized HTML (e.g. sanitize-html /
          DOMPurify at ingest). Never inject unsanitized API/user content — XSS risk. */}
      <div dangerouslySetInnerHTML={{ __html: post.contentSanitized }} />

      {/* Client island for interactivity */}
      <LikeButton postId={post.id} initialLikes={post.likes} />
    </article>
  )
}
```

## Context in Server/Client Components

```tsx
// app/providers.tsx
'use client'

import { ThemeProvider } from 'next-themes'

export function Providers({ children }: { children: React.ReactNode }) {
  return <ThemeProvider>{children}</ThemeProvider>
}

// app/layout.tsx
import { Providers } from './providers'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html>
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  )
}
```

## Third-Party Components

```tsx
// components/carousel-wrapper.tsx
'use client'

import { Carousel } from 'third-party-carousel'

export function CarouselWrapper({ items }: { items: Item[] }) {
  return <Carousel items={items} />
}

// app/page.tsx
import { CarouselWrapper } from './carousel-wrapper'

export default async function Page() {
  const items = await fetchItems()
  return <CarouselWrapper items={items} />
}
```

## Edge Runtime

```tsx
// app/api/route.ts
export const runtime = 'edge'

export async function GET() {
  return new Response('Hello from Edge!')
}

// app/page.tsx
export const runtime = 'edge'

export default async function Page() {
  return <div>Edge-rendered page</div>
}
```

## Quick Reference

| Capability | Server Component | Client Component |
|------------|------------------|------------------|
| Data fetching | ✅ Yes (apiClient) | ⚠️ React Query hooks (`feature-slice`) |
| Backend access | ✅ Yes (DB, files) | ❌ No |
| Event handlers | ❌ No | ✅ Yes |
| State/Effects | ❌ No | ✅ Yes |
| Browser APIs | ❌ No | ✅ Yes |
| Bundle size | 0 KB | Adds to bundle |
| Streaming | ✅ Yes | ❌ No |

## Best Practices

1. **Default to Server Components** - Only use 'use client' when needed
2. **Render ui-web** - Pages/layouts render `@repo/ui-web` components; never hand-build UI inline
3. **Move Client Components down** - Push them to leaves of component tree
4. **Delegate data** - `web-only`: fetch via `apiClient` and pass down. `web+native`: migratable
   components fetch via `@repo/core` React Query hooks (`feature-slice`), not server props
5. **Cache expensive operations** - Use React `cache()` to dedupe apiClient calls per render
6. **No local DB** - There is no Prisma/Postgres; all data is the Frappe `apiClient`
