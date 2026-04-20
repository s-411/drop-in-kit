# QA_PASS.md — Quick QA Pass

> **What this is:** a prompt template you paste into a fresh Claude Code session when you've just used the app and found things that need fixing. Not a pipeline stage — use it whenever, as many times as needed.
>
> **When to use:** after Stage 6 (main app built), after Stage 13 (TestFlight smoke test), or any time you sit down with the app for 5 minutes and find problems.
>
> **How it works:** you use the app, voice-memo or type everything that's wrong as you encounter it, paste the raw notes into the prompt below, hand it to a fresh agent. The agent parses, verifies, fixes, confirms — one issue at a time.

---

## The Prompt

Open a fresh Claude Code session in the repo. Paste the block below with your notes filled in.

```
SKILLS: Use self-check and building-native-ui skills for this work.

Read CLAUDE.md. This is a QA pass — I've used the app and found issues.
Your job: parse my notes, verify each issue in the code, fix it, confirm
the fix. One issue at a time, in order.

══════════════════════════════════════════════════════════════════
MY NOTES (raw — may be messy, verbal, stream-of-consciousness):

<PASTE YOUR NOTES HERE — voice memo transcription, typed notes,
whatever. Don't clean them up, the agent will parse them.>

══════════════════════════════════════════════════════════════════

STEP 1 — PARSE

Read my notes above. Extract every distinct issue into a numbered list.
For each issue, write:
  - Issue number
  - One-line summary of the problem
  - Where in the app it happens (screen, flow, component — as best
    you can infer from my description)
  - Severity: BROKEN (feature doesn't work), WRONG (works but incorrect
    behavior), UGLY (visual/UX problem), MINOR (polish)

Show me the full numbered list. STOP and wait for my confirmation
before proceeding. I may want to:
  - Remove items that aren't worth fixing right now
  - Add items I forgot
  - Clarify something you misunderstood
  - Reprioritize the order

Do NOT start fixing anything until I say "go" or "approved".

══════════════════════════════════════════════════════════════════

STEP 2 — VERIFY + FIX (one issue at a time, in order)

For each approved issue, follow this loop:

  a) LOCATE — find the relevant code. Name the file(s) and line(s).
     If you can't find it, say so — don't guess.

  b) VERIFY — confirm the issue is real by reading the code logic.
     Explain in 1-2 sentences WHY it's broken/wrong. If the code
     actually looks correct and my description might be wrong,
     say that — don't blindly "fix" working code.

  c) PLAN — describe the fix in 1-2 sentences. Do not implement yet.

  d) FIX — implement the fix. Minimal change — don't refactor
     surrounding code, don't add features, don't "improve" things
     that aren't in the issue list.

  e) TEST — do what you can to verify:
     - Run `npx tsc --noEmit` — zero errors
     - If the fix is logic-based, trace the code path mentally and
       confirm the new behavior matches what I described
     - If the fix is visual, describe what changed so I know what
       to look for when I check on device
     - Run /skill self-check if you've touched multiple files

  f) REPORT — for this issue, state:
     ✅ Issue #N — <one-line summary>
     Found: <file:line>
     Root cause: <one sentence>
     Fix: <what you changed>
     Confidence: HIGH (logic is clear) / MEDIUM (should work but
     test on device) / LOW (uncertain, flagging for manual check)

  Then move to the next issue. Do NOT batch — complete one issue
  fully before starting the next.

══════════════════════════════════════════════════════════════════

STEP 3 — FINAL REPORT

After all issues are fixed, provide:

  - Full list of all issues with their status (✅ fixed / ⚠️ needs
    manual verification / ❌ couldn't fix — explain why)
  - Files touched (deduplicated list)
  - `npx tsc --noEmit` result
  - Anything I should specifically test on device
  - Git commit message suggestion summarizing the QA pass

══════════════════════════════════════════════════════════════════

CONSTRAINTS:

- Use src/theme/theme.ts for all styling. No inline hex.
- Do not add features. Do not refactor beyond the fix. Do not
  touch code unrelated to the issues listed.
- If a fix requires an architectural change (e.g., navigation
  restructure, state management rethink), flag it and STOP —
  don't implement. I'll decide whether to do it now or park it
  in LATER.md.
- Terminal commands you produce must assume the terminal is already
  in the repo root — never prefix with cd <repo>. cd is only
  acceptable to move into a subdirectory of this repo.
```

---

## Tips for good notes

Your notes don't need to be organized. Just describe what you see as you use the app. Examples of useful notes:

> opened the app, went to dashboard, the calories card shows NaN instead of a number. tapped on the moon phase thing and nothing happened — should open the detail view. back button on the settings screen doesn't go back it just sits there. the font on the progress chart title is way too big compared to everything else. when i complete a workout and go back to the home screen the completion doesn't show up until i pull to refresh. the onboarding skip button is almost invisible on the dark background.

That's enough. The agent will parse "NaN on calories card" as issue #1, "moon phase tap doesn't navigate" as issue #2, etc.

**Voice memo workflow:** use your phone's voice memo or transcription app while you use the app. Talk through everything you see. Paste the transcription raw — don't clean it up.

---

## When NOT to use this

- **Don't use for feature requests.** If you want something new, that's Stage 5/6 work or a `LATER.md` item, not a QA fix.
- **Don't use for design overhauls.** If the whole screen needs a redesign, that's Stage 7 polish work.
- **Don't use for performance problems.** Slow screens, laggy scrolling, large bundle size — those need profiling, not a QA pass.

This is for "I used the app and things are broken or wrong." Keep it tactical.
