# PROMPTS.md — Copy/Paste Library, Indexed by Stage

> Paired with `PROCESS_GUIDE.md`. Find your stage, grab the prompt, paste, ship.
> Replace `<angle bracket>` placeholders before sending.

---

## Stage 0 — Triage + Reservations

No Claude prompt — manual setup. Open `BUNDLE_IDS.md`, fill it for every app, then create app records in App Store Connect for each.

---

## Stage 1 — Repo Spin-up

No Claude prompt — use `START_NEW.md` for NEW apps (paste-and-go), or manual unzip for EXISTING apps per `PROCESS_GUIDE.md` Stage 1.

---

## Stage 2 — Onboarding Clone & Build (NEW apps)

### 2a — Source the design (no Claude prompt, manual)
1. Take 3-4 screenshots of reference app onboarding
2. Upload to screendesigns.com or stitch.com → get HTML clones
3. Save HTML files into `app-references/html/` in the repo root
4. Save screenshot images into `app-references/images/` in the repo root

### 2b — Build in Cursor

Paste this into Cursor:

```
SKILLS: Use screen-cloner, building-native-ui, and design-consistency skills
for this work.

Read the HTML designs in app-references/html/ and the screenshot images in
app-references/images/ (both relative to repo root). Build React Native
onboarding screens with Expo, replicating the HTML designs exactly —
pixel-perfect copy of layout, spacing, icons, and structure.

Context: src/theme/theme.ts does NOT yet exist. Hardcoded inline styles are
EXPECTED at this stage. Tokenization happens at Stage 3, which will extract
every color/font/spacing value you use now into theme.ts and refactor these
screens to import from it.

RULES FOR THIS STAGE:
1. Hardcoded hex colors inline — fine
2. Hardcoded font names inline — fine
3. Hardcoded spacing values inline — fine
4. BUT: use the same hex/font/spacing for the same semantic role across all
   screens. e.g. don't use #FF6B35 on screen 1 and #FF6B33 on screen 2
   for the same "primary" button. Watch for near-duplicates like
   #121417 vs #0f1115 for bg — pick ONE value per role and reuse it.
   Consistency makes Stage 3 tokenization clean.
5. Use mock data only. No real API calls.

ASSET HANDLING (do this, do not skip):
The HTML mocks will reference external image URLs (screensdesign.com,
unsplash, stock sites, etc). These MUST NOT be hotlinked in the final code
— hotlinks break randomly, slow app boot, and fail offline.
- For EVERY external image URL in the HTML:
  a) Download it to assets/onboarding/<descriptive-name>.<ext>
     e.g. assets/onboarding/welcome-bg.png, assets/onboarding/quiz-hero.jpg
  b) Reference it via require('../../assets/onboarding/welcome-bg.png')
     — NOT via source={{ uri: 'https://...' }}
- Create assets/onboarding/ if it doesn't exist.
- If a URL 404s or can't be downloaded, STOP and tell me — do not silently
  substitute a placeholder.
- Use `curl -L -o <path> <url>` in bash to download.

NAVIGATION COMPLETENESS:
Every screen in the flow must be reachable AND exitable. Before marking any
screen done, verify:
- There is a way to advance from this screen to the next (button, CTA, form
  submit — not just a header "next" that doesn't exist).
- There is a way to go back from this screen (the OnboardingHeader back
  button counts, except on Screen 1 which is the entry point).
If a screen in the HTML mock is ambiguous about how to advance (e.g. a long
form with no obvious submit button), add a "Next" CTA at the bottom. Tell
me what you added.

BUILD PHASES:

Phase 2b.1 — OnboardingShell
Build: safe-area wrapper, top progress bar, back/next nav stack, persisted
onboarding step state via AsyncStorage. Shared components only — no screens
yet.

CRITICAL SHELL RULES (these prevent two specific bugs that have bitten past
builds — bake them in from the start, do not rediscover them):

a) CLAMP THE STEP COUNTER.
   `goNext` must NOT increment the step past the last valid index. Define
   MAX_STEP = <number of screens - 1> and clamp:
       const goNext = () => setStep(s => Math.min(MAX_STEP, s + 1));
       const goBack = () => setStep(s => Math.max(0, s - 1));
   Without this clamp, tapping "next" on the final screen persists an
   out-of-range step to AsyncStorage, and on reload the flow renders an
   empty black screen (no step matches).

b) Z-INDEX ON STACKED ABSOLUTE LAYERS.
   If a screen stacks layers (e.g. background image → gradient overlay →
   content), the content layer MUST either:
   - also use `StyleSheet.absoluteFill` so it sits ON TOP of the gradient, OR
   - be wrapped with explicit `zIndex` / `elevation` higher than the gradient.
   A gradient that includes 'transparent' stops does NOT let touches or
   visibility pass through to layers beneath it in React Native — it
   visually blocks content. This is counter-intuitive coming from web CSS.

Shared components only — no screens yet.
→ RUN /skill self-check on this phase.
→ STOP. Show me the diff and self-check result. Wait for my OK to continue.

Phase 2b.2 — Build each screen, ONE AT A TIME
For each screen, in order:
  a) Download any external images this screen uses (per ASSET HANDLING above).
  b) Build the screen.
  c) Verify navigation completeness (per NAVIGATION COMPLETENESS above).
  d) RUN /skill self-check on this screen.
  e) STOP. Show me the diff and self-check result. Wait for my OK.
Do NOT batch screens. Do NOT build screen 2 before I approve screen 1.

Phase 2b.3 — Wire navigation + App.tsx
Hook the screens into OnboardingFlow, update App.tsx, persist step across
reloads.
→ RUN /skill self-check.
→ Run `npx tsc --noEmit` — must return zero errors.
→ Confirm no remote image URIs remain: `grep -r "uri: 'http" src/` returns
  zero matches.
→ STOP. Show me results before marking complete.

Final handoff (only after all phases self-checked and approved):
- Give me the EXACT terminal command: `npx expo start` (LAN — default for
  home wifi, do not use --tunnel).
- Tell me which screen I should see first when I scan the QR.
- Do NOT mark complete until all self-checks pass AND all images are local.

The HTML designs are at app-references/html/ and the screenshot images are
at app-references/images/ — read them from disk, no attachments needed.
```

