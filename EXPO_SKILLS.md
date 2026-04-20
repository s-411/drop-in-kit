# EXPO_SKILLS.md — Official Expo Skills Reference

> **The official Expo skills install once globally on your Mac via Claude Code's plugin system. They are NOT copied into each repo.** This file is just a reference card.

---

## One-Time Install (run once, ever)

In any Claude Code session on your Mac:

```
/plugin marketplace add expo/skills
/plugin install expo
```

Done. All 12 official Expo skills are now available in every Claude Code session, in every repo, automatically. No per-repo setup.

**Source:** https://github.com/expo/skills (MIT license, maintained by the Expo team)

---

## Why NOT to copy `skills-main.zip` into each repo

- Skills are auto-discovered via the Claude Code plugin system globally
- Copying creates 15 stale copies you'd have to update by hand
- Pollutes repo with thousands of files unrelated to your app
- Defeats the point of skills as a global, versioned tool

The zip file is a fallback for non–Claude Code agents (e.g. Cursor's "Remote Rule" install). For your workflow, ignore the zip.

---

## The 12 Official Expo Skills

| Skill | Auto-invoked when… | Used in Stage |
|---|---|:-:|
| `building-native-ui` | Building screens, navigation, animations, tabs, layouts | 3, 4, 6 |
| `native-data-fetching` | Any network request, API call, React Query, error handling | 6 (later) |
| `expo-deployment` | Deploying to App Store or Play Store | 12, 13, 16 |
| `expo-dev-client` | Building dev client or distributing via TestFlight during dev | (rare) |
| `expo-cicd-workflows` | Setting up EAS workflow YAML files for CI/CD | (LATER.md) |
| `upgrading-expo` | Upgrading Expo SDK or fixing dependency conflicts | (as needed) |
| `expo-tailwind-setup` | Adding Tailwind CSS v4 / NativeWind to the project | (LATER.md) |
| `expo-api-routes` | Creating API routes in Expo Router with EAS Hosting | (LATER.md) |
| `expo-module` | Writing native Swift / Kotlin modules | (rare) |
| `use-dom` | Embedding web components in a native app | (rare) |
| `expo-ui-swift-ui` | Using SwiftUI components in Expo | (rare) |
| `expo-ui-jetpack-compose` | Using Jetpack Compose components in Expo | (rare) |

---

## Custom Skills in This Kit (per-repo)

These are YOUR custom skills, copied into each repo via the drop-in kit's `.claude/skills/` folder:

| Skill | What it does | Used in Stage |
|---|---|:-:|
| `design-consistency` | Audits codebase for theme drift (audit-only — does NOT generate `theme.ts` from scratch) | 7 |
| `screen-wiring` | Audits navigation; identifies missing screens/wiring | 4 |
| `app-backend-builder` | Reads onboarding → infers app → generates `BUILD_PLAN.md` | 5 |
| `self-check` | Verifies work before claiming any phase complete | every stage |
| `app-store-approval` | Phase 1: rejection audit. Phase 2: ASC metadata generation. | 11, 15 |
| `expo-publish` | Pre-build checklist + EAS production build | 12 |
| `sentry-setup` | Wires Sentry crash reporting from scratch | 9a |
| `analytics-posthog` | Wires PostHog analytics from scratch | 9b |

> **Deprecated:** `react-native-patterns` — replaced by Expo's official `building-native-ui`. Remove from kit if still present.

---

## Optional Per-Stack Skill Packs (install once globally, when relevant)

Skill packs from official upstream sources. Install ONLY the ones for stacks you're actually using.

### Convex (5 skills) — install before your first Convex app
```bash
npx skills add get-convex/agent-skills
```
Includes: `convex-quickstart` (scaffolding new projects), `convex-setup-auth` (Convex Auth + alternative providers), `convex-create-component`, `convex-migration-helper`, `convex-performance-audit`.

### Sentry (8 skills) — install when you start wiring Sentry
Install via instructions at: https://github.com/getsentry/sentry-for-ai

Includes: `sentry-sdk-setup` (universal entry), `sentry-react-native-sdk` (full Expo + RN setup), `sentry-nextjs-sdk`, `sentry-fix-issues` (debugging via MCP), `sentry-create-alert`, `sentry-workflow`.

### RevenueCat (4 skills + 4 agent files) — install before your first paid app
```bash
curl -fsSL https://raw.githubusercontent.com/RevenueCat/rc-claude-code-plugin/main/install.sh | bash
```
Includes: `status` (project overview), `create-app`, `create-product`, `apikey` retrieval. Plus reference files: `paywall-builder`, `troubleshoot`, `project-bootstrap`, `design-system-extractor`.

---

## Skills the agent reads but doesn't auto-invoke

Beyond skills, the agent reads ref-docs files at apps root (~/Documents/GitHub/) for technical reference. See `REF_DOCS_INDEX.md` for the stage-by-stage map of which files to consult.

---

## When Custom and Official Overlap

| Topic | Custom | Official | Use which |
|---|---|---|---|
| Building UI | (none) | `building-native-ui` | Official |
| Deploying | `expo-publish` | `expo-deployment` | Use both — `expo-publish` for the pre-flight + EAS commands, `expo-deployment` for deeper questions |
| App Store submission audit | `app-store-approval` | (none) | Custom — does the rejection audit no official skill covers |
| Generating ASC metadata | `app-store-approval` Phase 2 | (none) | Custom |
| Screen wiring audit | `screen-wiring` | `building-native-ui` | Custom for audit, official for new UI work |

---

## Verifying skills are installed

In any Claude Code session:

```
/plugin
```

You should see `expo` listed as installed. If not, re-run:

```
/plugin marketplace add expo/skills
/plugin install expo
```

---

## What to do if Expo ships skill updates

Re-run `/plugin install expo` periodically (every few weeks). It'll fetch the latest. No per-repo work required.

---

## Notes

- These skills are **fine-tuned for Opus models.** You're using Opus 4.6, so they'll perform optimally.
- Source repo: https://github.com/expo/skills
- License: MIT
