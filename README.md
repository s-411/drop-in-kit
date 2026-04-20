# drop-in-kit

Canonical source for the Expo / React Native app factory drop-in kit. Ships as plain files — drop them into any app repo to align it with the master pipeline.

## Install into a new or existing app repo

From the repo root:

```bash
npx degit s-411/drop-in-kit --force
```

Downloads the current state as plain files. No git history, no submodule. `--force` overwrites existing files. Re-run any time to refresh.

## Stack

Two profiles: **Local** (AsyncStorage, no auth) or **Convex** (Convex DB + Convex Auth). Clerk is intentionally not used in RN apps — see `STACK_PROFILES.md`.

## Kit conventions

- Bundle ID prefix: `com.astrum.<slug>`
- Terminal commands: always assume terminal is in repo root — never `cd <repo>`
- Nanobanana (Gemini) MCP: mandatory on every new repo (step 3a in `START_NEW.md`)
- All colors/fonts/spacing via `src/theme/theme.ts` — no inline hex
- Global `KeyboardHideButton` required on every app — mounted once at app root
- Stage 8a wires legal URLs into code only (pages 404 until Stage 8b)
- Stage 8b builds the landing page on applanding.co — MANDATORY before Stage 16
- Stage 14: mandatory iPad + iPhone screenshots, outpaint-first via Gemini MCP, never letterbox
- Stage 15: `asc-metadata.md` follows strict ASC navigation order
- Every stage prompt includes a `SKILLS:` line telling the agent which platform skills to invoke

## Related

- Web kit: [s-411/web-drop-in-kit](https://github.com/s-411/web-drop-in-kit) — Next.js + Convex + Clerk + Stripe variant