### 2c — Verify on device (no Claude prompt)
Run the terminal command, scan QR, walk through screens. If broken, restart 2b with the fix.

---

## Stage 3 — Brand Lock / Tokenize

> Runs after Stage 2 so it can extract tokens from the real screens you've built. Creates `src/theme/theme.ts` and refactors the onboarding screens to import from it.

```
SKILLS: Use design-consistency skill for this work.

Read CLAUDE.md. We're at Stage 3 — tokenize the onboarding screens.

Step 1 — Extract
Scan every file in src/screens/onboarding/ and the OnboardingShell component.
List every unique value I've hardcoded:
- Every hex color (and which semantic role it plays: bg, text, primary CTA,
  accent, border, muted text, etc.)
- Every font family
- Every font size (and what it's used for: heading, body, label, caption)
- Every spacing value (and round off-scale values to nearest 4/8/12/16/24/32/48)
- Every border radius

Show me the list before writing any code.

Step 2 — Propose theme.ts
Based on the extracted values, propose a src/theme/theme.ts with:
- Colors: bg, surface, text, textMuted, primary, primaryOn, accent, border,
  success, danger
- Typography: display font + body font, 6-step scale (xs/sm/base/lg/xl/2xl)
- Spacing scale: 4 / 8 / 12 / 16 / 24 / 32 / 48
- Radii: sm, md, lg, pill
- One elevation shadow token

TypeScript, single export, fully typed.

Show me 3 swatches I can eyeball:
1. Primary CTA on bg
2. Surface card with text + textMuted
3. Accent badge on surface

Wait for my approval before step 3.

Step 3 — Refactor
After approval: write theme.ts to src/theme/theme.ts, then refactor every
onboarding screen + OnboardingShell to import tokens from theme. No inline
hex, no inline font families, no off-scale spacing.

Verify:
- `grep -rE "#[0-9a-fA-F]{6}" src/screens/onboarding/` returns ZERO matches
- `grep -rE "#[0-9a-fA-F]{6}" src/components/` returns ZERO matches (aside
  from any inside theme.ts itself)
- App still runs on Expo Go, screens look identical to before

Run /skill self-check before marking done.
```

---

## Stage 4 — Onboarding Flow Hardening

**BEFORE pasting the prompt:** open `ONBOARDING_PATTERNS.md`. Pick:
- Which patterns to include (5-8 of the 12, ordered into an emotional arc)
- How many paywall variants to ship (1 / 2 / 3)

Then fill the TWO lists at the top of the prompt and paste.

```
SKILLS: Use screen-wiring and building-native-ui skills for this work.

Read CLAUDE.md and ONBOARDING_PATTERNS.md. Also read the existing onboarding
screens in src/screens/onboarding.

# MY SELECTIONS (fill before pasting):

APP CONTEXT (1 sentence describing the audience emotionally):
<e.g. "Men 20-35 who follow fitness influencers, want to get jacked, are
 tired of bro-science and want a premium system they can trust.">

PATTERNS TO INCLUDE (in order, from ONBOARDING_PATTERNS.md menu):
1. <e.g. Trust Signals>
2. <e.g. Quiz>
3. <e.g. Reinforcement>
4. <e.g. Before/After>
5. <e.g. Social Proof>
6. <e.g. Results>
7. <e.g. Features>
8. <e.g. Custom Plan>

PAYWALL VARIANTS: <1 | 2 | 3>
<1 = main only; 2 = main + discounted fallback after skip;
 3 = annual shown first, monthly fallback, discounted last-ditch>

EXISTING SCREENS TO KEEP:
<e.g. Welcome, Sign Up, Gender Selection, Macro Targets>
<or: "replace all existing screens, start fresh from the patterns above">

# END SELECTIONS

---

Your job: extend the existing onboarding into the FULL conversion flow based
on my selections above.

Rules (non-negotiable):
- Every pattern I listed becomes ONE dedicated screen, in the order I gave.
- Use src/theme/theme.ts exclusively. No inline hex, fonts, or spacing.
- Reuse OnboardingShell + OnboardingHeader + GradientBorderButton from Stage 3.
- Top progress bar spans the entire flow (update MAX_STEP in OnboardingFlow
  to match new total screen count — clamp goNext to prevent overflow).
- All images local (download to assets/onboarding/ — NO remote URIs).
- Quiz answers persist to AsyncStorage under 'onboarding_quiz' key and are
  read by downstream personalization screens (Results, Summary, Custom Plan,
  etc.) to inject user-specific content.
- Paywall variants: all have VISIBLE skip link. Real payment hookup goes to
  LATER.md.
- After the final paywall (or skip), enter a minimal MainAppShell with basic
  bottom tab navigation (2-3 tabs, placeholder content). Real main app is
  Stage 6.
- Anything requiring real auth, real backend, real payments → LATER.md.

Build phases (STOP + self-check + my approval between each):

Phase 4.1 — Plan the arc
List every screen I'll get, in order, with:
- Pattern name from ONBOARDING_PATTERNS.md
- One-line purpose
- Key personalization data from the quiz (if applicable)
- Proposed screen filename
Do NOT write code yet. Show me the plan. Wait for my OK.

Phase 4.2 — Quiz screen + data layer
Build the Quiz screen(s) and the AsyncStorage data layer FIRST, because every
downstream screen depends on quiz data.
→ Run /skill self-check. STOP for approval.

Phase 4.3 — Build remaining screens, ONE AT A TIME
For each screen in the plan:
- Download any external images
- Build the screen, pulling quiz data from AsyncStorage for personalization
- Run /skill self-check
- STOP for my approval before moving on

Phase 4.4 — Paywall variant(s) + skip logic
Build the N paywall variants I specified. Wire skip logic:
- Variant 1 skip → Variant 2 (if exists) or MainAppShell
- Variant 2 skip → Variant 3 (if exists) or MainAppShell
- Variant 3 skip → MainAppShell
→ Run /skill self-check. STOP for approval.

Phase 4.5 — MainAppShell + wire final navigation
Build a minimal bottom-tab MainAppShell (2-3 tabs, placeholder content per tab).
Wire OnboardingFlow → MainAppShell after final paywall/skip.
Update MAX_STEP. Verify goNext clamps correctly.

Create and mount a global KeyboardHideButton component at the app root (after
the navigator, as the last child before StatusBar). Pill-shaped, themed to
match the app, floats just above any keyboard/number pad (right-aligned),
fires Keyboard.dismiss() on tap. One mount — covers every screen in the app.
Do NOT add per-screen. Zero render cost when keyboard is hidden.

→ Run /skill self-check. Run `npx tsc --noEmit`.
→ Verify: `grep -rE "#[0-9a-fA-F]{6}" src/` returns ZERO matches outside theme.ts.
→ Verify: `grep -r "uri: 'http" src/` returns ZERO matches.
→ STOP for final approval.

Final handoff:
- Terminal command: `npx expo start` (LAN mode)
- What I should see: <first screen name>
- Total screen count in the flow
- Which screens read from quiz data for personalization
```

