# STACK_PROFILES.md — Architectural Decision

> **Picked at Stage 0, per app.** Locked through Stage 6. Changing your mind later means refactoring days of work — make this decision up front.
>
> The wrong call here is the most expensive mistake on any given app. A solo-user habit tracker on Convex with auth wastes setup time and cloud spend. A multi-device or multi-user app on AsyncStorage hits a wall the day you need cross-device sync.

---

## The 2 profiles

| Profile | Storage | Auth | Sample apps |
|---------|---------|------|-------------|
| **Local** | AsyncStorage (on-device) | None | Solo trackers, journaling, single-user habits |
| **Convex** | Convex DB (cloud, real-time) | Convex Auth (built-in) — anonymous device UUID OR email/password as needed | Multi-device same-user, leaderboards, real accounts, user-generated content |

> **Note on auth:** Convex Auth is the official auth library shipped with Convex (`@convex-dev/auth`). It handles anonymous device-scoped users AND real account sign-up/sign-in in one library. Clerk is intentionally not used in this kit — see "Why no Clerk" at the bottom.

---

## Decision tree

Answer in order. Stop at the first YES.

### 1. Does the app need cloud data — multi-device sync, leaderboards, shared content, OR real user accounts?
→ YES: **Convex**

### 2. Otherwise: solo user, on-device only.
→ **Local**

That's the whole tree. The Convex profile covers everything from anonymous-cloud to full account-based apps because Convex Auth handles both.

---

## What changes per profile

### Local
- **Storage:** `@react-native-async-storage/async-storage`
- **App.tsx:** no provider wrapping
- **Onboarding:** anonymous, no sign-up screens
- **Privacy disclosures (ASC):** "Data Not Collected"
- **Setup time:** zero — already in Expo
- **Cost at scale:** $0 forever

### Convex
- **Storage:** Convex DB
- **App.tsx:** wrap in `<ConvexAuthProvider>` (which itself wraps `<ConvexProvider>`)
- **Auth library:** `@convex-dev/auth` — handles anonymous device-UUID users out of the box, and real accounts (email/password, OAuth) when you wire them
- **Onboarding:**
  - Anonymous-only apps → no sign-up screens, device gets a UUID via Convex Auth automatically
  - Account-based apps → sign-up / sign-in screens before paywall
- **Account deletion in-app:** required by App Store if you have real accounts. Convex Auth + a server-side `deleteMyData` mutation covers this. (See `convex-auth-setup.md`.)
- **Privacy disclosures (ASC):**
  - Anonymous: "User Content" + "Identifiers (Device ID)" — both linked to user, not used for tracking
  - With accounts: above + "Email Address" (and "Name" if collected)
- **Setup time:** ~30 min anonymous, ~60 min with email/password accounts
- **Cost at scale:** Free up to 1M function calls / 1GB storage. Then $25/mo + usage.
- **Reference docs:**
  - `convex-auth-setup.md` (in this kit) — setup, sign-up/sign-in patterns, account deletion
  - `convex-react-client.md` (in this kit) — queries/mutations/actions
  - `~/Documents/GitHub/ref-docs/integrations/convex-expo-quickstart.md` (when present — full Convex+Expo wiring)
  - `~/Documents/GitHub/ref-docs/llms-txt/convex-full.txt` (when present — deep API reference, 42k lines, load on demand only)

---

## How the profile affects each stage

| Stage | Local | Convex (anonymous) | Convex (with accounts) |
|-------|-------|--------------------|------------------------|
| 1. Brand lock | Same | Same | Same |
| 2. Repo spin-up | Same | Same | Same |
| 3. Onboarding clone | Same | Same | Add sign-up screens to flow |
| 4. Onboarding hardening | Same | Same | Sign-up before paywall |
| 5. Main app plan | AsyncStorage data shape | Convex schema design | Convex schema + auth-scoped queries |
| 6. Main app build | AsyncStorage CRUD | Convex queries/mutations, device UUID as user key | Same, with `userId` from Convex Auth identity |
| 7. Polish | Same | Same | Add account deletion screen |
| 8. Legal URLs | Same | Same | Same |
| 9. Analytics + Sentry | Same | Same | Same + identify user in PostHog/Sentry with Convex userId |
| 11. App Store audit | Privacy = none | Privacy = device ID + content | Privacy = email + name + content + Sign in with Apple if social login |
| 15. Metadata | Privacy = "Not collected" | Privacy = device ID labels | Privacy = full user data labels |

---

## Picking the profile

**Default to Local unless you have a clear "no, this needs cloud" reason.** Most solo trackers are right on Local — zero cost, zero setup, zero auth surface.

Reach for **Convex** when any of these are true:

- User needs the same data on iPhone + iPad
- App has leaderboards, social features, or shared content
- App has real user accounts (sign-in across devices, account portability)
- Backend logic that can't run on-device (server-side cron, webhooks, third-party API calls with secrets)
- Future-self knows it'll be cloud eventually and wants to start there

---

## Migration paths (if you change your mind)

These are for reference, not encouragement — pick right the first time.

### Local → Convex (mid-build)
- Cost: 1-2 days
- Process: install Convex + Convex Auth, define schema mirroring your AsyncStorage shape, write migration script, swap CRUD calls. The anonymous-device-UUID pattern keeps existing data scopable.

### Convex anonymous → Convex with accounts (mid-build)
- Cost: 0.5-1 day
- Process: enable email/password (or OAuth) provider in `convex/auth.ts`, add sign-up/sign-in screens, add account deletion screen for App Store. Existing data keyed by anonymous UUID can be migrated to the new account UUID on first sign-in.

### Anything → Local (downgrade)
- Cost: don't. You'd be throwing away features.

---

## When to re-check

If during Stage 5 (`app-backend-builder` reads onboarding and infers what the app does) the agent says "this needs multi-user data" or "this needs accounts" but your profile says Local, **stop and reconsider the profile.** Better to re-pick now than at Stage 6.

---

## Why no Clerk

Clerk is intentionally not in this kit. Reasons:

1. **Convex Auth covers it.** Sign-up/sign-in/account-deletion all work with the official `@convex-dev/auth` library. One less service to wire, one less dashboard, one less set of env vars.
2. **Clerk + Expo has rough edges** that are not in their docs — token cache quirks, EAS env var loading, native vs JS-only sign-in component decisions. Not worth the burn.
3. **One auth source for shared databases.** If a Next.js web app and an RN app share the same Convex DB, both ends must agree on auth. Convex Auth works on both. Mixing Clerk on web and Convex Auth on mobile would mean two identity systems pointing at the same data — a recipe for bugs.

If a future standalone Next.js app (not sharing a Convex DB) wants Clerk's billing/orgs/admin tooling, that's a separate decision for that project — not this kit.
