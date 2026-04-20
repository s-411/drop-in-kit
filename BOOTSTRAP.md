# BOOTSTRAP.md — Per-Repo Setup Verification

> **For incoming agents: this is the FIRST file you check in any repo. Run every check below before doing any other work. If anything fails, STOP and tell the user what to fix — do not attempt to repair it yourself unless explicitly told to.**
>
> For Steven: run this checklist mentally (or have an agent run it) the first time you open any new or existing repo. ~5 minutes. Better to catch a missing skill now than to discover it mid-Stage 5.

---

## How to invoke

In a fresh Claude Code session in the repo:

```
Read BOOTSTRAP.md. Run every check in order. Report results as a
pass/fail table. Do not attempt fixes — just report. Stop after the
report.
```

---

## Section A — Drop-in kit files present

Every repo must contain these 15 docs at the root:

- [ ] `BOOTSTRAP.md` (this file)
- [ ] `CLAUDE.md`
- [ ] `PROCESS_GUIDE.md`
- [ ] `PROMPTS.md`
- [ ] `LATER.md`
- [ ] `EXPO_SKILLS.md`
- [ ] `APP_LANDING_PAGE.md`
- [ ] `SCREENSHOT_WORKFLOW.md`
- [ ] `TESTFLIGHT.md`
- [ ] `START_NEW.md`
- [ ] `CREDENTIALS.md`
- [ ] `STACK_PROFILES.md`
- [ ] `REF_DOCS_INDEX.md`
- [ ] `ONBOARDING_PATTERNS.md`
- [ ] `AI_IMAGERY.md`

**Verify command:**
```bash
ls BOOTSTRAP.md CLAUDE.md PROCESS_GUIDE.md PROMPTS.md LATER.md \
   EXPO_SKILLS.md APP_LANDING_PAGE.md SCREENSHOT_WORKFLOW.md \
   TESTFLIGHT.md START_NEW.md CREDENTIALS.md STACK_PROFILES.md \
   REF_DOCS_INDEX.md ONBOARDING_PATTERNS.md AI_IMAGERY.md \
   convex-auth-setup.md convex-react-client.md nextjs-bridge.md
ls legal/privacy.template.md legal/terms.template.md
```

If any are missing → refresh the kit from GitHub: `npx degit s-411/drop-in-kit --force` (run from the repo root).

> Note: technical reference docs (Convex, Sentry, RevenueCat, Stripe setup) live in `~/Documents/GitHub/ref-docs/` at apps root, NOT inside each repo. `REF_DOCS_INDEX.md` is the per-repo pointer that maps stages to ref-docs files. Auth setup for Convex Auth lives in `convex-auth-setup.md` here in the kit (not in ref-docs).

---

## Section B — Custom skills present at the GLOBAL Mac location

