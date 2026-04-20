# AI_IMAGERY.md

> **Companion doc for Stage 7 (Polish Audit).** All placeholder gradients in the app get replaced with AI-generated imagery in one dedicated session per app. ~60-90 min per app.
>
> **Tool:** Nano Banana (Gemini 3 Pro Image) via the Gemini MCP already installed on your Mac for the Stage 14 screenshot skill.
>
> **Rule:** do NOT generate images during Stages 2-6. Use gradient placeholders. Image generation is a Stage 7 task. Filling images earlier = scope creep and slower iteration on features.

---

## When this runs

**Stage 7 — Polish Audit.** After the `design-consistency` skill audit, before moving to Stage 8 (Landing Page). One dedicated 60-90 min session per app.

Pre-req: all gradient placeholders are in place and functional. If the app still has broken screens, fix those first — do not paper over with imagery.

---

## The workflow (one-pass)

### 1. Audit what needs replacing

```bash
grep -rn "colors={\['" src/screens/main/ | grep -i "placeholder\|gradient"
# or more specifically:
grep -rn "LinearGradient" src/screens/main/ | grep -v "theme\."
```

Build a list of placeholder slots. For the gym app, typical inventory:
- **App icon** (`assets/icon.png`) — 1024×1024 PNG — REQUIRED for a professional TestFlight build
- 6 program hero images (ProgramsScreen + ProgramDetailScreen)
- 3 meal plan card backgrounds (NutritionScreen)
- 6 library category thumbnails (LibraryScreen)
- 2-4 featured content cards (HomeScreen)

**The app icon is the first imagery to generate.** It's the most visible asset (shows up on the home screen, in TestFlight, in the App Store listing) and it drives client perception of the whole build. Generate it FIRST, lock it, then proceed with in-app imagery.

### 2. Pick a style anchor

Before generating individual images, generate ONE "anchor" image that defines the visual language for the whole app. Every subsequent image references this anchor for consistency.

Example anchor for gym apps:
> *"Cinematic fitness photography, moody dark gym, dramatic side lighting with warm orange accents on #f1652a highlights, shallow depth of field, film grain, Kodak Portra 400 film stock aesthetic, magazine quality, no people's faces visible, 3:2 ratio"*

Save the anchor image + prompt as `assets/_anchor.jpg` and `assets/_anchor.txt`. Reference this in every prompt that follows.

### 3. Generate in bulk via Gemini MCP

Single Claude Code prompt (run while agent rotates to another repo):

```
Read AI_IMAGERY.md in this repo. Generate all placeholder images for this
app in one pass using Gemini.

Style anchor: read assets/_anchor.txt for the visual style to match.

Images to generate (list per app type — see below for specific briefs).

For each image:
1. Call Gemini with the prompt + anchor reference
2. Save to the specified path
3. Verify file exists and is <300KB
4. Continue to next

Generate all, then give me a summary report.
```

### 4. Wire images into code

```
Now replace every gradient placeholder with the generated images:
- Update each screen to use require('../../assets/...') instead of
  LinearGradient with placeholder colors
- Keep LinearGradient for UI overlays (text readability on images),
  just use the image underneath
- Run /skill self-check
- Run `npx tsc --noEmit`
```

### 5. Verify on device

Reload in Expo Go. Walk through every screen. Any image that looks off:
- Regenerate with refined prompt
- Or swap to a different variant

---

## Image specs (all asset types)

| Asset | Aspect | Size (px) | Format | Max KB |
|-------|--------|-----------|--------|--------|
| **App icon** | **1:1** | **1024 × 1024** | **PNG** | **500** |
| Program hero (full screen) | 3:4 | 1200 × 1600 | WebP | 300 |
| Program hero (card) | 16:9 | 1200 × 675 | WebP | 200 |
| Meal plan card bg | 16:9 | 1200 × 675 | WebP | 200 |
| Library category thumbnail | 1:1 | 800 × 800 | WebP | 150 |
| Featured content card | 4:5 | 800 × 1000 | WebP | 200 |

All @2x for retina. Use WebP over PNG unless transparency is needed.

Compress after generation:
```bash
for f in assets/programs/*.png; do
  cwebp -q 80 "$f" -o "${f%.png}.webp"
  rm "$f"
done
```

---

## Prompt library by app type

### App icon prompts (all app types)

App icons have different rules than in-app imagery. The icon must read clearly at 60×60 pixels on a phone home screen. Photo-realistic icons almost always fail this test — by the time you shrink a gym photograph to 60px it's mud.