---

## Stage 5 — Main App Plan (`app-backend-builder`)

```
SKILLS: Use app-backend-builder skill. If Stack Profile is Convex, also use
convex-quickstart and convex-setup-auth skills.

Read CLAUDE.md and REF_DOCS_INDEX.md. Note the Stack Profile in CLAUDE.md header
(Local / Convex).

If Stack Profile is Convex:
  - First read convex-auth-setup.md and convex-react-client.md (both at repo root)
  - Execute the Convex install + npx convex dev + EAS env var steps
  - Wire <ConvexAuthProvider> in _layout.tsx with SecureStore as the storage
    adapter (NOT AsyncStorage — auth tokens must be encrypted at rest)
  - Mount <AnonymousAutoSignIn /> for anonymous-only apps, OR set up the
    Password / OAuth provider in convex/auth.ts for account-based apps
  - For account-based apps with social login, confirm Sign in with Apple is
    wired (App Store rejection without it)
  - Confirm with me when setup is done. Do not proceed without my OK.

Then invoke the app-backend-builder skill.

Read every screen in src/screens/onboarding (especially the quiz questions and
custom plan screen). Infer:
- What the app actually does after the paywall
- Who the user is
- What core actions they perform daily/weekly
- What data needs to persist

Bypass the paywall for MVP — skip link always works.

Write BUILD_PLAN.md to repo root with:
- Inferred app purpose (one paragraph)
- Stack Profile (mirror from CLAUDE.md)
- Tab structure (2-4 tabs max)
- Primary screens per tab
- Data shape:
    - If Local: TypeScript types for AsyncStorage records
    - If Convex: Convex schema (defineSchema + authTables spread + indexes),
      with every user-scoped table keyed by `userId: v.id('users')` from
      Convex Auth identity
- Which onboarding answers feed which main-app surfaces
- Phased build plan (3-6 phases, each independently testable)
- For account-based Convex apps: include sign-up/sign-in screens and account
  deletion screen as explicit phases

Do NOT implement the main app yet — plan only. I will hand BUILD_PLAN.md to a
fresh agent in Stage 6.

When complete, return a summary of what you changed, files touched, and any
issues encountered.
```

---

## Stage 6 — Main App Build (fresh Claude Code session)

```
SKILLS: Use building-native-ui and native-data-fetching skills for all UI and
data work. If Stack Profile is Convex, also use the convex skill. Run
self-check skill at the end of every phase.

Read CLAUDE.md, BUILD_PLAN.md, and REF_DOCS_INDEX.md. Note the Stack Profile
in CLAUDE.md.

If Stack Profile is Convex: also read convex-auth-setup.md and
convex-react-client.md (both at repo root) before starting.

If BUILD_PLAN.md includes payments:
- RevenueCat only: read ~/Documents/GitHub/ref-docs/integrations/revenuecat-expo.md (when present)
- RevenueCat + Superwall: above + ~/Documents/GitHub/ref-docs/integrations/superwall-revenuecat.md

Enter plan mode.

Confirm:
- You understand the full scope of BUILD_PLAN.md
- You have any clarifying questions before starting
- You will pull all colors/fonts/spacing from src/theme/theme.ts only
- For Convex: every query/mutation will be properly indexed and scoped to the
  user via `getAuthUserId(ctx)` server-side. Never trust `userId` from a
  client arg.
- You will run /skill self-check at the end of every phase before continuing

Begin Phase 1. Show me the diff and self-check results before moving to Phase 2.
Do NOT add features not in BUILD_PLAN.md. Anything out of scope → LATER.md.

When complete with each phase, return a summary of what you changed, files
touched, and any issues encountered.
```

---

## Stage 7a — Polish Audit (`design-consistency --audit`)

