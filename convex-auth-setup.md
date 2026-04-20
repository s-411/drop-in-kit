# Convex Auth — Setup, Patterns, Account Deletion

> **When to use:** Stack Profile is `Convex` and the app needs auth (anonymous device-scoped, email/password, or OAuth). This is the official Convex auth library — `@convex-dev/auth`. Replaces the previous Clerk-based pattern entirely.
>
> **Source:** Convex official docs — https://labs.convex.dev/auth and https://docs.convex.dev/auth. Re-fetch from the labs URL if patterns drift; Convex Auth is still in active development.

---

## What this is, and why it replaces Clerk in this kit

`@convex-dev/auth` is auth that lives inside the Convex deployment. One library handles:

- **Anonymous users** — every device gets a stable user record automatically, no sign-up screen needed
- **Email/password** — standard sign-up/sign-in flow
- **OAuth** — Apple, Google, GitHub providers
- **Magic links / OTP** — passwordless

The `userId` lives in `ctx.auth.getUserIdentity()` exactly like the old Clerk pattern, so server-side code is nearly identical to what you'd write with Clerk.

**Why this over Clerk:**
- One service, one dashboard (the Convex dashboard), one set of env vars
- No Clerk+Expo token cache / EAS env var quirks
- Same library works on RN and Next.js when both share a Convex DB
- Anonymous-first is built in — no extra "device UUID in AsyncStorage" pattern needed

---

## Install

```bash
npm install @convex-dev/auth @auth/core
```

You'll need the latest Convex client too — check `package.json` for `convex` and ensure it's recent.

---

## Server-side setup

### 1. `convex/auth.config.ts`

This declares what providers are enabled. Start with anonymous; add others as needed.

```ts
import { convexAuth } from '@convex-dev/auth/server';
import Anonymous from '@auth/core/providers/anonymous';

// For email/password apps, add:
// import { Password } from '@convex-dev/auth/providers/Password';

// For Apple Sign-In (required by App Store if you also offer Google/email),
// add the Apple provider — see https://labs.convex.dev/auth/providers

export const { auth, signIn, signOut, store, isAuthenticated } = convexAuth({
  providers: [
    Anonymous,
    // Password({ /* options */ }),
  ],
});
```

### 2. `convex/auth.ts`

Export the auth handlers as HTTP actions so the client can hit them.

```ts
import { httpRouter } from 'convex/server';
import { auth } from './auth.config';

const http = httpRouter();
auth.addHttpRoutes(http);
export default http;
```

### 3. `convex/schema.ts`

Convex Auth needs its own tables. Spread `authTables` into your schema.

```ts
import { defineSchema, defineTable } from 'convex/server';
import { v } from 'convex/values';
import { authTables } from '@convex-dev/auth/server';

export default defineSchema({
  ...authTables,

  // Your app tables — every user-scoped table needs userId: v.id('users')
  tasks: defineTable({
    userId: v.id('users'),
    title: v.string(),
    completedAt: v.optional(v.number()),
  }).index('byUser', ['userId']),
});
```

The `users` table is provided by `authTables`. Your foreign keys reference it via `v.id('users')`.

---

## Client-side setup (Expo / React Native)

### 1. `app/_layout.tsx` (or wherever providers wrap)

```tsx
import { ConvexAuthProvider } from '@convex-dev/auth/react';
import { ConvexReactClient } from 'convex/react';
import * as SecureStore from 'expo-secure-store';

const convex = new ConvexReactClient(process.env.EXPO_PUBLIC_CONVEX_URL!, {
  unsavedChangesWarning: false,
});

export default function RootLayout() {
  return (
    <ConvexAuthProvider client={convex} storage={SecureStore}>
      <Stack>{/* ... */}</Stack>
    </ConvexAuthProvider>
  );
}
```

`SecureStore` is the storage adapter for React Native — tokens go in the iOS Keychain / Android Keystore. **Do not use AsyncStorage for auth tokens** — it's not encrypted at rest.

Install: `npx expo install expo-secure-store`

### 2. Anonymous sign-in (no UI needed)

For apps where every user is anonymous, trigger sign-in once on first launch:

```tsx
import { useAuthActions } from '@convex-dev/auth/react';
import { useConvexAuth } from 'convex/react';
import { useEffect } from 'react';

export function AnonymousAutoSignIn() {
  const { isAuthenticated, isLoading } = useConvexAuth();
  const { signIn } = useAuthActions();

  useEffect(() => {
    if (!isLoading && !isAuthenticated) {
      signIn('anonymous');
    }
  }, [isLoading, isAuthenticated]);

  return null;
}
```

Mount `<AnonymousAutoSignIn />` once near the root of the app, inside `ConvexAuthProvider`. The user record is created on first call and persisted in SecureStore.

### 3. Email/password sign-in screen

```tsx
import { useAuthActions } from '@convex-dev/auth/react';
import { useState } from 'react';

export function SignInScreen() {
  const { signIn } = useAuthActions();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [flow, setFlow] = useState<'signIn' | 'signUp'>('signIn');

  async function handleSubmit() {
    try {
      await signIn('password', { email, password, flow });
    } catch (err) {
      // Show error to user
    }
  }

  return (/* TextInput for email, password, button */);
}
```

`flow: 'signUp'` creates an account; `flow: 'signIn'` authenticates an existing one.

### 4. Sign-out

```tsx
import { useAuthActions } from '@convex-dev/auth/react';

const { signOut } = useAuthActions();
// call signOut() from a button
```

---

## Reading the user in Convex functions (server-side)

Same pattern as before — `ctx.auth.getUserIdentity()`:

```ts
import { query } from './_generated/server';

export const getMyTasks = query({
  args: {},
  handler: async (ctx) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) {
      throw new Error('Not authenticated');
    }

    return await ctx.db
      .query('tasks')
      .withIndex('byUser', (q) => q.eq('userId', identity.subject as any))
      .collect();
  },
});
```

`identity.subject` is the Convex `users` table `_id`. Use it as the foreign key.

For convenience, there's also a helper:

```ts
import { getAuthUserId } from '@convex-dev/auth/server';

const userId = await getAuthUserId(ctx);
if (!userId) throw new Error('Not authenticated');
```

`getAuthUserId` returns the typed `Id<'users'>` directly (no string casting).

---

## Schema rules for Convex Auth apps

- Spread `authTables` into your `defineSchema({ ... })`
- Every user-scoped table needs `userId: v.id('users')`
- Every user-scoped query filters on `userId === getAuthUserId(ctx)`
- Indexes: `.index('byUser', ['userId'])`
- **Never trust a `userId` arg from the client.** Always derive it server-side from `getAuthUserId`.

---

## Account deletion (App Store requirement for account-based apps)

Apple requires in-app account deletion if your app has accounts. Convex Auth + a server-side mutation handles this.

### Server-side: `convex/users.ts`

```ts
import { mutation } from './_generated/server';
import { getAuthUserId } from '@convex-dev/auth/server';

export const deleteMyAccount = mutation({
  args: {},
  handler: async (ctx) => {
    const userId = await getAuthUserId(ctx);
    if (!userId) throw new Error('Not authenticated');

    // 1. Delete from every user-scoped table
    const tasks = await ctx.db
      .query('tasks')
      .withIndex('byUser', (q) => q.eq('userId', userId))
      .collect();
    for (const t of tasks) await ctx.db.delete(t._id);

    // Repeat for every user-scoped table in your schema...

    // 2. Delete the auth records (sessions, accounts, then user)
    const sessions = await ctx.db
      .query('authSessions')
      .withIndex('userId', (q) => q.eq('userId', userId))
      .collect();
    for (const s of sessions) await ctx.db.delete(s._id);

    const accounts = await ctx.db
      .query('authAccounts')
      .withIndex('userIdAndProvider', (q) => q.eq('userId', userId))
      .collect();
    for (const a of accounts) await ctx.db.delete(a._id);

    await ctx.db.delete(userId);
  },
});
```

### Client-side: Settings screen

```tsx
import { useMutation } from 'convex/react';
import { useAuthActions } from '@convex-dev/auth/react';
import { api } from '@/convex/_generated/api';
import { Alert } from 'react-native';

const deleteAccount = useMutation(api.users.deleteMyAccount);
const { signOut } = useAuthActions();

async function handleDelete() {
  Alert.alert(
    'Delete Account',
    'This permanently deletes your account and all data. This cannot be undone.',
    [
      { text: 'Cancel', style: 'cancel' },
      {
        text: 'Delete',
        style: 'destructive',
        onPress: async () => {
          await deleteAccount();   // server data + auth records gone
          await signOut();          // local tokens cleared
          // navigate to landing
        },
      },
    ]
  );
}
```

For anonymous-only apps, the same pattern works — there's still a `users` row to delete and SecureStore tokens to clear.

---

## Common pitfalls

| Mistake | Fix |
|---------|-----|
| Used AsyncStorage for `storage` prop | Tokens leak to plaintext storage. Use `expo-secure-store`. |
| Forgot to spread `authTables` into schema | Convex throws — auth tables are missing. Add `...authTables`. |
| Sent `userId` as a client mutation arg | Client can spoof. Always derive via `getAuthUserId(ctx)` server-side. |
| Forgot to delete auth records in account deletion | User's `users` row orphaned. Delete sessions + accounts + user, not just app data. |
| Forgot to mount `<AnonymousAutoSignIn />` for anon apps | First queries fail with "Not authenticated". Mount it once near root. |
| Used `convex/react` `<ConvexProvider>` instead of `<ConvexAuthProvider>` | Auth never reaches Convex. Use the auth-aware provider. |
| Didn't add Apple provider when using social login | App Store rejection — Sign in with Apple is required if you offer Google/email/etc. |

---

## EAS builds — env vars

Convex Auth requires `EXPO_PUBLIC_CONVEX_URL` at build time. Add it to `eas.json`:

```jsonc
{
  "build": {
    "preview": {
      "env": {
        "EXPO_PUBLIC_CONVEX_URL": "https://your-deployment.convex.cloud"
      }
    },
    "production": {
      "env": {
        "EXPO_PUBLIC_CONVEX_URL": "https://your-deployment.convex.cloud"
      }
    }
  }
}
```

The dev URL (from `npx convex dev`) and the production URL (from `npx convex deploy`) are different. Use the production URL for production builds.

---

## Privacy disclosures (App Store)

Anonymous-only Convex Auth app:
- "Identifiers" → "User ID" — linked to user, not used for tracking
- "User Content" — linked to user, not used for tracking

Email/password Convex Auth app:
- Above + "Contact Info" → "Email Address" — linked to user, not used for tracking
- If you collect a name: + "Contact Info" → "Name"

OAuth Apple/Google:
- Above + whatever the provider returns. Apple's "Hide My Email" returns a relay address — still counts as Email Address.

---

## Reference

- Convex Auth labs: https://labs.convex.dev/auth
- Providers list: https://labs.convex.dev/auth/providers
- Convex auth concepts: https://docs.convex.dev/auth
- For deeper Convex usage: `convex-react-client.md`
- For Convex+Expo wiring not covered here: `~/Documents/GitHub/ref-docs/integrations/convex-expo-quickstart.md` (when present)
