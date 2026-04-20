# App Landing Page — applanding.co

> Companion doc for **Stage 8** of `PROCESS_GUIDE.md`. Every app gets a landing page. Takes ~10 minutes. Do this *before* Stage 11 (App Store audit).

---

## What It Is

https://applanding.co — a tool that generates a simple, clean landing page for your app, plus the legal pages Apple requires.

**Why every app needs one:**
- App Store requires a live **Support URL** — this is it
- App Store requires a live **Privacy Policy URL** — hosted here
- App Store requires a live **Terms of Use URL** — hosted here
- Gives the app a real web presence (useful for ads later)
- Apple reviewers click these URLs — they must resolve to real pages, not 404s

---

## When to Do This

**Stage 8 in `PROCESS_GUIDE.md`** — after polish audit (Stage 7) and before analytics setup (Stage 9).

The App Store audit at Stage 11 will fail if these URLs aren't live. Don't skip ahead.

---

## What to Generate

When setting up on applanding.co, you need:

1. **App name** — from `app.json`
2. **One-line description** — what the app does (from `BUNDLE_IDS.md`)
3. **App icon** — from `assets/icon.png`
4. **Screenshots** (optional at this stage — skip, add after Stage 14 generates the real ones)
5. **Support email** — your contact email

The tool generates 3 URLs:
- Landing page (your **Support URL** + **Marketing URL**)
- **Privacy Policy** page (required for App Store)
- **Terms of Use** page (required for App Store)

---

## After Generation

Use the prompt from `PROMPTS.md` Stage 8 to wire the URLs into:

### 1. `app.json`
```json
{
  "expo": {
    "extra": {
      "supportUrl": "https://applanding.co/your-app",
      "privacyPolicyUrl": "https://applanding.co/your-app/privacy",
      "termsOfUseUrl": "https://applanding.co/your-app/terms"
    }
  }
}
```

### 2. Paywall screen (and Settings if it exists)
```tsx
import * as Linking from 'expo-linking'
import { theme } from '@/theme/theme'
import Constants from 'expo-constants'

const { privacyPolicyUrl, termsOfUseUrl } = Constants.expoConfig?.extra ?? {}

<View style={{ flexDirection: 'row', gap: theme.spacing.md }}>
  <TouchableOpacity onPress={() => Linking.openURL(privacyPolicyUrl)}>
    <Text style={{ color: theme.colors.textMuted }}>Privacy Policy</Text>
  </TouchableOpacity>
  <TouchableOpacity onPress={() => Linking.openURL(termsOfUseUrl)}>
    <Text style={{ color: theme.colors.textMuted }}>Terms of Use</Text>
  </TouchableOpacity>
</View>
```

### 3. App Store Connect metadata (Stage 15 picks these up automatically)
The Stage 15 `app-store-approval` Phase 2 prompt reads `app.json` and pulls these URLs into the generated `asc-metadata.md`. Don't paste them manually unless that step fails.

---

## Checklist (mark in `CLAUDE.md` Stage 8 row)

- [ ] Landing page created on applanding.co
- [ ] Support URL live and resolves correctly
- [ ] Privacy Policy URL live and resolves correctly
- [ ] Terms of Use URL live and resolves correctly
- [ ] All 3 URLs added to `app.json` under `expo.extra`
- [ ] Privacy + Terms links visible in paywall screen
- [ ] Privacy + Terms links visible in settings screen (if it exists)
- [ ] Self-check passed after wiring

---

## Notes

- Do this for every app — it takes 10 minutes and unblocks Stage 11
- Apple reviewers will click these links — they must load real content, not 404s
- After Stage 14 (screenshots), come back and update the landing page with the real App Store screenshots — gives it some polish for any ad traffic
- If applanding.co is down or you want self-hosted, Vercel + a single Next.js page works in 20 minutes (worth a future kit addition — added to cross-app roadmap in `LATER.md`)
