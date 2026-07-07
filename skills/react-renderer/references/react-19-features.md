# React 19 Features

> **Web is React 19; native is React 18** — these are **web-only**. Client-side features
> (`use()`, `useOptimistic`, `ref`-as-prop) are usable in `ui-web`. **`useActionState` /
> `useFormStatus` are server-action forms → app-shell only (`apps/web/app/**`), never in
> `ui-web`** (which stays client + React Query). Don't use any of these in `ui-native`.

## use() Hook

> **Not for data fetching in this monorepo.** All server data goes through `@repo/core`
> React Query hooks (`feature-slice`) — they own caching, retries, and the isError/isLoading
> states. Calling `fetchComments(postId)` in render (below) creates a new promise every
> render and has no cache; it's shown only to illustrate the API. Reserve `use()` for
> promises/context handed to you by the framework or a parent, not ad-hoc fetches.

```tsx
import { use, Suspense } from 'react';

// Read promises in render
function Comments({ commentsPromise }: { commentsPromise: Promise<Comment[]> }) {
  const comments = use(commentsPromise);
  return (
    <ul>
      {comments.map(c => <li key={c.id}>{c.text}</li>)}
    </ul>
  );
}

// Parent creates promise, child reads it
function Post({ postId }: { postId: string }) {
  const commentsPromise = fetchComments(postId);

  return (
    <article>
      <PostContent id={postId} />
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments commentsPromise={commentsPromise} />
      </Suspense>
    </article>
  );
}

// Read context conditionally
function Theme({ children }: { children: React.ReactNode }) {
  if (someCondition) {
    const theme = use(ThemeContext);
    return <div className={theme}>{children}</div>;
  }
  return children;
}
```

## useActionState / useFormStatus — app-shell only, NEVER in `ui-web`/`ui-native`

These are server-action form hooks. In `web+native` they belong **exclusively** in the
Next app shell (`apps/web/app/**`) — the layer that is never migrated. A component with a
native twin must run identically without a server, so `useActionState`, `useFormStatus`,
and inline `'use server'` actions are **banned from `packages/ui-web`** (and don't exist in
RN). In `ui-web`, form submission is a React Query mutation (`useApiMutation`,
`feature-slice`). The examples below are app-shell patterns.

## useActionState

```tsx
'use client';
import { useActionState } from 'react';

interface FormState {
  error?: string;
  success?: boolean;
}

async function submitAction(prevState: FormState, formData: FormData): Promise<FormState> {
  'use server';
  const email = formData.get('email') as string;

  try {
    await subscribe(email);
    return { success: true };
  } catch {
    return { error: 'Failed to subscribe' };
  }
}

function NewsletterForm() {
  const [state, formAction, isPending] = useActionState(submitAction, {});

  return (
    <form action={formAction}>
      <input name="email" type="email" required disabled={isPending} />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Subscribing...' : 'Subscribe'}
      </button>
      {state.error && <p className="text-danger-500">{state.error}</p>}
      {state.success && <p className="text-success-500">Subscribed!</p>}
    </form>
  );
}
```

## useFormStatus

```tsx
'use client';
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending, data, method, action } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  );
}

// Must be used inside a <form>
function ContactForm() {
  return (
    <form action={submitAction}>
      <input name="message" />
      <SubmitButton />
    </form>
  );
}
```

## useOptimistic

> **Prefer the data layer's optimistic path.** In this monorepo optimistic UI is normally
> a React Query mutation with `onMutate`/rollback, and offline writes reconcile through
> `@repo/offline-kit`'s **outbox** — both owned by `feature-slice`. That path is shared,
> migratable, and survives offline. Reach for `useOptimistic` only for ephemeral,
> view-local optimism that never needs to persist or sync. The example below uses a server
> action purely to show the primitive — in `ui-web` the persist call is a `useApiMutation`,
> not `createTodo`.

```tsx
'use client';
import { useOptimistic } from 'react';

function TodoList({ todos }: { todos: Todo[] }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo: Todo) => [...state, newTodo]
  );

  async function addTodo(formData: FormData) {
    const text = formData.get('text') as string;

    // Immediately update UI
    addOptimisticTodo({ id: 'temp', text, completed: false });

    // Then persist
    await createTodo(text);
  }

  return (
    <>
      <ul>
        {optimisticTodos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
      <form action={addTodo}>
        <input name="text" />
        <button>Add</button>
      </form>
    </>
  );
}
```

## ref as Prop (No forwardRef)

```tsx
// React 19: ref is just a prop
function Input({ ref, ...props }: { ref?: React.Ref<HTMLInputElement> }) {
  return <input ref={ref} {...props} />;
}

// No need for forwardRef anymore
function Form() {
  const inputRef = useRef<HTMLInputElement>(null);
  return <Input ref={inputRef} placeholder="Enter text" />;
}
```

## Quick Reference

| Hook | Purpose |
|------|---------|
| `use()` | Read promise/context in render |
| `useActionState()` | Form action state + pending |
| `useFormStatus()` | Form pending state (child) |
| `useOptimistic()` | Optimistic UI updates |

| Pattern | When |
|---------|------|
| `use(promise)` | Suspense data fetching |
| `use(context)` | Conditional context read |
| `useActionState` | Server Actions with state |
