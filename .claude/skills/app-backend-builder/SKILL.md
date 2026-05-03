---
name: app-backend-builder
description: Reads an incomplete React Native / Expo app that has onboarding and/or a paywall but no core app functionality, and generates a comprehensive PRD and phased build plan for the backend of the app. Use this skill whenever an app has onboarding screens but is missing its core loop, dashboard, settings, navigation, or main features. Invoke this skill when the user says an app is 50% done, has onboarding but no app, needs its backend figured out, or needs to know what to build next. This skill interprets what the app is about from existing screens and generates what a complete MVP would look like — without payments.
---

# App Backend Builder Skill

## Overview

This skill reads what exists, figures out what the app is actually *for*, and generates a complete plan for building the rest of it — the parts that make it a real app, not just an onboarding flow.

Output is a `BUILD_PLAN.md` file the user can hand to a fresh Claude Code agent to execute.

---

## Step 1 — Read Everything That Exists

Scan the entire codebase. Document:

### Existing Screens
List every screen file found. For each one note:
- Screen name / file path
- What it appears to do (1 line)
- Which category it falls into: `onboarding` / `paywall` / `auth` / `app` / `unknown`

### App Identity
From the screen content, copy, assets, and any constants/config files, extract:
- **App name**
- **Target user** (who is this for?)
- **Core problem** (what does this app help them do or stop doing?)
- **Tone** (clinical, warm, motivational, gamified, minimal, etc.)
- **Any features partially built** (e.g. a tracker screen that exists but isn't wired)

### Tech Stack
From `package.json`, `app.json`, `eas.json`:
- Navigation library in use
- State management (if any)
- Backend / database (Supabase, Firebase, AsyncStorage only, etc.)
- Any SDKs already installed that imply features (e.g. `react-native-health`, `expo-notifications`)

---

## Step 2 — Paywall & Onboarding Handling

If a paywall screen exists and payments are NOT being implemented yet:

- [ ] Identify which screen in the onboarding flow leads to the paywall
- [ ] The paywall must be **bypassed** — add a "Continue for free" or "Skip" button, or reroute the navigation so the paywall is not shown
- [ ] Do NOT delete the paywall screen — just remove it from the active navigation flow
- [ ] Note the bypass in the build plan so it can be reversed when payments are added later

---

## Step 3 — Infer the Core App

Based on what the app is about, reason through what a complete MVP needs. Use the reference file for domain-specific feature patterns:

→ Read `.claude/skills/app-backend-builder/references/app-patterns.md` for common app archetypes and their standard feature sets.

Then define for THIS app:

### Core Loop
What does the user actually *do* every time they open the app? (e.g. log something, check progress, complete a task, read content)

### Bottom Navigation Tabs
What are the 3–5 main sections? Typical structure:
- **Home / Dashboard** — the first thing they see after onboarding
- **[Core feature tab]** — the main action (log, track, browse, etc.)
- **Progress / Stats** — history and streaks
- **Settings / Profile** — account, preferences, notifications

Propose the exact tab names and icons appropriate to this app's tone.

### Screens to Build
List every screen that needs to be created, with:
- Screen name
- Purpose (1 line)
- Key UI elements
- Data it needs to display or capture

### Data Model
What data does this app need to store? For each entity:
- Name
- Fields
- Where it's stored (AsyncStorage for simple local data, Supabase/Firebase if already configured)

### Notifications
Does this app benefit from push notifications? If yes:
- What triggers them (daily reminder, streak alert, milestone, etc.)
- `expo-notifications` is already available in Expo — note if it needs to be added

---

## Step 4 — Generate BUILD_PLAN.md

Write the full build plan to `BUILD_PLAN.md` in the repo root.

Use the template in `.claude/skills/app-backend-builder/references/build-plan-template.md`.

The plan must be:
- **Phased** — broken into phases a Claude agent can execute one at a time
- **Specific** — each task names the exact file, component, or screen to create
- **Verifiable** — each phase ends with a verification step the agent can run
- **Scoped** — no payments, no advanced features, MVP only

---

## Step 5 — Handoff Instructions

After writing `BUILD_PLAN.md`, output this message to the user:

```
BUILD_PLAN.md has been written to the repo root.

To execute it, start a fresh Claude Code session and use this prompt:

"Read CLAUDE.md and BUILD_PLAN.md. Enter plan mode. 
Review the build plan — do you have any questions before we start? 
Once confirmed, begin Phase 1. After completing each phase, 
run the self-check skill to verify everything works before moving to the next phase."

Do not start building in this session — hand it to a fresh agent with full context.
```

---

## Notes

- If the app has both onboarding AND some partial app screens, include those partial screens in the plan and note what's missing from them
- If no backend SDK is configured (no Supabase, no Firebase), default to `AsyncStorage` for MVP data persistence — it's already available in Expo and requires no setup
- Notifications should be included in the plan if the app's core loop benefits from daily engagement (habit, health, productivity, journaling apps almost always do)
- The goal is a working app a real user could use — not a demo, not a prototype