```
SKILLS: Use design-consistency, self-check, and simplify skills for this work.

Read CLAUDE.md. Invoke design-consistency in audit mode.

Audit the entire codebase against src/theme/theme.ts. Flag every instance of:
- Hex/rgb/rgba colors not pulled from theme.ts
- Spacing values not on the 4/8/12/16/24/32/48 scale
- Font families not from theme.ts
- Touch targets under 44pt
- Screens missing SafeAreaView (or equivalent)
- Async surfaces missing loading states
- Lists missing empty states
- Primary CTAs missing haptic feedback (expo-haptics)
- console.log statements in shipped paths
- TODO/FIXME comments in shipped paths

Output as a fix list ranked: critical (breaks UX) → major (looks off) → minor
(polish). Do NOT auto-fix — I want to review first.
```

---

## Stage 7b — AI Imagery Replacement (Gemini MCP)

> Replaces every gradient placeholder in `src/screens/` with AI-generated imagery via Nano Banana (Gemini 3 Pro Image). Read `AI_IMAGERY.md` first — it has the full workflow, prompt library by app type, and all image specs. Run AFTER Stage 7a fixes are merged.

### Step 1 — Audit placeholders + pick style anchor

Paste this into a fresh Claude Code session in the repo:

```
SKILLS: Use the nanobanana MCP (Gemini) for all image generation in this stage.

Read AI_IMAGERY.md and CLAUDE.md.

Step 1: Audit all gradient placeholders in src/screens/. Run:
  grep -rn "LinearGradient" src/screens/ | grep -v "theme\."
Build a list of every placeholder slot that needs a real image.
Group by screen and asset type (hero, card bg, thumbnail, etc.).
Show me the full inventory.

Step 2: Based on the app's audience (read CLAUDE.md), recommend ONE
style anchor from AI_IMAGERY.md's prompt library (fitness, quit-habit,
wellness, tracking, productivity, food, content). Show me your
reasoning + the exact anchor prompt you'll use.

Step 3: Generate the anchor image using Gemini MCP. Save to
assets/_anchor.jpg. Save the prompt to assets/_anchor.txt for future
reference.

STOP after the anchor is generated. Show me the image inline. Wait
for my approval before bulk-generating.
```

### Step 2 — Bulk generate + wire in

After approving the anchor:

```
Anchor approved. Bulk-generate every image from the inventory.

For each placeholder slot:
1. Build the prompt: anchor + specific brief from AI_IMAGERY.md's
   prompt library for this app type
2. Call Gemini MCP, save to the path specified in AI_IMAGERY.md's
   image specs table
3. Compress to WebP if not already (target sizes per AI_IMAGERY.md)
4. Verify file exists and is under the size limit

After all images generated:
- Replace every LinearGradient placeholder in src/screens/ with
  Image source={require('../../assets/...')}
- Keep LinearGradient ONLY for text-overlay readability gradients
  on top of the new images
- Run /skill self-check
- Run `npx tsc --noEmit`
- Verify total assets/ size stays under 10MB
- Add winning prompts (the ones that worked first try) back to
  AI_IMAGERY.md under the relevant app-type section

Show me a summary report:
- Image count generated
- Total assets/ size
- Any prompts that needed iteration (note for AI_IMAGERY.md updates)
- Any slots that failed to generate (need manual intervention)
```

### Step 3 — Verify on device

Reload Expo Go. Walk every screen. Any image that misses the anchor
aesthetic — note the screen + slot, regenerate that one with a refined
prompt. First pass usually gets 80% right; the other 20% needs 1-2
iterations each.

---

## Stage 8a — Legal URLs Wiring (domain-only input)

You only need to give Claude the domain. Convention is `<slug>.applanding.co`. URLs are always `<domain>`, `<domain>/privacy`, `<domain>/terms`.

```
Read CLAUDE.md. We're at Stage 8a — wiring legal URLs.

Domain: <slug>.applanding.co

Step 1 — Fill legal templates.
- Read legal/privacy.template.md and legal/terms.template.md
- Replace placeholders using values from CLAUDE.md:
  {{APP_NAME}} → app's display name
  {{CONTACT_EMAIL}} → support email from CLAUDE.md (default to support@<slug>.com if not present, flag for me)
  {{LAST_UPDATED}} → today's date in "Month D, YYYY" format
  {{DOMAIN}} → the domain I gave above
- Write the filled output to legal/privacy.md and legal/terms.md
- Do not invent legal language not in the templates. If a placeholder is missing from CLAUDE.md, leave a clearly marked TODO and flag it.

Step 2 — Wire URLs (construct from domain, never ask me):
- supportUrl       = https://<domain>
- privacyPolicyUrl = https://<domain>/privacy
- termsOfUseUrl    = https://<domain>/terms

Update:
1. app.json under expo.extra:
   {
     "expo": {
       "extra": {
         "supportUrl": "...",
         "privacyPolicyUrl": "...",
         "termsOfUseUrl": "..."
       }
     }
   }

2. The paywall screen — add visible "Privacy Policy" and "Terms of Use" links
   that open the URLs in the system browser via Linking.openURL().
   Read URLs from Constants.expoConfig.extra so they stay in sync with app.json.

3. The settings screen if it exists — same two links, same source.

Use src/theme/theme.ts for all styling. Run /skill self-check when done.

When complete, return a summary of what you changed, files touched, and any
issues encountered (especially missing CLAUDE.md values).
```

**After the agent finishes:** commit the legal templates and URL wiring. The applanding.co page itself is NOT built at this stage — that happens at Stage 8b after ASC metadata exists. The URLs will 404 until then; that's expected.

---

## Stage 8b — Landing Page Build (manual, AFTER Stage 15, BEFORE Stage 16)

