# AgriLokal: Expo SDK 58 / React Native 0.88 Upgrade

This branch (`chore/expo-54-to-58-migration`) upgrades the app from **Expo SDK 54 / React Native 0.81.5** to **Expo SDK 58 / React Native 0.88.1**. We went one SDK at a time (54 → 55 → 56 → 57 → 58).

This file lists what changed and why. For step-by-step migration notes and troubleshooting, see [EXPO_SDK_58_MIGRATION.md](EXPO_SDK_58_MIGRATION.md). For new platform features that we haven't adopted yet, see [UPGRADE_FEATURE_ADOPTION.md](UPGRADE_FEATURE_ADOPTION.md).

---

## 1. Platform versions

| | Before | After |
|---|---|---|
| Expo SDK | 54.0.27 | **58.0.26** |
| React Native | 0.81.5 | **0.88.1** |
| React / React DOM | 19.1.0 | **19.3.0** |
| TypeScript | 5.8.3 | **6.0.3** |
| Android compileSdk | 36 | **37** (targetSdk stays 36, minSdk 24) |
| Android Gradle Plugin | 8.x | **9.x** (R8 full mode) |
| Architecture | New Arch via `newArchEnabled` flag | New Arch only (bridgeless; the flag is gone) |
| JS engine | Hermes | Hermes V1 (always on) |

## 2. Dependency upgrades

### Major-version bumps (these needed code or config changes)

| Package | Before | After |
|---|---|---|
| `@react-native-firebase/*` (app, analytics, crashlytics, messaging, remote-config) | 22.4.0 | 27.1.1 |
| `react-native-appsflyer` | 6.17.8 | 7.0.4 |
| `react-native-moengage` | 11.2.2 | 12.9.1 |
| `react-native-gesture-handler` | 2.28.0 | 3.3.0 |
| `react-native-pager-view` | 6.9.1 | 9.0.7 |
| `react-native-webview` | 13.16.1 | 14.0.1 |
| `react-native-razorpay` | 2.3.0 | 3.0.0 |
| `react-native-blob-util` | 0.24.5 | 1.1.1 |
| `@react-native-community/netinfo` | 11.4.1 | 12.0.1 |
| `@stream-io/react-native-webrtc` | 125.4.3 | 145.4.2 |
| `@config-plugins/react-native-blob-util` / `-pdf` | 12.0.0 | 14.0.2 |
| `@config-plugins/react-native-webrtc` | 13.0.0 | 15.0.2 |
| `@typescript-eslint/*` | 5.62.0 | 8.73.0 |
| `eslint-plugin-unused-imports` | 2.0.0 | 4.4.1 |
| `eslint-config-expo` | 7.1.2 | 58.0.4 |

### Expo packages

Expo packages now share the SDK's version number (`~58.0.x`). That covers `expo-audio`, `expo-build-properties`, `expo-constants`, `expo-crypto`, `expo-dev-client`, `expo-file-system`, `expo-font`, `expo-image`, `expo-image-manipulator`, `expo-image-picker`, `expo-linear-gradient`, `expo-linking`, `expo-localization`, `expo-location`, `expo-status-bar`, `expo-system-ui`, `expo-tracking-transparency`, `expo-updates`, `expo-video`, `@expo/metro-runtime` and `@expo/config`.

`expo-asset` is now listed as an explicit dependency.

### Minor or patch bumps

| Package | Before | After |
|---|---|---|
| `react-native-reanimated` | 4.2.0 | 4.7.1 |
| `react-native-worklets` | (not listed) | 0.13.0 |
| `react-native-screens` | 4.16.0 | 4.30.1 |
| `react-native-safe-area-context` | 5.6.2 | 5.11.1 |
| `react-native-svg` | 15.12.1 | 15.16.0 |
| `react-native-keyboard-controller` | 1.18.5 | 1.22.7 |
| `@shopify/flash-list` | 2.1.0 | 2.3.2 |
| `@stream-io/video-react-native-sdk` | 1.21.1 | 1.47.3 |
| `@gorhom/bottom-sheet` | 5.2.6 | 5.2.8 |
| `@react-navigation/*` | 7.1–7.8 | 7.1.28–7.14 |
| `@tanstack/react-query` | 5.52.1 | 5.90.20 |
| `axios` | 1.7.5 | 1.13.4 |
| `react-hook-form` | 7.53.0 | 7.71.1 |
| `zod` | 3.23.8 | 3.25.76 |
| `lottie-react-native` | 7.3.4 | 7.5.1 |
| `react-native-date-picker` | 5.0.12 | 5.0.13 |
| `prettier` | 3.3.3 | 3.8.1 |

### Removed

- `eslint-plugin-react-compiler`. Its rules now ship in `eslint-plugin-react-hooks` 7, which `eslint-config-expo` includes.

## 3. App configuration

