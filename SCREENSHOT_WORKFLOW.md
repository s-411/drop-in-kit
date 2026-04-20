# Screenshot Workflow

> Companion doc for **Stage 14** of `PROCESS_GUIDE.md`. App Store screenshots handled by Adam Lyttle's Claude Code skill. One-time Mac setup, then ~20 minutes per app.

---

## When to Do This

**Stage 14 in `PROCESS_GUIDE.md`** — after the app is on TestFlight (Stage 13) and verified working. Before metadata generation (Stage 15) and submission (Stage 16).

You need a working app to screenshot. Don't try to do this earlier.

---

## One-Time Mac Setup (already done — keep for reference)

Run these once. Works across all repos after that.

```bash
# 1. Install the skill globally into Claude Code
git clone https://github.com/adamlyttleapps/claude-skill-aso-appstore-screenshots \
  ~/.claude/skills/aso-appstore-screenshots

# 2. Python dependency
pip3 install Pillow

# 3. SF Pro Display Black font
# Download from https://developer.apple.com/fonts/
# Move to: /Library/Fonts/SF-Pro-Display-Black.otf

# 4. Gemini MCP (powers the AI enhancement step)
npm install -g @houtini/gemini-mcp
```

Then add Gemini MCP to `~/.claude/settings.json`:
```json
{
  "mcpServers": {
    "gemini": {
      "command": "npx",
      "args": ["-y", "@houtini/gemini-mcp"]
    }
  }
}
```

**Verify all 4 are installed before starting Stage 14 on the first app.** If any of these break, every app's Stage 14 breaks until fixed.

---

## Per-App Workflow (~20 min)

### 1. Read the target dimensions from ASC FIRST

Before running the skill, open the ASC record for this app and note the exact iPhone slot dimensions and (if universal) iPad slot dimensions. See "Apple's Screenshot Requirements — ASC record age matters" below. You'll pass these into the skill explicitly — don't let the skill's default win.

### 2. Grab Simulator Screenshots
1. Run the app: `npx expo start --ios` (Simulator opens)
2. In Simulator: navigate to each key screen
3. `Cmd + S` to save each screenshot
4. Screenshots land in `~/Desktop` by default

**Aim for 5–6 screens, ranked by impact:**
1. Hero / value prop screen (the one that explains the app in one glance)
2. Quiz / personalization (proof the app adapts to the user)
3. Custom plan / main feature screen
4. Secondary feature screen
5. Social proof / reviews screen (if present)
6. Onboarding hook screen (if visually strong)

### 3. Run the skill with explicit dimensions + outpaint-first fill

The skill is invoked via `/aso-appstore-screenshots` but the per-app prompt (see `PROMPTS.md` Stage 14) includes three mandatory inputs:

- **Target dimensions** for each device class (iPhone, iPad) — read from ASC in step 1
- **Both device sets are mandatory** — iPhone AND iPad. iPad is NOT optional. If the skill tries to skip iPad output, STOP and surface the error. A Stage 14 with only iPhone screenshots is incomplete.
- **Outpaint-first fill strategy** — any aspect-ratio mismatch between a source screenshot and the target output **MUST be filled by outpainting via the nanobanana (Gemini) MCP**. Never letterbox with solid color bars (black, white, or theme-color). If the nanobanana MCP is not available in the session, STOP and surface the error — do not fall back to letterboxing.
- **Visual treatment tier** — the prompt specifies zoom/magnify, depth, callouts, and composition variety so output rises above flat "phone frame + title" defaults. See `PROMPTS.md` Stage 14 for the full treatment spec.

Follow the skill's interactive prompts after that. It will:
1. Read the codebase → identify 3–5 core benefits
2. Pair each simulator screenshot with the best benefit
3. Generate 3 versions per screenshot (pick the best)
4. Output finals at the explicit dimensions you passed in

### 4. Pick Best of 3 Per Screen
The skill outputs 3 variants per screen. Walk through them, pick the strongest. Finals land in `screenshots/final/<device>/` with the `iphone/` and `ipad/` subfolders Stage 15 expects.

### 5. Upload at Stage 16
Finals from `screenshots/final/iphone/` and `screenshots/final/ipad/` upload directly to App Store Connect — no cropping, no resizing needed.

---

## Fill strategy for aspect mismatches — OUTPAINT, never letterbox

When a source screenshot's aspect ratio doesn't match the target output (especially iPad from iPhone sources, but also iPhone when the source came from a different device size), the fill strategy is **mandatory outpaint via the nanobanana (Gemini) MCP**.

- ❌ **Never** letterbox with solid black, white, or theme-color bars
- ❌ **Never** stretch/squash the source to fit
- ❌ **Never** crop the source to fit (unless specifically requested by the user for a single screenshot)
- ✅ **Always** extend the scene naturally — Gemini outpaints background gradients, textures, UI chrome, surrounding context — so the final looks native at the target size

If the nanobanana MCP is not available in the current Claude Code session, STOP the workflow and surface the error. Do not silently fall back to letterboxing — that produces screenshots that look like an app built by someone who didn't care, and it's been the single biggest quality loss in Stage 14.

