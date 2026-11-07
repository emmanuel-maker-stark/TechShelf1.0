# TechShelf — Product and Implementation Plan

**Status:** Interview complete; this document records the agreed v1 and the implementation sequence. No application code has been written yet.

## 1. Product summary

TechShelf is an Android-first app for developers and everyday technology learners to discover, watch, and organize YouTube videos and course-like series across eight technology topics. It provides a focused discovery experience, personalized recommendations, saved videos and playlists, and optional account sync.

YouTube video-file downloads are excluded from v1. The YouTube API Services Developer Policies prohibit copying YouTube audiovisual content or enabling offline playback without YouTube’s prior written approval. The app will use YouTube search and the official embedded player, with an explicit option to open a video in YouTube. The original Download tab, 6.5 GB quota, and quality selector are therefore removed; Library replaces Storage. See the YouTube policy and playback requirements in References.

## 2. Agreed v1 scope

### In scope

- Android only for the initial release; design configuration and data boundaries so more regions and languages can be added later.
- Four native tabs: **Home, Explore, Library, Profile**.
- Modern, clean, minimal mobile UI with few visible borders; use NativeWind for styling.
- The eight initial topics:
  1. Artificial Intelligence (AI & Machine Learning)
  2. Software Development & Programming
  3. Cybersecurity & Ethical Hacking
  4. Cloud Computing & DevOps
  5. Gadgets & Consumer Technology
  6. Tech Careers & Learning
  7. Data Science & Analytics
  8. Robotics, Electronics & IoT
- Topic selection during onboarding; users can follow or unfollow topics later. If onboarding topic selection is skipped, show a mix of all eight.
- Home feed with topic-based content and a **For You** section. Feature course-like series alongside individual videos.
- Explore search with topic shortcuts and free-text YouTube search. Search defaults to the Android device region, and users can choose a preferred region.
- Refresh results only on an explicit search or refresh action; do not issue a YouTube search request for every keystroke.
- Video preview from in-app discovery or a pasted/shared YouTube link. Actions: save to Watch Later, play inside the app, or open in YouTube.
- The official YouTube embedded player for in-app playback. Preserve YouTube’s required player controls, branding, and ads. The app must not enable background playback or downloads.
- Library with a default Watch Later list and user-created playlists. Users can create, rename, reorder, and delete custom playlists and add or remove videos.
- Anonymous browsing, topic preferences, viewing activity, saves, and playlists stored on the current device. No sign-in wall for these capabilities.
- Optional Google sign-in through Clerk. On sign-in, merge anonymous Library and preferences into the account and sync Library, playlists, profile, topic preferences, and viewing activity through Convex.
- Profile editing for display name and photo.
- For You personalization based on in-app video starts, saved videos, and followed topics. External YouTube viewing activity is not available to this app. Users can clear viewing activity.
- Account deletion removes the synced profile, Library, playlists, and viewing activity. Before deleting, offer to retain device data as an anonymous local library or delete it too.
- Free service with no ads in v1.
- Nigeria-first launch, English primary language, no confirmed launch date. Do not restrict the product to Nigeria.

### Out of scope

- Downloading, caching for offline playback, or exporting YouTube video/audio files.
- A Download tab, a 6.5 GB video quota, quality selection, and app-managed video-file storage.
- Paid plans, subscriptions, advertising, AI-generated content, and iOS/web clients in v1.
- YouTube account authorization; Google sign-in is for TechShelf accounts through Clerk, not for accessing a user’s YouTube account.

## 3. Primary user journeys

### First launch and onboarding

1. Launch directly into onboarding without requiring sign-in.
2. Show the eight topics with an option to select favorites or skip.
3. If skipped, initialize the feed with a balanced mix of all eight topics.
4. Ask for no unnecessary Android permissions. Request photo access only when the user chooses a profile photo, using the system photo picker.
5. Enter Home with the chosen or default topic mix. Sign-in remains available from Profile.

### Home and For You

1. Load topic sections and For You with clear skeleton/loading states.
2. Use the current topic preferences and in-app activity to rank results.
3. Offer explicit refresh. A refresh must respect YouTube quota limits.
4. Show videos and course-like playlists as distinct cards/sections, with unmodified YouTube metadata and attribution.
5. A card opens the video preview/player flow.

### Explore and search

