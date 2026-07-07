# Server Actions

> **App shell + `web-only` only.** Server actions live in `apps/web/app/**` and are used only
> in `web-only` projects. They are **banned from `packages/ui-web`/`ui-native`** (§5 boundary):
> a component with a native twin must run without a server. In **`web+native`**, form
> submission is a React Query mutation (`useApiMutation`, `feature-slice`), with offline writes
> via the outbox (`@repo/offline-kit`).
>
> **No local DB.** Mutations call the Frappe `apiClient` from `@repo/core` — never Prisma/`db`.
> Validate input with Zod before the call.

## Basic Server Action

```tsx
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'
import { apiClient, buildEndpoint } from '@repo/core'

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string
  const content = formData.get('content') as string

  await apiClient.post(buildEndpoint('post.create'), { title, content })

  revalidatePath('/posts')
}
```

## Form with Server Action

```tsx
// app/posts/new/page.tsx
import { createPost } from '../actions'

export default function NewPost() {
  return (
    <form action={createPost}>
      <input name="title" required />
      <textarea name="content" required />
      <button type="submit">Create Post</button>
    </form>
  )
}
```

## Server Action with Validation

```tsx
// app/actions.ts
'use server'

import { z } from 'zod'
import { revalidatePath } from 'next/cache'
import { apiClient, buildEndpoint } from '@repo/core'

const CreatePostSchema = z.object({
  title: z.string().min(3).max(100),
  content: z.string().min(10),
})

export async function createPost(formData: FormData) {
  const validatedFields = CreatePostSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content'),
  })

  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
    }
  }

  const { title, content } = validatedFields.data

  await apiClient.post(buildEndpoint('post.create'), { title, content })

  revalidatePath('/posts')
  return { success: true }
}
```

## Client Component with Server Action

```tsx
// app/posts/create-post-form.tsx  (app shell, web-only — NOT in ui-web)
'use client'

import { useActionState } from 'react' // React 19: replaces the deprecated useFormState
import { useFormStatus } from 'react-dom'
import { createPost } from './actions'

const initialState = {
  errors: {},
}

function SubmitButton() {
  const { pending } = useFormStatus()

  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Creating...' : 'Create Post'}
    </button>
  )
}

export function CreatePostForm() {
  const [state, formAction] = useActionState(createPost, initialState)

  return (
    <form action={formAction}>
      <div>
        <input name="title" />
        {state.errors?.title && <p>{state.errors.title[0]}</p>}
      </div>

      <div>
        <textarea name="content" />
        {state.errors?.content && <p>{state.errors.content[0]}</p>}
      </div>

      <SubmitButton />
    </form>
  )
}
```

## Server Action with Redirect

```tsx
// app/actions.ts
'use server'

import { redirect } from 'next/navigation'
import { revalidatePath } from 'next/cache'
import { apiClient, buildEndpoint } from '@repo/core'

export async function createPost(formData: FormData) {
  const { message } = await apiClient.post(buildEndpoint('post.create'), {
    title: formData.get('title') as string,
    content: formData.get('content') as string,
  })

  revalidatePath('/posts')
  redirect(`/posts/${message.data.name}`) // Frappe docname
}
```

## Optimistic Updates

> **In this monorepo, optimistic UI is a React Query mutation** (`onMutate` + rollback) owned
> by `feature-slice`, with offline writes reconciled through the `@repo/offline-kit` outbox —
> that path is shared with native and survives offline. The `useOptimistic` primitive below is
> a `web-only` app-shell convenience for ephemeral optimism that never needs to persist or
> sync; the persist call is still the Frappe `apiClient`, never a local `db`.

```tsx
// app/todos/todo-list.tsx  (app shell, web-only)
'use client'

import { useOptimistic } from 'react'
import { createTodo } from './actions'

export function TodoList({ todos }: { todos: Todo[] }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo: Todo) => [...state, newTodo]
  )

  async function handleSubmit(formData: FormData) {
    const title = formData.get('title') as string
    const newTodo = { id: crypto.randomUUID(), title, completed: false }

    // Optimistically update UI
    addOptimisticTodo(newTodo)

    // Send to server
    await createTodo(formData)
  }

  return (
    <div>
      <ul>
        {optimisticTodos.map(todo => (
          <li key={todo.id}>{todo.title}</li>
        ))}
      </ul>

      <form action={handleSubmit}>
        <input name="title" />
        <button type="submit">Add</button>
      </form>
    </div>
  )
}
```

## Server Action with Authentication

```tsx
// app/actions.ts
'use server'

import { redirect } from 'next/navigation'
import { revalidatePath } from 'next/cache'
import { getServerSession } from '@repo/core/auth'
import { apiClient, buildEndpoint } from '@repo/core'

export async function createPost(formData: FormData) {
  const session = await getServerSession() // Frappe session

  if (!session) {
    redirect('/login')
  }

  await apiClient.post(buildEndpoint('post.create'), {
    title: formData.get('title') as string,
    content: formData.get('content') as string,
    author: session.user.id,
  })

  revalidatePath('/posts')
}
```

## Inline Server Action