**MANDATORY.** Apple reviewers tap Privacy/Terms links during review. If they 404, the app is rejected. Do NOT submit at Stage 16 without completing this.

No Claude prompt — this is manual work on applanding.co (~10 min):

1. Go to https://applanding.co
2. Create a page using the agreed `<slug>.applanding.co` domain
3. Let the builder pull from the saved ASC record (it auto-composes from icon, screenshots, name, description)
4. Open `legal/privacy.md` from the repo, paste contents into the `/privacy` sub-page
5. Open `legal/terms.md`, paste into the `/terms` sub-page
6. Verify all three URLs resolve in a browser:
   - `<slug>.applanding.co` → landing page
   - `<slug>.applanding.co/privacy` → privacy policy
   - `<slug>.applanding.co/terms` → terms of use
7. If any 404: fix now. Do NOT proceed to Stage 16.

See `PROCESS_GUIDE.md` Stage 8b for full context on why this can't happen earlier.

---

## Stage 9a — Sentry (`sentry-setup`)

```
SKILLS: Use sentry-setup and sentry-react-native-sdk skills for this work.

Read CLAUDE.md. Invoke the sentry-setup skill.

Set up Sentry from scratch:
- Install @sentry/react-native and configure for Expo
- Add DSN to env (placeholder if I don't have one yet — flag this for me)
- Wire Sentry.init() in App.tsx with proper Expo integration
- Add a test crash trigger (long-press a hidden button on home screen, or a
  dev-only button) so I can confirm Sentry catches it post-TestFlight install

Run /skill self-check when done. Tell me explicitly:
- The Sentry DSN env var name to set
- The test crash trigger location and how to fire it
- What I should see in the Sentry dashboard when it works
```

## Stage 9b — PostHog (`analytics-posthog`)

```
SKILLS: Use analytics-posthog skill for this work.

Read CLAUDE.md. Invoke the analytics-posthog skill.

Set up PostHog from scratch:
- Install posthog-react-native
- Add PostHog API key to env (placeholder if I don't have one yet — flag this)
- Wire PostHog provider in App.tsx
- Fire these events at the right surfaces:
  - app_opened (App.tsx mount)
  - onboarding_started (first onboarding screen mount)
  - quiz_answered (with question_id + answer, on each quiz answer)
  - onboarding_completed (on paywall mount)
  - paywall_shown (paywall mount)
  - paywall_skipped (skip link tap)
  - paywall_purchased (stub — fires on tap of subscribe button, no real payment yet)

Use src/theme/theme.ts for any UI changes. Run /skill self-check when done.
Tell me the PostHog API key env var name and what I should see in the PostHog
dashboard when it works.
```

## Stage 9c — FunnelMob

No prompt — add to `LATER.md`:
```
- [ ] Wire FunnelMob analytics when React Native SDK ships — https://docs.funnelmob.com
```

---

## Stage 10 — Self-Check Gate (`self-check`)

```
SKILLS: Use self-check skill for this work.

Read CLAUDE.md. Invoke the self-check skill.

Run the FULL pre-App-Store self-check on this codebase:

1. Boots clean on Expo Go (no warnings, no red box, no console errors)
2. Onboarding → paywall → main app loop completes without crash
3. Zero TypeScript errors: run `npx tsc --noEmit` and report
4. No console.log, TODO, FIXME in shipped paths (flag every instance)
5. LATER.md exists at repo root and contains all parked work
6. src/theme/theme.ts is the single source of truth for colors/fonts/spacing
   (cross-check with `grep -rE "#[0-9a-fA-F]{6}" src/` — should return nothing
   except inside theme.ts)
7. Sentry initialized and test crash trigger exists
8. PostHog initialized and core events wired
9. applanding.co URLs wired into app.json AND paywall AND settings
10. Bundle ID in app.json matches BUNDLE_IDS.md entry

Output pass/fail per check. Do NOT proceed to Stage 11 until all 10 pass or
I explicitly waive a specific check with justification.
```

---

## Stage 11 — App Store Audit (`app-store-approval` Phase 1)

```
SKILLS: Use app-store-approval skill (Phase 1). If Stack Profile is Convex,
also use convex-performance-audit to check query performance before submission.

Read CLAUDE.md. Invoke the app-store-approval skill, Phase 1 only — the
rejection risk audit.

Do NOT touch any code yet. Audit and report only.

Check:
- Info.plist (via app.json plugins) has usage description strings for every
  permission used by any installed package
- Paywall has VISIBLE Restore Purchases, Privacy Policy, and Terms links
- Account deletion path exists if there's any auth (even stubbed)
- No placeholder / lorem / test content visible to user
- No broken links, no links to unfinished or missing pages
- App icon set (not default Expo)
- Splash screen set (not default Expo)
- No default Expo branding visible anywhere
- Bundle ID in app.json matches BUNDLE_IDS.md AND App Store Connect record
- Version + build number in app.json are bumped vs. last submission (or set
  to 1.0.0 / 1 for first submission)
- Privacy/Terms links in paywall actually open the applanding.co URLs

Output a checklist ranked critical → major → minor.

After I review, I'll tell you to fix the critical and major items in priority
order, one at a time.
```

After review, follow up with:

```
Work through every CRITICAL and MAJOR item from the audit, in priority order.
After each fix, confirm what you changed and why. When all critical+major are
resolved, run /skill self-check and report results before I confirm proceeding
to Stage 12.
```

---

## Stage 12 — EAS Production Build (`expo-publish`)

