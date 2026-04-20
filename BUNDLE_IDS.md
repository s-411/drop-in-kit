# BUNDLE_IDS.md — Cross-app Tracker

> Single source of truth for which apps you're building, what they're called, what their architecture is (Stack Profile), and where they are in the pipeline.
>
> Working columns are filled at Stage 0. Stack Profile is also locked at Stage 0 (changing later costs days). Final naming columns are locked at Stage 11.

---

## Naming model

**Phase 1 — Working (Stages 0-10)**: pick whatever slug pops in your head. Bundle ID defaults to `com.astrum.<slug-without-hyphens>`. Rename freely.

**Phase 2 — Final (Stage 11+)**: lock the final name + slug + bundle ID. Reserve in App Store Connect at Stage 12.

---

## Stack Profile (locked at Stage 0)

See `STACK_PROFILES.md` for the decision tree.

- **Local** — AsyncStorage, no auth (default for solo trackers)
- **Convex** — Convex DB + Convex Auth (anonymous device-scoped OR real accounts via `@convex-dev/auth`)

---

## Tracker

| # | Working name | Working slug | Stack Profile | Final name | Final slug | Final bundle ID | Status | Stage | ASC reserved? | One-line purpose |
|---|--------------|--------------|---------------|------------|------------|-----------------|--------|-------|:-:|------------------|
| 1 | Vetted | `vetted` | Local | | | | WORKING | 0 | ☐ | |
| 2 | Retardmaxxing | `retardmaxxing` | Local | | | | WORKING | 0 | ☐ | |
| 3 | CPN | `cpn` | **Convex** (with accounts) | | | | WORKING | 0 | ☐ | User content + identity |
| 4 | Sip Less | `sip-less` | Local | | | | WORKING | 0 | ☐ | Quit drinking, women |
| 5 | Peptide Her | `peptide-her` | Local | | | | WORKING | 0 | ☐ | |
| 6 | Lockin | `lockin` | Local | | | | WORKING | 0 | ☐ | |
| 7 | Headsup | `headsup` | Local | | | | WORKING | 0 | ☐ | |
| 8 | Bloom and Breath | `bloom-and-breath` | Local | | | | WORKING | 0 | ☐ | |
| 9 | Sovereign | `sovereign` | Local | | | | WORKING | 0 | ☐ | Quit porn |
| 10 | Male Peptide Tracker | `male-peptide-tracker` | Local | | | | WORKING | 0 | ☐ | |
| 11 | Gym Template | `gym-template` | Local | | | | WORKING | 0 | ☐ | Cloning a fitness influencer's app |
| 12 | _NEW APP 2_ | `tbd-2` | TBD | | | | WORKING | 0 | ☐ | |
| 13 | _NEW APP 3_ | `tbd-3` | TBD | | | | WORKING | 0 | ☐ | |
| 14 | _NEW APP 4_ | `tbd-4` | TBD | | | | WORKING | 0 | ☐ | |
| 15 | _NEW APP 5_ | `tbd-5` | TBD | | | | WORKING | 0 | ☐ | |

---

## Conventions

### Slugs
- Lowercase, hyphenated only: `sip-less`, `peptide-her`
- No dots, no underscores, no spaces

### Bundle IDs
- Format: `com.astrum.<slug-with-hyphens-removed>`
- E.g. `sip-less` → `com.astrum.sipless`
- Once reserved in ASC, **cannot be reused**, ever. Triple-check before reserving at Stage 12.

### Stack Profiles
- **Default to Local** unless you have a clear "this needs accounts or multi-user" reason
- Pick at Stage 0 — changing later costs days
- See `STACK_PROFILES.md` migration paths if you must change

### Stage 0 vs Stage 12
- **Stage 0:** fill working name + working slug + Stack Profile + one-line purpose
- **Stage 12:** lock final name + slug + bundle ID, reserve in ASC

---

## When you finalize a name (at Stage 11)

1. Add the final name + slug + bundle ID to the row above
2. Mark Status: LOCKED
3. Update `app.json` in the repo (`expo.name`, `expo.slug`, `expo.ios.bundleIdentifier`)
4. Remove the `_naming_status` working-name comment from `app.json`
5. Update `CLAUDE.md` header in the repo
6. Optionally rename the GitHub repo (Repository → Repository Settings)
7. Reserve the bundle ID in App Store Connect:
   - appstoreconnect.apple.com → My Apps → ➕ → New App
   - Bundle ID: the final one
   - SKU: the final slug
   - Primary Language: English (US)
   - Platform: iOS
8. Note the **ascAppId** App Store Connect assigns (10-digit number)
9. Update `eas.json` `submit.production.ios.ascAppId` with that ID
10. Tick the ASC reserved box above
