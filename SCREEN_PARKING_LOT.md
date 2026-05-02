# SCREEN_PARKING_LOT.md — Bulk Port from External Design Tools

> **What this is:** a reusable methodology for ingesting batches of externally-designed screens (Claude Design, Figma-to-React exports, design-canvas tools, etc.) into a working React Native + Expo repo as registered, viewable screens — without committing to routing, auth, or data wiring yet.
>
> **When to use:** any time a design tool hands you 10+ screens at once as bulk JSX/HTML exports and the repo already has a working drop-in kit baseline (registry + gallery + tokens + primitives).
>
> **Where this fits in the pipeline:** Slots into Stage 2 onward of the standard 16-stage pipeline as an alternative to "build screens incrementally per stage" when screens are already designed elsewhere. The pattern can also run multiple times across the build life of an app — each new design batch from the design tool is a fresh ingest cycle.
>
> **Recommended location:** drop this file into the drop-in kit so every new app picks it up automatically.

---

## Why this pattern exists

External design tools produce screens faster than any traditional per-stage workflow can absorb them. A typical batch arrives as one or more `.jsx` files where each file exports 4-20 full screen components. Naive approaches that try to wire each screen as it's ported (routes, auth gates, mock data, navigation) compound decisions and slow the port to a crawl.

The parking lot pattern decouples **porting** (visual translation web JSX → RN) from **wiring** (routing, data, auth). You port everything visual into a holding section first, where each screen is reachable from a dev gallery for review on a real device. Wiring, categorization, and integration happen later as separate phases.

Benefits:

- Agent has one job per port: translate JSX. No decision load on architecture.
- You see every screen on a real device before any routing decisions.
- Re-categorization is cheap once the visual foundation is settled.
- Multiple ingest cycles compose — drop a new batch in any time.
- The "parking lot" name itself signals "this is unsorted, sort later" — prevents premature categorization debates.

---

## Repo prerequisites

Before running this pattern, the repo must have:

1. **Expo + expo-router scaffolded.** `app/_layout.tsx` and a flat or grouped route structure.
2. **A gallery index page.** Typically `app/index.tsx` rendering a `SectionList` driven by a registry.
3. **A generic screen viewer.** Typically `app/screen/[id].tsx` that loads any registered screen by id and renders it inside a phone-frame wrapper. The viewer should swallow load errors with a fallback panel ("Screen not yet ported — <error>") so a broken port doesn't crash the gallery.
4. **A registry file.** Typically `src/screens/registry.ts` exporting:
   - A `ScreenSection` union (string literal types of section labels)
   - A `SECTIONS` array (ordered for gallery display)
   - A `REGISTRY` array of `{ id, label, section, load: () => require('./path/Screen') }` rows
5. **A tokens file.** Typically `src/theme/tokens.ts` with semantic color, typography, spacing, radius, and shadow roles.
6. **A primitives layer.** Typically `src/components/primitives/` with shared `Heading`, `Body`, `PrimaryBtn`, `Card`, `Input`, `CloseBtn`, etc.
7. **A staging directory.** Typically `app-reference/new-screens/` where bulk JSX exports are dropped before porting. (Keep separate from any existing `app-reference/` so duplicates against earlier batches are obvious.)

If any of these are missing, fix the foundation first. This pattern assumes them.

---

## The four phases

### Phase 0 — Stage the export

Drop the design tool's output (typically a folder of `.jsx` files) into `app-reference/new-screens/`. Don't merge with `app-reference/` — keep them separate so duplicates against earlier batches are detectable.

No agent involvement at this phase.

### Phase 1 — Audit + port plan (read-only)

Run a single audit prompt that produces a port plan as inline markdown (no file writes). The plan must include:

- **Inventory** — every full-screen export across every staged file, with source file path, export name, and a one-line UI description.
- **Dedupe** — each export marked DUPLICATE (matches an existing ported screen, cite the matching file) or NEW. Conservative: when unsure, mark NEW and flag in judgment calls.
- **Port plan** — proposed parking-lot section header, target folder, file naming scheme, registry id naming scheme, and a complete mapping table (source export → target file → registry id → registry label).
- **Porting notes** — web-only patterns to translate (Tailwind, CSS Grid, keyframes, radial gradients, etc.), new data shapes that hint at future schema work, anything broken or half-finished in source.
- **Judgment calls** — ambiguous duplicates, naming choices, anything requiring author input.

