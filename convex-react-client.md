# Convex React Client — Reference

> **When to use:** building queries, mutations, actions during Stage 6. Pair with `docs/convex-expo-setup.md` for initial setup.
>
> **Source:** Convex official docs — trimmed to what's needed during builds.

---

## Three function types

| Type | Purpose | Hook | Reactive? |
|------|---------|------|-----------|
| **Query** | Read data | `useQuery(api.X.fn)` | Yes — auto re-renders on data change |
| **Mutation** | Write data | `useMutation(api.X.fn)` | Triggers re-render of any subscribed query |
| **Action** | Call third-party APIs, side effects | `useAction(api.X.fn)` | No, no auto-retry |

Define them in `convex/<filename>.ts`. The agent generates `convex/_generated/api.ts` automatically when `npx convex dev` is running.

---

## Reading data — useQuery

```tsx
import { useQuery } from 'convex/react';
import { api } from '@/convex/_generated/api';

export function TaskList() {
  const tasks = useQuery(api.tasks.get);
  if (tasks === undefined) return <Loading />;
  return tasks.map(t => <Text key={t._id}>{t.text}</Text>);
}
```

### With arguments

```tsx
const tasks = useQuery(api.tasks.getByUser, { userId: '...' });
```

### Conditional / skip

You can't put hooks in conditionals, so use `'skip'`:

```tsx
const tasks = useQuery(api.tasks.getByUser, userId ? { userId } : 'skip');
// returns undefined when skipped
```

### Loading state

`useQuery` returns `undefined` until data loads. Always handle this case.

---

## Writing data — useMutation

```tsx
import { useMutation } from 'convex/react';
import { api } from '@/convex/_generated/api';

export function AddTask() {
  const addTask = useMutation(api.tasks.add);

  async function onPress() {
    try {
      await addTask({ text: 'New task', deviceId });
    } catch (e) {
      console.error(e);
    }
  }

  return <Button onPress={onPress} title="Add" />;
}
```

### Mutations are reactive
After a mutation runs, any `useQuery` watching the affected data automatically re-renders. **Don't manually re-fetch.**

### Auto-retry
Convex retries mutations until confirmed written. Each call only executes once even with retries. Convex also warns the user if they try to close the tab/app with pending mutations.

### Optimistic updates (advanced)
Apply a temporary local change before the server confirms — see Convex docs for `withOptimisticUpdate`. **Skip in MVP unless UX demands it.**

---

## Calling third-party APIs — useAction

```tsx
import { useAction } from 'convex/react';

export function FetchExternal() {
  const fetchData = useAction(api.external.fetchSomething);
  return <Button onPress={() => fetchData()} title="Fetch" />;
}
```

Actions:
- Run server-side (Node.js environment in Convex's cloud)
- Can call third-party HTTP APIs
- Can read/write Convex data (via `ctx.runQuery` / `ctx.runMutation`)
- **No auto-retry, no optimistic updates** — handle errors yourself

---

## Defining functions (server-side, in `/convex/`)

### Query
```ts
// convex/tasks.ts
import { query } from './_generated/server';
import { v } from 'convex/values';

export const get = query({
  args: { deviceId: v.string() },
  handler: async (ctx, args) => {
    return await ctx.db
      .query('tasks')
      .withIndex('byUser', q => q.eq('deviceId', args.deviceId))
      .collect();
  },
});
```

### Mutation
```ts
import { mutation } from './_generated/server';

export const add = mutation({
  args: { deviceId: v.string(), text: v.string() },
  handler: async (ctx, args) => {
    return await ctx.db.insert('tasks', {
      deviceId: args.deviceId,
      text: args.text,
      isCompleted: false,
      createdAt: Date.now(),
    });
  },
});
```

### Action
```ts
import { action } from './_generated/server';

export const fetchSomething = action({
  args: {},
  handler: async (ctx) => {
    const res = await fetch('https://example.com/api');
    return await res.json();
  },
});
```

---

## Common patterns

### Pagination
```tsx
import { usePaginatedQuery } from 'convex/react';

const { results, status, loadMore } = usePaginatedQuery(
  api.tasks.list,
  { deviceId },
  { initialNumItems: 20 }
);
```

Server-side:
```ts
export const list = query({
  args: { deviceId: v.string(), paginationOpts: paginationOptsValidator },
  handler: async (ctx, args) => {
    return await ctx.db
      .query('tasks')
      .withIndex('byUser', q => q.eq('deviceId', args.deviceId))
      .order('desc')
      .paginate(args.paginationOpts);
  },
});
```

### One-off query (in event handler, not subscribed)
```tsx
import { useConvex } from 'convex/react';

const convex = useConvex();
const result = await convex.query(api.tasks.get, { deviceId });
```

Use this when you need data on a button press, not for rendering.

---

## Schema design rules of thumb

1. **Always index** the field you query by. Convex queries without indexes scan the whole table.
2. **Index name convention:** `byX` where X is the field (e.g. `byUser`, `byCreatedAt`).
3. **Compound indexes** for queries with multiple filters: `defineTable({...}).index('byUserAndDate', ['userId', 'date'])`.
4. **Don't store derived data** that can be computed from other tables — query and join instead.

---

## Errors and connection handling

- WebSocket reconnects automatically on internet drop
- Mutations queue until reconnected
- Errors thrown in handlers propagate to the client as rejected Promises

```tsx
const addTask = useMutation(api.tasks.add);
try {
  await addTask({ text: 'foo', deviceId });
} catch (err) {
  // handle error — show toast, etc.
}
```

---

## What NOT to do

- ❌ Manually re-fetch after a mutation — queries are already reactive
- ❌ Call mutations from inside a `useEffect` without guards — they'll fire on every render
- ❌ Store sensitive data in tables without auth — anyone with the deployment URL can read
- ❌ Use `useQuery` without handling the `undefined` loading state
- ❌ Skip the index — your query will get slow with >100 rows

---

## Reference

Full docs: https://docs.convex.dev/client/react

For Convex Auth integration: `convex-auth-setup.md`
