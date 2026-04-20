# REF_DOCS_INDEX — Pointer to the Tech Stack Library

> **Where it lives:** `~/Documents/GitHub/ref-docs/` at apps root. NOT inside individual repos.
>
> **What it is:** a curated, iterative library of official docs for Convex, Sentry, RevenueCat, Superwall, Stripe — covering the cross-stack integrations that are hardest to find. Built up over time as you hit (and solve) painful integration bugs.
>
> **When to use:** an agent at Stage 5/6/9/11 looks here for technical reference. Workflow always lives in this kit (`PROCESS_GUIDE.md`, `PROMPTS.md`); deep technical reference lives in ref-docs. Auth-specific patterns for Convex Auth live in `convex-auth-setup.md` here in the kit (no ref-docs lookup needed).

---

## Token discipline

These files are **NOT auto-loaded.** Read them on demand only — never "just in case." The biggest llms-txt dumps (Convex full ~42k lines, Stripe full ~14k lines) will blow context if loaded speculatively. Use this index to pick the smallest relevant file and load just that.

---

## Quick lookup: stage → which file

### Stage 5 — Main App Plan (architecture decisions)

| Stack Profile | Files to read |
|---------------|---------------|
| Local | None — no external services |
| Convex (anonymous or accounts) | `convex-auth-setup.md` (in this kit) + `ref-docs/integrations/convex-expo-quickstart.md` (when present) |

If billing is in scope:
- Mobile via App Store IAP: `ref-docs/integrations/revenuecat-expo.md`
- Mobile with paywall A/B testing: above + `ref-docs/integrations/superwall-revenuecat.md`
- Web via Next.js: `ref-docs/integrations/stripe-nextjs.md`
- Web/Convex with webhooks: `ref-docs/integrations/stripe-convex.md`

### Stage 6 — Main App Build (implementation)

| Stack Profile | Files to read |
|---------------|---------------|
| Local | None |
| Convex | `convex-auth-setup.md` + `convex-react-client.md` (both in this kit) + `ref-docs/llms-txt/convex-full.txt` (when present, for API surface — load only if needed) |

### Stage 9 — Analytics + Sentry

| File | When |
|------|------|
| `ref-docs/integrations/sentry-expo.md` | Always — Expo-specific setup |
| `ref-docs/integrations/expo-using-sentry.md` | EAS Build source maps + EAS Update setup |
| `ref-docs/integrations/sentry-react-native.md` | If you need bare React Native (no Expo) — rare |

### Stage 11 — App Store Audit

If app uses RevenueCat for IAP: cross-check `ref-docs/integrations/revenuecat-expo.md` against your implementation for the iOS purchase capability + Android billing permission.

If app uses Convex Auth with accounts: re-check the account deletion section of `convex-auth-setup.md` — App Store will reject without in-app account deletion.

### Stage 12 — EAS Build

If anything goes wrong with EAS env vars: `ref-docs/llms-txt/expo.txt` (when present) or `ref-docs/gotchas/eas-env-vars.md` (when added).

---

## High-value docs by length / depth

