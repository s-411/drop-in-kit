---
name: self-check
description: Claude reviews and verifies its own work before marking any task complete. Use this skill after completing any significant change — new feature, bug fix, screen wiring, or refactor. Invoke this skill whenever the user asks Claude to check its work, verify output, or before calling a task done. Do not mark work as complete without running this skill.
---

# Self-Check Skill

## Overview

Before marking any task done, run this verification pass. Be honest. If something is broken or incomplete, say so — do not paper over it.

---

## Step 1 — Restate What Was Asked

In one paragraph, restate:
- What the user asked for
- What you actually built or changed
- Any scope you cut or deferred

If these don't match, flag it before continuing.

---

## Step 2 — Code Review Pass

Re-read every file you touched. Check for:

- [ ] **No hardcoded values** that should be environment variables or constants
- [ ] **No commented-out code** left behind from previous attempts
- [ ] **No console.log** statements left in production code
- [ ] **No TODO comments** added and left unresolved
- [ ] **No imports** that are unused
- [ ] **No files** created but never imported or referenced
- [ ] **No duplicate logic** — same function written twice in different places
- [ ] **Error handling** — async calls have try/catch or `.catch()`, API responses are checked before use
- [ ] **Null safety** — navigation params, API responses, and optional chained values are guarded

---

## Step 3 — Feature Verification

For the specific task completed, confirm:

- [ ] The feature works for the **happy path** (normal user flow)
- [ ] The feature handles the **empty state** (no data, first load)
- [ ] The feature handles **errors** (network failure, bad input)
- [ ] Any new screen is **reachable** from the existing navigation
- [ ] Any new screen has a **way back** (back button, tab, or explicit close)
- [ ] UI matches the **existing design system** — same fonts, colours, spacing patterns as surrounding screens

---

## Step 4 — Regression Check

List every file you modified. For each one, confirm:
- The change was intentional
- The change doesn't break any other screen or feature that depended on that file
- If you changed a shared component, every screen that uses it still works correctly

---

## Step 5 — Output

Produce a short summary:

```
✅ Task completed: [what was built]

Files changed: [list]

Verified:
- Happy path: ✅ / ❌
- Empty state: ✅ / ❌
- Error handling: ✅ / ❌
- Navigation reachable: ✅ / ❌
- No regressions: ✅ / ❌

Issues found: [list any ⚠️ or ❌ items and what needs fixing]

Ready to proceed: YES / NO
```

If **NO**, fix the issues before marking the task done.

---

## Notes

- This skill is not optional. Run it after every significant change.
- "It looks right" is not verification. Trace the actual code path.
- If you can't verify something (e.g. can't run the app), say so explicitly — don't assume it works.
