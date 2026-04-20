# CREDENTIALS.md — Walkthrough

> One-time Mac setup. ~10 minutes. After this, every Claude Code agent in every repo can run `eas` commands without browser auth or password prompts.

---

## What you already have (good news)

Looking at your EAS dashboard, you already have these uploaded server-side:

| Credential | Status | Where it lives |
|------------|--------|----------------|
| Apple Distribution Certificate | ✅ Valid (uploaded Jan 13, 2026) | EAS server |
| Apple Push Key | ✅ Uploaded Jan 16, 2026 | EAS server |
| App Store Connect API Key | ✅ Admin role (uploaded Jan 13, 2026) | EAS server |
| Apple Team ID | ✅ `FUHV534M4K` (Leverage Ventures) | Visible in EAS dashboard |

**EAS handles all four of these for you when you run `eas submit`.** You do not need to download the `.p8` file to your Mac. You do not need to set up `EXPO_ASC_*` env vars locally. The CLI calls the EAS server, the server has the credentials, done.

This means CREDENTIALS.md is much simpler than originally written. You only need to set up 2 things locally.

---

## What you need to set up (this walkthrough)

| Credential | Purpose | Time |
|------------|---------|------|
| `EXPO_TOKEN` | So `eas` commands run without browser login | 3 min |
| `SENTRY_AUTH_TOKEN` | Source map uploads on EAS builds (skip if not using Sentry yet) | 3 min |

Both go in the file you already created at `~/Documents/GitHub/.env-apps`.

---

## Step 1 — Get your EXPO_TOKEN (3 min)

1. Open this URL in your browser:
   **https://expo.dev/accounts/[your-username]/settings/access-tokens**
   (replace `[your-username]` with your Expo username — the one you see in the top-right of expo.dev)

2. Click **Create token**

3. Name it: `mac-local-cli`

4. Click **Create**

5. **Copy the token immediately** — it looks like `exp_xxxxxxxxxxxxxxxxxxxxxxxx`. You cannot see it again after closing the dialog.

6. Hold onto it for Step 3.

---

## Step 2 — Get your SENTRY_AUTH_TOKEN (3 min) — OPTIONAL FOR NOW

**Skip this step if you don't have a Sentry account yet** — you can do it later when you reach Stage 9a on your first app. The agent will prompt you.

If you do have a Sentry account:

1. Open: **https://sentry.io/settings/account/api/auth-tokens/**

2. Click **Create New Token**

3. Name it: `mac-eas-source-maps`

4. Scopes: tick exactly these three:
   - `project:read`
   - `project:releases`
   - `org:read`

5. Click **Create Token**

6. Copy the token (`sntrys_xxxxxxxxxxxxxxxxxx`)

---

## Step 3 — Fill in your `.env-apps` file

Open the file in your editor:
```bash
open -e ~/Documents/GitHub/.env-apps
```

