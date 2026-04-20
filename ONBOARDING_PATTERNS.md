# ONBOARDING_PATTERNS.md

> **Menu of onboarding screen patterns** proven to lift conversion. Pick from this list at Stage 4 to build the emotional arc for each app.
>
> Source: screensdesign.com patterns (what top-grossing apps actually ship).

---

## How to use this file

At Stage 4 (onboarding hardening), pick 5–8 patterns from the menu below. Order them into an emotional arc: **hook → personalize → validate → convince → convert.**

Paste the pattern names (or numbers) into the Stage 4 prompt. The agent builds one screen per pattern, using your existing `theme.ts` tokens.

Also decide: **how many paywall variants?** (1 = just the main paywall, 2 = main + discounted fallback, 3 = main + discounted + annual)

---

## The 12 patterns

### 1. Educational — "Build knowledge"
Teach the user something they didn't know (or confirm something they suspected). Primes them to trust the app as a source of truth.
**Example:** "Most men lose 1% of muscle per year after 30 — here's why"
**Use when:** audience is unfamiliar with the domain, or app solves a non-obvious problem

### 2. Social Proof — "Show experiences"
Real users, real quotes, real faces. Stars and numbers. The "others like me succeeded" moment.
**Example:** "Over 50,000 men have gained 15+ lbs using this system"
**Use when:** you need trust, especially for paid apps — this is the #1 conversion lever

### 3. Trust Signals — "Build authority"
Logos, certifications, press mentions, expert endorsements. "As seen in…"
**Example:** Mike Thurston's credentials, "10 years of training experience"
**Use when:** creator-led apps, apps with any medical/health angle, premium pricing

### 4. Quiz — "Interactive Q&A"
Multiple-choice questions that feel like discovery for the user but are secretly data-gathering for personalization.
**Example:** "What's your current training experience? / How many days per week can you commit? / What's your primary goal?"
**Use when:** ALWAYS — foundational. The quiz data drives every downstream personalization screen.

### 5. Reinforcement — "Validate choices"
After each quiz answer (or at key points), show validation: "Great choice — this is what most successful users pick."
**Example:** User picks "strength" → "Strength-focused training is the #1 most effective approach for body recomposition"
**Use when:** longer quizzes (5+ questions) where drop-off risk is high; keeps momentum

### 6. Results — "Show results"
Graphs, numbers, before-and-after metrics. "Users like you gained X in Y weeks."
**Example:** Projected strength curve over 12 weeks based on quiz answers
**Use when:** quantifiable outcomes exist (weight loss, strength, sleep quality, savings). Essential for fitness/health/finance apps.

### 7. Summary — "Recap info"
"Here's everything we know about you" — reflects the quiz answers back. Creates ownership.
**Example:** "Age: 28 / Goal: Build muscle / 4 days/week / Moderate experience"
**Use when:** long onboarding flows where user needs to feel the system "got" them before paying

### 8. Before/After — "Transformation"
Split-screen visual of current state vs potential future state. Heavy emotional lever.
**Example:** "Skinny you today / Jacked you in 6 months" (illustrated or photo)
**Use when:** visual transformation is the goal (fitness, skincare, home improvement). Be careful with medical.

### 9. FOMO — "Create urgency"
Limited-time offer, seat count, streak countdown. Pressure at the paywall moment.
**Example:** "This offer expires in 24 hours" / "Only 47 spots left at this price"
**Use when:** the paywall variant needs a kick. Use sparingly — overuse breaks trust.

### 10. Gamification — "Game elements"
Streaks, levels, progress rings, achievements. Makes "daily use" feel like winning.
**Example:** "You're on track to unlock Level 1 — just complete your first workout"
**Use when:** habit-forming apps (tracking, journaling, language learning). Less useful for one-shot utility apps.

### 11. Problem/Solution — "Pain points"
Name the user's pain in their words, then position the app as the fix.
**Example:** "Tired of bro-science? Confused about macros? Stuck at the same weight?" → "Here's the system that actually works"
**Use when:** commodity category where differentiation matters (yet another habit tracker, yet another meditation app)

### 12. Features — "Show benefits"
The classic "here's what you get" screen. Icons + one-liners per feature.
**Example:** "📊 Custom workout plan / 🍽️ Macro calculator / 🏆 Weekly progress tracking"
**Use when:** late in the flow, just before paywall. Converts abstract value into concrete.

---

## Canonical emotional arcs (pick one or remix)

### The Premium Fitness arc (THRST, Caliber, Future)
```
Welcome → Sign Up → Quiz → Reinforcement → Trust Signals → Before/After
→ Results → Social Proof → Features → Custom Plan → Paywall (1-2 variants)
```

### The Habit App arc (Finch, Fabulous, Streaks)
```
Welcome → Problem/Solution → Quiz → Results → Summary → Gamification
→ Features → Paywall (1-2 variants)
```

### The Short-and-Sweet arc (for utility apps)
```
Welcome → Quiz → Results → Features → Paywall (1 variant)
```

### The Social Proof Heavy arc (for creator-led apps)
```
Welcome → Trust Signals → Social Proof → Quiz → Before/After → Summary
→ Paywall (2-3 variants)
```

---

## Paywall variants — how many to ship

| Variants | When to use |
|----------|-------------|
| **1** (main only) | Testing / v1 / you don't have pricing tiers figured out |
| **2** (main + discounted) | User skips main → gets "wait, here's a better offer" second chance |
| **3** (main + discounted + annual) | Full conversion funnel — annual is shown first as "save 60%", main monthly is the fallback |

Default for apps: **1 variant in v1, add discounted fallback at v1.1** (park in `LATER.md`).

---

## Rules for all patterns

Regardless of which patterns you pick:

- Every screen uses `src/theme/theme.ts` tokens — no inline hex
- Top progress bar across the whole flow (already exists from Stage 3)
- Quiz answers persist to AsyncStorage and feed every downstream screen's personalization
- Paywall has visible skip link (real payment hookup is in `LATER.md`)
- Max 8–9 screens total — longer than that kills conversion
- Every screen has both a way forward AND a way back (except Welcome)

---

## Per-app guidance cheat sheet

| App type | Recommended patterns | # paywalls |
|----------|---------------------|------------|
| Fitness / training | 2, 3, 4, 6, 8, 12 | 1-2 |
| Quit-bad-habit (porn, drinking, smoking) | 1, 4, 7, 10, 11, 12 | 2 |
| Tracker (habits, journal) | 4, 6, 7, 10, 12 | 1 |
| Creator-led content | 2, 3, 4, 6, 12 | 2-3 |
| Medical / health logging | 1, 3, 4, 6, 7, 12 | 1 |
| Productivity / focus | 4, 7, 10, 11, 12 | 1 |

---

## When you update this file

After each app ships, note which patterns converted best. Add findings to `LATER.md` under "cross-app roadmap" and promote winning combinations here. This file gets smarter with every app.
