---
name: app-store-approval
description: Full App Store submission workflow for React Native / Expo apps. Use this skill whenever the user wants to prepare an app for App Store submission, check for rejection risks, generate App Store Connect metadata, audit a codebase before submitting, or needs to know what's missing for App Store approval. Invoke this skill at the start of any App Store submission work — do not guess or rely on memory, read the codebase first.
---

# App Store Approval Skill

## Overview

This skill has two phases. Run them in order:

1. **Phase 1 — Rejection Risk Audit**: Read the codebase and flag every known rejection trigger
2. **Phase 2 — ASC Metadata Generator**: Generate every field needed to complete the App Store Connect submission

Do not guess. Do not assume. Read the actual code — all screens, navigation flows, IAP logic, auth flows, permissions, and metadata files (`app.json`, `app.config.js`, `eas.json`, `package.json`).

---

## PHASE 1 — Rejection Risk Audit

Flag each item as:
- ✅ **CONFIRMED** — found evidence this is handled correctly
- ⚠️ **NEEDS REVIEW** — found something partial or ambiguous
- ❌ **MISSING** — found no evidence this exists

### Critical Rejection Triggers (fix before submitting)

- [ ] **Restore Purchases button** — must be present on every paywall screen with a subscription or non-consumable IAP
- [ ] **Account deletion in-app** — if any account creation exists, deletion must be accessible inside the UI (not email-only, not "contact support")
- [ ] **Sign in with Apple** — required if any third-party social login is used (Google, Facebook, Twitter, etc.)
- [ ] **Subscription paywall disclosure** — paywall must show: price, billing period, privacy policy link, terms of use link, and auto-renewal terms
- [ ] **Block and report** — required if any user-generated content or user-to-user interaction exists
- [ ] **Privacy policy URL** — must be live, public, and present both in-app and in App Store Connect metadata
- [ ] **Terms of use URL** — must be live, public, and linked on the paywall and/or in App Store Connect

### Common Rejection Triggers

- [ ] **No placeholder text** — scan all UI strings for "beta", "coming soon", "placeholder", "lorem ipsum"
- [ ] **No competing platform references** — scan all UI strings and metadata for "Android", "Google Play", or competitor names
- [ ] **Permission usage strings** — every permission in `app.json` / `Info.plist` must have a clear, specific usage description
- [ ] **Privacy manifest** — check if any required reason APIs are used: UserDefaults, file timestamps, system boot time, disk space, active keyboard list. If yes, `PrivacyInfo.xcprivacy` must be present with correct reason codes
- [ ] **No obvious crashes** — check for unhandled promise rejections, missing null checks on navigation params, unguarded API calls
- [ ] **Demo account or demo mode** — reviewers must be able to access all features without real credentials or real device setup

### Metadata Checks

- [ ] **App name** — 30 characters max, matches what appears inside the app
- [ ] **Screenshots** — must show real app UI, not splash screens or login screens only
- [ ] **Age rating** — check for: mature themes, user-generated content, unrestricted web access, gambling, medical info
- [ ] **Support URL** — must resolve to a live, real page
- [ ] **No keyword stuffing** — app name and subtitle must not contain pricing info or irrelevant terms

### Phase 1 Output Format