1. Show eight topic shortcuts and a free-text search field.
2. Submit a search explicitly; do not call YouTube on every keystroke.
3. Send the selected topic or query, English language preference, and preferred region to the backend search action.
4. Show search results from YouTube; support empty, loading, quota-exhausted, offline, and retry states.
5. Refresh results on explicit request. Do not promise trending or personalized YouTube results beyond what the official API returns.

### Pasted or shared video link

1. Accept a YouTube URL pasted into Explore or shared through Android’s share sheet.
2. Validate and extract the video or playlist identifier; reject unrelated or malformed URLs with a clear message.
3. Resolve current metadata through YouTube’s official API when available, then show a preview.
4. Allow saving to Watch Later, playing in-app, or opening YouTube.
5. If the video is unavailable, private, deleted, or restricted, explain that and keep the user’s saved list recoverable.

### Playback

1. Start an official embedded YouTube player after the user taps play.
2. Keep standard player controls, metadata, advertisements, captions/settings, and related behavior intact.
3. Provide a separate “Open in YouTube” action. When opening externally, launch the YouTube app if installed and otherwise use the system browser.
4. Track an in-app play-start signal for For You. Do not claim to track playback after a user leaves for YouTube.
5. Respect player failures and content restrictions with a retry or external YouTube action.

### Library and playlists

1. Show Watch Later and custom playlists.
2. Add a saved video by YouTube video ID; make the save action idempotent.
3. Custom playlists support create, rename, reorder, delete, and add/remove video.
4. Deleting a custom playlist removes its membership records, not a YouTube video. Watch Later cannot be deleted.
5. Anonymous Library and playlist changes persist locally. Signed-in changes sync through Convex.

### Sign-in, sync, and deletion

1. Sign in with Google through Clerk, initiated from Profile.
2. Merge the anonymous Library, playlists, topic preferences, and activity with the signed-in account.
3. Make the merge idempotent so retrying after a network failure cannot create duplicate saves or playlists.
4. Sync signed-in preferences and activity across devices.
5. Provide clear viewing activity, separate from deleting saved videos or followed topics.
6. Account deletion requires confirmation and offers: delete all device data, or keep it as an anonymous local library. Delete all synced account data either way.

## 4. Navigation and interface

- Use Expo Router’s native tab navigator with Home, Explore, Library, and Profile.
- Use NativeWind classes and a restrained visual system: clear typography, strong spacing, subtle surface separation, minimal borders, and accessible contrast.
- Keep search and Library usable with one hand on common Android screen sizes.
- Support Android back behavior for nested preview, player, playlist, and profile routes.
- Do not add an in-app download button or suggest offline video availability.
- Add empty states for no favorites, empty Watch Later, empty playlists, no search results, and anonymous/signed-in Profile states.

## 5. Architecture and integrations

### Client

- Expo + React Native + TypeScript, Expo Router, NativeWind.
- Android native tabs via the SDK 57 Expo Router native-tabs entry point.
- Use a persistent local database for anonymous data and a separate, account-scoped local cache/mirror for signed-in data. SQLite is the recommended local store because the app has related lists, playlist membership, ordering, and activity.
- Store auth tokens with Clerk’s supported secure storage integration.
- Use Android share intents for inbound YouTube links and a normal paste action. Do not read the clipboard automatically in the background.
- Use the official YouTube embedded player in the Android app. Build and test with a native development build; do not rely only on Expo Go for final playback verification.

### Authentication and application data

- **Clerk:** Google sign-in, account identity, session lifecycle, and editable identity fields.
- **Convex:** authenticated profile preferences, saved video references, playlists, and viewing activity. Protect every user-scoped query and mutation by Clerk identity.
- **Anonymous mode:** local-only records, namespaced separately from every signed-in account.
- **YouTube Data API v3:** public video/playlist search and current metadata lookup. No YouTube OAuth scopes are needed for this use case.
- Make YouTube API calls from Convex actions so the API key is not embedded in the Android bundle. Add server-side request controls and quota monitoring.
- Do not add another analytics, storage, AI, or monetization provider in v1.

### Data model

Use stable IDs, owner scope, timestamps, and explicit indexes. The exact Convex schema is written during implementation.

