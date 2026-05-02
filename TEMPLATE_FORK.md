# TEMPLATE_FORK.md — Building a New App from the Template

> **What this is:** the complete workflow for creating a new app by forking the design-complete template repo. Replaces `START_NEW.md` for template-based apps.
>
> **When to use:** every time you create a new app from the template. Do NOT use `START_NEW.md` — that's for blank-slate `create-expo-app` projects.
>
> **What you keep:** every main-app screen layout, every component, the entire theme/token system, the color scheme editor, KeyboardHideButton, all UI patterns.
>
> **What you build fresh:** onboarding (Stages 2-4) — different for every app.
>
> **What you change:** brand identity (one file), demo content/fixtures (one file), prompt library (one file), navigation wiring (which screens connect to which), and app.json identity fields.

---

## One-time setup: push the template to GitHub

Before your first fork, the template needs to be on GitHub so future apps can pull it with `degit`. Do this once:

1. Create a new public repo on GitHub: `s-411/hinge-template` (or whatever you want to call it)
2. From the template codebase on your Mac:
   ```bash
   git remote add template-origin https://github.com/s-411/hinge-template.git
   git push template-origin main --force
   ```
3. Done. Every fork from now on pulls with `npx degit s-411/hinge-template --force`.

When you update the template (fix something that should flow to future forks), push again from the template repo. Existing forks are unaffected — they've already diverged.

---

## Pre-flight (before you start each fork)

1. **Template repo Phase A is done.** `lib/brand.ts`, `lib/fixtures.ts`, `lib/prompts.ts` all exist. If they don't, the template isn't ready — go back and finish Phase A first.
2. **You know what the app is.** Name, slug, one-sentence purpose, and a rough idea of which screens from the template you'll use and how they connect.
3. **One-time Mac setup is done.** `CREDENTIALS.md` completed (Gemini key, EAS token, etc.).

---

## Stage 0 — Triage (~5 min)

Same as the standard pipeline. Pick:
- **Name** (working — finalized at Stage 11)
- **Slug** (e.g. `fitness-tracker`, `finance-app`)
- **Purpose** (one sentence)
- **Stack Profile** — always **Local** for template forks (the template uses AsyncStorage, no Convex)

---

## Stage 1 — Fork + Identity Swap (~15 min)

This replaces the standard Stage 1 entirely. No `create-expo-app`, no kit pull from `s-411/drop-in-kit`.

### Before pasting the prompt

1. **Create a new empty repo** on GitHub Desktop (or `git init` in a new folder). Name it the working slug.
2. **Pull the template** into the empty repo:
   ```bash
   npx degit s-411/hinge-template --force
   ```
   This downloads the full design-complete template (all screens, theme system, components, kit docs) in one shot. The drop-in-kit docs are already included — no separate `degit s-411/drop-in-kit` needed.
3. **Open the repo in VS Code / Cursor. Open Claude Code.**

### The Prompt

Fill in the **3 required fields**, paste into Claude Code in the new repo.