| File | Change | Why |
|---|---|---|
| `app.config.ts` | Removed `newArchEnabled` | New Arch is the only architecture now |
| `app.config.ts` | Moved the top-level `notification` block (icon and color) to `plugins/withNotificationIcon.js` | SDK 55+ removed the top-level key |
| `app.config.ts` | Removed `experiments.typedRoutes` | The app doesn't use expo-router |
| `app.config.ts` | Added the `expo-asset`, `expo-image` and `expo-status-bar` plugins | Added by `expo install` for SDK 58 |
| `app.config.ts` | `compileSdkVersion` 36 → 37 | Required by upgraded native dependencies |
| `app.config.ts` | Added a ProGuard keep rule for `net.time4j.**` | AGP 9's R8 stripped Time4J (used by `react-native-date-picker`), which crashed release builds at startup |
| `plugins/withNotificationIcon.js` | New config plugin | Sets the Android notification icon and accent color `#009B55` |
| `tsconfig.json` | Removed `baseUrl` and the `.expo/types` / `expo-env.d.ts` includes | `baseUrl` is deprecated in TS 6. Path aliases still work through `paths` |
| `.eslintrc.js` | Dropped the react-compiler plugin. Moved `consistent-type-imports` to a TS-only override. Set `caughtErrors: 'none'`. Set the new React Compiler rules (`react-hooks/refs`, `immutability`, `purity`, `set-state-in-effect`, `preserve-manual-memoization`) to `warn` | Compatibility with typescript-eslint 8 and eslint-config-expo 58. The existing code has about 100 React Compiler warnings, which we kept visible rather than failing on them |

## 4. Code changes

### Runtime fixes (the app breaks without these)

| Area | Files | Change |
|---|---|---|
| `StyleSheet.absoluteFillObject` removed in RN 0.88 | `src/lib/polyfills/absoluteFillObject.ts`, loaded first in `index.js` | Restores the property for third-party libraries that still read it: confirmation-code-field (OTP input), bottom-sheet, bootsplash, moti, stream-chat, flash-message and walkthrough-tooltip |
| Same | `ImageCarousel.tsx`, `WeatherWrapper.tsx`, `VideoPlayer.tsx`, and the booking screens (`InCallChat`, `ConnectingBooking`, `BookingCallSession`, `CallOverlay`, `BookingChatOverlay`, `IncomingBooking`) | Our own code switched to `StyleSheet.absoluteFill` |
| `InteractionManager` removed in RN 0.88 | `LazyFeedbackMetaRow.tsx` | Replaced with `requestIdleCallback` / `cancelIdleCallback` |
| Firebase v27 removed the namespaced API (`messaging()`, `remoteConfig()`) | `getFcmToken.ts`, `setupPushListeners.ts`, `useRegisterStreamPushToken.ts`, `initRemoteConfig.ts`, `readRemoteConfig.ts` | Moved to the modular API (`getMessaging`, `getToken`, `onMessage`, `getRemoteConfig`, `getValue`, …) |
| AppsFlyer v7 replaced callbacks with promises | `initAppsFlyer.ts`, `appsFlyerClient.ts` | New startup order: deep-link listener → `init()` → other listeners → `start()` after the session is ready. `logEvent`, `setCustomerUserId` and `anonymizeUser` now take an object and return a promise. `appId` is passed on iOS only |
| expo-file-system 58 changed its default API | `usePdf.ts` | Imports from `expo-file-system/legacy` |

### Type-only fixes (needed for `tsc` to pass)

| Change | Files |
|---|---|
| `FirebaseMessagingTypes.RemoteMessage` → `RemoteMessage` | `src/lib/push/**` |
| `TextInput` ref → `TextInputInstance` | `src/components/ui/input/*` |
| `ViewToken` → `ListViewToken` | `useImageCarousel.ts`, `useQueueListViewability.ts` |
| `WebView` → `WebView<object>` | `CropGuideSection.tsx`, `useCropGuideSection.ts` |
| Player type now comes from `expo-video` | `useVideoPlayerEvents.ts`, `useVideoPlayerRef.ts` |
| Cast for masked-view's old class types | `MaskedText.tsx` |
| Cast to `MotiTransitionProp` (moti vs Reanimated 4.7 types) | `activeStrip/index.tsx`, `useMarqueeStrip.ts` |
| Auth options shaped for Stream's typed union | `streamCallClient.ts` |
| Ambient `declare const jest` for AppsFlyer 7's TypeScript source | `src/types/vendor-globals.d.ts` |

## 5. Patches (`patches/`, applied by pnpm)

| Patch | Status | Why |
|---|---|---|
| `react-native.patch` | Kept | Existing |
| `@gorhom__bottom-sheet.patch` | **Removed** | Written for 5.2.6. The fix is upstream in 5.2.8 |
| `react-native-date-picker@5.0.13.patch` | **New** | AGP 9 no longer accepts `proguard-android.txt`; switched to `proguard-android-optimize.txt` |
| `react-native-moengage@12.9.1.patch` | **New** | RN 0.88 removed the deep `TurboModule` import. Also exposes the MoEngage BOM as `api` so the app's Gradle build resolves it |
| `react-native-walkthrough-tooltip@1.6.0.patch` | **New** | Replaces `InteractionManager` with a `requestIdleCallback` shim |

## 6. Building after the upgrade

```bash
pnpm install
pnpm run prebuild -p android --clean   # always prebuild clean after pulling this branch
```

- **Stale JS bundle in release builds.** Gradle can reuse an old bundle. If a release build behaves like old code, run `rm -rf android/app/build/generated/assets/react` and build again.
- **"No apps connected" on reload.** A release APK is installed, and release builds don't connect to Metro. Install a debug build instead.
- **Which API a build uses.** Release builds load `.env.production` unless `EXPO_NO_DOTENV=1` is set. For a release build that uses dev-api:
  ```bash
  pnpm run prebuild:development -p android --clean
  cd android && APP_ENV=development EXPO_NO_DOTENV=1 ./gradlew assembleRelease && cd ..
  ```
- **Staging builds are blocked.** `com.agrilokal.kisan.staging` isn't registered in Firebase. The client is missing from `google/google-services.staging.json`, so `processReleaseGoogleServices` fails. Register the app in Firebase and replace that file (and `GoogleService-Info.staging.plist` for iOS).


