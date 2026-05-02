# Mobile App Onboarding Deep Dive

*A screen-by-screen breakdown of a $40k/month app's onboarding flow.*

---

## Your Onboarding Flow Is A Story

Mau Baron built a mobile app that makes $40K/month. The secret? His onboarding flow.

When he expanded his onboarding from 5 screens to 29, his conversion rate went from 3% to 12-15%. And it changed everything.

Most app builders treat onboarding like a formality. A few screens to get through before the "real" product starts. That's the mistake. Your onboarding IS the product. It's your one shot to convince a stranger that your app is worth their time, attention, and money — before they've seen any of it.

This is a screen-by-screen breakdown of the exact onboarding flow Mau uses today: the design decisions, the psychology behind each screen, and why the order matters as much as the content.

Think of it as a story. Introduction, climax, conclusion. Every screen has a job. Here's what each one is doing.

---

## 01 — The Introduction

### Screen 1 — The Welcome Screen

**What it is:** A simple, minimalistic, colorful welcome screen that greets the user with a "Hey."

**Why it works:** You never want to hit a new user with a login screen right away. That's pure friction. Instead, you want to make them feel welcomed and spark just enough curiosity to keep going. This screen does exactly that — it's warm, it's on-brand, and it costs the user nothing to get past it.

**How to apply it:** Keep it simple. Match it to your brand. The goal is just to open the door.

---

### Screens 2 & 3 — Problem, The Solution

**What it is:** Screen 2 paints the problem: "Ever feel like your phone gets more attention than God?" Screen 3 immediately presents the solution: Prayer Lock helps you put God first — every day you pray to unlock your apps.

**Why it works:** You want the user to know exactly what your app does within the first three screens. No confusion, no vagueness. Problem, solution, clarity. This also sets up everything that comes after.

**How to apply it:** Lead with the pain your user already feels, then immediately show them you have the answer. Don't make them guess what your app does.

---

### Screen 4 — Ask For Their Name

**What it is:** A simple screen asking for the user's name.

**Why it works:** Personalization builds trust. The moment you address someone by their name, the experience stops feeling like an app and starts feeling like it was made for them. We didn't have this in the first version of our onboarding. When we added it as part of our full onboarding overhaul, our conversion rate went from 3% to 12-15% — more than a 3x increase.

**How to apply it:** Ask for the name early. Use it throughout the onboarding. Make the user feel like you built this for them specifically.

---

### Screens 5 & 6 — Age and Phone Usage

**What it is:** We ask the user their age and how many hours a day they spend on their phone.

**Why it works:** These answers set up the most important screen in the introduction — the bombshell. But beyond that, every question you ask in your onboarding should serve a purpose. These two are doing the groundwork.

**How to apply it:** Don't ask questions just to fill time. Every question should either make the user reflect or set up something bigger.

---

### Screen 7 — The Bombshell (First Aha Moment)

**What it is:** Based on their answers, we show the user a data snapshot.

**Why it works:** This is your first "aha!" moment, and you want it to land as early as possible — ideally within the first minute. We're not telling the user they have a problem. We're showing them the math and letting them arrive at that conclusion themselves. That's a completely different psychological experience. The user stops, thinks, and feels it.

**How to apply it:** Find the number, stat, or realization that makes your user go "wow." Calculate it based on their answers so it feels personal. The aha moment should be specific, surprising, and emotionally resonant with the problem you're solving.

---

### Screen 8 — The Bridge

**What it is:** We follow the bombshell with: "It doesn't have to be this way. Do you have five minutes for God each day? Let's build a plan for you."

**Why it works:** You just made the user feel the weight of their problem. Now you immediately offer them a way out — and you frame it as something personalized and achievable. The phrase "let's build a plan for you" is doing a lot of work here. It signals that what comes next is just for them.

**How to apply it:** After your aha moment, give the user hope. Don't leave them sitting in the problem. Show them the door out.

---

### Screens 9-15 — The Question Bank

**What it is:** A series of questions. With these questions are reflection screens that mirror the user's answers back at them, and a final reflection screen on screen 15 where each line fades in one by one.

**Why it works:** Here's the thing most people get wrong about onboarding questions — they think the questions are there so the app can learn about the user. That's secondary. The real purpose is to make the user learn about themselves. You want them to sit with their answers and slowly convince themselves that they have a problem worth solving. You are not hard selling anything. You are creating the conditions for the user to sell themselves.

**How to apply it:** Be extremely deliberate with both your questions and your answer choices. The answers you provide should reflect the exact struggles your target user already has. Don't make them generic. The goal is for the user to read an answer option and think "that's literally me." That moment of recognition is what builds the emotional momentum you need going into the climax.

> *Bonus content referenced in the source: "Learn more about why the question bank is so important" — additional material not included in this excerpt.*

---

### Screens 16-19 — Analytics Questions, Final Reflection, and the Chart

**What it is:** Two questions primarily for analytics purposes, followed by another screen mirroring their answers back, and finally a screen with a small chart and a quote confirming that Prayer Lock is the solution.

**Why it works:** By this point the user knows they have a problem, they feel heard, and now we want to remind them one more time that we are the answer. The chart gives it a visual, almost clinical confirmation. The quote reinforces it emotionally. Together they close out the introduction of the story.

**How to apply it:** End your introduction with clarity. The user should walk out of this section knowing their problem, feeling understood, and believing your app can fix it.