The most substantive integration docs (worth the agent's time when present in `ref-docs/integrations/`):

| Doc | Approx words | Why it matters |
|-----|--------------|----------------|
| `superwall-revenuecat.md` | 1,113 | The full purchase pipeline — `CustomPurchaseControllerProvider`, callbacks, customer info syncing |
| `stripe-convex.md` | 751 | End-to-end payment flow — checkout session → webhook → transactional fulfillment |
| `revenuecat-expo.md` | 646 | Full RC setup including dev build requirements, EAS profiles, Web Billing |
| `stripe-nextjs.md` | 508 | Stripe Checkout in Next.js with route handlers + success page |
| `sentry-expo.md` | 396 | Expo plugin config + Metro config + privacy manifest |

---

## llms-txt files: when to consult

The `ref-docs/llms-txt/` folder will hold full doc dumps for use as deep context (when populated):

| File | Approx lines | Use when |
|------|--------------|----------|
| `convex-full.txt` | 42,778 | Agent needs the complete Convex API surface — biggest file |
| `stripe-full.txt` | 14,400 | Agent needs deep Stripe details — webhooks, lifecycle, advanced patterns |
| `expo.txt` | 856 | Finding a specific Expo library or API |
| `convex-short.txt` | 741 | Quick Convex orientation before loading the full doc |
| `revenuecat-full.txt` | 390 | Navigating the RevenueCat doc index |

**Never load `convex-full.txt` or `stripe-full.txt` "just in case"** — they're huge. Load only when you need them and you know the keyword to grep.

---

## Skills available globally

Optional skill packs you can install once on your Mac. See `EXPO_SKILLS.md` for install commands.

- **Convex** (5 skills) — including `convex-quickstart`, `convex-setup-auth`, `convex-performance-audit`
- **Sentry** (8 skills) — including `sentry-react-native-sdk`, `sentry-fix-issues`
- **RevenueCat** (4 skills + 4 agent files) — including `paywall-builder`, `troubleshoot`

---

## MCP servers available

Optional MCP servers you can wire per-repo. See `CREDENTIALS.md` for setup commands. Nanobanana is the only mandatory one (see `START_NEW.md` step 3a).

- **Nanobanana MCP** (mandatory, every new repo) — Gemini image generation for in-app imagery + App Store screenshots
- **Stripe MCP** — payments API, checkout sessions, docs search
- **Sentry MCP** — debug issues, search errors, Seer AI analysis
- **RevenueCat MCP** — manage subscriptions, products, projects

---

## How this library grows

This is an iterative library, not a one-time scrape. The protocol is in `ref-docs/README.md`:

1. Agent gets stuck on something
2. You unblock it manually with the official docs
3. Save the relevant doc(s) under `integrations/` or `gotchas/`
4. Add a row to this index so future agents find it before getting stuck

The first 5 entries pay for themselves the first time each prevents an hour of agent flailing.

---

## Refreshing the library

When you have populated llms.txt files, add a `refresh.sh` script in `~/Documents/GitHub/ref-docs/` and re-run every ~60 days to catch upstream changes.

```bash
cd ~/Documents/GitHub/ref-docs
./refresh.sh
```

---

## What's NOT in ref-docs (look elsewhere)

- React Native core docs — no llms.txt available, see https://reactnative.dev
- Next.js core docs — already loaded via Vercel plugin in Claude Code
- Convex Auth setup patterns — in this kit's `convex-auth-setup.md`, not in ref-docs
- Adapty (paywall infra alternative) — parked in `LATER.md`
- Branch.io (deep linking + referrals) — parked in `LATER.md`
- Mixpanel (analytics) — we use PostHog, parked in `LATER.md`

---

## Where this fits in the architecture

```
~/Documents/GitHub/
├── BUNDLE_IDS.md            ← cross-app tracker (apps-root)
├── .env-apps                ← shell creds (apps-root, gitignored)
├── ref-docs/                ← THIS LIBRARY (apps-root)
│   ├── README.md
│   ├── integrations/
│   ├── gotchas/
│   └── llms-txt/
├── gym-template/            ← per-app repo (has the kit)
│   ├── BOOTSTRAP.md
│   ├── CLAUDE.md
│   ├── PROCESS_GUIDE.md
│   ├── PROMPTS.md
│   ├── STACK_PROFILES.md
│   ├── REF_DOCS_INDEX.md   ← THIS FILE — points at ref-docs
│   ├── convex-auth-setup.md
│   ├── convex-react-client.md
│   ├── nextjs-bridge.md
│   ├── legal/
│   │   ├── privacy.template.md
│   │   └── terms.template.md
│   └── ...
└── cpn/                     ← another per-app repo
    └── ...
```

The kit travels with the repo. ref-docs stays at apps root and is referenced by path.
