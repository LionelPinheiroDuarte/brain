# EAS — Expo Application Services

EAS is Expo's cloud build and submission service for React Native apps. It handles
Android/iOS compilation in the cloud, keystore management, and automated Play Store /
App Store submissions — no local Android Studio or Xcode required.

---

## Key Concepts

| Term | What it is |
|------|-----------|
| **EAS Build** | Cloud compilation — produces `.aab` (Android) or `.ipa` (iOS) |
| **EAS Submit** | Uploads a built artifact to Play Store or App Store |
| **EAS Update** | OTA JS updates (no build needed) — not covered here |
| **Profile** | A named build configuration in `eas.json` (e.g. `development`, `production`) |
| **Keystore** | Android signing certificate — EAS generates and stores it automatically |

---

## Setup

```bash
npm install -g eas-cli
eas login
eas init          # links the project, injects projectId into app.json
```

---

## eas.json

The central config file at the project root:

```json
{
  "cli": {
    "version": ">= 16.0.0",
    "appVersionSource": "remote"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "production": {
      "android": {
        "buildType": "app-bundle",
        "autoIncrement": true
      }
    }
  },
  "submit": {
    "production": {
      "android": {
        "track": "internal"
      }
    }
  }
}
```

### Key settings

- **`appVersionSource: "remote"`** — versionCode is managed by EAS, not from `app.json`. Required when using `autoIncrement`.
- **`autoIncrement: true`** — EAS increments `versionCode` automatically on each build. Never set `versionCode` manually in `app.json`.
- **`buildType: "app-bundle"`** — produces `.aab` (required by Play Store since 2021).
- **`track: "internal"`** — submits to the Internal Testing track on Play Console.

---

## Common Commands

```bash
# Build for Android (production profile)
eas build --platform android --profile production

# Submit the latest build to Play Store
eas submit --platform android

# Submit a specific local file
eas submit --platform android --path ./build.aab

# Check build status
eas build:list
```

---

## Build Failures — Lessons Learned

### 1. npm ci peer dependency conflict

**Symptom:**
```
npm error code ERESOLVE
npm error ERESOLVE could not resolve
```

EAS uses `npm ci` (strict mode) — `^` ranges that work locally can fail on the build server.

**Fix:**
```ini
# .npmrc
legacy-peer-deps=true
```

Or, better: pin the conflicting package to an exact version instead of a range.
```json
"react": "19.2.3",
"react-dom": "19.2.3"
```

!!! tip
    `react-dom` must be pinned to the exact same version as `react`. Using `^19.2.3`
    causes an ERESOLVE conflict on EAS even though it works locally.

---

### 2. React Native architecture mismatch (Paper APIs removed)

**Symptom:** Gradle compilation fails with errors mentioning `UIManagerModule`,
`LayoutAnimationController`, or similar Paper-era APIs.

**Root cause:** React Native 0.75+ removed the Paper bridge. Libraries compiled against
Paper APIs (like `react-native-reanimated` v3) fail to build.

**Fix for reanimated v3 → v4:**

```bash
npx expo install react-native-reanimated react-native-worklets
```

Update `babel.config.js`:
```js
// Before (v3)
plugins: ['react-native-reanimated/plugin']

// After (v4)
plugins: ['react-native-worklets/plugin']
```

!!! warning
    Always use `npx expo install` (not `npm install`) for Expo-managed packages.
    It pins to the version declared in `bundledNativeModules.json` for your SDK.

---

### 3. expo-doctor version mismatches

Run before every build:

```bash
npx expo-doctor
```

If mismatches are found:
```bash
npx expo install --check    # auto-corrects all mismatched versions
```

`bundledNativeModules.json` (inside `node_modules/expo/`) is the source of truth for
expected package versions per SDK.

---

## Version Management

- **`version`** in `app.json` → user-visible string (e.g. `"2.0.0"`) — update manually
- **`versionCode`** → internal Android integer — let EAS manage it with `autoIncrement: true`
- Never set both `appVersionSource: "remote"` and a hardcoded `versionCode` in `app.json`

---

## Play Store Submission Flow

```
EAS Build → .aab artifact
    ↓
EAS Submit (or manual upload)
    ↓
Play Console — Internal Testing track
    ↓
Promote to Closed Testing → Open Testing → Production
```

For automated submission, EAS needs a Google Service Account JSON:

1. Play Console → Setup → API access → Create service account
2. Grant "Release manager" role
3. Download JSON → provide path to `eas submit`

---

## CI/CD with GitHub Actions

```yaml
# .github/workflows/release.yml
on:
  push:
    tags: ['v*']

jobs:
  build-and-submit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EAS_TOKEN }}
      - run: eas build --platform android --profile production --non-interactive
      - run: eas submit --platform android --non-interactive
```

Required GitHub secrets:
- `EAS_TOKEN` — from expo.dev → Account Settings → Access Tokens
- `GOOGLE_SERVICE_ACCOUNT_KEY` — service account JSON (base64 or raw)

---

## Checklist Before First Build

- [ ] `eas.json` has `appVersionSource: "remote"`
- [ ] `versionCode` not hardcoded in `app.json`
- [ ] `.npmrc` has `legacy-peer-deps=true` if needed
- [ ] `npx expo-doctor` passes (21/21)
- [ ] `npm run typecheck && npm run lint` — 0 errors
- [ ] `app.json` has correct `bundleIdentifier` (iOS) and `package` (Android)