```
TEMPLATE FORK — NEW APP FROM DESIGN TEMPLATE

# REQUIRED:
APP_NAME: <e.g. Fitness Tracker>
WORKING_SLUG: <e.g. fitness-tracker>
PURPOSE: <one sentence>

This repo was forked from the design template. The UI, theme system, and
all screen layouts are ready to use. Your job is to swap the identity and
content — NOT rebuild any UI.

Execute the following steps in order. STOP and ask if anything fails.

═══════════════════════════════════════════════════════════════════
STEP 1 — IDENTITY SWAP (app.json + package.json)
═══════════════════════════════════════════════════════════════════

Edit app.json:
  - expo.name → APP_NAME
  - expo.slug → WORKING_SLUG
  - expo.scheme → WORKING_SLUG (no hyphens, lowercase)
  - expo.ios.bundleIdentifier → com.astrum.<slug-without-hyphens>
  - expo.extra.eas.projectId → leave as-is (re-linked at EAS init)
  - Add/confirm: expo.ios.config.usesNonExemptEncryption = false

Edit package.json:
  - name → WORKING_SLUG

═══════════════════════════════════════════════════════════════════
STEP 2 — BRAND SWAP (lib/brand.ts — ONE file edit)
═══════════════════════════════════════════════════════════════════

Open lib/brand.ts. Replace every value with the new app's brand:
  - brand.name → APP_NAME
  - brand.tagline → a tagline that fits the new app
  - brand.memberLabel → "<APP_NAME> Member" (or equivalent)
  - brand.plusName → "<APP_NAME>+" (or remove if no paywall tiers)
  - brand.proName → "<APP_NAME> Pro" (or remove)

Verify: grep -rn "Connect" app/ components/ — should return ZERO hits
outside of lib/brand.ts. If any remain, the template wasn't fully
centralised — fix them now by importing from brand.ts.

═══════════════════════════════════════════════════════════════════
STEP 3 — CONTENT SWAP (lib/fixtures.ts — ONE file edit)
═══════════════════════════════════════════════════════════════════

Open lib/fixtures.ts. Replace all demo profile data, sample content,
and paywall benefit lists with content appropriate for the new app's
vertical. Keep the same data shape — just change the values.

═══════════════════════════════════════════════════════════════════
STEP 4 — PROMPT LIBRARY SWAP (lib/prompts.ts — ONE file edit)
═══════════════════════════════════════════════════════════════════

Open lib/prompts.ts. Replace the dating-specific prompt questions with
prompts relevant to the new app's domain. Keep the same structure
(array of { label, answer } or similar) — just change the content.

If the new app doesn't use a prompt/quiz concept, empty the array and
leave the file as a stub — don't delete it.

═══════════════════════════════════════════════════════════════════
STEP 5 — ASSETS (icons + splash)
═══════════════════════════════════════════════════════════════════

Replace these files with the new app's assets (or leave as placeholders
and replace at Stage 7b with AI-generated versions):
  - assets/icon.png (1024×1024)
  - assets/adaptive-icon.png (1024×1024)
  - assets/splash-icon.png
  - assets/favicon.png

If app.json splash.backgroundColor doesn't match the new app's
theme/tokens.ts colors.bg, update it now.

═══════════════════════════════════════════════════════════════════
STEP 6 — NANOBANANA MCP (MANDATORY)
═══════════════════════════════════════════════════════════════════

Check if nanobanana is already connected (it's usually system-wide):
  claude mcp list | grep nanobanana
If output shows ✓ Connected → skip the install, move to step 7.
If NOT connected → extract the key from a prior install:
  grep -o '"GEMINI_API_KEY": *"[^"]*"' ~/.claude.json | head -1
Use that value to install:
  claude mcp add nanobanana uvx nanobanana-mcp-server@latest -e GEMINI_API_KEY=<the-key-value>
Verify: claude mcp list | grep nanobanana → ✓ Connected
If no key exists in ~/.claude.json, STOP and tell me.
If you had to install (not just verify), exit Claude Code and restart.

═══════════════════════════════════════════════════════════════════
STEP 7 — EAS INIT
═══════════════════════════════════════════════════════════════════

Run: eas init
This creates a new EAS project and updates expo.extra.eas.projectId.
If it asks for credentials or fails, STOP — CREDENTIALS.md not done.

Then patch eas.json submit.production.ios:
  "appleTeamId": "FUHV534M4K"
  "ascAppId": "TBD-set-at-Stage-11"

═══════════════════════════════════════════════════════════════════
STEP 8 — FILL CLAUDE.md
═══════════════════════════════════════════════════════════════════

Update the CLAUDE.md header:
  - Name: APP_NAME
  - Purpose: PURPOSE
  - Slug: WORKING_SLUG
  - Bundle ID: com.astrum.<slug-without-hyphens>
  - Stack Profile: Local
  - Current stage: 1
  - Status: TEMPLATE FORK

In the stage history, note: "Stages 2-4 required — onboarding built per-app."

═══════════════════════════════════════════════════════════════════
STEP 9 — COMMIT + VERIFY
═══════════════════════════════════════════════════════════════════

  git add -A
  git commit -m "Fork from template — <APP_NAME>"

Run BOOTSTRAP.md verification. Report results.

Verify identity swap:
  grep -rni "theme picker" app.json README.md CLAUDE.md package.json
  → should return ZERO hits (or only legitimate ones)

Verify brand swap:
  grep -rn "Connect" app/ components/ lib/
  → only lib/brand.ts should define the literal

Run: npx tsc --noEmit → zero errors

═══════════════════════════════════════════════════════════════════
STEP 10 — HANDOFF
═══════════════════════════════════════════════════════════════════

Tell me:
  - The command to start: npx expo start
  - What I should see on the welcome screen (should show the new
    APP_NAME and tagline, not "Connect")
  - Confirm all identity fields are swapped
  - Confirm we are ready for Stage 5 (Main App Plan)

Do NOT begin Stage 5 work. Stop after step 10.

Terminal commands you produce must assume the terminal is already in the
repo root — never prefix with cd <repo>. cd is only acceptable to move
into a subdirectory of this repo.

When complete, return a summary of what you changed, files touched, and
any issues encountered.
```

