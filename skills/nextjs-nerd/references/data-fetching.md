# Data Fetching & Caching

> **In this monorepo, server-side data is the Frappe `apiClient` from `@repo/core`** — not raw
> `fetch` to arbitrary URLs and never a local DB. The `apiClient` is fetch-based and forwards
> Next's `cache` / `next: { revalidate, tags }` options, so every caching strategy below
> applies — pass the options through the `apiClient` call. **Client-side** data is React Query
> hooks (`feature-slice`), never `useState`+`useEffect` or ad-hoc SWR. In **`web+native`**, a
> component with a native twin always uses the React Query hook so both platforms share one path.

## Cache / revalidate options (forwarded through apiClient)

```tsx
// app/products/page.tsx
import { apiClient, buildEndpoint } from '@repo/core'

async function getProducts() {
  // apiClient forwards the same cache/next options Next extends fetch with
  const { message } = await apiClient.get(buildEndpoint('product.list'), {
    next: { revalidate: 60 }, // ISR: revalidate every 60s
  })
  return message.data
}

export default async function Page() {
  const products = await getProducts()
  return <ProductList products={products} />
}
```

## Cache Options

```tsx
import { apiClient, buildEndpoint } from '@repo/core'
const endpoint = buildEndpoint('data.get')

// 1. Force cache (Static Site Generation)
apiClient.get(endpoint, { cache: 'force-cache' }) // Default behavior

// 2. No cache (Server-Side Rendering)
apiClient.get(endpoint, { cache: 'no-store' }) // Always fetch fresh data

// 3. Revalidate (Incremental Static Regeneration)
apiClient.get(endpoint, { next: { revalidate: 3600 } }) // Revalidate every hour

// 4. Revalidate with tags
apiClient.get(endpoint, { next: { tags: ['posts'] } })
```

## Revalidation Methods

### Time-based Revalidation (ISR)

```tsx
import { apiClient, buildEndpoint } from '@repo/core'

// Revalidate every 60 seconds
async function getPosts() {
  const { message } = await apiClient.get(buildEndpoint('post.list'), {
    next: { revalidate: 60 },
  })
  return message.data
}

// Route segment config
export const revalidate = 60 // seconds

export default async function Page() {
  const posts = await getPosts()
  return <PostList posts={posts} />
}
```

### On-Demand Revalidation

```tsx
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache'
import { NextRequest } from 'next/server'

export async function POST(request: NextRequest) {
  const path = request.nextUrl.searchParams.get('path')

  if (path) {
    revalidatePath(path)
    return Response.json({ revalidated: true, now: Date.now() })
  }

  return Response.json({ revalidated: false })
}

// Usage in Server Action (web-only, shell only)
'use server'

import { revalidatePath } from 'next/cache'
import { apiClient, buildEndpoint } from '@repo/core'

export async function createPost(formData: FormData) {
  await apiClient.post(buildEndpoint('post.create'), { title: formData.get('title') })

  // Revalidate specific path
  revalidatePath('/posts')

  // Revalidate entire layout
  revalidatePath('/posts', 'layout')
}
```

### Tag-based Revalidation

```tsx
import { apiClient, buildEndpoint } from '@repo/core'

// Tag apiClient reads so they can be revalidated together
async function getPosts() {
  const { message } = await apiClient.get(buildEndpoint('post.list'), { next: { tags: ['posts'] } })
  return message.data
}

async function getAuthors() {
  const { message } = await apiClient.get(buildEndpoint('author.list'), { next: { tags: ['authors'] } })
  return message.data
}

// Revalidate by tag (e.g. from a server action or the revalidate webhook)
import { revalidateTag } from 'next/cache'

export async function refreshPosts() {
  revalidateTag('posts') // invalidates all reads tagged 'posts'
}
```

## Route Segment Config

```tsx
// app/posts/page.tsx

// Force dynamic rendering
export const dynamic = 'force-dynamic' // 'auto' | 'force-dynamic' | 'error' | 'force-static'

// Revalidation interval
export const revalidate = 3600 // false | 0 | number (seconds)

// Fetch cache
export const fetchCache = 'auto' // 'auto' | 'default-cache' | 'only-cache' | 'force-cache' | 'force-no-store' | 'default-no-store' | 'only-no-store'

// Runtime
export const runtime = 'nodejs' // 'nodejs' | 'edge'

// Preferred region
export const preferredRegion = 'auto' // 'auto' | 'home' | 'edge' | string | string[]

export default async function Page() {
  return <div>Posts</div>
}
```

## Parallel Data Fetching

```tsx
import { apiClient, buildEndpoint } from '@repo/core'

export default async function Page() {
  // Fetch in parallel with Promise.all — no client-side waterfall
  const [user, posts, comments] = await Promise.all([
    apiClient.get(buildEndpoint('user.current')),
    apiClient.get(buildEndpoint('post.list')),
    apiClient.get(buildEndpoint('comment.list')),
  ])

  return (
    <div>
      <UserInfo user={user.message.data} />
      <Posts posts={posts.message.data} />
      <Comments comments={comments.message.data} />
    </div>
  )
}
```

## Sequential Data Fetching

```tsx
import { apiClient, buildEndpoint } from '@repo/core'

// When one fetch depends on another (Next 15: params is async)
export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params
  // First fetch
  const user = (await apiClient.get(buildEndpoint('user.get', { id }))).message.data
  // Second fetch depends on first
  const posts = (await apiClient.get(buildEndpoint('user.posts', { id: user.id }))).message.data

  return (
    <div>
      <h1>{user.name}</h1>
      <Posts posts={posts} />
    </div>
  )
}
```

## Streaming with Suspense

