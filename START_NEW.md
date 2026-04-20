# START_NEW — Brand New App Bootstrap

> **What this is:** a single prompt you paste into Claude Code to take an empty repo from zero to fully set up and ready for Stage 2 (Onboarding Clone). All in one go.
>
> **When to use:** every time you create a new app repo. Handles everything in `PROCESS_GUIDE.md` Stage 1 NEW path.
>
> **When NOT to use:** for an existing repo that already has code. For those, run `npx degit s-411/drop-in-kit --force` from the repo root, then run `BOOTSTRAP.md`.

---

## Pre-flight (do these first, takes 30 sec)

1. **One-time setup done?** If you haven't done `CREDENTIALS.md` yet, do that first. Without it, `eas build` and `eas submit` will fail later. ~30 minutes one-time.

2. **Drop-in kit is pulled from GitHub** — no zip file to maintain. The kit lives at https://github.com/s-411/drop-in-kit and `npx degit` downloads the current state straight into the repo. Requires no auth (repo is public) and no git history pollution (`degit` strips it).

3. **Pick a working slug** — whatever pops in your head. `gym-template`, `quit-drinking-thing`, `fitness-clone`. **It does not have to be final.** You can rename the repo, change the bundle ID, and rebrand the app any time before Stage 11. The only naming that's permanent is what you reserve in App Store Connect at Stage 12.

4. **GitHub Desktop -> File -> New Repository:**
   - Name: working slug from above
   - Local path: wherever you keep repos
   - **IMPORTANT: Uncheck "Initialize this repository with a README"** — leave it completely empty. (If you forgot, the prompt handles it.)
   - Click Create Repository

5. **Open the new (empty) repo folder in Cursor / VS Code.**

6. **Open Claude Code in the integrated chat panel.**

---

## The Prompt

Fill in the **2 required lines** at the top. The rest have safe defaults.

```
START NEW APP

# REQUIRED — fill these in:
WORKING_SLUG: <e.g. gym-template — does NOT have to be final>
PURPOSE: <one sentence about what the app does; can be vague>
STACK_PROFILE: <Local | Convex — see STACK_PROFILES.md>

# OPTIONAL — these have safe defaults:
DISPLAY_NAME: <leave blank to default to title-cased WORKING_SLUG>
WORKING_BUNDLE: <leave blank to default to com.astrum.<slug-without-hyphens>>

# DO NOT WORRY ABOUT FINAL NAMING. The bundle ID is a placeholder.
# Final naming and ASC reservation happen at Stage 11. Rename freely until then.
# DO NOT change STACK_PROFILE later — it's locked. Read STACK_PROFILES.md before
# answering. Default to Local unless you have a clear "needs accounts" reason.

Execute the following steps in order. After each step, briefly confirm what
happened. STOP and ask me if anything fails or seems off — do not improvise
fixes silently.

1. Print pwd. Confirm we are in the new empty repo (folder name should roughly
   match WORKING_SLUG). If we are somewhere weird, STOP.

2. Clear any GitHub Desktop starter files so create-expo-app has a clean directory:
     rm -f README.md .gitignore LICENSE
   (these are safe to remove — Expo will regenerate the right .gitignore)

3. Initialize the Expo project:
     npx create-expo-app@latest . --template blank-typescript --yes
   This takes ~60 seconds. When done, confirm package.json, app.json, and
   App.tsx exist.

3a. **Install the Nanobanana MCP for this repo (MANDATORY).** Claude Code 2.1.84+ scopes MCPs per-project, so this must be re-run on every new repo. Used throughout the build for in-app imagery AND for App Store screenshots — not just at Stage 7b.
     claude mcp add nanobanana uvx nanobanana-mcp-server@latest -e GEMINI_API_KEY=$GEMINI_API_KEY
   Verify:
     claude mcp list | grep nanobanana
   Expected: `nanobanana: uvx nanobanana-mcp-server@latest - ✓ Connected`
   If $GEMINI_API_KEY is empty, STOP — `.env-apps` not sourced. See CREDENTIALS.md.
   **The MCP loads at session start.** After installing, exit this Claude Code session and restart it before continuing — otherwise nanobanana tools won't be visible to the agent.

4. Pull the drop-in kit from GitHub into the repo root:
     npx degit s-411/drop-in-kit --force
   This downloads the current state of the kit (no git history, no .git folder)
   and overwrites existing files in place. Re-run this same command any time
   to refresh to the latest kit state.
   Verify the kit files landed:
     ls BOOTSTRAP.md CLAUDE.md PROCESS_GUIDE.md PROMPTS.md EXPO_SKILLS.md \
        APP_LANDING_PAGE.md SCREENSHOT_WORKFLOW.md LATER.md START_NEW.md \
        CREDENTIALS.md STACK_PROFILES.md REF_DOCS_INDEX.md \
        ONBOARDING_PATTERNS.md AI_IMAGERY.md convex-auth-setup.md \
        convex-react-client.md nextjs-bridge.md TESTFLIGHT.md BUNDLE_IDS.md
     ls legal/privacy.template.md legal/terms.template.md
   If degit fails with "could not find commit", confirm the repo exists at
   https://github.com/s-411/drop-in-kit and is public.

5. Edit CLAUDE.md — fill in ONLY the header section. Set:
     - Name: DISPLAY_NAME (or title-cased WORKING_SLUG if blank)
     - Purpose: PURPOSE
     - Slug: WORKING_SLUG (working — final TBD at Stage 11)
     - Bundle ID: WORKING_BUNDLE (working — final TBD at Stage 11)
     - Stack Profile: STACK_PROFILE (LOCKED — do not change after Stage 0)
     - Current stage: 1
     - Status: NEW

6. Edit app.json:
   - Set expo.name to the display name above
   - Set expo.slug to WORKING_SLUG
   - Set expo.ios.bundleIdentifier to the working bundle ID
   - If expo.ios block does not exist, add it
   - **CRITICAL:** add the export compliance flag so TestFlight doesn't
     hold future builds in "Missing Compliance" state:
       "ios": {
         "bundleIdentifier": "com.astrum.<slug>",
         "config": {
           "usesNonExemptEncryption": false
         }
       }
     This bakes ITSAppUsesNonExemptEncryption: false into the Info.plist.
     Every EAS build will auto-pass export compliance with no manual step.
   Add this comment at the top of the expo block:
     "_naming_status": "Working name — finalize at Stage 11 before EAS build"

7. Set up eas.json with EAS-managed credentials:
   - Run: eas init
     (this creates a basic eas.json and links the project to your EAS account)
   - Then patch eas.json so the production submit profile knows your Apple Team ID.
     The submit.production.ios block should look like this:
       "submit": {
         "production": {
           "ios": {
             "appleTeamId": "FUHV534M4K",
             "ascAppId": "TBD-set-at-Stage-11"
           }
         }
       }
   - That's all that's needed locally. EAS server-side has the Distribution
     Cert, Push Key, and ASC API Key — the CLI pulls them from there at build
     and submit time.
   - If `eas init` asks for credentials or fails because EXPO_TOKEN is empty,
     STOP — CREDENTIALS.md walkthrough wasn't completed.

8. Make the initial git commit:
     git add -A
     git commit -m "Initial Expo + drop-in kit setup (working name)"

9. Run BOOTSTRAP.md verification. Read the file, work through Sections A, B,
   C, D1, E, F (skip D2-D5 — those are Mac-level checks done in CREDENTIALS).
   Report results in the exact table format at the bottom of BOOTSTRAP.md.

10. Final handoff:
    - Tell me the exact terminal command to run myself to start the dev
      server. Default to LAN mode (no tunnel flag): `npx expo start`.
      Only suggest `--tunnel` if I explicitly mention being on cellular
      or a different network than my Mac.
    - Tell me what I should see when I scan with Expo Go (the default Expo
      "Open up App.tsx to start working" screen).
    - Remind me that the bundle ID and app name are working values — they
      get finalized at Stage 11.
    - Confirm we are ready to begin Stage 2 (Onboarding Clone) once I have
      verified the boot test on my phone.

Do NOT begin Stage 2 work in this session. Stop after step 10.
```

