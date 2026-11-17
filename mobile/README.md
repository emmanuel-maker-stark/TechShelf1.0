# TechShelf1.0

Android-first Expo app for discovering and organizing technology learning videos. YouTube playback uses the official embedded player; YouTube video downloads are not supported.

## Local setup

### Prerequisites

- Node.js 22.13 or later.
- Android Studio with an Android SDK and emulator, or a physical Android device.
- A Clerk application configured for the Native API and Google sign-in.
- A Convex deployment when backend sync or YouTube search is being developed.

### Install and configure

```bash
npm ci
```

Copy `.env.example` to `.env.local` and fill in the Clerk publishable key, Google OAuth client IDs, and Convex deployment URL. These client values are public identifiers; never put Clerk secret keys or the YouTube Data API key in `EXPO_PUBLIC_` variables. Configure the YouTube API key as a Convex server environment variable when the search actions are implemented.

Start Expo Go to open the current home screen:

```bash
npm start
```

Scan the terminal QR code with SDK 58 Expo Go. The login route and sign-in UI have been removed. Clerk dependencies, provider, token cache, and native plugins remain configured for future use. The native Google sign-in integration is not available in Expo Go. To use it later, build and install the Android development client, then start its QR server:

```bash
npx expo run:android
npm run start:dev-client
```

## Useful commands

```bash
npm run lint
npx tsc --noEmit
npx expo install --check
npx expo-doctor
```

## Main dependencies

- Expo SDK 58, React Native 0.88, Expo Router native tabs, and NativeWind 4.
- Clerk for identity and Google sign-in; Convex for authenticated backend data and server-side YouTube API calls.
- SQLite for on-device anonymous data, Expo SecureStore for auth session tokens, and Expo Sharing for inbound shared links.
- React Native WebView for the official YouTube embedded player.
- Expo ImagePicker and Localization for profile photos and device-region defaults.

Direct dependency versions are pinned in `package.json` and `package-lock.json`. Keep the Expo SDK-managed packages aligned with SDK 58 using `npx expo install`.