Author reviews the plan, approves or pushes back. No code is written until the plan is approved.

### Phase 2 — Port screens

Execute the approved plan in three sub-phases:

**2a — Scaffold.** Create the parking-lot folder. Add the new section to the `ScreenSection` union and the `SECTIONS` array (appended at end). Do NOT add registry entries yet — those land per-screen during 2b. Show diff. Stop.

**2b — Port screens, one at a time at first.** For each screen in the planned order:

- Translate web JSX → RN per the rules below.
- Add file header comment: `// Port of <source-file>.jsx → Screen<Name>()`.
- Add the registry entry using the planned id and label.
- Run `npx tsc --noEmit` — must pass before moving to next screen.
- Stop, show diff + tsc result, await approval.

After 3-4 screens the pattern is established and review fatigue sets in. Switch to small batches (2-4 screens at a time) for the rest of the flow. Don't batch across logical flow boundaries — finish a flow before starting the next.

**2c — Helpers.** If multiple screens in the same flow share helper components (e.g. tooltip, sandbox banner), extract to a single `_<flow>-helpers.tsx` inside the parking lot. Don't promote to `@/components/` until the screens leave the parking lot — promotion is a separate decision once you know the helper has earned its place across multiple flows.

### Phase 3 — Verification

Single agent run that confirms:

- `npx tsc --noEmit` returns zero errors.
- `grep -rE "uri: ['\"]http" src/screens/parking-lot/` returns zero matches (no remote URIs).
- New section is at the end of `SECTIONS`.
- All planned registry ids are present, no duplicates (`grep -oE "id: '[a-z0-9-]+'" registry.ts | sort | uniq -d` empty).
- Total registry count matches expected delta (pre-port + new screens).

Author reviews verification table. If anything fails, stop before fixes — root-cause first.

### Commit

Single commit with descriptive message listing flows + screen counts. Example:

```
Add 20 new onboarding-flow screens to parking lot

- Flow A (4): screens-a-list
- Flow B (4): screens-b-list
- Flow C (4): screens-c-list
- Flow D (4 + helpers): screens-d-list, _flow-d-helpers
- Flow E (4): screens-e-list

Registry: 88 -> 108 screens. tsc clean, no remote URIs.
```

---

## Naming conventions

**Section labels.** Recommendation for new apps: number all sections from the start. Use `Section 01 · <Topic>` through `Section 99 · <Topic>` for sort stability and unambiguous ordering. Don't mix `Section A`, `Section 1`, `Wellness · Mindset`, `Parking Lot · ...` — pick one scheme and stick to it.

The parking lot itself uses a different format: `Parking Lot · <Batch description>`. This signals "unsorted batch" visually in the gallery and makes it obvious which screens are awaiting categorization. If multiple parking-lot batches exist before re-categorization, distinguish them: `Parking Lot · Onboarding flows`, `Parking Lot · Settings v2`, etc.

**Folder paths.**

- Per-section folders: `src/screens/section-NN/` matching the section number with zero padding.
- Parking lot: `src/screens/parking-lot/` (single flat folder, easy to redistribute).

**File names.** PascalCase, no `Screen` prefix. Match the source export name minus the prefix:

- `ScreenPF2_Ideal` → `PF2Ideal.tsx`.
- `ScreenDashboard` → `Dashboard.tsx`.

**Registry ids.** Lowercase, hyphenated, prefixed by the source file abbreviation:

- `pf2-ideal`, `pf2-task`, `pl1-first-step`.

The id is permanent; the label can change freely. If you re-categorize a screen later, the file moves and the registry section changes, but the id stays. This means deep links and references to specific screens survive re-categorization.

**Registry labels.** `<PREFIX> · <Description>` or `<Code> · <Description>`. Used in the gallery list, so keep them short and human-readable:

- `PF2 · Ideal outcome`.
- `B3 · Girl Detail`.

---

## Translation rules (web JSX → React Native)

