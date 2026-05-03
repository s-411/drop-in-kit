---
name: analytics-posthog
description: Set up PostHog product analytics from scratch in a React Native / Expo app. Use this skill when adding PostHog to an app for the first time. Invoke after the core app is built and EAS build is confirmed working, before App Store submission. Handles installation, initialisation, event tracking patterns, and screen tracking.
---

# PostHog Analytics Setup — React Native / Expo

## Overview

PostHog tracks user behaviour: screen views, feature usage, funnels, retention.
Set up once per app, late in the build cycle — after features are stable.

---

## Step 1 — Install

```bash
npx expo install posthog-react-native
```

If the app uses Expo Go (not a custom dev client), also install:
```bash
npx expo install expo-file-system expo-application expo-device expo-localization
```

---

## Step 2 — Environment Variable

Add to `.env` / `app.config.js`:
```js
// app.config.js
export default {
  extra: {
    posthogApiKey: process.env.POSTHOG_API_KEY,
  }
}
```

Add to `.env.local`:
```
POSTHOG_API_KEY=phc_xxxxxxxxxxxxxxxxxxxx
```

Flag as MANUAL INPUT REQUIRED — get the API key from https://app.posthog.com/project/settings

---

## Step 3 — Initialise

Wrap the app in `PostHogProvider` at the root:

```tsx
// App.tsx or _layout.tsx
import { PostHogProvider } from 'posthog-react-native'
import Constants from 'expo-constants'

export default function App() {
  return (
    <PostHogProvider
      apiKey={Constants.expoConfig?.extra?.posthogApiKey}
      options={{
        host: 'https://app.posthog.com',
      }}
    >
      {/* rest of app */}
    </PostHogProvider>
  )
}
```

---

## Step 4 — Screen Tracking

Add automatic screen tracking via navigation listener:

```tsx
// In your root navigator
import { usePostHog } from 'posthog-react-native'
import { useNavigationContainerRef } from '@react-navigation/native'

export function NavigationTracker() {
  const posthog = usePostHog()
  const navigationRef = useNavigationContainerRef()

  return (
    <NavigationContainer
      ref={navigationRef}
      onReady={() => {
        // track initial screen
        const routeName = navigationRef.getCurrentRoute()?.name
        if (routeName) posthog.screen(routeName)
      }}
      onStateChange={() => {
        const routeName = navigationRef.getCurrentRoute()?.name
        if (routeName) posthog.screen(routeName)
      }}
    >
      {/* navigators */}
    </NavigationContainer>
  )
}
```

---

## Step 5 — Event Tracking

Track key user actions throughout the app:

```tsx
import { usePostHog } from 'posthog-react-native'

export function SomeFeature() {
  const posthog = usePostHog()

  const handleAction = () => {
    posthog.capture('feature_used', {
      feature_name: 'whatever_it_is',
      // any relevant properties
    })
  }
}
```

### Events to track (minimum for every app)
- `onboarding_started`
- `onboarding_completed`
- `onboarding_skipped` (if applicable)
- `sign_up`
- `sign_in`
- `core_action_completed` (whatever the main thing the app does is)
- `paywall_viewed` (when payments are added)

---

## Step 6 — Identify Users

After sign in, identify the user:

```tsx
const posthog = usePostHog()

// After successful auth
posthog.identify(userId, {
  email: user.email,
  name: user.name,
})

// On sign out
posthog.reset()
```

---

## Step 7 — Verify

1. Run the app in development
2. Complete the onboarding flow
3. Go to https://app.posthog.com → Live Events
4. Confirm events are appearing in real time

---

## Notes

- Do not track PII (full name, email) as event properties — only as `identify` traits
- PostHog is free up to 1M events/month — no billing required for MVP
- Feature flags and A/B testing are available post-launch via PostHog — add to LATER.md
