# LATER.md

> **Everything out of scope for v1 lives here.** When an agent is tempted to scope-creep, it parks the work here instead and returns to the current stage.
>
> Review this file before starting v1.1 of any app — top items become future apps's work.

---

## Auth

- [ ] Sign in with Apple OAuth via Convex Auth (mandatory if any social login is added later — App Store rejects social-login apps without it)
- [ ] Account deletion path (required by App Store — Stage 11 will flag if missing)
- [ ] Password-less / magic link option

## Backend

- [ ] Convex schema + queries
- [ ] Replace mock data with real persistence
- [ ] Background sync
- [ ] Conflict resolution for offline edits

## Payments

- [ ] RevenueCat hookup (replace skip-able paywall)
- [ ] Restore Purchases working end-to-end
- [ ] Subscription tiers (monthly + annual)
- [ ] Discounted paywall variant (cohort-based)
- [ ] Promo codes

## Growth

- [ ] Push notifications (Expo Notifications)
- [ ] Deep linking (Expo Router with universal links)
- [ ] Share sheet integration
- [ ] App Clips (where applicable)
- [ ] Referral system

## Analytics

- [ ] Wire FunnelMob analytics — currently no React Native SDK, check back: https://docs.funnelmob.com
- [ ] Wire real PostHog dashboards (currently events fire but no funnel set up)
- [ ] Funnel: install → onboarding complete → paywall view → purchase
- [ ] Cohort analysis (D1, D7, D30 retention)
- [ ] Revenue tracking via RevenueCat → PostHog

## Native polish

- [ ] Dark mode (`theme.ts` already has the structure — needs `dark` variant)
- [ ] Haptics on every secondary interaction (currently only primary CTAs)
- [ ] Custom splash animation
- [ ] App icon variants (seasonal, premium)
- [ ] Widgets (iOS 17+ interactive widgets)

## CI / CD

- [ ] EAS workflows YAML for auto-build on main push (use `expo-cicd-workflows` skill)
- [ ] Auto-submit to TestFlight on green build
- [ ] Sentry source maps uploaded automatically

## Growth tools to evaluate (decide before v1.1)

- [ ] **Superwall** — paywall A/B testing on top of RevenueCat. Use if conversion rate matters and you want to test paywall variants without rebuilds. https://superwall.com
- [ ] **Adapty** — alternative to RevenueCat (subscription infra + paywall builder in one). Smaller community than RC, evaluate only if RC has a specific pain point. https://adapty.io
- [ ] **Branch.io** — deep linking + referral attribution. Use when you want "user X referred user Y" tracking or universal links from web → app. https://branch.io
- [ ] **Mixpanel** — only consider if PostHog hits a wall (it won't at MVP scale)

## Cross-app roadmap (not app-specific)

- [ ] Build a library of pre-made theme files (`female-soft.ts`, `female-bold.ts`, `masculine-dark.ts`, `masculine-clean.ts`, etc.) → skip Stage 1 in future apps by picking a file
- [ ] Build a library of pre-baked onboarding archetypes (quiz-driven habit, journaling, tracker, calculator) → Stage 3 becomes "pick archetype + customize" instead of clone-from-scratch
- [ ] Add a `BRAND_GUIDE.md` template that captures the swatches + typography decisions per app for future-you reference
- [ ] Pre-baked `eas.json` profiles per app archetype

---

## Notes

- Anything added here gets a one-sentence context — what it is, why it's deferred. Don't park bare TODOs.
- When a bullet gets done, delete it (don't leave checked items cluttering the file).
- If something here keeps getting re-mentioned across multiple apps, promote it to "cross-app roadmap" and consider a kit update.
