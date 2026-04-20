# CLAUDE.md

> **For incoming agents: read this file first. Then read `PROCESS_GUIDE.md`. Then act.**

---

## This app

- **Name:** <APP NAME>
- **Purpose:** <ONE SENTENCE — what does this app do and for whom>
- **Slug:** <slug from BUNDLE_IDS.md>
- **Bundle ID:** `com.astrum.<slug>`
- **Stack Profile:** <Local | Convex — see STACK_PROFILES.md>
- **Current stage:** <0–16, per PROCESS_GUIDE.md>
- **Status:** <NEW | EXISTING-onboarding | EXISTING-mainapp | EXISTING-testflight>

---

## How to work in this repo

1. **Run `BOOTSTRAP.md` first** if this is your first session in this repo, or if anything seems off (missing skills, build errors, missing files). It's a 5-minute verification that catches problems before they cascade.
2. **Read `PROCESS_GUIDE.md`.** It defines all 16 stages with skip conditions.
3. **Find the current stage above.** Do only that stage's work. Do not get ahead.
4. **Use the prompt from `PROMPTS.md`** for the current stage.
5. **Self-check after every phase.** Invoke `/skill self-check` before claiming any phase done.
6. **Park scope creep in `LATER.md`.** Never expand scope inside the current stage.
7. **Read context first. Plan mode before building.** No code without a plan I've approved.

---

## Source-of-truth files

| File | What it is |
|------|-----------|
| `BOOTSTRAP.md` | Per-repo setup verification — run first, every time. |
| `PROCESS_GUIDE.md` | The 16-stage pipeline. Authoritative. |
| `STACK_PROFILES.md` | Architecture decision (Local / Convex). Picked at Stage 0. |
| `REF_DOCS_INDEX.md` | Map of stages → which file to read in the ref-docs library. |
| `ONBOARDING_PATTERNS.md` | Menu of 12 onboarding screen patterns. Pick 5-8 at Stage 4. |
| `AI_IMAGERY.md` | Prompt library + Gemini MCP workflow for Stage 7b image replacement. |
| `~/Documents/GitHub/ref-docs/` | Tech stack reference library at apps root (~/Documents/GitHub/) (NOT per-repo). Convex, Sentry, RevenueCat, Stripe official docs. |
| `PROMPTS.md` | Copy-paste prompts indexed by stage. |
| `BUNDLE_IDS.md` | Cross-app tracker (lives in apps-root, not per-repo). |
| `EXPO_SKILLS.md` | Reference for the official Expo skills. |
| `APP_LANDING_PAGE.md` | applanding.co workflow for Stage 8a (legal URLs) and Stage 8b (branded page). |
| `convex-auth-setup.md` | Convex Auth (`@convex-dev/auth`) setup, sign-in patterns, account deletion. Read for Convex profile apps. |
| `convex-react-client.md` | Convex queries/mutations/actions reference. Read at Stage 6 for Convex profile apps. |
| `nextjs-bridge.md` | When/how to add a Next.js companion to a Convex-backed app. |
| `legal/privacy.template.md` | Privacy policy template — filled at Stage 8a. |
| `legal/terms.template.md` | Terms of use template — filled at Stage 8a. |
| `SCREENSHOT_WORKFLOW.md` | App Store screenshot tool workflow for Stage 14. |
| `TESTFLIGHT.md` | Internal + External tester setup, ASC group creation, troubleshooting. Stage 13. |
| `LATER.md` | Everything out of scope for v1. Keeps current stages clean. |
| `BUILD_PLAN.md` | (created at Stage 5) phased plan for the main app build. |
| `asc-metadata.md` | (created at Stage 15) App Store Connect copy-paste content. |

---

## Hard constraints

- **No inline hex colors.** Every color comes from `src/theme/theme.ts` via import.
- **No magic spacing values.** Every spacing comes from `theme.ts` spacing scale (`4/8/12/16/24/32/48`).
- **No inline font families.** Every font comes from `theme.ts`.
- **For Local profile:** no real auth, no real backend, no real payments in v1.
- **For Convex profile:** Convex DB IS the v1 backend. Real payments still parked in `LATER.md`. Auth via @convex-dev/auth — anonymous by default, accounts when needed. See `convex-auth-setup.md`.
- **One agent per repo at a time.** If another agent is already working here, stop.
- **Global keyboard dismiss button required.** A `KeyboardHideButton` component must be mounted once at the app root (after the navigator). Appears above any keyboard/number pad, right-aligned, themed to match the app. One mount covers every screen — do not add per-screen.

---

## When in doubt

Stop. Ask. Don't guess. The cost of asking is 30 seconds; the cost of guessing wrong is undoing 2 hours of work.

---

## Stage history (update as you progress)

| Stage | Completed | Notes |
|-------|:-:|-------|
| 0. Triage | ☐ | |
| 1. Repo spin-up | ☐ | |
| 2. Onboarding clone & build | ☐ | |
| 3. Brand lock / tokenize | ☐ | |
| 4. Onboarding hardening | ☐ | |
| 5. Main app plan | ☐ | |
| 6. Main app build | ☐ | |
| 7. Polish audit | ☐ | |
| 8a. Legal URLs wired | ☐ | |
| 9. Analytics + Sentry | ☐ | |
| 10. Self-check gate | ☐ | |
| 11. App Store audit | ☐ | |
| 12. EAS build | ☐ | |
| 13. TestFlight + smoke test | ☐ | |
| 14. Screenshots | ☐ | |
| 15. Metadata generation | ☐ | |
| 8b. Landing page build | ☐ | (MANDATORY — after Stage 15, before Stage 16) |
| 16. ASC submission | ☐ | |