(or open in Cursor / VS Code — it's a plain text file)

**Delete whatever is in there now** (you said it contains the markdown of CREDENTIALS.md — that's not what should be in the file). Replace with this exact content, substituting your real tokens:

```bash
# Shell credentials — sourced by ~/.zshrc on every new terminal session
# DO NOT commit this file to git. It contains secrets.

# Expo CLI auth — run `eas` without browser login
export EXPO_TOKEN="paste-your-EXPO_TOKEN-here"

# Gemini API key — powers the nanobanana MCP for in-app imagery + App Store screenshots
# Required by START_NEW.md step 3a on every new repo
export GEMINI_API_KEY="AIzaSyBX5SLxGn8f4RZnfwJKMp8grWVx7Y0QcKA"

# Sentry source map uploads (optional — leave commented if not using Sentry yet)
# export SENTRY_AUTH_TOKEN="paste-your-SENTRY_AUTH_TOKEN-here"

# Apple Team ID — your team in EAS (already correct)
export EXPO_APPLE_TEAM_ID="FUHV534M4K"
```

Save the file.

---

## Step 4 — Make your shell load this file on every terminal session

Right now, the file exists but your shell doesn't know about it. We need to add one line to your shell config so it sources `.env-apps` on every new terminal.

1. Open your `.zshrc`:
   ```bash
   open -e ~/.zshrc
   ```

2. Scroll to the bottom. Add this single line:
   ```bash
   source ~/Documents/GitHub/.env-apps
   ```

3. Save the file.

4. **Reload your shell** (so it actually picks up the changes):
   ```bash
   source ~/.zshrc
   ```

---

## Step 5 — Verify everything works

In your terminal, run:

```bash
echo "EXPO_TOKEN starts with: ${EXPO_TOKEN:0:6}..."
echo "EXPO_APPLE_TEAM_ID: $EXPO_APPLE_TEAM_ID"
```

You should see:
```
EXPO_TOKEN starts with: exp_xx...
EXPO_APPLE_TEAM_ID: FUHV534M4K
```

If both print, you're done. If either is empty, the source line in `.zshrc` didn't work — run `source ~/Documents/GitHub/.env-apps` directly and check your file paths.

Then verify EAS picks it up:

```bash
eas whoami
```

Should print your Expo username (no login prompt). If it prompts you to log in, `EXPO_TOKEN` isn't being loaded.

---

## Step 6 — Protect the file

Make sure the `.env-apps` file isn't readable by other users on your Mac:

```bash
chmod 600 ~/Documents/GitHub/.env-apps
ls -la ~/Documents/GitHub/.env-apps
```

The output should start with `-rw-------` — that means only you can read/write it.

Also, since you put it inside `~/Documents/GitHub/`, make sure no repo subfolder accidentally tries to commit it. Add this safety net to your global gitignore:

```bash
echo ".env-apps" >> ~/.gitignore_global
git config --global core.excludesfile ~/.gitignore_global
```

This blocks `.env-apps` from being committed in any repo, anywhere on your Mac.

---

## What about the .p8 file from App Store Connect?

You already uploaded it to EAS in January 2026 (visible as Key ID `XH49FJ996T` in your dashboard). EAS uses it server-side. **You do not need it on your Mac.**

If you ever need to download it again (e.g. switching to a different developer team), Apple does NOT let you re-download — you have to revoke and create a new one. But for now, you're good.

---

## What's happening when EAS runs

When you (or an agent) runs `eas build --platform ios --profile production`:

1. EAS CLI authenticates to expo.dev using `$EXPO_TOKEN` from your shell
2. EAS server pulls the Apple Distribution Cert + Push Key from its credential store
3. EAS server signs the build with those credentials
4. Build completes on EAS server, you download the `.ipa`

When you run `eas submit --platform ios --profile production`:

1. EAS CLI authenticates with `$EXPO_TOKEN`
2. EAS server uses the App Store Connect API Key (already uploaded) to upload to TestFlight
3. Apple processes it, you see it in TestFlight ~5-15 min later

No browser auth, no password prompts, no per-app credential setup. That's the win.

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `eas` asks for username/password | `echo $EXPO_TOKEN` is empty. Your `.zshrc` source line isn't loading. Re-run `source ~/.zshrc`. |
| `EXPO_APPLE_TEAM_ID` empty | Same as above — env vars not loading. Check the path in your `.zshrc` source line is right. |
| `eas whoami` works but `eas build` says "credentials not found" | EAS-managed credentials issue — open EAS dashboard, confirm cert/key/ASC API key are still listed and valid. |
| Sentry stack traces minified after launch | `SENTRY_AUTH_TOKEN` not set when you ran `eas build`. Set it, rebuild. |
| You want to revoke a token | Delete it on the dashboard where you created it. Generate new one, update `.env-apps`, run `source ~/.zshrc`. |
| You're on a new Mac | Re-run this whole walkthrough. Tokens are stored in your shell, not portable. |

---

## Per-app credentials (heads up, not for now)

Some credentials are per-app and live in each repo's `.env.local`:

- **Sentry DSN** (different per app — created at Stage 9a)
- **PostHog API key** (different per app — created at Stage 9b)

You don't set these up here. The agent walks you through each one when you reach the relevant stage on the first app, and the pattern repeats per app.

These go into `.env.local` inside each repo (NOT into `.env-apps`), and `.env.local` is gitignored by default in Expo projects.

---

## Optional: MCP Servers (one-time setup, when relevant)

MCP servers give Claude Code direct access to a tool's API/dashboard. Install ONLY the ones for stacks you're using.

### Nanobanana MCP — REQUIRED before Stage 7b (AI imagery)

Powers Gemini / Nano Banana image generation for app icons and in-app imagery. Install once per repo (MCPs in Claude Code 2.1.84+ are project-scoped).

**Prereqs:**
1. `uvx` installed on Mac: `curl -LsSf https://astral.sh/uv/install.sh | sh`
2. `GEMINI_API_KEY` exported in your shell — already set in the `.env-apps` template above. Verify with `echo $GEMINI_API_KEY` (should print the key, not be empty). If empty, you skipped the Step 3/4 walkthrough above — go back and finish it.
3. Billing activated on Google Cloud project the key is in (at https://console.cloud.google.com/billing) — image gen models now require billing even on free tier. Google gives $300 credit for new billing accounts (~7,500 images at Flash rates). If you already have a Google Cloud paid account from Maps/other services, you pay standard rates: ~$3 per app for 75-image passes.

**Install (run from inside each repo that needs it):**
```bash
claude mcp add nanobanana uvx nanobanana-mcp-server@latest -e GEMINI_API_KEY=$GEMINI_API_KEY
```

**Verify from inside the repo:**
```bash
claude mcp list
# should show: nanobanana: uvx nanobanana-mcp-server@latest - ✓ Connected
```

**Set a budget alert so you don't get surprised:**
- https://console.cloud.google.com/billing/budgets → Create Budget → $25/month → alerts at 50/90/100%

**IMPORTANT:** MCPs only load at session start. If you add nanobanana while a Claude Code session is running, the running session won't see it — you need to close and reopen Claude Code.

### Stripe MCP — when you start your first paid web (Next.js) app
```bash
claude mcp add --transport http stripe https://mcp.stripe.com/
```
- Auth: OAuth (browser-based, recommended) OR `STRIPE_SECRET_KEY` env var
- Get keys: Stripe Dashboard → Developers → API keys
- Capabilities: create/list customers, subscriptions, checkout sessions, payment links; search Stripe docs

### Sentry MCP — when you wire Sentry on your first app
```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```
- Auth: OAuth via browser (cloud) — no API key needed for cloud Sentry
- For self-hosted: create access token with scopes `org:read`, `project:read`, `project:write`, `team:read`, `team:write`, `event:write`
- Capabilities: search issues/errors with stack traces, query events in natural language, Seer AI root-cause analysis, debug from pasted Sentry URLs

### RevenueCat MCP — when you wire RevenueCat on your first paid app
```bash
claude mcp add --transport http revenuecat https://mcp.revenuecat.ai/mcp \
  --header "Authorization: Bearer YOUR_API_V2_SECRET_KEY"
```
- Auth: API v2 Secret Key required
- Get key: RevenueCat Dashboard → Project → API Keys → Create new v2 key (write-enabled or read-only)
- Capabilities: view project status (apps, products, entitlements, offerings), create apps + products, manage subscriptions

### Verifying MCP installs
```bash
claude mcp list
```
Should show each server you've added with status `connected` or `ready`.

### When NOT to install
Don't install MCPs you won't use right now. They consume Claude Code's tool budget and add latency. The Stack Profiles map tells you which stacks each app uses — install accordingly.