**Rules for every app icon:**
- 1024×1024 square PNG, no transparency, no rounded corners (Apple rounds them)
- Maximum 2-3 visual elements
- Strong silhouette readable in greyscale at 60px
- Single brand color + one accent color, matched to `theme.ts` primary
- NO text on the icon (Apple's design guidance)
- NO gradients that wash out when shrunk

**Generation approach:**

1. Open the app's `src/theme/theme.ts` and note the primary and accent colors
2. Prompt Gemini with this template, filled in with app-specific details:

```
App icon design, 1024x1024 square, flat vector illustration style.
Subject: [SINGLE ICONIC ELEMENT for this app — dumbbell / water drop /
flame / brain / lotus / etc, pick ONE]
Primary color: [theme.ts primary hex]
Accent color: [theme.ts accent hex]
Background: solid color, [theme.ts bg or surface hex]
Style: bold geometric shapes, modern minimalist, high contrast,
strong silhouette readable at small sizes.
NO text, NO letters, NO words, NO logos, NO watermarks.
NO gradient fills on the background.
NO photorealistic imagery.
```

3. Generate 3-4 variants. View each at 60px thumbnail (macOS Finder preview works).
4. Pick the one that reads clearly at 60px.
5. Save to `assets/icon.png`.

**Per-app icon subjects** (add more as the you build more):

| App type | Icon subject |
|----------|--------------|
| Gym / lifting | Single dumbbell, barbell, or flexed-arm silhouette |
| Quit drinking | Water drop, clean glass, or crossed-out bottle |
| Quit porn | Shield, lock, or sword |
| Habit tracker | Flame (streaks), calendar checkmark, or tree growing |
| Meditation | Lotus flower, single curved line, or circle with dot |
| Journal | Open book, pen nib, or quote mark |
| Nutrition | Apple silhouette, single leaf, or plate with fork |
| Focus / productivity | Target, arrow, or mountain peak |
| Pregnancy | Gentle curve (belly silhouette) or flower bud |

Per-app accent colors already come from the app's theme file — don't override.

---

### Fitness / training apps (gym, lifting, cardio)

**Style anchor:**
> Cinematic fitness photography. Moody dark gym interior. Dramatic warm side-lighting on subjects, deep shadow contrast. Warm orange color grade (#f1652a hue), slight film grain, shallow depth of field. Editorial magazine aesthetic — think Men's Health × Kinfolk. No faces visible (crop above neck or show from behind). Empty equipment, dumbbells, plates, barbells, cable machines as primary subjects. 3:2 cinematic aspect ratio.

**Program hero prompts (swap the action for each program):**
- Upper/Lower split: *"[anchor] — man mid-deadlift at olympic barbell, plates loaded heavy, hands chalked"*
- Push/Pull/Legs: *"[anchor] — arrangement of dumbbells in ascending weight, gym floor, shot from above"*
- Full body: *"[anchor] — squat rack with barbell loaded, empty gym, early morning light through window"*
- Beach body / cutting: *"[anchor] — lean silhouette doing cable flyes, dramatic backlighting"*
- Aesthetic Phase 1: *"[anchor] — back muscles during lat pulldown, sweat detail, rim lighting"*
- Aesthetic Phase 2: *"[anchor] — barbell curl against mirror, forearm veins, dramatic contrast"*

### Quit-bad-habit apps (alcohol, porn, nicotine)

**Style anchor:**
> Minimal cinematic lifestyle photography. Morning light, clean modern interiors, soft neutrals with a single accent color. Subtle hope and renewal aesthetic — think Apple ad × Kinfolk lifestyle. Sparse compositions, lots of negative space. No faces, or shot from behind. Muted earth tones.

### Wellness / mindfulness / journaling

**Style anchor:**
> Serene editorial photography. Soft natural light, cool muted palette, botanicals and ceramics, linen and wood textures. Journalistic morning-routine feel. Shallow depth of field. Hands, objects, spaces — no faces. Instagram slow-living aesthetic.

### Tracking / logging apps (habit, mood, water)

**Style anchor:**
> Clean geometric illustration. Flat vector style with subtle gradient shading, primary brand color with one accent. Isometric or 3/4 perspective. Friendly, not corporate. Think Dribbble top-shot circa 2024 — bold shapes, limited palette.

(Tracking apps often suit illustration over photography — reads cleaner on small cards.)

### Productivity / focus apps

**Style anchor:**
> Minimal studio photography of premium objects. Soft window light, neutral backdrops (linen, concrete, plaster). Mechanical keyboards, leather notebooks, fountain pens, matcha, analog timers. Muji × Monocle aesthetic. Single-color accents matching brand.

### Nutrition / meal plan content (food photography)

**Style anchor:**
> Editorial food photography. Natural window light, 3/4 overhead angle, rustic wood or linen backdrop. Color-matched to meal category (protein = warm tones, veggie = cool greens, carbs = earth tones). Shallow depth of field. NYT Cooking × Bon Appétit aesthetic.

**Specific prompts:**
- High Carb plan: *"[anchor] — bowl of oatmeal with berries, banana slices, honey drizzle, morning light"*
- Low Carb: *"[anchor] — grilled chicken breast, roasted asparagus, avocado slices, lemon wedge"*
- Balanced: *"[anchor] — grain bowl with salmon, quinoa, greens, tahini dressing, overhead"*

### Content library / educational

**Style anchor:**
> Conceptual illustration with limited palette. Bold geometric shapes representing abstract concepts (strength, recovery, nutrition, mindset). Single accent color (brand primary). Minimal, editorial — think New Yorker column illustrations × Figma Config branding.

---

## Gemini MCP invocation patterns

Check the MCP is live:
```bash
claude mcp list | grep gemini
```

In Claude Code, invocation shape:
```
Use Gemini MCP to generate an image.

Prompt: "[paste anchor + specific brief]"
Aspect: 3:2
Output path: /absolute/path/to/assets/programs/upper-lower.png
Negative prompt: "no faces, no watermarks, no text, no logos"
```

If the MCP tool name differs, check the available tools:
```
Show me all Gemini-related tools available.
```

---

## Iteration strategy

First pass generates 80% usable. The other 20% need iteration.

**Green flag:** image matches anchor aesthetic, reads at thumbnail size, no weird artifacts.

**Red flags requiring regeneration:**
- Hallucinated anatomy (extra fingers, warped proportions on visible bodies)
- Text baked into image (Gemini sometimes adds fake logos)
- Wrong subject (asked for deadlift, got bench press)
- Wrong mood (too bright / too cheerful for the anchor)

**Prompt refinement:**
- If too generic: add specificity (camera, film stock, location detail)
- If too chaotic: add "minimal, sparse composition, negative space"
- If wrong color: add explicit hex to the prompt (Gemini respects `#f1652a` in prompts)
- If faces appeared when not wanted: add to negative prompt

---

## Don't generate — source real imagery

Some content needs real photography, not AI:

- **Before/After transformation photos** — AI can't fake realistic body transformation without looking weird. Stage 14 screenshot skill can add these in ASCII-art placeholder style if needed. For in-app: use generic "fitness journey" stock or commission real photos post-launch.
- **Real people testimonials** — never generate fake faces with fake testimonials. This violates App Store guidelines. Use avatar initials or abstract shapes until you have real users.
- **Real brand partnerships / logos** — generate your own app branding, but don't generate fake third-party logos (Bioniq, Whoop, Apple Health, etc.)

Park all of these in LATER.md → "source real imagery before v1 ship."

---

## Post-generation checklist

Before marking Stage 7 imagery complete:

- [ ] Every `LinearGradient` placeholder in `src/screens/` replaced with real image (except text-overlay gradients)
- [ ] `assets/` directory size under 10MB total (keeps binary small)
- [ ] All images compressed to WebP where possible
- [ ] No faces on people used in commercial stock-style unless explicitly AI-generated
- [ ] No baked-in text that duplicates what's rendered as actual Text components
- [ ] Anchor image + prompt saved to `assets/_anchor.jpg` + `assets/_anchor.txt` for future app reference
- [ ] Winning prompts added to this doc under the relevant app-type section (compounds across all apps)

---

## After shipping — feed discoveries back

The prompts above are starting points. After each app, add:

- The exact prompts that produced shippable images on first try → promote to this file
- The prompts that needed iteration → note the iteration pattern (what you changed and why)
- New app types not yet covered (e.g. "crypto portfolio tracker") → new section

This file gets better every app. By app 5, you should have a locked prompt library that produces shippable imagery in 30-40 min instead of 90.

---

## Cross-app roadmap (add to LATER.md if not there)

- [ ] After 3 apps shipped: audit winning prompts, build a `~/app-imagery/` library of anchor images per aesthetic (masculine-dark, feminine-soft, minimalist, editorial) — new apps pick an anchor in 30 seconds instead of re-generating
- [ ] Evaluate whether an "imagery skill" is worth building — would codify this doc into an auto-invokeable Claude Code skill