```tsx
// app/page.tsx
import { Suspense } from 'react'
import { apiClient, buildEndpoint } from '@repo/core'
import { PostList, Spinner } from '@repo/ui-web'

async function Posts() {
  const { message } = await apiClient.get(buildEndpoint('post.list'), { cache: 'no-store' })
  return <PostList posts={message.data} />
}

export default function Page() {
  return (
    <div>
      <h1>Posts</h1>
      <Suspense fallback={<Spinner />}>
        <Posts />
      </Suspense>
    </div>
  )
}
```

## React cache for Deduplication

```tsx
// lib/data.ts
import { cache } from 'react'
import { apiClient, buildEndpoint } from '@repo/core'

export const getUser = cache(async (id: string) => {
  const { message } = await apiClient.get(buildEndpoint('user.get', { id }))
  return message.data
})

// components/user-profile.tsx
export async function UserProfile({ userId }: { userId: string }) {
  const user = await getUser(userId) // Cached
  return <div>{user.name}</div>
}

// components/user-posts.tsx
export async function UserPosts({ userId }: { userId: string }) {
  const user = await getUser(userId) // Uses cached result
  return <div>{user.posts.length} posts</div>
}

// app/page.tsx
export default function Page() {
  return (
    <>
      <UserProfile userId="123" />
      <UserPosts userId="123" /> {/* Same fetch, deduplicated */}
    </>
  )
}
```

## Backend data — Frappe apiClient (no local DB)

There is **no Prisma/Postgres** in the web app. Server components read from the Frappe REST
backend through the `@repo/core` `apiClient` (Frappe-shaped: `buildEndpoint`, `message.data`
envelope, snake_case), then render `ui-web` components.

```tsx
// app/posts/page.tsx
import { apiClient, buildEndpoint } from '@repo/core'
import { PostList } from '@repo/ui-web'

export const revalidate = 60 // Revalidate every 60 seconds

export default async function PostsPage() {
  const { message } = await apiClient.get(buildEndpoint('post.list', { order_by: 'creation desc' }))
  return <PostList posts={message.data} />
}
```

## Error Handling

```tsx
import { apiClient, buildEndpoint } from '@repo/core'

export default async function Page() {
  // apiClient throws on a non-OK Frappe response → activates the closest error.tsx
  const { message } = await apiClient.get(buildEndpoint('data.get'))
  return <div>{message.data.title}</div>
}

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
    Sentry.captureException(error)
  }, [error])

  return <ErrorState onRetry={reset} /> // friendly ui-web fallback, never the raw error
}
```

## Loading States

```tsx
// app/posts/loading.tsx
import { Spinner } from '@repo/ui-web'
export default function Loading() {
  return <Spinner />
}

// app/posts/page.tsx
import { apiClient, buildEndpoint } from '@repo/core'
import { PostList } from '@repo/ui-web'

export default async function PostsPage() {
  const { message } = await apiClient.get(buildEndpoint('post.list'))
  return <PostList posts={message.data} />
}
```

## Client-Side Data Fetching — React Query, not SWR

Client components never fetch with `useState`+`useEffect` or ad-hoc SWR. They use the
`@repo/core` React Query hooks owned by `feature-slice` (`useApiQuery` wrapping the Frappe
`apiClient`). The hook lives in `@repo/core` so the native twin shares it verbatim.

```tsx
'use client'

import { usePosts } from '@repo/core/features/posts' // React Query hook (feature-slice)

export function Posts() {
  const { data, isError, isLoading } = usePosts()

  if (isError) return <ErrorState />
  if (isLoading) return <Spinner />

  return <PostList posts={data} />
}
```

## Preloading Data

```tsx
// lib/data.ts
import { cache } from 'react'
import { apiClient, buildEndpoint } from '@repo/core'

export const preload = (id: string) => {
  void getUser(id) // Trigger fetch without awaiting
}

export const getUser = cache(async (id: string) => {
  const { message } = await apiClient.get(buildEndpoint('user.get', { id }))
  return message.data
})

// app/users/user.tsx
import { getUser, preload } from './data'

export async function User({ id }: { id: string }) {
  const user = await getUser(id)
  return <div>{user.name}</div>
}

// app/users/page.tsx
import { User } from './user'
import { preload } from './data'

export default async function Page() {
  preload('123') // Start loading immediately
  return <User id="123" />
}
```

## Static Generation with Dynamic Routes

```tsx
// app/posts/[slug]/page.tsx
type Post = {
  slug: string
  title: string
  content: string
}

import { apiClient, buildEndpoint } from '@repo/core'

export async function generateStaticParams() {
  const { message } = await apiClient.get(buildEndpoint('post.list'))
  return message.data.map((post: Post) => ({ slug: post.slug }))
}

// Next 15: params is async
export default async function Post({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params
  const { message } = await apiClient.get(buildEndpoint('post.get', { slug }))
  const post = message.data

  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  )
}
```

## Quick Reference

| Strategy | Config | Use Case |
|----------|--------|----------|
| **SSG** | `cache: 'force-cache'` | Static content |
| **SSR** | `cache: 'no-store'` | Always fresh data |
| **ISR** | `next: { revalidate: 60 }` | Periodic updates |
| **Tag-based** | `next: { tags: ['posts'] }` | On-demand revalidation |
| **Dynamic** | `export const dynamic = 'force-dynamic'` | Per-request data |

## Best Practices

1. **Default to caching** - Use force-cache for static content
2. **Use ISR** - Revalidate periodically for semi-dynamic content
3. **Parallel fetching** - Use Promise.all for independent requests
4. **Deduplicate** - Use React cache() for repeated calls
5. **Stream with Suspense** - Show content progressively
6. **Tag your fetches** - Enable granular revalidation
7. **Handle errors** - Use error.tsx for graceful degradation
