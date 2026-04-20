# PROCESS_GUIDE.md — Expo App Pipeline, Stages 0–16

> **Purpose:** one linear path you run for every app. Find your stage. Do only that stage. Move on. Designed to be run 15× without surprises.
>
> **Golden rules**
> 1. Read context first → plan mode → phased builds → self-check → next phase
> 2. Anything out of scope → `LATER.md`, never the current build
> 3. One agent per repo. 3–4 repos in parallel max. Never two agents in one repo.
> 4. Every stage has a skip condition. Read it before starting.
> 5. Companion files: `PROMPTS.md` (copy-paste prompts), `BUNDLE_IDS.md` (tracker), `CLAUDE.md` (per-repo orientation), `EXPO_SKILLS.md` (official skills reference)

---

## Status legend (for the spreadsheet)

`✅ done` / `🔄 in progress` / `⏭ skipped (not applicable)` / `❌ blocked`

---

# Stage 0 — Triage + Stack Profile (one-time, ~25 min for every app)

**Do this once, in a single sitting, before opening any repo.**

1. Open `BUNDLE_IDS.md`. For each app, fill in the **WORKING** columns:
   - Working name (whatever's in your head — can change)
   - Working slug (lowercase, hyphenated, e.g. `sip-less`)
   - One-line purpose (vague is fine)
   - Status: WORKING

2. **Pick a Stack Profile per app.** Read `STACK_PROFILES.md` and decide:
   - **Local** (default for most apps — solo trackers, journals, habit apps)
   - **Convex** (multi-device same user, leaderboards, OR real user accounts via Convex Auth — anonymous and account-based both supported)

   This decision LOCKS at Stage 0. Changing later costs days. Default to Local unless you have a clear "this needs cloud" reason.

3. Do NOT reserve bundle IDs in App Store Connect yet. ASC reservation is deferred to **Stage 11 per app**, when you've decided the final name. Reserving early forces premature naming decisions and creates dead bundle IDs you can't reuse.

4. Confirm `CREDENTIALS.md` has been completed once on your Mac. If not, do it now (~10 min). Without it, Stages 12 and 13 will fail.

**Out:** `BUNDLE_IDS.md` filled with working names + Stack Profiles. CREDENTIALS one-time setup done. Ready to start Stage 1 on your highest-priority app.

> **Naming note:** don't reserve bundle IDs up front. ASC bundle IDs cannot be reused once reserved, and forcing final names at Stage 0 creates friction. Defer naming to Stage 11.

---

# Stage 1 — Repo Spin-up (~5 min/app)

> **Before doing anything else in this stage, run `BOOTSTRAP.md` to verify the repo is set up correctly.** If anything fails, fix it before proceeding to Stage 2. Re-run BOOTSTRAP at the end of this stage to confirm the green light.

### NEW app:
1. GitHub Desktop → New Repo → name = slug from Stage 0
2. Open in Cursor / VS Code
3. Use the `START_NEW.md` prompt — it handles Expo init + kit unzip + CLAUDE.md fill in one go
4. Run `npx expo start` → confirm green QR code, scan with Expo Go, confirm boots

### EXISTING app:
1. Open repo in Cursor
2. Unzip your **mobile drop-in kit** into the repo root (skip files that already exist if you've customized them)
3. Open `CLAUDE.md`, fill in the header (Name, Purpose, Slug, Bundle ID, Stack Profile, current Stage)
4. Run `npx expo start` → confirm boots

**Out:** repo opens in Expo Go on your phone with no errors. `CLAUDE.md` filled. No `theme.ts` yet — that's Stage 3.

---

# Stage 2 — Onboarding Clone & Build (~60–90 min/app, NEW only)

**Skip if:** EXISTING app already has working onboarding screens. Run them in Expo Go to confirm, then skip to Stage 3 (or Stage 4 if Stage 3 also done).

This is the stage where you turn competitor screenshots into 3–4 pixel-perfect RN onboarding screens running in Expo Go. Colors and fonts are hardcoded inline at this stage — they get tokenized at Stage 3.

### 2a. Source the design (manual, ~15 min)
1. Take screenshots of 3–4 onboarding screens from a reference app (App Store screenshots, real app, etc.)
2. Go to **screendesigns.com** or **stitch.com**
3. Upload the screenshots, get HTML clones of each screen
4. Save the HTML + the original screenshots into a folder: `~/app-references/<slug>/`

### 2b. Build in RN (Cursor, ~30–60 min)
1. Open the repo in Cursor
2. Paste the prompt from `PROMPTS.md` Stage 2 with HTML + screenshots attached
3. Agent works through:
   - Phase 2b.1: shell + safe-area + nav stack + progress bar
   - Phase 2b.2: each screen one at a time (3–4 screens) — colors/fonts hardcoded
   - Phase 2b.3: back/next wiring + state persistence
4. Self-check after each phase

### 2c. Verify on device (the non-negotiable gate)
1. Terminal: `npx expo start` (LAN mode — default for home wifi)
2. QR code appears → scan with Expo Go on your phone
3. Walk through all 3–4 screens — they must match the references and respond to back/next without errors
4. **If anything is broken, do not proceed to Stage 3.** Restart Stage 2b with the fix.

**Out:** onboarding running on your phone via Expo Go, pixel-close to the references, no console errors. Hardcoded styles are expected at this stage.

---

# Stage 3 — Brand Lock / Tokenize (~20 min/app)

**Why this order:** Stage 2 builds the screens with hardcoded styles so you can see them on your phone first. Stage 3 extracts those styles into a single `theme.ts` and refactors the screens to import from it. From this point forward, every new screen imports tokens — no inline hex.

**Skip if:** EXISTING app and `src/theme/theme.ts` already exists and covers all the tokens needed.

**Skill:** None — the `design-consistency` skill is audit-only and runs at Stage 7 to catch drift. For initial token extraction, the Stage 3 prompt is self-contained.

**Process:**
1. Agent scans the 3–4 onboarding screens you just built
2. Extracts all hardcoded colors, fonts, spacing values into a proposed `src/theme/theme.ts`
3. Shows you 3 swatches so you can eyeball the palette
4. After approval: refactors every onboarding screen to import from `theme.ts`

**Theme file must contain:**
- Colors: `bg`, `surface`, `text`, `textMuted`, `primary`, `primaryOn`, `accent`, `border`, `success`, `danger`
- Typography: one display font, one body font, 6-step type scale
- Spacing scale: `4 / 8 / 12 / 16 / 24 / 32 / 48`
- Radii: `sm`, `md`, `lg`, `pill`
- One soft elevation/shadow token

**After tokenizing:** scan `src/screens/onboarding/` with `grep -rE "#[0-9a-fA-F]{6}"` — should return zero results outside `theme.ts`.

**Out:** `src/theme/theme.ts` exists with real tokens. All onboarding screens refactored to use it. Changing the brand color is now a one-line edit.

> **Roadmap (in LATER.md):** build a library of pre-made theme files (`female-soft.ts`, `female-bold.ts`, `masculine-dark.ts`, `masculine-clean.ts`, etc.) so future apps can skip the extraction step.

---

# Stage 4 — Onboarding Flow Hardening (~30–45 min/app)

**Skip if:** onboarding already covers the full flow (Quiz → Results → Custom Plan → Paywall) end to end.

Take the 3–4 screens from Stage 3 and extend them into the full conversion flow.

**Skill:** `building-native-ui` (Expo official) for building new screens, then `screen-wiring` (custom) to audit navigation between them.

**Prompt:** `PROMPTS.md` Stage 4

**Pre-work:** open `ONBOARDING_PATTERNS.md`. Pick 5–8 of the 12 patterns, ordered into an emotional arc for your specific app. Also pick the number of paywall variants (1/2/3). Fill these into the Stage 4 prompt's selection block before pasting.

**Canonical arcs are listed in `ONBOARDING_PATTERNS.md`** — start from one of those if you're not sure.

**Critical settings:**
- Paywall **visible but skippable** for now (skip link always works, RevenueCat hookup is in `LATER.md`)
- Quiz answers persist and feed personalization on ALL downstream screens (Results, Summary, Custom Plan, etc.)
- Top progress bar across the entire flow — remember to update `MAX_STEP` in OnboardingFlow to match new total screen count
- All images local (`assets/onboarding/`), no remote URIs
- Self-check after every phase

**Out:** complete onboarding flow running on Expo Go, paywall skip works, MainAppShell stub reachable.

---

# Stage 5 — Main App: Plan (~20-45 min/app)

**Skip if:** EXISTING app already has a working main app post-paywall.

**Skill:** `app-backend-builder`

**Process changes by Stack Profile:**

### If Local profile:
1. Skill reads onboarding, infers app, writes `BUILD_PLAN.md` with AsyncStorage data shapes
2. ~20 min

### If Convex profile (anonymous — no sign-up screens):
1. **Before invoking the skill**, run Convex setup. Read `convex-auth-setup.md` (in this kit) and execute the install + `npx convex dev` + EAS env var steps + `<ConvexAuthProvider>` wiring + `<AnonymousAutoSignIn />` mount.
2. Then invoke `app-backend-builder`. Skill reads onboarding, infers app, writes `BUILD_PLAN.md` with **Convex schema** instead of AsyncStorage shapes.
3. Records keyed by `userId` from `getAuthUserId(ctx)` — Convex Auth issues an anonymous user automatically on first device launch.
4. ~45 min total

### If Convex profile (with real accounts — sign-up/sign-in required):
1. **Before invoking the skill**, run Convex setup with the Password (or OAuth) provider enabled in `convex/auth.ts`. Read `convex-auth-setup.md` (in this kit) — covers anonymous AND account-based flows.
2. If using OAuth (Apple/Google), confirm Sign in with Apple is wired — App Store rejection if you offer Google/email without Apple.
3. Then invoke `app-backend-builder`. Skill reads onboarding, infers app, writes `BUILD_PLAN.md` with Convex schema using `userId` from Convex Auth identity + sign-up/sign-in screen requirements + account deletion screen (App Store mandatory for accounts).
4. ~75 min total

### If billing is in scope at Stage 5 (decision-time only):
- For mobile IAP: `ref-docs/integrations/revenuecat-expo.md` informs architecture (when present)
- For Next.js web payments: `ref-docs/integrations/stripe-convex.md` (direct Stripe + Convex webhooks, full control)

**Prompt:** `PROMPTS.md` Stage 5 (the prompt branches based on profile — the agent reads `CLAUDE.md` to know which profile to use)

**Out:** `BUILD_PLAN.md` exists in repo root with 3–6 phases. Provider wiring complete (if Convex). Schema designed (if Convex). Ready for Stage 6.

---

# Stage 6 — Main App: Build (~2–6 hrs/app, mostly agent time)

**Skip if:** EXISTING app already has a working main app post-paywall.

**Process:**
1. **Open a fresh Claude Code session** (clean context)
2. Paste prompt from `PROMPTS.md` Stage 6
3. Agent reads `BUILD_PLAN.md`, `CLAUDE.md`, `REF_DOCS_INDEX.md`, and the relevant ref-docs reference files (auto-determined from CLAUDE.md Stack Profile)
4. Agent enters plan mode, executes phase by phase
5. After every phase, agent invokes `self-check` skill
6. You check in between phases — review diff, approve, continue

**Reference files the agent reads (per profile):**
- Local: none beyond `BUILD_PLAN.md`
- Convex: `convex-auth-setup.md` + `convex-react-client.md` (both in this kit) for query/mutation patterns and auth-scoped queries; `~/Documents/GitHub/ref-docs/llms-txt/convex-full.txt` (when present) for deep API surface — load on demand only, it's 42k lines

**If adding payments at Stage 6:**
- RevenueCat only: `ref-docs/integrations/revenuecat-expo.md` (when present)
- Superwall + RevenueCat: above + `ref-docs/integrations/superwall-revenuecat.md` (most detailed integration doc — covers `CustomPurchaseControllerProvider`, callback wiring, customer info syncing)
- Direct Stripe (web/Next.js): `ref-docs/integrations/stripe-nextjs.md`
- Stripe + Convex pipeline (webhooks): `ref-docs/integrations/stripe-convex.md`

**While this runs:** rotate to another repo. This is the longest stage and the easiest to parallelize.

**Out:** main app working. Onboarding → paywall (skipped or sign-in) → main app loop completes on Expo Go without crash.

---

# Stage 7 — Polish Audit (~90 min/app, two parts)

Stage 7 has two parts: design consistency audit, then AI-generated imagery replacement.

## Stage 7a — Design Audit (~20 min)

**Skill:** `design-consistency` (audit mode)

**Prompt:** `PROMPTS.md` Stage 7

**Audit checklist** (skill enforces):
- [ ] No hex/rgb colors outside `theme.ts`
- [ ] No spacing values off the `4/8/12/16/24/32/48` scale
- [ ] All touch targets ≥ 44pt
- [ ] All screens use `SafeAreaView` properly
- [ ] All async surfaces have loading states
- [ ] All lists have empty states
- [ ] Primary CTAs have haptic feedback
- [ ] No `console.log` left in shipped paths
- [ ] No TODO/FIXME comments in shipped paths

**Out:** audit returns zero criticals. Majors fixed or added to `LATER.md` with justification.

## Stage 7b — AI Imagery Replacement (~60-90 min)

**Doc:** `AI_IMAGERY.md` (full workflow, prompt library by app type, Gemini MCP patterns, app icon prompts)

**Tool:** Nano Banana (Gemini 3 Pro Image or Gemini 2.5 Flash Image) via the nanobanana MCP installed at system level.

**Why now:** placeholder gradients shipped through Stages 2-6 are functional but don't read "premium" on device. Real imagery is the biggest visual leap in the whole pipeline. It also means the Stage 12 EAS build has a real app icon, not the default Expo logo.

**Process:**
1. Read `AI_IMAGERY.md`
2. **Generate app icon FIRST** — `assets/icon.png`, 1024×1024 PNG, flat vector style (not photorealistic — icons need to read at 60px). Lock this before in-app imagery. See `AI_IMAGERY.md` "App icon prompts" section.
3. Pick the style anchor that matches your app type (fitness / wellness / tracking / productivity / etc.)
4. Generate ONE anchor image, save to `assets/_anchor.jpg` + prompt to `assets/_anchor.txt`
5. Bulk-generate all placeholder slots via nanobanana MCP in one Claude Code session
6. Agent swaps gradient placeholders for `require('../../assets/...')` references
7. Verify on device — iterate on any image that misses the anchor aesthetic
8. Add winning prompts back to `AI_IMAGERY.md` — it gets better every app

**Out:** app icon set + every gradient placeholder in `src/screens/` replaced with real imagery. Assets directory under 10MB total. Anchor + winning prompts saved for reuse next app.

---

# Stage 8a — Legal URLs Wiring (~5 min/app)

**What this does:** fills the legal templates in the repo and wires the three URLs into the app code. The applanding.co page itself is NOT created at this stage — that happens at Stage 8b after ASC metadata exists.

**Why the split:** applanding.co's builder pulls metadata from the App Store Connect record (icon, screenshots, name, description). That data doesn't exist until after Stage 14/15. So 8a wires the URLs into code now, and 8b builds the actual landing page later. Between 8a and 8b, the URLs will 404 — that's expected and not a bug.

### 8a workflow

The agent does this in two parts. You give it ONE input: the app's domain on applanding.co.

**Convention** — domain follows app slug:
- `pregnancytracker.applanding.co`
- `bloomandbreathe.applanding.co`
- `vetted.applanding.co`

URLs are always:
- `<domain>` → support/landing
- `<domain>/privacy` → privacy policy
- `<domain>/terms` → terms of use

You never give Claude the full URLs. The agent constructs them from the domain you provide.

**Step 1 — Fill legal templates in the repo.**

Templates live at `legal/privacy.template.md` and `legal/terms.template.md` in this kit. The agent fills placeholders ({{APP_NAME}}, {{CONTACT_EMAIL}}, {{LAST_UPDATED}}) using values from `CLAUDE.md` and outputs `legal/privacy.md` and `legal/terms.md` in the repo.

**Step 2 — Wire the three URLs.** The agent updates:
- `app.json` → `expo.extra.{supportUrl, privacyPolicyUrl, termsOfUseUrl}`
- Paywall screen → "Privacy Policy" and "Terms" links
- Settings screen (if exists) → same links

**There is no Step 3 at this stage.** The applanding.co page is created at Stage 8b after ASC metadata is filled.

**Out:** legal templates filled and committed. URLs wired into app code. The URLs themselves will 404 until Stage 8b — that's expected.

---

# Stage 8b — Landing Page Build (~10 min/app, AFTER Stage 15, BEFORE Stage 16)

**This is MANDATORY before Stage 16.** Apple reviewers tap the Privacy Policy and Terms links during review. If they 404, the app is rejected. The landing page MUST be live before you click "Submit for Review."

**Why it can't happen earlier:** applanding.co's page builder pulls metadata from the saved ASC record — app name, subtitle, description, icon, screenshots, category. That data doesn't exist until Stage 15 saves it to ASC. Stage 8b runs after Stage 15 is complete.

### 8b workflow (manual, ~10 min, no Claude prompt)

1. Go to https://applanding.co
2. Create a page using the agreed domain (`<slug>.applanding.co`)
3. Let the builder pull from the ASC record — it auto-composes the landing page from the saved metadata (icon, screenshots, name, description)
4. Open `legal/privacy.md` from the repo (filled at Stage 8a), paste contents into the `/privacy` sub-page
5. Open `legal/terms.md` from the repo, paste contents into the `/terms` sub-page
6. Verify all three URLs resolve in a browser:
   - `<slug>.applanding.co` → landing page with app info
   - `<slug>.applanding.co/privacy` → privacy policy
   - `<slug>.applanding.co/terms` → terms of use
7. If any URL doesn't resolve, fix it now. Do NOT proceed to Stage 16 with broken URLs.

**Out:** all three URLs live and resolving. Landing page matches ASC listing. Legal text pasted. Ready for Stage 16 submission.

---

# Stage 9 — Analytics + Sentry (~30 min/app)

**Bug-fixed from old "Step 7.6".** Analytics + crash reporting must be in the binary that goes to TestFlight (Stage 13), otherwise you can't smoke-test that events fire. Old order had this *after* TestFlight which made smoke-test impossible without rebuilding.

**Run all three prompts in one Claude Code session** (sequential, they don't conflict):

1. **Sentry** (`PROMPTS.md` Stage 9a) — invoke `sentry-setup` skill
2. **PostHog** (`PROMPTS.md` Stage 9b) — invoke `analytics-posthog` skill
3. **FunnelMob** — no React Native SDK exists yet. Add to `LATER.md`: *"Wire FunnelMob analytics when RN SDK ships — https://docs.funnelmob.com"*

**Minimum events firing through PostHog:**
- `app_opened`
- `onboarding_started`
- `quiz_answered` (with `question_id` + `answer`)
- `onboarding_completed`
- `paywall_shown`
- `paywall_skipped`
- `paywall_purchased` (stub, no real payment)

**Out:** Sentry capturing test crash. PostHog firing test event. Both visible in their dashboards.

---

# Stage 10 — Self-Check Gate (~10 min/app)

**Skill:** `self-check`

**Prompt:** `PROMPTS.md` Stage 10

**This is a hard gate.** If anything fails, fix it or move to `LATER.md` with explicit justification. Do not proceed with red flags into App Store stages.

**Checks:**
- Boots clean on Expo Go (no warnings, no red box)
- Onboarding → paywall → main app loop completes without crash
- Zero TypeScript errors (`npx tsc --noEmit`)
- No `console.log`, `TODO`, `FIXME` in shipped paths
- `LATER.md` exists and contains all parked work

**Out:** all checks pass.

---

# Stage 11 — App Store Audit + Final Naming Lock (~45 min/app)

**This is the naming-lock point.** Up to here you've been using working names. At Stage 11 you decide the final name, slug, and bundle ID — because the next stage (Stage 12 EAS build) requires them to be locked and registered in App Store Connect.

### 11a — Lock the final name (~15 min)

1. Decide the final name. If you've shown the working name to test users / friends, this is when their feedback gets incorporated.
2. Update the FINAL columns in `BUNDLE_IDS.md`:
   - Final name
   - Final slug (lowercase, hyphenated)
   - Final bundle ID (`com.astrum.<slug-no-hyphens>`)
   - Status: LOCKED
3. **Reserve the bundle ID in App Store Connect:**
   - appstoreconnect.apple.com → My Apps → ➕ → New App
   - Bundle ID: the final one
   - SKU: the final slug
   - Primary Language: English (US)
   - Platform: iOS
4. Note the **ascAppId** (10-digit number) ASC assigns. Save it in `BUNDLE_IDS.md` and `eas.json`.
5. Update the repo:
   - `app.json` → `expo.name`, `expo.slug`, `expo.ios.bundleIdentifier`
   - `app.json` → remove the `_naming_status` working-name comment
   - `CLAUDE.md` header
   - `eas.json` → `submit.production.ios.ascAppId`
6. Optionally rename the GitHub repo (Repository Settings → Rename).

### 11b — Run the audit

**Skill:** `app-store-approval` Phase 1

**Prompt:** `PROMPTS.md` Stage 11

**What it checks:**
- `Info.plist` has usage description strings for every permission used
- Paywall has visible Restore Purchases, Terms, Privacy links
- Account deletion path exists (if any auth, even stubbed)
- No placeholder / lorem / test content visible to user
- No broken links, no links to unfinished pages
- App icon set, splash screen set, no default Expo branding visible
- Bundle ID in `app.json` matches `BUNDLE_IDS.md` final column AND App Store Connect record
- Version + build number in `app.json` are bumped vs. last submission

Work through every ❌ item. Common fixes:
- Add account deletion screen
- Add `NSCameraUsageDescription`-style strings to `app.json` plugins
- Wire Privacy/Terms links to applanding.co URLs (should already be done in Stage 8a — the URLs will 404 until Stage 8b, that's expected at this stage, just verify the links are wired in code)

**Out:** name LOCKED in BUNDLE_IDS.md, bundle ID reserved in ASC, zero criticals from audit.

---

# Stage 12 — EAS Production Build (~15–25 min/app, mostly EAS server time)

**Skill:** `expo-publish` (custom) + `expo-deployment` (Expo official)

**Prompt:** `PROMPTS.md` Stage 12

**Process:**
1. Bump version + build number in `app.json` (confirm with you first)
2. Verify EAS project linked: `eas project:info`
3. Run: `eas build --platform ios --profile production`
4. Wait for build (~15–25 min). **Rotate to another repo's Stage 1.**
5. Check status: `eas build:list --platform ios --limit 5` or expo.dev dashboard

**Out:** build succeeded on EAS, `.ipa` artifact ready.

---

# Stage 13 — Submit to TestFlight + Smoke Test (~30 min/app)

**Doc:** `TESTFLIGHT.md` (full tester setup + troubleshooting)

**Submit:**
```bash
eas submit --platform ios --profile production
```

Wait 5–15 min for Apple processing. Check App Store Connect → TestFlight.

## 13a — One-time per-app ASC setup (~2 min, first build only)

If this is the first build for this app on TestFlight, you need to create the Internal Testing group. New apps do NOT auto-create this — each app starts with zero groups. (Services like Rork did this automatically via the ASC API; EAS does not.)

1. ASC → your app → **TestFlight** tab → left sidebar → **Internal Testing** → click **"+"**
2. Group name: `Team (Expo)` (standardize across every app in this project)
3. **Tick "Enable automatic distribution"** — critical, or future builds won't auto-link
4. Add yourself as tester (email)
5. For external testers: create `External Beta` group, add emails, Beta App Review runs on first build (~few hours)

See `TESTFLIGHT.md` for full tester setup, role guidance, and troubleshooting.

## 13b — Smoke test on real device (the part most people skip)

1. Install via TestFlight on your phone
2. Walk through onboarding → paywall → main app
3. Confirm Sentry shows session in dashboard
4. Confirm PostHog shows events firing
5. Tap Privacy Policy link in paywall → confirm it OPENS the URL in the browser (the page itself will 404 at this stage — that's expected, the landing page isn't built until Stage 8b after ASC metadata is saved). You're verifying the link is wired, not that the page exists.
6. Tap Terms link → same check (opens URL, 404 expected)
7. If any crashes, Sentry catches them — fix and rebuild

**Out:** app installed via TestFlight on real device, full loop works, analytics + crash reporting confirmed live. Internal group set up with auto-distribution — every future build for this app auto-invites testers, no per-build manual step.

---

# Stage 14 — Screenshots (~20 min/app)

**Doc:** `SCREENSHOT_WORKFLOW.md` (full setup procedure)

**Prereqs (one-time, already done on your Mac):** Pillow, SF Pro Display Black at `/Library/Fonts/`, Gemini MCP in `~/.claude/settings.json`, screenshot skill cloned to `~/.claude/skills/aso-appstore-screenshots`.

**Per-app:**
1. `npx expo start --ios` → Simulator opens
2. Navigate to 5–6 key screens, `Cmd+S` each one to save to Desktop
3. In Claude Code in repo: `/aso-appstore-screenshots`
4. Skill reads codebase → identifies 3–5 core benefits → pairs each screenshot with one → generates 3 versions per screenshot
5. Pick best of 3 per screen
6. Finals land in `screenshots/final/` at exact App Store dimensions (1290×2796px or 1320×2868px depending on device)

**Required minimum:** 3 screenshots. **Recommended:** 5–7.
**Suggested scenes:**
1. Hero / value prop
2. Quiz / personalization
3. Custom plan / main feature
4. Secondary feature
5. Social proof / reviews

**Out:** 5–7 final screenshots in `screenshots/final/` ready to upload.

---

# Stage 15 — ASC Metadata Generation (~15 min/app)

**Skill:** `app-store-approval` Phase 2

**Prompt:** `PROMPTS.md` Stage 15

**Output:** `asc-metadata.md` in repo root, structured in **exactly the order you'll click through ASC at Stage 16** — four top-level sections matching the four ASC pages:

1. **iOS App > Distribution** (the version page) — screenshots, promo text, description, keywords, URLs, version, copyright, app review info
2. **App Information** — name, subtitle, category, content rights, age rating, encryption, server notifications
3. **App Privacy** — privacy URL, data types collected, granular per-category fields
4. **Pricing and Availability** — price, base region, country list

The doc IS the paste-script. No table of contents, no summary at top. Manual-input items only appear above SECTION 1, clearly flagged.

**Defaults baked in (override in CLAUDE.md if needed):**
- Copyright: `© 2026 Leverage Ventures Limited Liability Company`
- Reviewer contact: Steve Harris / .voice / SteveHarris411@gmail.com
- Demo credentials placeholder for any Convex profile with accounts

**Out:** `asc-metadata.md` exists with all four sections populated in ASC navigation order, every field grounded in the actual codebase, manual-input items flagged at the top.

---

# Stage 16 — App Store Submission (~20 min/app, manual in ASC)

**No Claude prompt — this is clicks in App Store Connect.**

**Open `asc-metadata.md` and `screenshots/final/` from Stage 14/15. Work top-to-bottom through the doc — it's been ordered to match the ASC UI exactly.**

**Click order in ASC:**

- [ ] Open the app record created in Stage 0, create new version (matches `app.json` version)

**iOS App > Distribution (the version page) — `asc-metadata.md` SECTION 1:**
- [ ] Upload iPhone screenshots from `screenshots/final/iphone/`
- [ ] Upload iPad screenshots from `screenshots/final/ipad/`
- [ ] Promotional text
- [ ] Description
- [ ] Keywords
- [ ] Support URL
- [ ] Marketing URL
- [ ] Version number (auto-set from app.json)
- [ ] Copyright (`© 2026 Leverage Ventures Limited Liability Company` unless overridden)
- [ ] Build — select build from Stage 12 (now in TestFlight)
- [ ] App Review Information: sign-in required + demo credentials (if applicable) + contact info + notes for reviewer + attachments
- [ ] **CLICK SAVE**

**App Information — `asc-metadata.md` SECTION 2:**
- [ ] Name + Subtitle (Localizable Information)
- [ ] Primary + Secondary category
- [ ] Content rights
- [ ] Age rating questionnaire
- [ ] App encryption documentation (export compliance — auto-passes if `app.json` has `usesNonExemptEncryption: false`)
- [ ] App or server notifications — production + sandbox URLs (only if app has IAP)
- [ ] **CLICK SAVE**

**App Privacy — `asc-metadata.md` SECTION 3:**
- [ ] Privacy Policy URL
- [ ] Data Types Collected (top-level Yes/No per category)
- [ ] Per-category granular fields (Linking / Tracking / Purposes)
- [ ] **CLICK SAVE**

**Pricing and Availability (under Monetization) — `asc-metadata.md` SECTION 4:**
- [ ] Price schedule (Free / Tier 0)
- [ ] Base country or region
- [ ] App availability
- [ ] **CLICK SAVE**

**Pre-submit: verify Stage 8b landing page is live (MANDATORY)**
- [ ] Open `<slug>.applanding.co` in a browser → landing page loads with app info from ASC
- [ ] Open `<slug>.applanding.co/privacy` → privacy policy text displays
- [ ] Open `<slug>.applanding.co/terms` → terms of use text displays
- [ ] If ANY of these 404: STOP. Go back to Stage 8b. Do not submit with broken URLs — Apple will reject.

**Final:**
- [ ] Submit for Review

Apple review typically takes 24–48 hours. Status will read "Waiting for Review" → "In Review" → "Pending Developer Release" or "Approved".

**Out:** "Waiting for Review" 🎉. Open the next repo's Stage 1.

---

# Stage Map by App Status

| Stage | NEW (5 apps) | EXISTING with onboarding only | EXISTING with main app | EXISTING on TestFlight |
|-------|:-:|:-:|:-:|:-:|
| 0. Triage + bundle ID | ✅ | ✅ | ✅ | ✅ |
| 1. Repo spin-up | ✅ | ✅ kit only | ✅ kit only | ✅ kit only |
| 2. Onboarding clone & build | ✅ | ⏭ | ⏭ | ⏭ |
| 3. Brand lock / tokenize | ✅ | ⏭ if theme.ts good | ⏭ if theme.ts good | ⏭ if theme.ts good |
| 4. Onboarding hardening | ✅ | ✅ | ⏭ if full flow | ⏭ |
| 5. Main app plan | ✅ | ✅ | ⏭ | ⏭ |
| 6. Main app build | ✅ | ✅ | ⏭ | ⏭ |
| 7. Polish audit | ✅ | ✅ | ✅ | ✅ |
| 8. Landing page | ✅ | ✅ | ✅ | ✅ |
| 9. Analytics + Sentry | ✅ | ✅ | ✅ | ⏭ if already in build |
| 10. Self-check gate | ✅ | ✅ | ✅ | ✅ |
| 11. App Store audit | ✅ | ✅ | ✅ | ✅ |
| 12. EAS build | ✅ | ✅ | ✅ | ✅ if rebuild needed |
| 13. TestFlight + smoke test | ✅ | ✅ | ✅ | ⏭ if already there + verified |
| 14. Screenshots | ✅ | ✅ | ✅ | ✅ |
| 15. Metadata generation | ✅ | ✅ | ✅ | ✅ |
| 16. ASC submission | ✅ | ✅ | ✅ | ✅ |

---

# The 4-Slot Parallel Loop

Never sit waiting. While one agent works, you work on another.

```
Slot A: Repo 1 → Stage 6 (agent building main app, 2-4 hr) ← agent working
Slot B: Repo 2 → Stage 12 (EAS build running, 15-25 min)   ← EAS working
Slot C: Repo 3 → Stage 16 (you in ASC pasting metadata)    ← you working
Slot D: Repo 4 → Stage 3 (Cursor agent building screens)   ← agent working
```

Rotate when an agent finishes. Check in, review output, advance to next stage, set off again.

**Hard rules:**
- Never more than 4 things running at once
- Never two agents in the same repo
- Never start a new stage in a repo without confirming the previous stage's "Out:" condition is met

---

# Recovery: when an agent goes off-track

1. **Stop it.** Don't try to correct mid-session.
2. Open a fresh Claude Code window.
3. Paste:
   ```
   Read CLAUDE.md and PROCESS_GUIDE.md.
   We were on Stage <N>. Here's what happened: <one paragraph>.
   Enter plan mode. Confirm understanding before touching anything.
   ```
4. Fresh context = fresh eyes.

---

# Common blockers

| Problem | Fix |
|---------|-----|
| EAS build fails | `expo-publish` skill → check error table |
| App Store rejects | Re-run `app-store-approval` Phase 1 — something was missed |
| Agent breaks working code | `git stash` or revert, restart with tighter scope |
| Paywall showing in app | `app-backend-builder` handles bypass — check `App.tsx` nav |
| Screenshots wrong size | Use `screenshots/final/` only, ignore working files |
| Expo Go won't connect | Try `--tunnel` flag instead of LAN |
| Stage 3 onboarding looks off vs reference | Re-do Stage 3a — the HTML clone wasn't faithful enough |