```
SKILLS: Use expo-publish and expo-deployment skills for this work.

Read CLAUDE.md. Invoke the expo-publish skill.

Pre-build checklist (do not start the build until all pass):
1. Confirm version + buildNumber in app.json — propose a bump and wait for my
   approval before changing
2. Confirm EAS project linked: run `eas project:info` and report
3. Confirm `eas.json` has a "production" profile for iOS
4. Confirm bundle ID matches BUNDLE_IDS.md entry
5. Confirm there are no uncommitted changes (run `git status`, report)

Once all pre-checks pass and I confirm the version bump, run:
  eas build --platform ios --profile production

Tell me:
- The build URL on expo.dev
- Estimated completion time
- The exact `eas submit` command to run after the build completes
```

---

## Stage 13 — Submit + Smoke Test

```bash
# Once EAS build succeeded:
eas submit --platform ios --profile production
```

After Apple processing (~5-15 min), install via TestFlight on your phone, then walk through:
- Onboarding → paywall → main app
- Test crash trigger (Sentry)
- Confirm PostHog events firing in dashboard
- Tap Privacy Policy and Terms links

If anything fails, fix and rebuild (back to Stage 12). No Claude prompt for this stage — it's manual verification.

---

## Stage 14 — Screenshots

**Before pasting the prompt below**, you do two manual things in ~30 seconds:

1. **Read the ASC record's slot dimensions.** Open App Store Connect → this app → App Store tab → current version → App Previews and Screenshots. Note the iPhone slot label AND the iPad slot label.
2. **Grab 5–6 simulator screenshots.** `npx expo start --ios`, navigate to key screens, `Cmd+S` on each. They land in `~/Desktop`.

Then paste the prompt below into Claude Code, with the two `<fill>` lines replaced. Do not remove or soften any of the `MANDATORY` blocks — they exist because agents skip them otherwise.

```
SKILLS: Use aso-appstore-screenshots skill. Use nanobanana MCP (Gemini) for
all outpainting. Use self-check when reviewing final output.

Invoke the aso-appstore-screenshots skill for this app's Stage 14.

══════════════════════════════════════════════════════════════════
TARGET DIMENSIONS — fill from the ASC record (do not guess, do not
use the skill's defaults):

  iPhone: <fill>×<fill>
  iPad:   <fill>×<fill>

Reference (pick the row matching the ASC slot label you saw):
  iPhone 6.9" Display  → 1320×2868
  iPhone 6.7" Display  → 1290×2796
  iPhone 6.5" Display  → 1242×2688  or  1284×2778  (grandfathered)
  iPad   13"           → 2064×2752
  iPad   12.9"         → 2048×2732

Grandfathered 6.5" ASC records SILENTLY REJECT 1320×2868 uploads.
If the user didn't fill the iPhone line above, STOP and ask them.
══════════════════════════════════════════════════════════════════

MANDATORY — BOTH DEVICE SETS

Generate both iPhone AND iPad screenshot sets. iPad is NOT optional.
If the skill tries to skip iPad, or you cannot produce iPad output,
STOP and surface the error. Do not mark Stage 14 complete without
both sets present.

══════════════════════════════════════════════════════════════════

MANDATORY — OUTPAINT, NEVER LETTERBOX

Any aspect-ratio mismatch between a source simulator screenshot and
the target output (most common: iPhone source → iPad output) MUST
be filled by OUTPAINTING via the nanobanana (Gemini) MCP. Extend
the scene naturally — backgrounds, gradients, UI chrome, surrounding
context — so the final looks native at the target size.

Forbidden fallbacks:
  ✗ Letterbox bars (black, white, theme-color, any solid fill)
  ✗ Stretch or squash the source
  ✗ Crop to fit (unless I explicitly ask for a specific screen)

If the nanobanana MCP is not available in this session, STOP and
tell me immediately. Do not proceed with any fallback. Verify with:
  claude mcp list | grep nanobanana
Should show: ✓ Connected

══════════════════════════════════════════════════════════════════

MANDATORY — VISUAL TREATMENT (polished indie studio, not Figma default)

The skill's default output is "phone frame + title text + flat
screenshot" — that's the floor, not the ceiling. Push every screen
toward the visual quality top apps ship. Apply these treatments
liberally, varying across the set so the 5–6 screens don't all look
the same:

  • Zoom/magnify key UI elements that demonstrate the benefit
    (e.g., enlarge a chart, pull out a calendar detail, blow up the
    primary CTA). Don't just show the screen — point at what matters.

  • Add subtle depth: drop shadows on the device frame, gradient
    backgrounds tuned to the app's theme palette, light glow on key
    elements. Backgrounds should feel designed, not stock.

  • Use callout annotations where they clarify: arrows, circles,
    spotlight/vignette effects drawing attention to the feature
    being highlighted. Use sparingly — one callout per screen max.

  • Vary compositions across the 5–6 screens. Not every screenshot
    should be "centered phone + title above." Mix in: angled/tilted
    devices, partial crops showing just the key UI area, split-screen
    before/after, layered elements at different scales, two devices
    side-by-side showing different states.

  • The title/caption text should be punchy and benefit-led, not
    feature-led. "Know when to push harder" beats "Workout tracker."

Goal: when I scroll through final/iphone/ and final/ipad/, the set
reads as considered and premium — the work of someone who cares.
Flat skill-default output is a failure state, not success.

══════════════════════════════════════════════════════════════════

OUTPUT STRUCTURE

Finals must land in these exact paths (Stage 15's asc-metadata.md
reads them):

  screenshots/final/iphone/01-<benefit-slug>.png
  screenshots/final/iphone/02-<benefit-slug>.png
  ...
  screenshots/final/ipad/01-<benefit-slug>.png
  screenshots/final/ipad/02-<benefit-slug>.png
  ...

══════════════════════════════════════════════════════════════════

PROCESS

1. Read the codebase → identify 3–5 core benefits to headline
2. Pair each simulator screenshot with the best benefit
3. Generate 3 variants per screen per device at the exact target
   dimensions above, applying the visual treatments above, using
   outpainting for any aspect-ratio fill
4. Save all variants under screenshots/<benefit-slug>/ for my review
5. After I pick, copy my chosen variant into screenshots/final/iphone/
   and screenshots/final/ipad/ under matching filenames

When complete, return a summary of:
  - Number of screens per device (must be equal)
  - Confirmation outpainting was used for every aspect mismatch
  - Confirmation zero letterboxed output
  - Which visual treatments were applied to which screens
  - Final output paths

Terminal commands you produce must assume the terminal is already
in the repo root — never prefix with cd <repo>. cd is only
acceptable to move into a subdirectory of this repo.
```