- **Profile:** Clerk subject, display name, photo reference, preferred search region, language, creation/update time.
- **Topic preference:** owner scope, topic key, selected/followed state, update time. Seed with the eight agreed topic keys.
- **Playlist:** owner scope, stable playlist ID, Watch Later/custom type, title, sort position, creation/update time.
- **Playlist item:** playlist ID, YouTube video ID, position, added time. Unique by playlist and video ID.
- **Saved video reference:** owner scope and canonical YouTube video ID; avoid storing redundant video metadata here.
- **Viewing activity:** owner scope, event type (in-app play start, save, topic follow/unfollow), relevant topic/video reference when permitted, event time. The clear-activity operation deletes these events.
- **YouTube metadata cache:** video or playlist ID, returned fields, fetch time, expiry time. Keep API-derived metadata separate from user-created Library records so it can be refreshed or deleted under YouTube policy.

### Source of truth and merge policy

- Clerk is the identity source of truth. Convex is the source of truth for signed-in TechShelf data. SQLite is the source of truth for anonymous data and the local mirror for the active signed-in account.
- On sign-in, merge local items into the cloud account. Deduplicate the same video within a playlist by YouTube video ID. Import local custom playlists as distinct records; preserve their item order. Keep cloud records and order intact, and append nonduplicate local items. The merge must be transactional or checkpointed and idempotent.
- Keep local anonymous and signed-in account namespaces separate to prevent cross-account data leaks.
- On sign-out, retain cloud data in Convex, clear the active account’s private local mirror, and return to the anonymous local namespace.
- On account deletion, perform the selected local-retention action and delete account-owned Convex documents.

## 6. Search, recommendations, and data flow

### YouTube search

- Use official YouTube Data API search for video and playlist results. Include query, topic context, English language relevance, and the user’s preferred region.
- Device region initializes the preference. The user can change it in Profile/settings; store it with the profile when signed in and locally when anonymous.
- Search only when the user submits a query, opens a topic section, or explicitly refreshes.
- Debounce local input presentation only; do not issue remote API calls for every typed character.
- Monitor actual project quota from Google Cloud. Avoid hard-coding assumptions about quota allocation or pricing; handle quota errors explicitly.
- Display the most recent data returned by YouTube. Preserve required YouTube branding, links, titles, thumbnails, and player behavior.

### For You

- Start with declared interests and followed topics.
- Add weight from in-app play starts and saved videos; do not infer that an external YouTube open was watched.
- Use a deterministic, explainable topic/video ranking first; no machine-learning service is required for v1.
- Anonymous ranking state stays local. Signed-in state syncs through Convex.
- Clearing viewing activity removes play-start history used for ranking while preserving Library items and topic preferences.
- Do not create or store statistical metrics derived from YouTube API data unless current YouTube policy permits it.

### YouTube API retention and compliance