---

## Stages 2, 3, 4 — BUILD ONBOARDING (same as standard pipeline)

Onboarding is different for every app — these are NOT skipped.

- **Stage 2 (Onboarding build)** — use the standard prompt from `PROMPTS.md` Stage 2. Put HTML references in `app-references/html/` and screenshots in `app-references/images/`. The template's existing screens are for the main app, not onboarding.
- **Stage 3 (Brand lock / tokenize)** — use the standard prompt. Extract onboarding colors/fonts into `theme/tokens.ts` (which already exists — add entries, don't restructure).
- **Stage 4 (Onboarding hardening)** — use the standard prompt. Wire onboarding flow, paywall variants, MainAppShell entry point. Mount `KeyboardHideButton` if not already inherited from the template root (it should be — verify).

Mark these as complete in CLAUDE.md stage history only after they're actually built.

---

## Stage 5 — App Plan (MODIFIED for template forks)

Standard Stage 5 infers what the app does and writes a BUILD_PLAN.md from scratch. For template forks, the screens already exist — the agent's job is to decide which ones to USE, which to REMOVE, and how to REWIRE them.

```
SKILLS: Use app-backend-builder skill for this work.

Read CLAUDE.md. This is a TEMPLATE FORK — the UI screens already exist.
We are at Stage 5 — planning how to adapt the template into this app.

Scan every screen and component in the repo. Build an inventory:
  - List every screen (file path + one-line description of what it shows)
  - Group into: KEEP (fits this app's purpose), ADAPT (keep layout but
    change content/behavior), REMOVE (doesn't fit this app at all)

Then write BUILD_PLAN.md to repo root with:
  - App purpose (from CLAUDE.md)
  - Screen inventory with KEEP / ADAPT / REMOVE decisions
  - Navigation map — how the KEEP + ADAPT screens connect for this app's
    user flow (which screen leads to which, tab structure, modal triggers)
  - For each ADAPT screen: what specifically changes (copy, data source,
    behavior) and what stays (layout, components, styling)
  - Content plan — what goes in lib/fixtures.ts for this app's domain
  - Phased build plan (2-4 phases, each independently testable):
    Phase 1: Remove unused screens, rewire navigation
    Phase 2: Adapt content on remaining screens
    Phase 3: Any new screens needed that don't exist in the template
    Phase 4: Polish + verify everything connects

Do NOT implement anything. Plan only. I will review BUILD_PLAN.md and
approve before Stage 6.

Terminal commands you produce must assume the terminal is already in the
repo root — never prefix with cd <repo>.

When complete, return a summary of what you wrote and any questions.
```

---

## Stage 6 — Adapt + Build (MODIFIED for template forks)

```
SKILLS: Use building-native-ui and self-check skills. Run self-check at
the end of every phase.

Read CLAUDE.md and BUILD_PLAN.md. This is a TEMPLATE FORK — you are
ADAPTING existing screens, not building from scratch.

RULES:
- Do NOT rebuild screens that exist. Adapt their content.
- Do NOT change the theme system, tokens, or component styling patterns.
- All content changes flow through lib/brand.ts, lib/fixtures.ts, and
  lib/prompts.ts — not inline in screens.
- If you need a screen that doesn't exist in the template, build it
  following the EXACT same patterns (useMemo([role]), theme tokens,
  same component library).
- Navigation changes happen in app/_layout.tsx and the relevant
  (tabs)/_layout.tsx — don't scatter navigation logic.

Begin Phase 1 from BUILD_PLAN.md. Show me the diff before moving to
Phase 2. Do NOT batch phases.

Terminal commands you produce must assume the terminal is already in the
repo root — never prefix with cd <repo>.

When complete with each phase, return a summary of what you changed,
files touched, and any issues encountered.
```

---

## Full stage map for template forks

| Stage | Status |
|-------|--------|
| 0. Triage | Same as standard |
| 1. Fork + identity swap | **MODIFIED** — use TEMPLATE_FORK.md prompt, not START_NEW.md |
| 2. Onboarding build | **USE** — standard prompt from PROMPTS.md (onboarding is per-app) |
| 3. Brand lock / tokenize | **USE** — standard prompt (add onboarding tokens to existing theme) |
| 4. Onboarding hardening | **USE** — standard prompt |
| 5. Main app plan | **MODIFIED** — inventory + rewire existing screens, not build from scratch |
| 6. Main app adapt | **MODIFIED** — adapt content, rewire nav, new screens only if needed |
| 7a. Polish audit | **USE** — standard prompt |
| 7b. AI imagery | **USE** — replace template placeholders with app-specific imagery |
| 8a. Legal URLs | **USE** — standard prompt |
| 9a. Sentry | **USE** — standard prompt |
| 9b. PostHog | **USE** — standard prompt |
| 10. Self-check | **USE** — standard prompt |
| 11. App Store audit | **USE** — standard prompt |
| 12. EAS build | **USE** — standard prompt |
| 13. TestFlight | **USE** — standard prompt |
| 14. Screenshots | **USE** — standard prompt (1242×2688 iPhone, 2048×2732 iPad) |
| 15. ASC metadata | **USE** — standard prompt |
| 8b. Landing page | **USE** — mandatory before Stage 16 |
| 16. Submit | **USE** — standard checklist |

Use the standard prompts from `PROMPTS.md` for every **USE** stage. For the three **MODIFIED** stages (1, 5, 6), use the prompts in this doc.

---

## Quick reference: what changes per fork

| What | Where | How |
|------|-------|-----|
| App name, tagline, tier names | `lib/brand.ts` | Edit one file |
| Demo profiles, sample data, paywall benefits | `lib/fixtures.ts` | Edit one file |
| Quiz/prompt questions | `lib/prompts.ts` | Edit one file |
| App identity (slug, bundle ID, scheme) | `app.json` + `package.json` | Stage 1 prompt handles it |
| App icon, splash, favicon | `assets/` | Replace 4 files |
| Onboarding screens + flow | `app/onboarding/` or `src/screens/onboarding/` | Built from scratch at Stage 2-4 |
| Which main-app screens are used + how they connect | `app/_layout.tsx` + BUILD_PLAN.md | Stage 5/6 |
| Colors + fonts | Theme editor in-app or `theme/tokens.ts` | Already works, no code change needed |
| Legal pages | `legal/privacy.md` + `legal/terms.md` | Stage 8a fills from templates |

## What NEVER changes per fork

- `theme/tokens.ts` structure (add entries fine, don't restructure)
- `theme/ThemeProvider.tsx` (the engine)
- Component patterns (`useMemo([role])`, no inline hex)
- `KeyboardHideButton` (already global, works in every fork)
- The `src/theme/` folder architecture

---

## King Prompt for template fork sub-chats

Paste this when starting a new sub-chat in this project for a template fork app:

```
New app sub-chat — TEMPLATE FORK. Before doing anything else, run the
kickoff interview.

- App name / working slug:
- Purpose:
- Stack Profile: Local (template default)
- Current stage: 0
- Repo: forked from design template (use TEMPLATE_FORK.md, NOT START_NEW.md)
- Notes:
```