Document these as rules the agent applies during port. Most are mechanical.

**Tailwind utilities.** Replace with `StyleSheet.create({...})` referencing tokens from `src/theme/tokens.ts`. Existing port templates in the repo are the reference.

**Phone wrapper.** If the source wraps every screen in a `<Phone>` or `<FrameWrapper>` component, drop it — the gallery's `app/screen/[id].tsx` viewer wraps every screen in a phone frame already. Double-wrapping breaks measurements.

**Design tool helper kit.** References to `T` token objects, `Eyebrow`, `Heading`, `Body`, `PrimaryBtn`, `SecondaryBtn`, etc. should map to the repo's existing `@/components/primitives` and `@/components/shared` exports. Don't import from the design tool's helper namespace.

**CSS Grid.** Replace with flex:

- `gridTemplateColumns: 'repeat(N, 1fr)'` → flex row with `flex: 1` per child.
- `gridTemplateColumns: 'repeat(2, 1fr)'` for 2x2 → `flexDirection: 'row', flexWrap: 'wrap'` with `width: '48.5%'` per cell (gap leaves room for a small inter-cell gap).

**CSS keyframes.** Replace with Reanimated v4 (`react-native-worklets/plugin` must be in babel config):

- `useSharedValue` + `withRepeat(withTiming(...))` driving an `<Animated.View>` style.

**Linear gradients.** `expo-linear-gradient` with the same color stops. Direction prop maps from the CSS angle.

**Radial gradients.** No native RN equivalent. Two options:

- Approximate with `expo-linear-gradient` if the radial origin is on an edge (linear top→bottom often reproduces the visible wash).
- Use `react-native-svg <RadialGradient>` for true radial.

Pick whichever looks closer; note the choice in translation notes.

**Box shadow.** No 1:1 equivalent for spread/inset. Approximation strategies:

- Single drop shadow: iOS `shadowColor / shadowOffset / shadowRadius / shadowOpacity` + Android `elevation`.
- Spread halos (`box-shadow: 0 0 0 4px <color>`): wrap the element in a slightly larger sibling View with the halo color and matching `borderRadius`. Shadows on small elements often disappear; concentric-circle layering reads cleaner.
- Layered shadows: stack two or three concentric Views. Honest about the limitation: pixel-perfect parity isn't always achievable.

**Inline `<span>` for color/weight runs.** Nest `<Text>` inside `<Text>`. RN supports nested Text for inline style runs.

**External image URLs.** Download to `assets/` and reference via `require()`. Never hotlink — hotlinks break randomly, slow app boot, and fail offline. The verification grep for `uri: ['\"]http` catches misses.

**Unicode escapes.** Source often uses `\u{1F4C9}` etc. for emoji. Substitute literal glyphs (📉) for readability.

**Borderless dashed cards.** RN supports `borderStyle: 'dashed'` on Views directly.

**Percentage transforms.** RN doesn't support `translate(-50%, -50%)` percentage syntax. Use known pixel values: `transform: [{ translateX: -wrapperWidth/2 }, { translateY: -dotSize/2 }]`.

**Border radius shorthand.** `borderRadius: '3px 3px 0 0'` doesn't work. Use individual props: `borderTopLeftRadius`, `borderTopRightRadius`.

**Z-index on stacked absolute layers.** A gradient that includes 'transparent' stops does NOT let touches or visibility pass through to layers beneath it in RN — counter-intuitive coming from web CSS. Content sitting beneath a gradient overlay either needs `StyleSheet.absoluteFill` to sit on top, or explicit `zIndex`/`elevation` higher than the gradient.

---

## Token discipline during port

**Don't add new tokens for translucent variants.** Source designs use many opacities of accent (0.06, 0.08, 0.10, 0.12, 0.20). Adding all of them as tokens bloats `tokens.ts` for marginal gain. Inline `rgba(...)` at the use site, with a comment if it's intentional. The strict semantic tokens (`accent`, `accentInk`, `successBg`, `dangerBg`, etc.) cover the canonical solid uses.

**Do extend tokens** when a new semantic role appears that isn't covered (a different brand color, a new tone like `warning` if it didn't exist). Show the addition in the port diff so it's review-worthy.