- YouTube policy limits temporary storage of most API data to 30 calendar days, after which it must be refreshed or deleted; current data should be displayed. Implement expiry and refresh/deletion for API-derived metadata. [YouTube Developer Policies](https://developers.google.com/youtube/terms/developer-policies/)
- Persist minimal user-created references and playlist membership separately from API metadata. Before release, confirm that long-term storage of user-selected YouTube IDs and activity references is allowed for this product. If the policy does not allow the planned retention, obtain written approval or revise long-term Library behavior before release.
- Do not store audiovisual copies, stream URLs, or offline playback data. Do not disable or alter YouTube player controls/ads or allow background playback.
- Provide an app privacy policy explaining data collected, local/cloud storage, Clerk/Convex/YouTube data flows, deletion, and third-party processing.

## 7. Failure and edge-case behavior

- **Network unavailable:** keep local Library and topic preferences available. Show an explicit refresh error; use cached YouTube metadata only while within its allowed lifetime.
- **Search quota exhausted or rate-limited:** do not retry in a tight loop. Keep valid results if permitted, explain temporary unavailability, and provide a retry state.
- **No results:** show a helpful empty state and preserve the query.
- **Deleted/private/restricted video or playlist:** show unavailable status, retain the user’s Library reference if policy permits, and offer removal.
- **Malformed or non-YouTube link:** explain the accepted link format; do not silently open or save unknown URLs.
- **Cancelled/failed Google sign-in:** remain anonymous and retain local data.
- **Partial sync or retry:** idempotent mutations and merge checkpoints prevent data loss or duplicates.
- **Storage failure:** report a recoverable error; never claim a save succeeded until local persistence or Convex mutation succeeds.
- **Account switch:** isolate local caches by account and never display one account’s records under another identity.
- **Player failure:** show retry and Open in YouTube actions; do not replace the official player with an unofficial stream.
- **API metadata older than 30 days:** refresh or remove it before display; never continue using expired metadata as current.

## 8. Security, privacy, and quality

- Store only fields required for app behavior. Never log Clerk tokens, profile secrets, raw auth payloads, or full search histories.
- Keep Google/YouTube credentials in protected server configuration; restrict API key to the required API and backend environment.
- Validate every Convex mutation, input size, URL, topic key, playlist operation, and user ownership.
- Use authenticated access for signed-in Convex records; anonymous records remain on device.
- Provide controls to clear activity, sign out, delete account, and choose whether local data remains after account deletion.
- Follow platform accessibility basics: screen-reader labels, minimum touch targets, text scaling, color contrast, and clear loading/error announcements.
- **Developer quality:** modular feature-oriented TypeScript, shared domain types, explicit API boundaries, lint/type checks, deterministic core logic, and documentation for environment setup and data flows.
- No numeric performance, concurrency, or service-level objective has been supplied. Initial acceptance is responsive local navigation, predictable loading states, and no blocking UI during network or sync work.

## 9. Environments, delivery, and observability

- Maintain separate development and production configurations for Clerk, Convex, and YouTube API credentials. Keep secrets out of source control.
- Use an Expo development build for native integrations; keep a production Android build configuration reproducible.
- Recommended default delivery path: local Android development build first, then Expo Application Services for repeatable Android preview/release builds. Confirm the production distribution workflow when the repository and store account are available.
- CI provider is not yet selected. Once a repository is connected, add lint, typecheck, dependency verification, and Android build jobs.
- Record structured backend errors and operational metrics for YouTube request counts/quota failures, Convex sync errors, sign-in failures, and playback initialization failures. Avoid logging personal video history or raw query text.
- No external analytics or ad SDK is in v1.

## 10. Implementation sequence and gates

### Phase 0 — Compatibility and compliance spike

1. Create the minimal Expo SDK 57 / Expo Router 57 / NativeWind 4 setup with exact pins and a committed lockfile.
2. Confirm the Android New Architecture and peer dependency compatibility required by the selected packages.
3. Add only the native tabs shell, one NativeWind-styled screen, Clerk/Convex providers, and an official YouTube embedded player proof of concept.
4. Run dependency checks: Expo install compatibility check, Expo Doctor, package-manager lockfile integrity, and TypeScript/lint checks.
5. Build and smoke-test on an Android emulator: launch, native tab navigation, NativeWind styles, and player initialization.
6. Test Google sign-in and YouTube playback on a physical Android device with the app’s real package ID and signing configuration.
7. **Gate:** do not implement additional product features until all dependency checks and required Android builds/smoke tests pass. If the selected native-tabs API blocks the build or has unacceptable defects, pause and return with evidence before changing the agreed navigation choice.

### Phase 1 — Product foundation

- Establish project structure, theme tokens, reusable video/playlist cards, native tabs, onboarding, and empty/loading/error patterns.
- Add SQLite local schema and migrations for anonymous data; keep account scoping explicit.

### Phase 2 — Authentication and sync

- Integrate Clerk Google sign-in and Convex auth.
- Implement anonymous-to-account Library/preference merge, sign-out behavior, cross-device sync, activity clearing, and account deletion/local-retention choice.
- Verify ownership checks, duplicate handling, and retry safety.

### Phase 3 — YouTube discovery and playback

- Create Convex YouTube search and metadata actions, secret configuration, quota/error handling, region selection, expiry/refresh policy, and link parsing.
- Implement topic search, free-text Explore, feed refresh, player screen, and Open in YouTube fallback.
- Complete the YouTube policy review of persistent video references and personalized ranking before public release.

### Phase 4 — Library and personalization

- Implement Watch Later, custom playlist creation/edit/reorder/delete, and save/remove flows.
- Add topic follow/unfollow and deterministic For You ranking from approved local app signals.
- Add playback and search empty/failure states and Profile controls.

### Phase 5 — Release hardening

- Finish accessibility, privacy policy, data deletion, Android back/deep-link/share handling, analytics-free observability, and production environment configuration.
- Run the full acceptance checklist below on Android emulator and physical device.
- Build signed Android release artifact only after all checks pass.

## 11. Verification and acceptance checklist

### Dependency/build gate

- Exact pins for Expo SDK package, Expo Router, NativeWind, and every direct dependency; no floating caret or tilde ranges in the manifest.
- Lockfile is committed and reproducible.
- Expo compatibility check and Expo Doctor pass with no unexplained dependency mismatch.
- Android development build succeeds and launches on an emulator.
- Native tabs render the four agreed destinations; NativeWind styling is applied in release-compatible configuration.
- Physical device verifies Google sign-in and the official YouTube player with app identity/signing configured.

### Product flows

- First launch works without sign-in; topic selection and skip both reach Home.
- Skipping topics shows a mix of all eight; following/unfollowing changes feed preferences.
- Search by topic and free text works, supports region selection, and does not issue a request per keystroke.
- Refresh, no results, offline, quota error, and malformed link states are understandable.
- Pasted and shared YouTube links preview correctly and can be saved to Watch Later.
- In-app player and Open in YouTube both work; player controls and ads remain available.
- Watch Later and custom playlist create, rename, reorder, delete, add, remove, and empty states work.
- Anonymous state survives app restart; signing in merges without duplicates; account data syncs on another device.
- Clearing activity leaves saved videos and followed topics intact.
- Account deletion clears cloud data and honors the keep/delete local-data choice.
- API metadata expiry and refresh/delete rules work; no video files are written or exposed as downloadable.

### Quality checks

- Unit tests for URL parsing, ranking, playlist ordering, deduplication, merge idempotency, and data expiry.
- Convex tests for auth ownership, invalid inputs, sync retry, and deletion.
- UI/component tests for onboarding, search, Library, player errors, and Profile empty/error states.
- Manual Android smoke test for navigation, share intent, back behavior, text scaling, and network loss.

## 12. Decisions still open

- YouTube’s current policy interpretation for retaining saved video IDs and activity references long term, and whether written approval is needed. This is a release gate because Library persistence and personalization depend on it.
- Google Cloud YouTube API project creation, production quota allocation, and any quota increase.
- Developer/company name for developer or YouTube API registration: **Sato Techworks**. Contact email: [Satoworks45@gmail.com](mailto:Satoworks45@gmail.com). If a YouTube form separately requires an individual applicant’s legal name, that name is unknown and must be obtained from the user; do not substitute the app name **TechShelf1.0** or company name **Sato Techworks**.
- Final app package ID, Clerk production application, Convex production deployment, signing credentials, and Play Store account.
- Numeric performance/scale targets and CI provider; set these when deployment context is known.

## 13. Version pins and references

The compatibility strategy is the latest stable combination documented for NativeWind support, rather than SDK 58 with an unverified NativeWind stable pairing:

- Expo SDK package: **57.0.32**
- Expo Router: **57.0.30**
- NativeWind: **4.2.7**
- React Native and React: use the Expo SDK 57 compatibility pair and pin the resolved exact versions in the manifest and lockfile.
- NativeWind v4 uses Tailwind CSS 3; pin the exact compatible Tailwind CSS and supporting native package versions selected by Expo’s SDK 57 dependency map.
- Expo Router native tabs in SDK 57 use the **unstable** native-tabs entry point. This is an explicit accepted exception; test it in Phase 0 and do not silently substitute a different navigator.

References:

- [Expo SDK references and React Native compatibility](https://docs.expo.dev/versions/latest/)
- [Expo Router native tabs guide](https://docs.expo.dev/router/advanced/native-tabs/)
- [Expo SDK 57 native tabs package reference](https://docs.expo.dev/versions/v57.0.0/sdk/router/native-tabs/)
- [NativeWind stable installation and SDK 57 support](https://www.nativewind.dev/docs/getting-started/installation)
- [Clerk and Convex integration](https://docs.convex.dev/auth/clerk)
- [YouTube Data API Services Developer Policies](https://developers.google.com/youtube/terms/developer-policies/)
- [YouTube player policy guide](https://developers.google.com/youtube/terms/developer-policies-guide)
- [YouTube Android embedded player requirements](https://developers.google.com/youtube/iframe_api_reference)