---

## 02 — The Climax

### Screens 20-22 — The In-Onboarding App Experience

**What it is:** We walk the user through the core workflow of the app — the same two questions they'll answer every day (How's your relationship with God today? How are you feeling today?) — and then show them a carefully selected, pre-written prayer they recite, plus a Bible verse chosen specifically for this moment.

> Note: we do not generate a live AI prayer here. We use a curated prayer to save on LLM costs and to control the quality of the experience for non-paying users.

**Why it works:** This is the climax of your onboarding story. After all the reflection and buildup, the user finally gets to feel what the app actually is. They don't just see a demo — they do it. That hands-on experience builds trust in a way that no screenshot or description ever could. By the end of this sequence the user is excited. They want to keep going. They want to start their streak.

**How to apply it:** Let the user use your core feature during the onboarding. Don't just show them what the app does — make them do it. The more they interact, the more invested they become.

---

### Screen 23 — Congratulations and the Streak

**What it is:** We congratulate the user on completing their first prayer and show them their streak starting at day one.

**Why it works:** You just brought the user to the highest emotional point in the entire onboarding. They completed something. They feel accomplished. The streak makes that feeling concrete and gives them an immediate reason to come back tomorrow. This is the peak of the story — and it's exactly where you want to place your review modal.

**How to apply it:** Celebrate the user at their peak. Give them something to show for the work they just did in your onboarding.

---

### Screen 24 — The Review Modal

**What it is:** Immediately after the congratulations and streak screen, we ask the user to leave a review.

**Why it works:** Prayer Lock has a 12% review rate, meaning 1 in 8 users leaves a review. That number is almost unheard of. The reason is timing. Most apps show the review modal at the end of the onboarding, next to a social proof screen, when the user's energy is winding down. We show it at the peak of the climax, when the user is most excited and most emotionally engaged.

**How to apply it:** Show your review modal at the emotional peak of your onboarding, not at the end. Time it to the moment your user feels the best about your app.

> *Bonus content referenced in the source: "A practical reason for showing the review screen here" — additional material not included in this excerpt.*

---

## 03 — Conclusion

### The Transition — Loading Animation + Personalized Summary

**What it is:** A loading animation that gives the impression of personalizing the experience, followed by a summary screen showing the user where they currently are in their faith journey, where they want to go, and how Prayer Lock will get them there — with a specific goal like "you'll build a consistent prayer habit in 30 days."

**Why it works:** Nothing is actually being processed here, but that doesn't matter. The user feels like the app is working for them. The summary screen then makes everything feel tailored. It reiterates that we heard them, we saved their answers, and we built something just for them. The 30-day goal also sets a concrete, achievable expectation.

**How to apply it:** Use a loading screen as a transition into your conclusion. It signals that something meaningful just happened. Follow it with a summary that reflects the user's journey back to them one more time.

---

### Screens 28 & 29 — The Commitment Screens (Cialdini)

**What it is:** We ask the user "How committed are you to making this future happen?" with options ranging from "extremely committed" to "just trying it out." Based on their answer, we respond with tailored copy that makes them feel understood and affirmed.

**Why it works:** This comes directly from Robert Cialdini's principle of commitment and consistency. When you get someone to verbally or actively state that they are committed to something, they are significantly more likely to follow through. By making the user say out loud — even by tapping a button — that they are committed, you're increasing the likelihood that they'll pay, engage, and stick around.

**How to apply it:** Find a way to get your user to commit before they hit the paywall. Make it feel natural, not transactional. The goal is to prime them psychologically for the decision they're about to make.

---

### Screen 30 — The Faith Snapshot

**What it is:** A personalized screen summarizing where the user is and where they want to be — one final reflection before the paywall.

**Why it works:** You want the last thing the user sees before the paywall to be about them, not about you. This screen sends them in feeling understood, hopeful, and ready.

**How to apply it:** Close your onboarding story with the user at the center. Remind them of their goal. Make the paywall feel like the natural next step toward it.

---

### Configuration Screens — Notifications + Screen Time

**What it is:** The necessary setup screens — connecting Screen Time and enabling notifications — both with fade-in animations.

**Why it works:** These are functional requirements, but the fade-in slows them down and keeps the user invested. Every second of time spent in your onboarding is working in your favor.

**How to apply it:** Even your functional screens can be designed intentionally. Don't just dump a permissions prompt on the user. Frame it, pace it, and make it feel like part of the experience.

---

### Final Screen — Social Proof

**What it is:** A screen showing total downloads, active users, and ratings before the paywall.

**Why it works:** You want the user's last thought before the paywall to be "I'm not alone in this." Tens of thousands of people have been exactly where they are. That reassurance removes hesitation.

**How to apply it:** End with social proof. Real numbers, real credibility. Then hit the paywall.

---

### The Paywall

**What it is:** A hard paywall with a free trial, and a notification sent one day before the trial ends reminding the user it's about to expire.

**Why it works:** The pre-expiry notification isn't about pressure — it's about trust. You're telling the user upfront that you'll give them a heads up before they're charged. That transparency increases conversion. In tools like Superwall you can set this up once and forget about it.

**How to apply it:** In the early stages, don't overthink your paywall design or pricing. Focus on your onboarding and your distribution first. Once you have consistent traffic and a strong onboarding, then optimize the paywall. The notification reminder is one of the easiest high-leverage things you can add.