**Decorative one-offs stay inline.** Avatar palette colors (`#5BAAFF`, `#FF8A65`), single-use shadows, screen-specific gradients — inline. Tokens are for repeated, semantic uses, not for every hex that appears in the design.

---

## What this pattern defers

The parking lot ports visual screens. It does NOT do:

- **Routing.** Screens are reachable only via the gallery viewer. No tab bar, no stack pushes, no modal presentations. Cross-screen navigation doesn't exist after port.
- **Auth gates.** No signed-in vs signed-out distinction. Sign-in screens are pure visual.
- **Real data.** Mock data is inline in each screen (small arrays at the top of the file). No backend, no Convex, no AsyncStorage persistence.
- **Interactivity beyond visual state.** Buttons render. They don't navigate. Forms render. They don't submit. Toggles flip local component state at most.
- **Onboarding wiring.** Multi-step flows have all their step screens registered, but no flow logic ties them.

These all happen in subsequent phases. The parking lot's purpose is to make the visual foundation reviewable on a real device.

---

## After the parking lot — recommended phasing

Once a batch lands, the next phases (in order):

1. **Re-categorize** (lightweight). Move screens out of `parking-lot/` into final section folders. Update registry section labels. The dev gallery still works through this — it's just file moves + registry rewrites.
2. **Routing IA proposal.** Agent proposes the production route tree (tabs, stacks, modals, route groups). Author approves. Implementation = thin route shims in `app/` that re-export from `src/screens/section-NN/`.
3. **Auth + backend init.** If using Convex Auth + Convex DB: wire early, before any feature uses real data. Late wiring causes painful retrofitting.
4. **Schema definition.** Derive from screens. Forms, lists, and detail views encode data shapes. Better to scope schema with all screens visible.
5. **Feature slices, end-to-end, one at a time.** Each slice = schema fields + mutations/queries + screen wiring + nav guards. One feature working before starting the next.
6. **Onboarding wiring.** Late, deliberately. Onboarding collects profile data that needs to match settled schema.
7. **Monetization, referrals, polish, TestFlight, store submission.** Standard pipeline from here.

The parking lot retires (or is hidden behind a dev flag) before TestFlight. The gallery viewer is dev infrastructure, not production.

---

## Master prompts

The following prompts encode the pattern. Copy verbatim, fill in placeholders where bracketed, paste into the in-repo agent.

### Prompt 1 — Audit + port plan (read-only)

```
SKILLS: Use design-consistency, building-native-ui skills if available.
This is a SCAN-AND-REPORT prompt — do NOT port any screens, do NOT modify
src/screens/ or registry.ts, do NOT install packages.

Context: a fresh batch of web-JSX screen designs has been dropped into
app-reference/new-screens/. Existing ported screens live in
src/screens/section-*/ and are registered in src/screens/registry.ts. Many
existing screen files have header comments like
  // Port of <source>.jsx → Screen<Name>()
which tell you which JSX export they came from.

GOAL: produce an inventory + port plan. After approval, a follow-up prompt
will execute. This run is read-only.

1. NEW SCREEN EXPORTS
   Walk every .jsx/.tsx under app-reference/new-screens/. List every
   top-level full-screen export with: source file, export name, one-line UI.

2. DEDUPE
   Mark each export DUPLICATE (cite matching src/screens file) or NEW.
   Be conservative — when unsure, mark NEW and flag in judgment calls.

3. PORT PLAN (NEW only)
   - Proposed registry section header (label, e.g. 'Parking Lot · <topic>')
   - Target folder under src/screens/parking-lot/
   - Id-naming scheme
   - Complete mapping: source export → target file → registry id → label

4. PORTING NOTES
   Web-only patterns flagged (CSS Grid, keyframes, radial gradients, etc.),
   new data shapes referenced (will affect future schema), anything broken
   or half-finished in source.

5. JUDGMENT CALLS
   Bullet list of ambiguous duplicates, naming choices, screens that
   probably belong in an existing section vs. parking lot.

OUTPUT
Inline markdown report in your response. No file writes anywhere.

Terminal commands you produce must assume the terminal is already in the
repo root — never prefix with `cd <repo>`. `cd` is only acceptable to
move into a subdirectory of this repo.

When complete, return your inventory + plan + judgment calls as the summary.
```