The 8 custom skills live at `~/.claude/skills/` (your Mac's home directory), NOT inside each repo. They're available to every Claude Code session on this Mac automatically.

Verify all 8 are present:

```bash
ls ~/.claude/skills/{design-consistency,screen-wiring,app-backend-builder,self-check,app-store-approval,expo-publish,sentry-setup,analytics-posthog}/SKILL.md
```

Should return 8 paths with no "No such file" errors.

**Also check that `react-native-patterns` is NOT present** (deprecated — replaced by Expo's official `building-native-ui`):
```bash
ls ~/.claude/skills/react-native-patterns 2>/dev/null && echo "DEPRECATED — DELETE THIS FOLDER" || echo "OK"
```

If skills are missing → re-copy them from your master kit folder into `~/.claude/skills/`.

---

## Section C — Official Expo skills installed (Mac-wide, not per-repo)

These install once on the Mac via Claude Code's plugin system. Verify in any Claude Code session:

```
/plugin
```

Expected output includes: `expo` listed as installed.

If not installed:
```
/plugin marketplace add expo/skills
/plugin install expo
```

---

## Section D — Mac-level tools installed (one-time)

These should already be set up from the first time you ran the kit. If you're on a fresh Mac, install them now.

### D1 — Expo / EAS CLI
```bash
node --version    # expect v20+ or v22+
npm --version
npx expo --version
eas --version
```

If `eas` not found:
```bash
npm install -g eas-cli
eas login
```

### D2 — Expo Go on your phone
- iOS: install "Expo Go" from the App Store
- You should be logged into the same Expo account as your Mac

### D3 — Apple Developer credentials
- Apple Developer account active at developer.apple.com
- Logged into Xcode at least once (for iOS signing certificates)
- App Store Connect access at appstoreconnect.apple.com

### D4 — Screenshot tool prereqs (for Stage 14)
```bash
ls /Library/Fonts/SF-Pro-Display-Black.otf       # font
pip3 show Pillow                                  # python lib
ls ~/.claude/skills/aso-appstore-screenshots     # screenshot skill
```

### D4.5 — Image generation prereqs (for Stage 7b)
```bash
which uvx                                        # should return a path
cd ~/Documents/GitHub/<this-repo>
claude mcp list | grep nanobanana                # should show Connected
```

If uvx missing: `curl -LsSf https://astral.sh/uv/install.sh | sh`
If nanobanana missing from this repo: see CREDENTIALS.md "Nanobanana MCP" section for install command. Note MCPs are per-repo in Claude Code 2.1.84+ — each new app needs the install.

### D5 — Git + GitHub Desktop
- Repo connected to GitHub
- `git status` returns clean (or you know what's uncommitted)

### D6 — Shell credentials loaded in shell

The `~/Documents/GitHub/.env-apps` file should be sourced by your `~/.zshrc` so EAS commands run without auth prompts.

```bash
echo "EXPO_TOKEN starts with: ${EXPO_TOKEN:0:6}..."
echo "EXPO_APPLE_TEAM_ID: $EXPO_APPLE_TEAM_ID"
eas whoami
```

Expected output:
- `EXPO_TOKEN starts with: exp_xx...` (not empty)
- `EXPO_APPLE_TEAM_ID: FUHV534M4K` (not empty)
- `eas whoami` prints your Expo username with no login prompt

If any are empty or `eas whoami` asks you to log in → `CREDENTIALS.md` walkthrough wasn't completed. Stop and do it now (~10 min).

### D7 — ref-docs library present at apps root (~/Documents/GitHub/)

The ref-docs library should be at `~/Documents/GitHub/ref-docs/`. Useful for any Convex / payment / Sentry work that goes deeper than what this kit covers — NOT required for Local profile apps, and NOT required for Convex Auth setup (that lives in `convex-auth-setup.md` here in the kit).

```bash
ls ~/Documents/GitHub/ref-docs/README.md
```

Expected: file path returned with no error.

If the directory doesn't exist:
- For Local profile apps: skip this check, you don't need ref-docs
- For Convex apps: not a hard blocker — `convex-auth-setup.md` and `convex-react-client.md` in this kit cover Stage 5/6 setup. ref-docs becomes useful when you need deeper API surface (e.g., complex Convex queries, full Stripe webhook patterns). Set up the stub at any point — see `ref-docs/README.md` (in the stub) for the protocol.

---

## Section E — Project-level setup

### E1 — Expo project initialized
```bash
ls package.json app.json
cat package.json | grep '"expo"'
```

If `package.json` doesn't exist or doesn't contain `expo` → run:
```bash
npx create-expo-app@latest . --template blank-typescript
```

### E2 — Theme file in correct location
```bash
ls src/theme/theme.ts
```

If not present → copy from `~/app-themes/<slug>-theme.ts` (Stage 1 output).

### E3 — `app.json` has correct bundle ID + export compliance flag
Open `app.json` and confirm:
```json
{
  "expo": {
    "ios": {
      "bundleIdentifier": "com.astrum.<slug>",
      "config": {
        "usesNonExemptEncryption": false
      }
    }
  }
}
```

Cross-check bundle ID against the `BUNDLE_IDS.md` row for this app (lives in your apps-root, not per-repo).

**The `usesNonExemptEncryption: false` flag is critical.** Without it, every TestFlight build gets held in "Missing Compliance" state until you manually answer the export compliance question in ASC. Baking it into `app.json` means every build auto-passes compliance — no manual step per build.

### E3.5 — App icon exists
```bash
ls assets/icon.png
```

If no icon exists yet, that's fine at Stage 2 — icon generation happens at Stage 7b with Gemini Nano Banana. But `assets/icon.png` should exist before Stage 12 (EAS build) or the resulting TestFlight build looks unprofessional and may confuse reviewers.

### E4 — `eas.json` has production profile
```bash
cat eas.json | grep -q '"production"' && echo "OK" || echo "MISSING production profile"
```

If missing:
```bash
eas init
eas build:configure
```

### E5 — `CLAUDE.md` header filled
Open `CLAUDE.md` and confirm the top section has real values, not placeholders:
- Name (not `<APP NAME>`)
- Purpose (not `<ONE SENTENCE…>`)
- Slug
- Bundle ID
- Current stage
- Status

---

## Section F — Sanity boot test

```bash
npx expo start
```

Expected: green QR code appears, no errors. Scan with Expo Go on your phone, app loads (or shows the default Expo template if it's a fresh repo at Stage 2).

If this fails, **do not proceed past Stage 2.** Common fixes:
- `rm -rf node_modules && npm install`
- `npx expo install --fix`
- Check `app.json` for syntax errors

---

## Output format

When an agent runs this checklist, expected output:

```
BOOTSTRAP CHECK — <slug> — <date>

Section A — Kit files:        8/8 ✅
Section B — Custom skills:    7/8 ❌  (analytics-posthog missing)
Section C — Expo plugin:      ✅
Section D — Mac tools:        ✅
Section E — Project setup:    4/5 ❌  (eas.json missing production profile)
Section F — Boot test:        ✅

BLOCKERS (must fix before Stage 3):
  - Refresh drop-in kit via `npx degit s-411/drop-in-kit --force` to restore .claude/skills/analytics-posthog/
  - Run `eas build:configure` to add production profile to eas.json

NON-BLOCKERS (note for later):
  - None
```

If the report shows blockers, fix them and re-run BOOTSTRAP. Only proceed to Stage 3 once everything is ✅.

---

## When to re-run this checklist

- **Always:** when starting work on a repo for the first time
- **Always:** after pulling a new version of the drop-in kit
- **Always:** before Stage 12 (EAS build) — catches missing `eas.json` config that wastes a build
- **Sometimes:** if an agent reports "skill not found" or "command not found" — re-run BOOTSTRAP before debugging
