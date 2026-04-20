# TESTFLIGHT.md — Internal + External Tester Setup

> Companion doc for **Stage 13** of `PROCESS_GUIDE.md`. Everything you need to know about getting builds into testers' hands on iOS. One-time ASC setup per app, ~2 minutes.
>
> This doc exists because TestFlight has two completely separate tester systems and if you mix them up you'll waste hours.

---

## The two types of testers — don't confuse them

### Internal Testers
- **Who:** members of your ASC team (via Users and Access)
- **Max:** 100 per app
- **Review required:** none — can test any build immediately
- **Use for:** yourself, co-founder, dev who needs access to everything
- **Cost:** burns an ASC team seat per person

### External Testers
- **Who:** anyone with an email address, no ASC account needed
- **Max:** 10,000 per app
- **Review required:** Beta App Review on first build (few hours to 24h). Subsequent builds skip full review.
- **Use for:** clients, friends testing the demo, beta testers, real users
- **Cost:** zero

**Default answer:** if you're adding someone who isn't you, use **External**. Don't give random people ASC access.

---

## One-time per-app setup (first build of every new app)

### Internal Testing Group (for yourself)

1. ASC → your app → **TestFlight** tab → left sidebar → **Internal Testing** → click **"+"**
2. Group Name: `Team (Expo)` (standardize this across all apps in this project)
3. **Tick "Enable automatic distribution"** — this is the critical box
4. Save
5. Click into the group → **Testers** tab → **"+"** → add yourself by email

With auto-distribution on, every future EAS build for this app auto-links to this group. Testers get notified the moment the build processes. No manual step per build.

### External Testing Group (for anyone else)

1. ASC → your app → **TestFlight** → left sidebar → **External Testing** → click **"+"**
2. Group Name: `External Beta`
3. Add tester emails (up to 10,000)
4. On your first build for this app, Apple runs **Beta App Review** — usually a few hours, sometimes up to 24h
5. Once approved, testers get invite emails → they install TestFlight app → then your app

Subsequent builds only re-trigger Beta Review if you change metadata significantly (name, description, etc). Code-only updates skip review.

---

## If internal testers aren't getting invites (troubleshooting)

You just pushed an EAS build, it shows up in ASC, but no invite email. Three possible causes:

### 1. Export compliance not set (old builds)
If `app.json` is missing `expo.ios.config.usesNonExemptEncryption: false`, TestFlight holds the build until you manually answer the compliance question in ASC.

**Fix for this build:** ASC → build → click "Missing Compliance" banner → answer "Uses Encryption: No"
**Fix for all future builds:** add to `app.json`:

```json
{
  "expo": {
    "ios": {
      "config": {
        "usesNonExemptEncryption": false
      }
    }
  }
}
```

This bakes `ITSAppUsesNonExemptEncryption: false` into the Info.plist. No manual step per build.

### 2. No Internal Testing group exists for this app
Groups are per-app, not per-team. New app = create the group.

**Fix:** create the group as described above. Enable auto-distribution.

### 3. Group exists but isn't linked to THIS build
If auto-distribution wasn't on when the group was created, Apple won't auto-link old builds.

**Fix:** ASC → your app → TestFlight → Internal Testing → your group → **Builds** tab → click **"+"** → add the build manually. (Or turn on auto-distribution now for future builds.)

### 4. You're not actually in the group
Rare but happens if you create the group and forget to add yourself.

**Fix:** ASC → the group → Testers tab → add yourself.

---

## ASC user roles — if you must add someone as Internal

If someone actually needs ASC access (not just to test), pick the least-privilege role:

| Role | What they can do | When to use |
|------|-----------------|-------------|
| **Admin** | Everything except billing | Co-founder |
| **App Manager** | Manage specific apps | Senior engineer who owns a product line |
| **Developer** | Upload builds, view crashes, TestFlight | An engineer who just needs to ship |
| **Marketing** | App Store listing edits | Marketing person |
| **Sales** | Financial reports | Finance person |
| **Customer Support** | Customer emails only | Support rep |

**Default for team members: Developer.** Never grant App Manager or Admin unless they're running the business with you.

For non-team members (clients, friends, beta users) — use External Testers. They don't need a role at all.

---

## Quick reference — the setup flow for every new app

```
After EAS build uploads to TestFlight:

1. First time on THIS app:
   a. Create Internal Testing group "Team (Expo)" — auto-distribution ON
   b. Add yourself as internal tester
   c. If external testers needed: create "External Beta" group, add emails
   d. Wait for Beta App Review to approve external group (first build only)

2. Every build after that:
   a. Internal: auto-linked, email arrives within ~10 min of build processing
   b. External: auto-distributed after Beta Review (first build only)
```

Standardize the group names. "Team (Expo)" and "External Beta" for every app = muscle memory, no thinking per app.

---

## Demo-build vs pre-release-build distinction

**Demo build (pre-client-approval):**
- Internal Testing only
- You + maybe a co-founder or trusted reviewer
- Skip External group until you have real beta users
- No need for App Store metadata yet

**Pre-release build (close to submission):**
- Internal + External both set up
- External Testers getting real use (5-50 testers minimum ideal)
- Full ASC metadata drafted (even if not submitted yet)

---

## Email notification issues

If you're missing TestFlight emails even when the group/build is linked correctly:

- ASC → top-right profile → Edit Profile → Email Preferences → enable "TestFlight" notifications
- Check your email spam folder — Apple's invites sometimes get filtered
- On your phone, open the TestFlight app — invites also appear there even if email is lost

---

## What Rork.com / Expo Snack / other services did "magically"

Services like Rork auto-created the Internal Testing group via the ASC API on first build, which is why tested Rork-built apps "just worked" without this setup. EAS does NOT do this by default — you get the raw ASC behaviour where every new app starts with zero groups.

If you're moving off Rork-style services (you are, per your setup), expect to do this 2-minute per-app setup until you've done it enough times that it's reflex.

---

## Related docs

- `PROCESS_GUIDE.md` Stage 13 — where this fits in the pipeline
- `CREDENTIALS.md` — for EAS + ASC API key setup (one-time Mac setup)
- `BOOTSTRAP.md` Section E — `app.json` must have the encryption flag set
