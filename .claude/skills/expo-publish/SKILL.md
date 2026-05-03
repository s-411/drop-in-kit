---
name: expo-publish
description: How to correctly build and publish a React Native / Expo app using EAS. Use this skill whenever the user wants to create a new build, submit to TestFlight, bump a version number, configure eas.json, troubleshoot EAS build failures, or push an update via Expo Updates. Invoke this skill before running any eas build or eas submit command.
---

# Expo Publish Skill

## Overview

This skill covers the full EAS build and submission flow for React Native / Expo apps. Work through the checklist in order. Do not skip steps.

---

## Pre-Build Checklist

Read `app.json`, `app.config.js`, `eas.json`, and `package.json` before doing anything. Confirm:

- [ ] `expo.version` is correct and has been bumped if this is an update
- [ ] `expo.ios.buildNumber` has been incremented (must be higher than the last submitted build)
- [ ] `expo.ios.bundleIdentifier` matches what's registered in App Store Connect
- [ ] `expo.name` matches the app name in App Store Connect (max 30 chars)
- [ ] `expo.splash` and `expo.icon` point to real files that exist
- [ ] No `"beta"`, `"coming soon"`, or placeholder strings in `app.json`
- [ ] `eas.json` has a `production` build profile configured

### eas.json minimum production profile

```json
{
  "build": {
    "production": {
      "autoIncrement": true,
      "ios": {
        "simulator": false,
        "credentialsSource": "remote"
      }
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "MANUAL INPUT REQUIRED",
        "ascAppId": "MANUAL INPUT REQUIRED",
        "appleTeamId": "MANUAL INPUT REQUIRED"
      }
    }
  }
}
```

---

## Build Commands

### Production build (iOS)
```bash
eas build --platform ios --profile production
```

### Submit to TestFlight immediately after build
```bash
eas build --platform ios --profile production --auto-submit
```

### Submit an existing build to TestFlight
```bash
eas submit --platform ios --profile production
```

### Check build status
```bash
eas build:list --platform ios --limit 5
```

---

## Version Bumping Rules

- **Patch update** (bug fix, minor UI change): bump `version` patch digit, e.g. `1.0.0` → `1.0.1`
- **Feature update**: bump minor digit, e.g. `1.0.1` → `1.1.0`
- **`buildNumber`** must always increment — never reuse a build number
- If `autoIncrement: true` is set in `eas.json`, EAS handles `buildNumber` automatically

---

## Common Failures & Fixes

| Error | Fix |
|-------|-----|
| `bundleIdentifier does not match` | Check `expo.ios.bundleIdentifier` in `app.json` matches App Store Connect exactly |
| `build number already exists` | Increment `expo.ios.buildNumber` or enable `autoIncrement` |
| `missing push notification entitlement` | Add `"expo.ios.entitlements": { "aps-environment": "production" }` to `app.json` |
| `icon must be 1024x1024` | Replace icon asset, rerun build |
| `credentials not found` | Run `eas credentials` and follow prompts |
| `expo-updates not configured` | Add `expo-updates` to `app.json` plugins if using OTA updates |

---

## After Build Completes

1. Log into [App Store Connect](https://appstoreconnect.apple.com)
2. Navigate to your app → TestFlight
3. Wait for build processing (usually 5–15 min after EAS reports success)
4. Add the build to a TestFlight group for internal testing
5. If submitting for App Store review, invoke the `app-store-approval` skill first

---

## Notes

- Always use `--profile production` for App Store builds — never `development` or `preview`
- `eas submit` without `--auto-submit` lets you choose which build to submit from a list
- OTA updates via `expo-updates` do NOT require a new build — use `eas update` for JS-only changes
- If the build fails on EAS servers, check the full build log at `https://expo.dev`