---

## After the prompt finishes

1. **Run the dev server command** Claude gave you (`npx expo start` for LAN, which is the default).
2. **Scan the QR with Expo Go** on your phone. Confirm the default Expo welcome screen loads.
3. **If it boots:** Stage 1 is done. Open `PROMPTS.md`, copy the Stage 2 prompt, paste into the same chat. Onboarding clone begins.
4. **If it does not boot:** tell Claude in the chat. Don't proceed to Stage 2 until boot works.

---

## Stuck?

| Problem | Fix |
|---------|-----|
| `create-expo-app` fails | Directory wasn't empty. `ls -la` to see what's there, delete anything that isn't `.git`, retry. |
| `degit` fails | Confirm the kit repo at https://github.com/s-411/drop-in-kit is reachable in your browser. If you're offline or GitHub is down, the pull will fail — try again later. |
| `degit` command not found | You don't need to install it globally — `npx degit` ships with node. Confirm node is installed (`node -v`). |
| Step 7 says env vars not set | Complete `CREDENTIALS.md` first. ~30 min one-time. |
| CLAUDE.md edits look weird | Open it, manually fix the header. |
| Want to refresh kit in an existing repo mid-build | From the repo root: `npx degit s-411/drop-in-kit --force` — overwrites kit files, leaves your app code alone. |

---

## For EXISTING apps (repos that already have code)

Don't use this prompt — it's for fresh repos only. For existing apps, pull the current kit from the repo root:

```bash
npx degit s-411/drop-in-kit --force
```

Then open Claude Code and say:
```
This is <app name>, currently at Stage <N>, status EXISTING-<onboarding|mainapp|testflight>.
Run BOOTSTRAP.md and report what's missing or misconfigured before we proceed.
```

Re-running `npx degit s-411/drop-in-kit --force` at any point refreshes the kit to latest without touching your app code.

---

## Renaming later (when you've decided the final name)

At Stage 11 (App Store Audit), you lock the final name. When you do, you need to update:

1. **GitHub Desktop:** Repository -> Repository Settings -> Rename
2. **`app.json`:** `expo.name`, `expo.slug`, `expo.ios.bundleIdentifier`
3. **`CLAUDE.md`:** header lines
4. **`BUNDLE_IDS.md`:** the row for this app
5. **App Store Connect:** the bundle ID you reserve must match the new `expo.ios.bundleIdentifier`
6. **Remove the `_naming_status` comment** from `app.json`

The `app-store-approval` skill at Stage 11 will check all of these match.