### Prompt 2 — Execute port

```
SKILLS: Use design-consistency, building-native-ui skills if available.

Execute the port plan you produced. The plan is approved as written.

Build phases — STOP between each phase, show diff, wait for OK before
continuing.

PHASE 1 — Scaffold
- Create src/screens/parking-lot/ if it doesn't exist
- Add the new section header to ScreenSection union and SECTIONS array
  (appended at end) in src/screens/registry.ts
- Do NOT add REGISTRY entries yet — entries land per-screen in Phase 2

→ Show registry.ts diff. STOP.

PHASE 2 — Port screens, one at a time, in flow order
For EACH screen:
  a) Translate web JSX → React Native:
     - Tailwind → StyleSheet + tokens from src/theme/tokens.ts
     - Reuse @/components/primitives and @/components/shared where source
       references the design tool's helper kit (T, Phone, Eyebrow, Heading,
       Body, PrimaryBtn, etc.)
     - CSS keyframes → Reanimated withRepeat/withTiming
     - CSS Grid → flex rows / flex:1 / flexWrap
     - Radial gradient → expo-linear-gradient fallback or
       react-native-svg <RadialGradient>; pick whichever looks closer
     - Box shadow spread → concentric View layering or iOS shadow approx
     - Tooltips → no hardcoded px widths if avoidable, measure or use Modal
  b) Add file header comment: // Port of <source>.jsx → Screen<Name>()
  c) If a token doesn't exist for a new SEMANTIC role, EXTEND tokens.ts.
     Translucent variants of existing tokens stay inline as rgba(...).
  d) Helpers shared across screens in this flow → _<flow>-helpers.tsx in
     the parking lot, NOT promoted to @/components/.
  e) Add the registry entry using the planned id + label.
  f) Inline mock data is fine — match the source JSX.
  g) Run `npx tsc --noEmit` — must pass before moving to next screen.
  h) STOP. Show diff + tsc result. Wait for OK before next screen.

After 3-4 ports, the user may approve batching. Continue with batches of
2-4 screens within a single flow once approved. Never batch across flow
boundaries.

KNOWN HALF-FINISHED SOURCE BITS
Add // TODO comments rather than fabricating values. List specific TODOs
from the port plan here so they're not forgotten.

OUT OF SCOPE — DO NOT TOUCH
- No changes to existing screens outside parking-lot/
- No routing / navigation changes
- No package installs unless a screen is unbuildable without one (and
  then STOP and ask first)

Terminal commands you produce must assume the terminal is already in
the repo root — never prefix with `cd <repo>`. `cd` is only acceptable
to move into a subdirectory of this repo.

When complete, return a summary listing each ported screen + registry id,
any tokens added, any deviations from the plan, and the final tsc result.
```

### Prompt 3 — Verification

```
Run Phase 3 verification:
  1. `npx tsc --noEmit` returns zero errors
  2. `grep -rE "uri: ['\"]http" src/screens/parking-lot/` returns zero
     matches
  3. SECTIONS array has the new section at the end
  4. REGISTRY contains all new ids and no duplicates — list them
  5. Print the count: registry count before vs. after this port

Report results in a table. If anything fails, STOP and tell me before
attempting fixes.
```

### Prompt 4 — Commit

```
Stage all parking-lot work and commit. Use this exact message:

  Add <N> new <description> screens to parking lot

  - <Flow A> (<n>): <screen-list>
  - <Flow B> (<n>): <screen-list>
  ...

  Registry: <X> -> <Y> screens. tsc clean, no remote URIs.

Run:
  git add -A
  git status --short
  git commit -m "<message above>"
  git log -1 --stat

Terminal commands you produce must assume the terminal is already in
the repo root — never prefix with `cd <repo>`. `cd` is only acceptable
to move into a subdirectory of this repo.

When complete, return git status and git log output.
```

---

## Iteration tips

