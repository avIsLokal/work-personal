# AgriLokal Mobile App Setup & Development Guide

This guide details the complete environment setup required to run and develop the AgriLokal React Native Expo project on an Android Simulator (macOS).

---

## 1. System Prerequisites

| Tool | Version / Recommendation |
| :--- | :--- |
| **OS** | macOS (Apple Silicon M-series or Intel) |
| **Node.js** | `>= 20.x` (Recommended: Node 20 LTS or Node 24) |
| **Package Manager** | `pnpm` (v9.x – v12.x supported) |
| **Java Development Kit (JDK)** | OpenJDK 21 (Temurin JDK 21 recommended) |
| **Android Studio** | Latest version (Ladybug or newer) |
| **Android SDK Platform** | Android 15 / API 36 (`platforms;android-36`) |
| **Android NDK** | `27.1.12297006` |
| **Android Build-Tools** | `36.0.0` |
| **CMake** | `3.22.1` |

---

## 2. Environment Configuration (`~/.zshrc`)

Add the following environment variables to your shell profile (`~/.zshrc`):

```bash
# Node & Global Binaries
export PATH="$HOME/.npm-global/bin:$HOME/.local/bin:$PATH"

# Java Home (Temurin JDK 21)
export JAVA_HOME="/Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home"

# Android SDK
export ANDROID_HOME="$HOME/Library/Android/sdk"
export ANDROID_SDK_ROOT="$ANDROID_HOME"

# Android SDK Tools in PATH
export PATH="$PATH:$ANDROID_HOME/platform-tools"
export PATH="$PATH:$ANDROID_HOME/emulator"
export PATH="$PATH:$ANDROID_HOME/cmdline-tools/latest/bin"
```

After modifying, reload your shell:
```bash
source ~/.zshrc
```

Verify your setup:
```bash
node -v
pnpm -v
java -version
adb --version
emulator -list-avds
```

---

## 3. Android SDK & NDK Setup

If setting up on a fresh machine:

### A. Install Command-Line Tools
If `$ANDROID_HOME/cmdline-tools/latest` is missing:
```bash
TMP_DIR=$(mktemp -d)
curl -o "$TMP_DIR/cmdline-tools.zip" "https://dl.google.com/android/repository/commandlinetools-mac-11076708_latest.zip"
unzip -q "$TMP_DIR/cmdline-tools.zip" -d "$TMP_DIR"
mkdir -p "$ANDROID_HOME/cmdline-tools/latest"
cp -r "$TMP_DIR/cmdline-tools/"* "$ANDROID_HOME/cmdline-tools/latest/"
rm -rf "$TMP_DIR"
```

### B. Accept Licenses
```bash
yes | $ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager --licenses
```

### C. Install Required SDK Packages & NDK
React Native 0.81 and this codebase require API 36 and NDK 27.1:
```bash
sdkmanager "platform-tools" \
           "platforms;android-36" \
           "build-tools;36.0.0" \
           "ndk;27.1.12297006" \
           "cmake;3.22.1"
```

---

## 4. Android Virtual Device (Simulator) Setup

1. Open **Android Studio** → **Virtual Device Manager**.
2. Create a device (e.g. Pixel 8 / Medium Phone).
3. Select a system image matching your CPU architecture (`arm64-v8a` for Apple Silicon).
4. Launch the emulator or start it from terminal:
   ```bash
   emulator -avd <Your_AVD_Name>
   ```
5. Confirm it appears in ADB:
   ```bash
   adb devices
   # Output should show: emulator-5554 device
   ```

---

## 5. Project Installation

From the project root (`AgriLokal`):

1. **Install Node dependencies using pnpm:**
   ```bash
   pnpm install
   ```

2. **Environment Variables:**
   Ensure environment files exist in the project root:
   - `.env.development` (Default local development)
   - `.env.staging`
   - `.env.production`

3. **Google Services Config:**
   Verify that `google/google-services.development.json` exists for Android Firebase integration.

---

## 6. Prebuild Native Projects

The project uses Expo Prebuild with custom config plugins (`app.config.ts`). Generate the native `android` folder:

```bash
pnpm run prebuild:development
```

*(Note: The generated `android` and `ios` directories are git-ignored and can be regenerated anytime).*

---

## 7. Running the App

### Option A: Run directly via Expo CLI
Make sure your Android emulator is booted and detected:
```bash
pnpm run android
```
*(This builds the APK, installs it to the emulator, and connects to Metro).*

### Option B: Build APK & Start Metro Separately

1. **Build and install debug APK:**
   ```bash
   cd android
   ./gradlew :app:installDebug
   cd ..
   ```

2. **Reverse the Metro port for the emulator:**
   ```bash
   adb reverse tcp:8081 tcp:8081
   ```

3. **Start the Metro Bundler:**
   ```bash
   pnpm run start
   ```

4. **Launch the Development Client:**
   Press `a` in the Metro terminal, or run:
   ```bash
   adb shell am start -d "exp+agrilokal://expo-development-client/?url=http%3A%2F%2Flocalhost%3A8081" -a android.intent.action.VIEW com.agrilokal.kisan.development
   ```

---

## 8. Common Troubleshooting

| Issue | Cause | Resolution |
| :--- | :--- | :--- |
| `InstallFailedException: Failed to install ... ndk;27.1.12297006` | Missing NDK version required by React Native 0.81 | Run `sdkmanager "ndk;27.1.12297006"` and accept licenses with `yes \| sdkmanager --licenses`. |
| `Unable to locate a Java Runtime` or Gradle failure | `JAVA_HOME` not exported in shell | Set `export JAVA_HOME="/Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home"` in `~/.zshrc`. |
| `Could not connect to development server` in emulator | Port 8081 not forwarded to emulator | Run `adb reverse tcp:8081 tcp:8081`. |
| `pnpm doctor` store write error | Permission error on `~/Library/pnpm` | Run `mkdir -p ~/Library/pnpm/store/v11` and ensure user ownership. |
| Duplicate native module warning in `expo-doctor` | Transitive dependencies mismatch | Run `pnpm run doctor` to audit, or verify `pnpm-lock.yaml`. |

---

## 9. Useful npm/pnpm Scripts

- `pnpm run start` - Start Expo Metro bundler
- `pnpm run android` - Build and start app on Android
- `pnpm run prebuild:development` - Clean prebuild for development environment
- `pnpm run lint` - Run ESLint across TypeScript and JavaScript files
- `pnpm run type-check` - Run TypeScript compiler type checking (`tsc --noEmit`)
- `pnpm run doctor` - Run Expo doctor diagnostics
