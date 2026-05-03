---
name: sentry-setup
description: Set up Sentry error tracking and crash reporting from scratch in a React Native / Expo app. Use this skill when adding Sentry to an app for the first time. Invoke after the core app is built and EAS build is confirmed working, before App Store submission. Sentry captures crashes, unhandled errors, and performance issues in production.
---

# Sentry Setup — React Native / Expo

## Overview

Sentry catches crashes and errors you'd never see in development.
Set up before TestFlight so you catch issues during testing too.

---

## Step 1 — Create Project

1. Go to https://sentry.io — free tier covers indie apps
2. Create a new project → Platform: React Native
3. Get your DSN from Project Settings → Client Keys

---

## Step 2 — Install

```bash
npx expo install @sentry/react-native
```

Run the Sentry wizard to auto-configure native files:
```bash
npx @sentry/wizard@latest -i reactNative
```

The wizard handles:
- Adding the Sentry Expo config plugin to `app.json`
- Setting up source maps for readable stack traces
- Configuring iOS and Android native integrations

---

## Step 3 — Environment Variable

Add to `app.config.js`:
```js
export default {
  extra: {
    sentryDsn: process.env.SENTRY_DSN,
  },
  hooks: {
    postPublish: [
      {
        file: "sentry-expo/upload-sourcemaps",
        config: {
          organization: "YOUR_ORG_SLUG",
          project: "YOUR_PROJECT_SLUG",
        },
      },
    ],
  },
}
```

Add to `.env.local`:
```
SENTRY_DSN=https://xxxx@xxxx.ingest.sentry.io/xxxx
SENTRY_AUTH_TOKEN=sntrys_xxxx    ← for source map uploads, get from User Settings → Auth Tokens
```

Flag both as MANUAL INPUT REQUIRED.

---

## Step 4 — Initialise

```tsx
// App.tsx or _layout.tsx — as early as possible
import * as Sentry from '@sentry/react-native'
import Constants from 'expo-constants'

Sentry.init({
  dsn: Constants.expoConfig?.extra?.sentryDsn,
  enableInExpoDevelopment: false,  // only track in production builds
  debug: false,
  tracesSampleRate: 0.2,           // 20% of sessions for performance monitoring
  environment: __DEV__ ? 'development' : 'production',
})

export default Sentry.wrap(App)    // wrap root component
```

---

## Step 5 — Wrap Root Component

```tsx
// Wrap your root App component with Sentry
export default Sentry.wrap(function App() {
  return (
    // ... your app
  )
})
```

---

## Step 6 — Custom Error Boundaries

For sections of UI you want to gracefully handle:

```tsx
import * as Sentry from '@sentry/react-native'

export function FeatureErrorBoundary({ children }) {
  return (
    <Sentry.ErrorBoundary
      fallback={({ error, resetError }) => (
        <View>
          <Text>Something went wrong</Text>
          <Button onPress={resetError} title="Try again" />
        </View>
      )}
    >
      {children}
    </Sentry.ErrorBoundary>
  )
}
```

---

## Step 7 — Identify Users

After sign in, tag errors with the user:

```tsx
import * as Sentry from '@sentry/react-native'

// After auth
Sentry.setUser({
  id: userId,
  // do NOT send email or name — keep PII minimal
})

// On sign out
Sentry.setUser(null)
```

---

## Step 8 — Verify

1. Temporarily add a test crash in development:
```tsx
<Button onPress={() => { throw new Error('Sentry test crash') }} title="Test Sentry" />
```
2. Run a production build (`eas build --profile production`)
3. Install on device, trigger the crash
4. Confirm it appears in Sentry dashboard with a readable stack trace
5. Remove the test button

---

## EAS Build Config

Add to `eas.json` production profile to upload source maps automatically:
```json
{
  "build": {
    "production": {
      "env": {
        "SENTRY_AUTH_TOKEN": "sntrys_xxxx"
      }
    }
  }
}
```

---

## Common Issues

| Problem | Fix |
|---------|-----|
| Crashes show minified stack traces | Source maps not uploading — check `SENTRY_AUTH_TOKEN` and `eas.json` |
| Nothing appearing in Sentry | Check DSN is correct, `enableInExpoDevelopment: false` only blocks dev mode |
| Wizard failed on native files | Run `npx expo prebuild` first, then re-run wizard |
| Too many events on free tier | Lower `tracesSampleRate` to `0.05` or disable performance monitoring |

---

## Notes

- Free tier: 5,000 errors/month — more than enough for MVP
- Source maps are critical — without them stack traces are unreadable
- Never log PII in Sentry events — user IDs are fine, emails/names are not
- Sentry also handles performance monitoring (slow screens, API latency) — useful post-launch but optional for MVP