- **Start one-at-a-time, batch after 3-4.** Visual review fatigue is real. Once the agent has the pattern (you've reviewed 3-4 ports and they're all clean), switch to batches of 2-4 screens within a logical flow. Tell the agent explicitly when switching modes.
- **Never batch across flow boundaries.** Finish a flow before starting the next, even when batching. Different flows have different patterns (animation, gradients, layouts) and a flow boundary is a natural review checkpoint.
- **Token additions are review-worthy.** Any new entry in `tokens.ts` should be flagged in the port summary — don't bury it.
- **Watch for primitive promotion candidates.** When a helper appears in 3+ screens with the same shape, it's earned a spot in `@/components/`. Until then, keep it screen-local or flow-local.
- **Track TODOs from source.** Hardcoded values that should later derive from earlier flow steps (e.g. "30%" in a validation screen that should derive from a goal selection) get `// TODO:` comments at the use site. These become the wiring backlog when this flow leaves the parking lot.
- **Inline mock data is intentional.** Don't extract to fixtures yet. The data shapes inform future schema work — keeping them at the use site keeps them visible.
- **Same-flow screens often share patterns.** If flow A's screen 1 used a particular gradient or helper, the agent should reuse the exact same value in flow A's screens 2-4. Cross-flow consistency is less important.

---

## Multiple ingest cycles

Nothing prevents running this pattern more than once per app. A typical app ingest cycle looks like:

```
Cycle 1: Initial design batch (50-100 screens) → numbered sections directly,
         or parking-lot if categorization is ambiguous.
Cycle 2: Conversion/onboarding flows from design tool (10-20 screens)
         → parking-lot.
Cycle 3: New feature surfaces (5-15 screens) → parking-lot.
... (re-categorize between cycles as needed)
```

Each cycle uses the same four phases. Section labels distinguish batches if multiple parking-lot ingests happen before re-categorization (`Parking Lot · Onboarding flows`, `Parking Lot · Settings v2`, etc.).

---

## Failure modes to avoid

- **Wiring during port.** "While I'm in this screen anyway, let me hook up the button to navigate to..." — no. Routing is a separate phase. Stop the agent if it starts adding navigation imports.
- **Promoting helpers too early.** The first screen with a tooltip helper does not need that tooltip in `@/components/`. Wait until 3+ uses confirm the shape.
- **Coining tokens for opacity variants.** `accent06`, `accent08`, `accent10`... no. Inline `rgba(...)`.
- **Skipping the audit phase.** "Just port them" produces ports that miss duplicates and have inconsistent ids. The audit takes 5 minutes and prevents hours of cleanup.
- **Committing mid-port.** Commit once, after Phase 3 verification passes. Half-ported flows in main are noise.
- **Trying to make screens interactive during port.** Visual port only. Buttons render but don't navigate. State is local component state at most. Save interactivity for the wiring phases.
- **Letting the agent improve the design.** The job is faithful translation, not redesign. If the source has a quirky 14-day pip tracker, port it as-is. Improvements happen later via design conversation, not silently during port.

---

## Reference checklist (in order)

- [ ] Pre-flight: drop-in kit baseline present (gallery, registry, tokens, primitives, viewer).
- [ ] Phase 0: bulk JSX in `app-reference/new-screens/`.
- [ ] Prompt 1: audit + port plan, author approves.
- [ ] Prompt 2 Phase 1: scaffold (folder + section), diff approved.
- [ ] Prompt 2 Phase 2: ports, one-at-a-time → batched, each tsc-clean.
- [ ] Prompt 2 Phase 2c: helpers extracted to flow-local file if applicable.
- [ ] Prompt 3: verification table, all checks green.
- [ ] Prompt 4: single commit with descriptive message.
- [ ] Re-categorize when ready (separate phase, separate prompts).

---

## Closing notes

The first time through, the prompts look like walls of text. By the second batch in the same app, they're rote. By the third app, this pattern is the way you bring screens into a repo.

The pattern's value comes from its **constraint**: the agent does one thing, well, repeatedly. Visual translation. Nothing else. That constraint is what makes high-volume screen ingestion work without architectural decisions piling up faster than they can be made.

Re-categorization, routing, auth, schema, and feature wiring all come after. They are easier decisions to make once you can see every screen on a real device.
