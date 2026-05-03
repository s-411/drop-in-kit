# App Patterns Reference

Common app archetypes and their standard MVP feature sets. Use this to infer what needs to be built based on what the app is about.

---

## Health & Habit Apps
*(quit drinking, reduce alcohol, quit smoking, fitness streaks, meditation, sleep)*

**Core loop:** User logs something daily (a drink, a cigarette, a mood, a workout)

**Standard screens:**
- Dashboard — today's status, streak counter, motivational message
- Log / Check-in — primary action screen (quick, frictionless)
- Progress — calendar heatmap or chart of streaks over time
- Milestones — badges or achievements for hitting goals
- Settings — daily reminder time, goal settings, notification preferences

**Data model:**
- `entries` — date, value logged, notes
- `streak` — current streak, longest streak, last entry date
- `goal` — target (e.g. max drinks per week, days sober)
- `user_preferences` — reminder time, notification on/off

**Notifications:** Daily reminder at user-set time. Streak milestone alerts.

**Nav tabs:** Home, Log, Progress, Settings

---

## Journaling & Reflection Apps
*(gratitude, mood tracking, daily reflection, mental health check-ins)*

**Core loop:** User writes or selects responses to prompts daily

**Standard screens:**
- Home — today's prompt or mood check-in CTA
- Write / Reflect — the journal entry screen
- History — past entries, browsable by date
- Insights — mood trends over time (simple chart)
- Settings — reminder time, prompt preferences, export

**Data model:**
- `entries` — date, prompt, response, mood score
- `moods` — score, label, emoji
- `user_preferences` — reminder time, prompt style

**Notifications:** Daily journaling reminder.

**Nav tabs:** Home, Write, History, Insights, Settings

---

## Learning & Skill-Building Apps
*(language learning, courses, flashcards, reading habits)*

**Core loop:** User completes a lesson or practice session

**Standard screens:**
- Home — daily lesson / next up / streak
- Learn — the lesson or exercise screen
- Library — all available content, organised by topic or level
- Progress — lessons completed, time spent, streak
- Settings — daily goal, reminder, difficulty

**Data model:**
- `lessons` — id, title, content, category, completed
- `progress` — lesson_id, completed_at, score
- `streak` — current, longest
- `user_preferences` — daily goal (minutes or lessons), reminder time

**Notifications:** Daily learning reminder. Streak at-risk alert.

**Nav tabs:** Home, Learn, Library, Progress, Settings

---

## Productivity & Task Apps
*(to-do, focus timer, goal tracker, project management lite)*

**Core loop:** User adds tasks, completes them, sees progress

**Standard screens:**
- Home — today's tasks / focus area
- Tasks — full task list, add/complete/delete
- Focus — pomodoro or timer screen (if applicable)
- Stats — tasks completed over time
- Settings — work session length, break length, notifications

**Data model:**
- `tasks` — id, title, due_date, completed, priority, category
- `sessions` — start_time, end_time, tasks_completed
- `user_preferences` — session length, break length

**Notifications:** Daily planning reminder. Task due reminders.

**Nav tabs:** Today, Tasks, Focus, Stats, Settings

---

## Finance & Money Apps
*(expense tracking, savings goals, budget, spending awareness)*

**Core loop:** User logs an expense or checks their budget

**Standard screens:**
- Dashboard — spending this month vs budget, savings progress
- Add Expense — quick log screen
- Transactions — history, filterable by category
- Budget — set and edit category budgets
- Settings — currency, budget reset day, categories

**Data model:**
- `transactions` — date, amount, category, notes
- `budgets` — category, monthly_limit
- `savings_goals` — name, target_amount, current_amount, deadline

**Notifications:** Budget alert (approaching limit). Weekly spending summary.

**Nav tabs:** Dashboard, Add, Transactions, Budget, Settings

---

## Social & Community Apps
*(shared goals, accountability partners, group challenges)*

**Core loop:** User logs progress and sees others' progress

**Standard screens:**
- Feed — activity from connections or group
- My Progress — personal stats and log
- Leaderboard or Group — shared view
- Profile — user's own profile and history
- Settings — privacy, notifications, profile edit

**Data model:**
- `users` — id, name, avatar, join_date
- `entries` — user_id, date, value, note
- `connections` — user_id, friend_id, status

**Notifications:** Friend activity. Challenge milestones. Weekly recap.

**Nav tabs:** Feed, Log, Leaderboard, Profile, Settings

---

## Content & Discovery Apps
*(recipes, workouts, meditation sessions, articles)*

**Core loop:** User browses, saves, and consumes content

**Standard screens:**
- Home — featured / recommended content
- Browse / Explore — filtered content library
- Detail — individual content item (recipe, workout, article)
- Saved / Favourites — bookmarked items
- Settings / Profile

**Data model:**
- `content` — id, title, category, tags, body/url, duration
- `saved` — user_id, content_id, saved_at
- `completed` — user_id, content_id, completed_at

**Notifications:** New content alert. Streak or completion milestone.

**Nav tabs:** Home, Explore, Saved, Profile, Settings

---

## General MVP Principles

Regardless of app type, every MVP needs:

1. **A home screen** — the first thing after onboarding. Shows today's status, primary CTA, and streak/progress if relevant.
2. **The core action screen** — the main thing the user does. Should be reachable in 1 tap from home.
3. **A history/progress screen** — proves the app is working. Even a simple list of past entries.
4. **A settings screen** — at minimum: notification toggle + time, and a way to reset or log out.
5. **An empty state** for every list — tells the user what to do when there's no data yet.
6. **A loading state** for any async data fetch.
