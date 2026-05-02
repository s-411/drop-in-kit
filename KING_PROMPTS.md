# King Prompts — paste at the start of every new sub-chat

Pick the scenario that matches, fill in the fields, paste the filled block as your first message in a fresh sub-chat. If every field is filled, the kickoff interview is satisfied and Claude proceeds straight to the next action.

---

## Scenario A — Brand New App

```
New app sub-chat. Before doing anything else, run the kickoff interview.

- App name / working slug:
- Purpose:
- Stack Profile (Local | Convex):
- Current stage: 0
- Repo: brand new (use START_NEW.md)
- Notes:
```

Action implied: walk through `START_NEW.md` step by step. Nanobanana MCP install at step 3a is mandatory.

---

## Scenario B — Resuming a Paused App

```
New app sub-chat. Before doing anything else, run the kickoff interview.

- App name / working slug:
- Purpose:
- Stack Profile (Local | Convex):
- Current stage:
- Repo: existing, last worked on:
- Notes:
```

Action implied: run `BOOTSTRAP.md` first to verify kit state (especially if the gap since last work is more than a few days, since the kit may have been refreshed). Then resume from the stage listed above.

---

## Scenario C — Active Build (recent work, state is fresh)

```
New app sub-chat. Before doing anything else, run the kickoff interview.

- App name / working slug:
- Purpose:
- Stack Profile (Local | Convex):
- Current stage:
- Repo: existing, worked on recently
- Notes:
```

Action implied: skip `BOOTSTRAP.md`, go straight to the next prompt for the current stage from `PROMPTS.md`.

---

## Field reference

- **Stack Profile** — `Local` (AsyncStorage, no auth, no cloud) or `Convex` (Convex DB + Convex Auth, anonymous device-scoped OR account-based). Default to Local unless there's a clear "needs cloud" reason. See `STACK_PROFILES.md`.
- **Current stage** — `0` through `16`. Stage 8 is split into `8a` (legal URLs wiring, in pipeline position) and `8b` (branded landing page, after Stage 15). See `PROCESS_GUIDE.md`.
- **Notes** — anything non-default: existing wiring (RevenueCat / Sentry / PostHog already in?), brand colors locked already, client constraints, timeline pressure, or anything weird about the repo state.