For each item above, output the flag (✅ / ⚠️ / ❌) and a one-line explanation of what you found (or didn't find) in the code.

Then output a **Prioritised Fix List**: all ❌ items first, then ⚠️ items, with the specific file, component, or line where the fix needs to go.

---

## PHASE 2 — App Store Connect Metadata Generator

Read the entire codebase, then generate every field below. Output in clearly labelled, copy-paste-ready sections. Do not truncate. Do not use placeholder text. Flag anything you can't determine from the codebase.

### 1. App Information

- **App Name** (30 chars max)
- **Subtitle** (30 chars max — one punchy line that complements the name)
- **Primary Category**
- **Secondary Category** (optional)
- **Bundle ID** — read from `app.json` / `eas.json`
- **SKU** — suggest based on bundle ID convention

### 2. App Store Listing — English (US)

- **Promotional Text** (170 chars max — can update without a new build)
- **Description** (4000 chars max — lead with core value prop, explain key features naturally, end with CTA. No competitors. No "coming soon". No "beta".)
- **Keywords** (100 chars max, comma-separated, no spaces after commas — prioritise discoverability, avoid terms already in the app name)
- **Support URL** — read from codebase or flag as MANUAL INPUT REQUIRED
- **Marketing URL** — read from codebase or SKIP
- **Privacy Policy URL** — read from codebase or flag as MANUAL INPUT REQUIRED

### 3. Version Information

- **Version Number** — read from `app.json` / `package.json`
- **Copyright** — format: © YEAR CompanyName
- **What's New** (4000 chars max — output "Initial release." for v1.0)

### 4. App Review Information

- **Notes for App Review** — explain non-obvious features, how to reach key functionality, flows requiring specific state, any restricted API usage
- **Demo Account Username** — read from codebase test credentials or flag as MANUAL INPUT REQUIRED
- **Demo Account Password** — flag as MANUAL INPUT REQUIRED
- **Contact details** — flag as MANUAL INPUT REQUIRED

### 5. Age Rating Questionnaire

Answer based on actual app content:

- Cartoon or Fantasy Violence: NONE / INFREQUENT / FREQUENT
- Realistic Violence: NONE / INFREQUENT / FREQUENT
- Sexual Content or Nudity: NONE / INFREQUENT / FREQUENT
- Profanity or Crude Humor: NONE / INFREQUENT / FREQUENT
- Mature/Suggestive Themes: NONE / INFREQUENT / FREQUENT
- Horror/Fear Themes: NONE / INFREQUENT / FREQUENT
- Medical/Treatment Information: YES / NO
- Alcohol, Tobacco, or Drug Use: NONE / INFREQUENT / FREQUENT
- Gambling and Contests: YES / NO
- Unrestricted Web Access: YES / NO
- User-Generated Content: YES / NO

**Resulting Age Rating:** [generate based on above]

### 6. Privacy — Data Nutrition Labels

For each data type, state COLLECTED / NOT COLLECTED. If collected, state purpose and whether linked to identity or used for tracking.

Read codebase for: auth flows, analytics SDKs, crash reporting, third-party SDKs, network calls, HealthKit, location, camera, contacts, etc.

Data types to cover: Name, Email Address, Phone Number, Physical Address, Other User Contact Info, Health & Fitness, Financial Info, Location (Precise), Location (Coarse), Sensitive Info, Contacts, User Content, Browsing History, Search History, Identifiers (User ID / Device ID), Purchase History, Usage Data, Diagnostics (Crash / Performance)

**Privacy Manifest Required:** YES / NO — list which required reason APIs are used and what reason codes apply.

### 7. In-App Purchases (if applicable)

For each IAP product found in the codebase:
- Product ID, Reference Name, Type, Price Tier
- Display Name (30 chars max), Description (45 chars max)
- Subscription Group Name, Duration, Free Trial: YES / NO

### 8. Pricing & Availability

- **Price:** FREE / Tier [X] — read from codebase or infer from IAP/paywall structure
- **Availability:** All territories or list exceptions
- **Pre-Order:** YES / NO

### 9. Flags & Manual Inputs Required

Numbered checklist of every field that could not be determined from the codebase.

---

## Notes

- Run Phase 1 first. Do not proceed to Phase 2 until all ❌ critical items are resolved.
- If no IAP exists, skip Section 7.
- If no App Clip exists, skip App Clips entirely.
- For account deletion: the in-app flow is non-negotiable. A link to a web form is not sufficient.
- For Sign in with Apple: if Google or Facebook login exists anywhere in the app, Sign in with Apple is required — no exceptions.