```tsx
// app/posts/page.tsx
import { apiClient, buildEndpoint } from '@repo/core'
import { revalidatePath } from 'next/cache'

export default async function Posts() {
  const { message } = await apiClient.get(buildEndpoint('post.list'))
  const posts = message.data

  async function deletePost(formData: FormData) {
    'use server'

    const id = formData.get('id') as string
    await apiClient.delete(buildEndpoint('post.delete', { id }))
    revalidatePath('/posts')
  }

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          {post.title}
          <form action={deletePost}>
            <input type="hidden" name="id" value={post.id} />
            <button type="submit">Delete</button>
          </form>
        </li>
      ))}
    </ul>
  )
}
```

## Programmatic Server Action Call

```tsx
// app/posts/delete-button.tsx  (app shell, web-only)
'use client'

import { deletePost } from './actions'

export function DeleteButton({ postId }: { postId: string }) {
  async function handleDelete() {
    if (confirm('Are you sure?')) {
      await deletePost(postId)
    }
  }

  return (
    <button onClick={handleDelete}>
      Delete
    </button>
  )
}

// app/posts/actions.ts
'use server'

import { revalidatePath } from 'next/cache'
import { apiClient, buildEndpoint } from '@repo/core'

export async function deletePost(postId: string) {
  await apiClient.delete(buildEndpoint('post.delete', { id: postId }))
  revalidatePath('/posts')
}
```

## Revalidation Strategies

```tsx
// app/actions.ts
'use server'

import { revalidatePath, revalidateTag } from 'next/cache'
import { apiClient, buildEndpoint } from '@repo/core'

export async function updatePost(id: string, data: UpdatePostData) {
  await apiClient.put(buildEndpoint('post.update', { id }), data)

  // Revalidate specific path
  revalidatePath('/posts')
  revalidatePath(`/posts/${id}`)

  // Revalidate all paths in a layout
  revalidatePath('/posts', 'layout')

  // Revalidate by cache tag
  revalidateTag('posts')
}
```

## Server Action with File Upload

```tsx
// app/profile/actions.ts
'use server'

import { apiClient, buildEndpoint } from '@repo/core'

export async function uploadAvatar(formData: FormData) {
  const file = formData.get('avatar') as File
  if (!file) {
    return { error: 'No file uploaded' }
  }

  // Forward the file to the Frappe backend — never write to the app's public/ dir
  const upload = new FormData()
  upload.append('file', file)
  const { message } = await apiClient.post(buildEndpoint('upload_file'), upload)

  return { success: true, url: message.file_url }
}

// app/profile/upload-form.tsx
'use client'

import { uploadAvatar } from './actions'

export function UploadForm() {
  async function handleSubmit(formData: FormData) {
    const result = await uploadAvatar(formData)
    if (result.success) {
      // handle result.url
    }
  }

  return (
    <form action={handleSubmit}>
      <input type="file" name="avatar" accept="image/*" />
      <button type="submit">Upload</button>
    </form>
  )
}
```

## Error Handling

```tsx
// app/actions.ts
'use server'

import { apiClient, buildEndpoint } from '@repo/core'

export async function createPost(formData: FormData) {
  try {
    await apiClient.post(buildEndpoint('post.create'), {
      title: formData.get('title') as string,
      content: formData.get('content') as string,
    })

    revalidatePath('/posts')
    return { success: true }
  } catch (error) {
    // Return a safe message; report the raw error to your service, not the user
    return { error: 'Failed to create post' }
  }
}

// components/form.tsx
'use client'

export function CreatePostForm() {
  const [error, setError] = useState<string | null>(null)

  async function handleSubmit(formData: FormData) {
    const result = await createPost(formData)

    if (result.error) {
      setError(result.error)
    } else {
      // Success
      router.push('/posts')
    }
  }

  return (
    <form action={handleSubmit}>
      {error && <p className="text-danger-500">{error}</p>}
      {/* form fields */}
    </form>
  )
}
```

## Server Action with Cookies

```tsx
// app/actions.ts
'use server'

import { cookies } from 'next/headers'

export async function setTheme(theme: 'light' | 'dark') {
  cookies().set('theme', theme, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    maxAge: 60 * 60 * 24 * 365, // 1 year
    path: '/',
  })
}

export async function getTheme() {
  return cookies().get('theme')?.value ?? 'light'
}
```

## Rate Limiting

```tsx
// app/actions.ts
'use server'

import { getServerSession } from '@repo/core/auth'
import { ratelimit } from '@repo/core/ratelimit'

export async function createPost(formData: FormData) {
  const session = await getServerSession()
  const { success } = await ratelimit.limit(session.user.id)

  if (!success) {
    return { error: 'Rate limit exceeded' }
  }

  // Create post via apiClient...
}
```

## Quick Reference

| Capability | Usage |
|------------|-------|
| **Define** | Add 'use server' at top of file or function |
| **Form** | Pass action to `<form action={serverAction}>` |
| **Programmatic** | Call directly: `await serverAction(data)` |
| **Validation** | Use Zod/TypeBox before mutations |
| **Revalidate** | `revalidatePath()` or `revalidateTag()` |
| **Redirect** | `redirect()` after mutation |
| **Errors** | Return error objects, handle in client |
| **Files** | Access via `formData.get()` as File |

## Best Practices

1. **Always validate** - Use Zod/TypeBox for type-safe validation
2. **Revalidate** - Call revalidatePath() after mutations
3. **Handle errors** - Return error objects instead of throwing
4. **Auth checks** - Verify session before mutations
5. **Rate limiting** - Protect against abuse
6. **Type safety** - Define input/output types
7. **Optimistic updates** - Use useOptimistic for better UX
