# Next.js Bridge

> **When to use:** when you build a Next.js web app, especially one that shares a Convex DB with an existing React Native app in this kit.
>
> **Status:** REFERENCE — not a full quickstart. Points at the right official docs and notes the kit-specific decisions (Convex Auth on both ends, no Clerk).

---

## The core decision: Convex Auth on web too

For any Next.js app in this kit, **auth is `@convex-dev/auth`** — the same library used on the React Native side. Reasons (also in `STACK_PROFILES.md`):

- If the Next.js app shares a Convex DB with an RN app, both ends MUST agree on auth. Mixing two identity systems pointing at the same data is a recipe for bugs.
- One auth dashboard, one set of env vars, one library to debug.
- Convex Auth has a Next.js-specific package (`@convex-dev/auth/nextjs`) with middleware and server-side helpers — same patterns as the RN side, different import path.

If a future standalone Next.js app (NOT sharing a Convex DB with an RN app) wants Clerk's billing/orgs/admin tooling, that's a separate decision for that project — not this kit.

---

## What stays the same when you go from RN to web

- `STACK_PROFILES.md` decision logic — Local doesn't really apply on web (no AsyncStorage); default web profile is **Convex**
- Convex backend code — identical. Same `convex/` folder works for both web and mobile. `defineSchema`, `query`, `mutation`, `action`, `getAuthUserId` — all unchanged.
- Convex Auth providers (`Anonymous`, `Password`, OAuth) — same `convex/auth.config.ts` works for both clients
- The 16-stage process — most stages still apply, just with Vercel instead of EAS for deployment

---

## What changes

| Concept | Mobile (Expo) | Web (Next.js) |
|---------|---------------|---------------|
| Framework | Expo + React Native | Next.js (App Router) |
| Routing | Expo Router | Next.js App Router |
| Auth client | `@convex-dev/auth/react` | `@convex-dev/auth/nextjs` (server) + `@convex-dev/auth/react` (client components) |
| Token storage | `expo-secure-store` (Keychain/Keystore) | HTTP-only cookies (set by middleware) |
| Provider wrapper | `<ConvexAuthProvider>` in `_layout.tsx` | `<ConvexAuthNextjsProvider>` in `app/layout.tsx` + `convexAuthNextjsMiddleware()` in `middleware.ts` |
| Build | EAS | Vercel |
| Submit | App Store / Play Store | Vercel deploy (no app store) |
| Stages 8a/8b (landing/legal), 13 (TestFlight), 14 (screenshots), 16 (ASC) | Apply | Skip — the Next.js app IS the landing/web presence |
| Stage 11 (App Store audit) | Apply | Skip |
| Stage 9 (Sentry/PostHog) | `@sentry/react-native`, PostHog RN | `@sentry/nextjs`, PostHog JS — different packages, same concepts |
| Payments | RevenueCat (App Store IAP) | Direct Stripe (Stripe Checkout) |

---

## Quickstart pointers (read these when you start your first Next.js app)

The official docs are kept current — read from source rather than copying snippets here:

- **Convex + Next.js setup:** https://docs.convex.dev/quickstart/nextjs
- **Convex Auth on Next.js:** https://labs.convex.dev/auth/setup/nextjs
- **Convex Auth providers** (Anonymous, Password, OAuth): https://labs.convex.dev/auth/providers
- **Vercel + Convex deployment:** https://docs.convex.dev/production/hosting/vercel
- **Stripe + Next.js (App Router):** https://docs.stripe.com/checkout/quickstart?lang=node (when Stripe is in scope)
- **Stripe + Convex webhook pipeline:** `~/Documents/GitHub/ref-docs/integrations/stripe-convex.md` when present (covers checkout session → webhook → mutation)

---

## Three Next.js gotchas worth knowing up front

1. **Middleware is mandatory for cookie-based auth.** Convex Auth's Next.js setup relies on `convexAuthNextjsMiddleware()` running before every protected route. Forget it and `getAuthUserId(ctx)` returns null on the server.

2. **Server components vs client components.** Auth-aware queries inside server components use `fetchQuery(api.x.y, { token: await convexAuthNextjsToken() })`. Client components use the same `useQuery` hook from RN, just imported differently. Mixing them up causes hydration errors.

3. **Env vars.** `NEXT_PUBLIC_CONVEX_URL` for the client, `CONVEX_DEPLOYMENT` for `npx convex dev`/`deploy`. Add both to Vercel project settings (not just `.env.local`) before the first production deploy or it'll fail at build time.

---

## Stage map for Next.js apps

When you build a Next.js companion (or standalone Next.js app), the pipeline collapses:

| Stage | Status for Next.js |
|-------|-------------------|
| 0. Triage + Stack Profile | Apply (Convex profile by default) |
| 1. Repo spin-up | Apply — `npx create-next-app@latest` instead of `create-expo-app` |
| 2. Onboarding clone | Apply if onboarding flow exists; often skipped for marketing/dashboard sites |
| 3. Brand lock | Apply — `theme.ts` pattern works the same |
| 4. Onboarding hardening | Apply if Stage 2 ran |
| 5. Main app plan | Apply — `BUILD_PLAN.md` works the same |
| 6. Main app build | Apply |
| 7. Polish audit | Apply |
| 7b. AI imagery | Apply (still nanobanana via MCP) |
| 8a. Legal URLs wiring | Adapted — privacy/terms can live on `/privacy` and `/terms` routes within the Next.js app itself; no applanding.co needed |
| 8b. Branded landing | N/A — the app IS the landing |
| 9. Sentry + PostHog | Apply with Next.js variants of the SDKs |
| 10. Self-check gate | Apply |
| 11. App Store audit | SKIP |
| 12. EAS build | SKIP — `vercel deploy` instead |
| 13. TestFlight | SKIP — Vercel preview deployments serve the same role |
| 14. Screenshots | SKIP |
| 15. ASC metadata | SKIP |
| 16. ASC submission | SKIP |

---

## Shared-DB pattern (the reason this file exists)

When a Next.js app and an RN app point at the same Convex deployment:

1. **Single `convex/` folder lives in one of the two repos** (or in a shared package). Pick the canonical home — usually the RN repo since it's the more active codebase. The Next.js repo imports the generated API.

2. **Both apps run `npx convex dev` against the same deployment URL** during development. Schema changes flow to both clients automatically via the generated `_generated/api.ts`.

3. **`convex/auth.config.ts` lists every provider both apps need.** If RN uses Anonymous + Password and Next.js uses Password + Apple OAuth, declare all three in one place.

4. **Run schema migrations carefully.** Both clients must be redeployed after a breaking schema change. The web side redeploys instantly via Vercel; the mobile side needs an EAS Update or new TestFlight build.

5. **Web sessions and mobile sessions are independent** — the same user signing in on web and on mobile gets two separate session records in `authSessions` pointing at the same `users` row. That's normal and correct.
