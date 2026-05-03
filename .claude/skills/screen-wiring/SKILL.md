---
name: screen-wiring
description: Audit all screens in a React Native / Expo app to ensure they are connected, navigable, and reachable. Use this skill whenever screens may be missing from navigation, buttons don't route anywhere, or the app needs a full navigation audit before submission. Invoke this skill before any App Store submission or when the user says screens aren't wired up, flows are broken, or navigation needs checking.
---

# Screen Wiring Skill

## Overview

Read the entire codebase — every screen file, every navigator, every navigation call — and produce a complete map of what's connected and what isn't. Fix broken wiring. Do not modify screen UI or logic.

---

## Step 1 — Map Every Screen

Find all screen components. Look in:
- `screens/`, `src/screens/`, `app/` (Expo Router), `pages/`
- Any file that renders a full-screen view and is referenced in a navigator

List every screen found with its file path.

---

## Step 2 — Map Every Navigator

Find all navigators:
- Stack navigators (`createNativeStackNavigator`, `createStackNavigator`)
- Tab navigators (`createBottomTabNavigator`)
- Drawer navigators (`createDrawerNavigator`)
- Expo Router file-based routes (`app/` directory)

For each navigator, list:
- Which screens are registered in it
- The screen name used in navigation calls

---

## Step 3 — Audit Navigation Calls

Search the entire codebase for:
- `navigation.navigate(`
- `router.push(` / `router.replace(` (Expo Router)
- `<Link href=` (Expo Router)
- `navigation.goBack()`
- Any `onPress` that should trigger navigation

For each call, confirm the target screen name exists in a navigator. Flag any that don't match.

---

## Step 4 — Check Every Entry Point

Confirm a user can reach every screen from the app's normal flow:

- [ ] Every tab in the tab bar routes to a real screen
- [ ] Every button labelled with an action (e.g. "View Details", "Settings", "Back") navigates somewhere
- [ ] No screen is an island — every screen is reachable from at least one other screen
- [ ] The initial route is correctly set and loads without error
- [ ] Auth flow correctly gates protected screens (logged-out users can't reach authenticated screens)
- [ ] Back navigation works on every screen that isn't a root tab

---

## Step 5 — Flag & Fix

Output a table:

| Screen | Reachable? | Issue | Fix |
|--------|-----------|-------|-----|
| HomeScreen | ✅ | — | — |
| ProfileScreen | ❌ | Not registered in any navigator | Add to StackNavigator in `App.tsx` |
| SettingsScreen | ⚠️ | Registered but no button navigates to it | Add Settings button to ProfileScreen header |

Then apply all fixes. Only touch navigation wiring — do not modify screen content, styles, or business logic.

---

## Step 6 — Verify

After fixing:
1. Trace a complete user journey from app open → every main feature → back to home
2. Confirm no `undefined` screen name warnings would appear in the console
3. Confirm the tab bar (if present) shows the correct icons and labels for every tab

---

## Notes

- If using Expo Router, the file structure IS the navigation structure — missing files mean missing routes
- A screen imported but not added to a navigator will silently never appear
- `navigation.navigate('ScreenName')` is case-sensitive — mismatches cause silent failures
- Modals must be registered in the navigator that contains them, not a child navigator
