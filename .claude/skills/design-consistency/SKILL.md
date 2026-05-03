---
name: design-consistency
description: Ensure new and modified screens stay consistent with the existing design system. Use this skill before and after building any new screen or UI component, when reviewing screens for visual consistency, or when Claude has drifted from the existing design. Invoke this skill whenever adding UI to prevent visual inconsistency across the app.
---

# Design Consistency Skill

## Overview

Before writing any UI code, read the existing codebase to understand the design system in use. Then build to match it — do not invent new patterns.

---

## Step 1 — Extract the Design System

Read the codebase and document what's already in use:

### Colours
Find the colour constants file (e.g. `constants/Colors.ts`, `theme.ts`, `styles/colors.ts`). List:
- Background colours
- Primary / accent colours
- Text colours (primary, secondary, muted)
- Error / success colours
- Border colours

If no constants file exists, extract the hex values used most frequently across existing screens.

### Typography
Find the font configuration. Document:
- Font family / families in use
- Font sizes used for: headings, body, labels, captions, buttons
- Font weights used
- Line heights if specified

### Spacing
Identify the spacing scale in use (e.g. multiples of 4, 8, or 16). Document:
- Standard padding values for screens
- Standard margin values between elements
- Gap between list items

### Components
List the shared/reusable components already in the codebase:
- Buttons (primary, secondary, destructive)
- Input fields
- Cards
- Headers / navigation headers
- Empty states
- Loading indicators
- Modals / bottom sheets

---

## Step 2 — Consistency Checklist (Before Building)

Before writing any new UI:

- [ ] I know which colour constants to use for background, text, and accents
- [ ] I know the correct font sizes for headings and body text
- [ ] I know the standard screen padding value
- [ ] I know which existing components to reuse instead of creating new ones
- [ ] I have checked that a similar screen already exists that I can reference

---

## Step 3 — Build to Match

When writing new UI:

- Import colours from the constants file — never hardcode hex values
- Use the same `StyleSheet.create` pattern as surrounding files
- Reuse existing components — do not create a new Button if `components/Button.tsx` exists
- Match the header style of adjacent screens exactly
- Use the same loading indicator (spinner, skeleton) as the rest of the app
- Use the same empty state pattern as the rest of the app

---

## Step 4 — Consistency Audit (After Building)

Compare the new screen against 2–3 existing screens. Check:

- [ ] Background colour matches
- [ ] Font sizes match for equivalent content types
- [ ] Padding/margins match
- [ ] Button styles match
- [ ] Header style matches (title size, back button, right actions)
- [ ] No new colours introduced that don't exist in the constants file
- [ ] No new font sizes introduced that don't match the existing scale
- [ ] Icons use the same icon library as the rest of the app

---

## Step 5 — Output

```
Design system extracted from: [file paths]

New screen: [screen name]
Colours used: [list — all from constants? ✅ / ❌]
Fonts used: [list — matches existing scale? ✅ / ❌]
Spacing: [matches existing pattern? ✅ / ❌]
Reused components: [list]
New components created: [list — justified? ✅ / ❌]

Consistency with adjacent screens: ✅ consistent / ⚠️ minor drift / ❌ significant drift
Issues: [list anything that needs fixing]
```

---

## Notes

- If there is no design system yet, document what you find and propose a minimal constants file — but don't build one unless asked
- "Close enough" is not consistent — match exactly
- Dark mode: if the app supports it, check both modes
- Do not change existing screen styles to match new ones — new screens adapt to the existing system, not the other way around
