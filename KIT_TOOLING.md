# KIT_TOOLING.md — System-Wide Skills & MCPs

> **What this is:** the agent-facing inventory of skills and MCP servers available in any repo running this kit. Read this on session start so you know what's available before reaching for workarounds.
>
> **Two skill locations to know about:**
>
> 1. **Per-repo kit skills** at `.claude/skills/` — 8 skills that ship with the kit via degit. Version-locked to whatever kit version this repo last pulled. Listed below.
> 2. **User-scope vendor skills** at `~/.claude/skills/` — Convex, Clerk, Vercel, Next.js, Expo plugins. Installed once per Mac. Listed below.
>
> **Verify installation any time:**
> ```bash
> claude mcp list && echo "---PER-REPO SKILLS---" && ls .claude/skills/ && echo "---USER-SCOPE SKILLS---" && ls ~/.claude/skills/
> ```
>
> **If something listed here is missing or fails to invoke:** flag it and stop. Don't work around it — refresh the kit (`npx degit s-411/drop-in-kit --force`) for per-repo skills, or check `CREDENTIALS.md` for user-scope MCPs.

---

## Per-repo kit skills (.claude/skills/)

These ship with the kit. Refresh via `npx degit s-411/drop-in-kit --force` in the repo root.

- **design-consistency** — Stage 7a (and any time UI is added/changed). Audits new screens against the existing design system to prevent visual drift.
- **screen-wiring** — Pre-submission navigation audit. Confirms every screen is reachable, every button routes, no orphaned screens.
- **app-backend-builder** — Stage 5/6. Reads an app with onboarding but no core loop, generates a phased PRD + build plan for the dashboard, settings, navigation, main features. Has reference resources at `references/app-patterns.md` and `references/build-plan-template.md`.
- **self-check** — After any significant change. Claude reviews its own work before claiming a task done. Invoke as `/skill self-check`. Mandatory at Stage 10 gate.
- **app-store-approval** — Stage 11. Full submission workflow: rejection-risk audit, ASC metadata, codebase review for App Store guidelines.
- **expo-publish** — Stage 12+. EAS build, TestFlight, version bumps, eas.json config, build failure troubleshooting, OTA updates via Expo Updates.
- **sentry-setup** — Stage 9a. Add Sentry crash reporting from scratch. Run after core app is built and EAS build is confirmed working, before App Store submission.
- **analytics-posthog** — Stage 9b. Add PostHog product analytics from scratch. Same timing window as sentry-setup.

---

## User-scope vendor skills (~/.claude/skills/)

Installed once per Mac via vendor marketplaces. Auto-load based on relevance. Knowing they exist lets you reach for them deliberately rather than improvising.

### Expo (13)

The Expo team's official skills, installed via `git clone https://github.com/expo/skills.git ~/.claude/skills/expo-skills` (it's a plugin-repo layout, not a single skill). Reach for these on any Expo CLI, EAS, native build, or platform-specific Expo issue.

Highest-relevance for this kit's RN apps:

- **building-native-ui** — RN UI patterns with Expo Router. Core reference for Stage 2 onboarding builds and Stage 6 main app builds.
- **native-data-fetching** — network requests, API calls, data fetching in RN. Stage 6 when wiring Convex queries/mutations.
- **expo-deployment** — App Store / Play Store / web hosting deploys. Stage 12-16.
- **expo-cicd-workflows** — EAS workflow YAML.
- **upgrading-expo** — SDK version upgrades.
- **eas-update-insights** — OTA update health metrics post-launch.

Other Expo skills available: `expo-api-routes`, `expo-dev-client`, `expo-module`, `expo-tailwind-setup`, `expo-ui-jetpack-compose`, `expo-ui-swift-ui`, `use-dom`.

### Convex (6)

- **convex-quickstart** — initial Convex setup (Stage 5)
- **convex-setup-auth** — Convex Auth wiring (Stage 5/6, Convex profile only). Pair with this kit's `convex-auth-setup.md`.
- **convex-create-component** — reusable Convex components
- **convex-performance-audit** — query/mutation performance analysis
- **convex-migration-helper** — schema migration patterns
- **convex** — general Convex patterns reference

### Clerk (19 — Next.js bridge apps only, not RN)

Full set: `clerk`, `clerk-setup`, `clerk-nextjs-patterns`, `clerk-react-patterns`, `clerk-expo-patterns`, `clerk-backend-api`, `clerk-billing`, `clerk-orgs`, `clerk-webhooks`, `clerk-testing`, `clerk-custom-ui`, `clerk-android`, `clerk-swift`, `clerk-astro-patterns`, `clerk-chrome-extension-patterns`, `clerk-nuxt-patterns`, `clerk-react-router-patterns`, `clerk-tanstack-patterns`, `clerk-vue-patterns`.

Highest-relevance for this kit's Next.js bridge: `clerk-setup`, `clerk-nextjs-patterns`, `clerk-billing`, `clerk-webhooks`.

### Next.js (3)

- **next-best-practices** — App Router, Server Components, common patterns
- **next-cache-components** — Next 16 / App Router caching
- **next-upgrade** — version upgrade workflows

### Vercel (7)

- **deploy-to-vercel** — deployment workflows
- **vercel-cli-with-tokens** — token-based deploys
- **vercel-composition-patterns** — multi-app composition
- **vercel-react-best-practices** — React patterns
- **vercel-react-native-skills** — RN-flavored Vercel patterns (RN generally doesn't deploy to Vercel; limited relevance)
- **vercel-react-view-transitions** — view transition API
- **web-design-guidelines** — Vercel's design system guidelines

---

---

## MCP servers

### Mandatory at Stage 1 of every repo

- **nanobanana** — image generation via Gemini API. Used for app icons (Stage 7b), in-app illustrations (Stage 7b), and App Store screenshot outpainting (Stage 14). Install verification is step 3a of `START_NEW.md`. If `claude mcp list` doesn't show it as `✓ Connected`, stop and reinstall.

### Used during RN app builds

- **convex** — Convex DB introspection. Stage 5/6 — inspect schema, query state, debug auth wiring. Multi-project mode (auto-routes to whichever Convex project the current repo is linked to).
- **revenuecat** — IAP introspection. Stage 12 — check entitlements, products, customer state. Skip for Local Stack Profile apps.

### Used during Next.js companion app builds

- **vercel** (plugin) — deploy + env management. Slash commands: `/deploy`, `/env`, `/status`, `/bootstrap`. Auto-activates on Vercel/Next.js project detection.
- **clerk** — Clerk docs MCP for SDK snippet lookup. Note: RN apps in this kit do NOT use Clerk (Convex Auth only). This MCP is only relevant for Next.js bridge apps.

### Not used by Claude Code in this kit

Google Drive, Notion, Gmail, Calendar are claude.ai consumer connectors. Ignore them in build sessions.

---

## Updating this inventory

- **Per-repo kit skills:** edit them in the kit repo at `.claude/skills/` and push. Each app picks them up on the next `npx degit s-411/drop-in-kit --force`.
- **User-scope vendor skills or new MCPs:** append to the relevant section here, then bump the kit version in the `README.md`.

Agents trust this list — keep it accurate.