After the agent finishes: you walk through `screenshots/<benefit-slug>/` folders and pick the strongest of 3 per screen. Your picks get promoted to `screenshots/final/iphone/` and `screenshots/final/ipad/` — those are what Stage 16 uploads to ASC.

For the full reference on ASC slot eras, outpaint policy rationale, and troubleshooting, see `SCREENSHOT_WORKFLOW.md`. But you do not need to read it to run Stage 14 — the prompt above is self-contained.

---

## Stage 15 — ASC Metadata (`app-store-approval` Phase 2)

The output of this stage is `asc-metadata.md` in the repo root. **It MUST be ordered exactly the way the App Store Connect UI is laid out**, so you can paste field-by-field top-to-bottom without scrolling around the doc. The skill is responsible for enforcing that order in the file it writes.

```
SKILLS: Use app-store-approval skill (Phase 2) for this work.

Read CLAUDE.md. Note the Stack Profile in the header. Invoke the
app-store-approval skill, Phase 2.

Generate full App Store Connect metadata for this app and save to
asc-metadata.md in repo root.

CRITICAL — output ordering:
The asc-metadata.md MUST follow the ASC UI navigation order EXACTLY, top to
bottom, so I can paste field-by-field as I scroll the doc. Use these four
top-level sections in this order, with sub-fields inside each in the order
listed below. Do not reorder, do not group differently, do not add a generic
"summary" at the top — the doc is a paste-script, not a report.

================================================================
SECTION 1 — iOS App > Distribution (the main version page)
================================================================

  1.1  Screenshots — paths in repo
       - iPhone screenshots folder: screenshots/final/iphone/
         Required ASC iPhone sizes (one set is enough — pick the device size
         the user actually targeted at Stage 14):
           6.7" Display:  1290 × 2796 (portrait) or 2796 × 1290 (landscape)
           6.5" Display:  1242 × 2688 (portrait) or 2688 × 1242 (landscape)
           6.5" alt:      1284 × 2778 (portrait) or 2778 × 1284 (landscape)
       - iPad screenshots folder: screenshots/final/ipad/
         Required ASC iPad sizes:
           13":           2064 × 2752 (portrait) or 2752 × 2064 (landscape)
           12.9":         2048 × 2732 (portrait) or 2732 × 2048 (landscape)
       - Verify both folders exist and contain images at the right resolution.
         If either is missing or wrong size, STOP and tell me — I'll fix at
         Stage 14 before continuing.

  1.2  Promotional text (≤170 chars)
  1.3  Description (≤4000 chars: lead with value prop, then features, then trust)
  1.4  Keywords (≤100 chars total, comma-separated, NO spaces, singular forms)
  1.5  Support URL (from app.json supportUrl — set in Stage 8a)
  1.6  Marketing URL (Stage 8b branded landing page if it exists, else same as support URL)
  1.7  Version number (from app.json expo.version — should be 1.0.0 for v1)
  1.8  Copyright (default: © 2026 Leverage Ventures Limited Liability Company)
       — use exactly this string unless CLAUDE.md specifies a different entity

  1.9  App Review Information
       1.9.1  Sign-in required? (Yes/No)
              - Local profile: No
              - Convex anonymous: No
              - Convex with accounts: Yes
              If Yes, provide demo credentials:
                 Email:    review@<slug>.com  (or whatever's in CLAUDE.md)
                 Password: <generate or pull from CLAUDE.md>
              Flag both under MANUAL INPUT REQUIRED if not in CLAUDE.md.

       1.9.2  Contact information (defaults — use unless CLAUDE.md overrides)
                 First name:    Steve
                 Last name:     Harris
                 Phone number:  .voice
                 Email:         SteveHarris411@gmail.com

       1.9.3  Notes for reviewer
              Short paragraph explaining: what the app does, how to use it
              (any non-obvious flow), and (for paywalled apps) "subscription
              is purely cosmetic / unlocks X feature; you can dismiss the
              paywall to use the free version".

       1.9.4  Attachments
              List any attachments needed (screenshots showing how to bypass
              auth, video walk-through, etc.). Default: none.

  1.10  ⚠ CLICK SAVE before navigating away from this page.

================================================================
SECTION 2 — App Information
================================================================

  2.1  Localizable Information
       2.1.1  Name (≤30 chars — must match app.json expo.name)
       2.1.2  Subtitle (≤30 chars)

  2.2  Category
       2.2.1  Primary category
       2.2.2  Secondary category (optional but recommended for discoverability)

  2.3  Content rights
       Does this app contain, show, or access third-party content? Yes/No
       Default: No (unless app embeds licensed content like music, video, etc.)

  2.4  Age rating
       Run through the ASC questionnaire and provide answers for every question
       — Cartoon Violence, Realistic Violence, Sexual Content, Profanity,
       Alcohol/Tobacco/Drug References, Mature/Suggestive Themes, Horror/Fear,
       Medical/Treatment Info, Gambling, Contests, Unrestricted Web Access.
       For most apps in this kit: all "None" → 4+ rating.

  2.5  App encryption documentation (export compliance)
       The kit defaults app.json to ITSAppUsesNonExemptEncryption: false, so
       the answer is: "No, your app does not use encryption beyond what's
       exempt." Confirm app.json has this set; if not, flag it.

  2.6  App or server notifications (App Server Notifications URL for IAP)
       - For apps WITHOUT in-app purchases: leave blank, skip this field
       - For apps WITH in-app purchases (RevenueCat or direct StoreKit):
           Production server URL: <RevenueCat or your webhook endpoint>
           Sandbox server URL:    <RevenueCat sandbox or your sandbox endpoint>

  2.7  ⚠ CLICK SAVE before navigating away from this page.

================================================================
SECTION 3 — App Privacy
================================================================

  3.1  Privacy Policy URL (from app.json privacyPolicyUrl — set in Stage 8a)

  3.2  Data Types Collected — top-level YES/NO gate per category.
       The agent must check the ACTUAL codebase (Info.plist permissions,
       PostHog events, Sentry config, Convex schema, auth provider config)
       to answer accurately. Do NOT default to "collected" without grounds.

       Stack Profile decision matrix:

       Local profile:
           Most categories: Not Collected
           "Usage Data" → Collected (PostHog events, anonymous, not tracking)
           "Crash Data" + "Performance Data" → Collected (Sentry, anonymous)

       Convex profile, anonymous:
           Above, plus:
           "User Content" → Collected (linked to anonymous device ID, not tracking)
           "User ID" → Collected (Convex Auth anonymous user ID, not tracking)

       Convex profile, with accounts:
           Above, plus:
           "Email Address" → Collected (linked, not tracking unless ads)
           "Name" → Collected ONLY if sign-up form actually collects it
                    (check the schema and sign-up screen)
           If using Apple/Google OAuth: confirm Sign in with Apple is wired
           — App Store rejection if you offer social login without Apple.

  3.3  For each category answered YES in 3.2, fill in the granular sub-fields:
       - Linking: Linked to user / Not linked to user
       - Tracking: Used for tracking / Not used for tracking
         (Default for this kit: NOT used for tracking — we don't ship ad SDKs)
       - Purposes: App Functionality / Analytics / Product Personalization /
         Developer's Advertising or Marketing / Third-Party Advertising /
         Other (pick the minimum that's actually true)

       Output this as a per-category subsection so I can paste each cluster
       into the corresponding ASC card without hunting.

================================================================
SECTION 4 — Pricing and Availability (under Monetization)
================================================================

  4.1  Price schedule
       - Free / Tier 0 unless this app has a non-IAP paid download (very rare)
       - For IAP-only apps (default for this kit): Free at the app level;
         IAP prices live in the In-App Purchases section, not here.

  4.2  Base country or region
       Default: United States (unless CLAUDE.md says otherwise).

  4.3  App availability
       Default: All countries and regions where the App Store operates.
       Flag if there are any reasons to exclude specific countries
       (regulatory, language, content) — usually none for this kit.

================================================================
END OF asc-metadata.md REQUIRED ORDERING
================================================================

Other ground rules:

- Read the ACTUAL codebase to ground every claim. No invented features. No
  aspirational copy about things that aren't built.
- If anything requires manual input from me, put it under a clearly marked
  "MANUAL INPUT REQUIRED" subsection at the very top of asc-metadata.md
  (before SECTION 1) so I see it before I start pasting.
- Do not create a "summary" or "table of contents" at the top — the doc IS
  the paste-script.
- Verify every URL referenced (supportUrl, privacyPolicyUrl) is actually
  present in app.json before claiming it. If missing, STOP and tell me.

When complete, return a summary of: which sections were fully populated,
which fields needed manual input, and any inconsistencies found between
CLAUDE.md / app.json / actual codebase.
```