The per-app Stage 14 prompt in `PROMPTS.md` enforces this at the invocation level, so the skill never has to "remember" to reach for Gemini — it's a hard requirement in every run.

---

## Output Structure

```
screenshots/
  01-benefit-slug/
    scaffold.png       ← layout before AI enhancement
    v1.png, v2.png, v3.png
  final/
    iphone/            ← upload these to the iPhone slots in ASC
      01-benefit-slug.png
      02-benefit-slug.png
      03-benefit-slug.png
      04-benefit-slug.png
      05-benefit-slug.png
    ipad/              ← upload these to the iPad slots in ASC (if universal)
      01-benefit-slug.png
      02-benefit-slug.png
      03-benefit-slug.png
      04-benefit-slug.png
      05-benefit-slug.png
  showcase.png         ← side-by-side preview (for your reference)
```

Stage 15's `asc-metadata.md` template (`PROMPTS.md` Stage 15, Section 1.1) expects `screenshots/final/iphone/` and `screenshots/final/ipad/` as the canonical paths. Keep this structure.

---

## Apple's Screenshot Requirements — ASC record age matters

**Before invoking the skill you MUST check which era your ASC record belongs to.** The skill defaults to 1320×2868 (Apple's current "new app" requirement), but ASC records created before the 6.9" requirement rolled out are grandfathered into the older 6.5" slots and **silently reject or fail** 1320×2868 uploads. Always confirm the accepted dimensions from the actual ASC listing for the app, then pass them into the skill explicitly.

### How to check the record's accepted size

1. Open App Store Connect → your app → **App Store** tab → the version you're working on
2. Scroll to **App Previews and Screenshots**
3. Look at the device-size picker (shows the size slots ASC has reserved for this record). The slot label tells you the era:

| ASC slot label | Accepted dimensions | Era |
|---|---|---|
| **6.9" Display** | 1320 × 2868 (portrait) or 2868 × 1320 (landscape) | New records (post-6.9" rollout) |
| **6.7" Display** | 1290 × 2796 or 2796 × 1290 | Mid-era records — still widely accepted |
| **6.5" Display** | 1242 × 2688 or 2688 × 1242 | Grandfathered records — older apps |
| **6.5" Display (alt)** | 1284 × 2778 or 2778 × 1284 | Grandfathered alt variant |

For iPad (if universal — required for iPad records):

| ASC slot label | Accepted dimensions |
|---|---|
| **13" iPad** | 2064 × 2752 or 2752 × 2064 |
| **12.9" iPad** | 2048 × 2732 or 2732 × 2048 |

**If the ASC record has a 6.5" slot (not 6.9"), you cannot upload 1320×2868 to it. Generate to the slot size or uploads fail.**

### Never infer — always read from ASC

Do NOT let the skill default to 1320×2868. Do NOT guess based on "this feels like a newer app." Open ASC, read the slot label, pass the exact dimensions into the skill invocation. Per-app `PROMPTS.md` Stage 14 carries a required target-dimensions line for this reason.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| ASC rejects the upload with "incorrect dimensions" | ASC record is grandfathered — check the slot label (6.5" vs 6.9") and regenerate at the slot's accepted size. See "Apple's Screenshot Requirements — ASC record age matters" above. |
| Output has solid color bars on iPad (letterboxing) | Outpaint step didn't run. Confirm `claude mcp list \| grep nanobanana` shows `✓ Connected`. If missing, install per `CREDENTIALS.md` Nanobanana section, restart Claude Code, re-run. Never accept letterboxed output. |
| Skill output size doesn't match what you asked for | The skill's default won. The per-app prompt must pass target dimensions as explicit input, not rely on the default. See `PROMPTS.md` Stage 14. |
| Font not found | Confirm SF Pro Black is at `/Library/Fonts/SF-Pro-Display-Black.otf` |
| Gemini MCP not connecting | Run `claude mcp list` from inside the repo. If nanobanana is missing, re-install per `CREDENTIALS.md`. MCP must be installed per-repo in Claude Code 2.1.84+. |
| Skill not found | Re-run the `git clone` command from "One-Time Mac Setup" |
| Generated copy doesn't match the app | The skill mis-identified benefits — re-run with explicit benefit list as input |
| Screenshots look generic | Pick stronger source screens — empty states or settings screens make weak ASO assets |

---

## Repo

https://github.com/adamlyttleapps/claude-skill-aso-appstore-screenshots

---

## Notes

- Do this *after* Stage 13 (TestFlight smoke test) so you're screenshotting the actual shipped UI, not a half-built one
- Don't update screenshots once submitted — Apple will re-review the listing if you do, which delays release
- **This doc is reference.** The actual Stage 14 invocation is a full copy-paste prompt block in `PROMPTS.md` Stage 14 — that's where the hard constraints (target dimensions, mandatory iPad, outpaint-first, visual treatment tier) live inline so the skill receives them at invocation time. Read this doc to understand *why*; read `PROMPTS.md` to know *what to paste*.
