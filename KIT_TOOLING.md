# KIT_TOOLING.md — System-Wide Skills & MCPs

> **What this is:** the agent-facing inventory of skills and MCP servers installed at user scope on Steve's Mac. Every agent in every repo has access to everything listed here without per-project setup. Read this on session start so you know what's available before reaching for workarounds.
>
> **Verify installation any time:**
> ```bash
> claude mcp list && echo "---" && ls -1 ~/.claude/skills/
> ```
>
> **If something listed here is missing or fails to invoke:** flag it and stop. Don't work around it — the user wants the kit fixed, not bypassed.

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

## Skills (auto-loaded, ~/.claude/skills/)

Skills auto-load based on relevance. Knowing they exist lets you reach for them deliberately rather than improvising.

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

## Updating this inventory

When a new skill or MCP is added to user scope, append it here and bump the kit version in the `README.md`. Agents trust this list — keep it accurate.