---

## Stage 16 — ASC Submission

No Claude prompt — manual in App Store Connect. Use `asc-metadata.md` from Stage 15 and `screenshots/final/` from Stage 14.

---

# Recovery Prompts (use anytime)

### Get unstuck mid-build
```
Stop. Read CLAUDE.md, PROCESS_GUIDE.md, and BUILD_PLAN.md (if it exists).
Tell me what stage I'm on, what's left in the current phase, and what the
next single action is. Do NOT write code.
```

### Force a self-check before claiming done
```
Before you mark this phase complete, invoke /skill self-check against the
work you just did. Report results. Only mark complete if it passes.
```

### Park scope creep
```
This is out of scope for the current stage. Add it to LATER.md under the
appropriate section with one sentence of context, then return to the current
stage's defined work.
```

### Agent went off-track
```
Stop. Do not make any more changes. Run /skill self-check and tell me exactly
what state the codebase is in. List every file you touched in this session
and what you changed.
```

### Starting fresh after a bad session
```
Read CLAUDE.md. Here's where we got to: <one paragraph of what was done>.
We need to continue from Stage <N>. Enter plan mode and confirm understanding
before touching anything.
```

### EAS build failing
```
SKILLS: Use expo-publish and expo-deployment skills for this work.

Read CLAUDE.md. Invoke the expo-publish skill.
The build is failing with this error: <paste full error>
Check the troubleshooting table and tell me the fix BEFORE applying it.
```

### App Store rejection received
```
SKILLS: Use app-store-approval skill (Phase 1) for this work.

Read CLAUDE.md. Invoke the app-store-approval skill, Phase 1.
We received a rejection. Apple's reason: <paste rejection reason and any
guideline references>
Find where this issue is in the codebase. Tell me the fix BEFORE applying it.
```
