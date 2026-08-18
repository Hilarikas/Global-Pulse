# Global Pulse

Real-time global news, social-trend & podcast intelligence — **design prototype**.

- `index.html` — the full app UI (Home / News / Podcasts / World / Settings). Runs standalone in any browser, pulls **live RSS** on load. Just double-click it.
- `android/` — a WebView wrapper that bundles the prototype into an installable app (**full features**).
- `android-native/` — a **native Jetpack Compose** app (Kotlin). All five tabs implemented natively: Home (Top 10 hierarchy), News (category filters), Podcasts (live iTunes), World (trend map + regional rankings), Settings (working theme switch + live feed status). Installs alongside the WebView app (different package id).
- `.github/workflows/` — cloud builds that produce real `.apk` files, no local tools needed:
  - `build-apk.yml` — WebView app (debug + optionally signed release)
  - `build-native-apk.yml` — native Compose app
  - `generate-keystore.yml` — one-time signing-key helper

## Which app is which?

| | `android/` (WebView) | `android-native/` (Compose) |
|---|---|---|
| Screens | Home, News, Podcasts, World, Settings | Home, News, Podcasts, World, Settings |
| UI | HTML/CSS inside a WebView | Real native Compose widgets |
| Extras in WebView only | For-You feed, story timeline animations, search | — (coming to native next) |
| Best for | Full prototype fidelity | The long-term native codebase |

Both fetch the same live feeds and share the same trend logic (ported to Kotlin in the native app).
Both workflows also produce a signed release APK when the keystore secrets are set.

## What's live vs. simulated

Headlines, publishers, timestamps, images, and multi-source **story clustering** are genuinely live from public RSS feeds (routed through a public CORS relay, with automatic fallback across several relays and, if all are down, a bundled offline sample set).

Trend-velocity percentages, sparklines, and social/podcast signals are **simulated and labeled as such** in the UI — real values require a backend that stores historical snapshots over time.

---

## Getting an APK

You cannot build an Android APK without the Android toolchain. Pick whichever path fits:

### Option A — GitHub Actions (no local install) ✅ recommended
1. Create a new GitHub repository.
2. Upload this whole folder (keep the structure — `index.html`, `android/`, `.github/` all at the repo root).
3. Push to the `main` branch. The **Build Global Pulse APK** workflow runs automatically
   (or trigger it from the **Actions** tab → *Run workflow*).
4. When it finishes, open the run → **Artifacts** → download `global-pulse-debug-apk`.
   Unzip to get `app-debug.apk`.

### Option B — Android Studio (one click)
1. Install [Android Studio](https://developer.android.com/studio).
2. **Open** the `android/` folder — or `android-native/` for the Compose app — (not the whole project). Let it sync Gradle.
3. **Build → Build Bundle(s) / APK(s) → Build APK(s)**.
4. Click *locate* → `app/build/outputs/apk/debug/app-debug.apk`.

The native app builds in CI too: **Actions → Build Global Pulse (native) APK → Run workflow** → artifact `global-pulse-native-debug-apk`.

### Option C — command line
Requires JDK 17 + Android SDK (`ANDROID_HOME` set).
```bash
cd android
gradle wrapper --gradle-version 8.7
./gradlew assembleDebug        # Windows: gradlew.bat assembleDebug
# → android/app/build/outputs/apk/debug/app-debug.apk
```

## Signed release APK (optional)

The debug APK is fine for sideloading. For a **signed release** build (needed for the
Play Store, and for stable app updates), you need a keystore:

1. In your GitHub repo → **Actions** → **Generate signing keystore** → **Run workflow**.
2. Open the finished run → **Artifacts** → download `signing-keystore-DELETE-AFTER-USE`.
   It contains `SIGNING-SECRETS.txt` with four values.
3. Repo → **Settings → Secrets and variables → Actions** → add each as a secret:
   `KEYSTORE_BASE64`, `KEYSTORE_PASSWORD`, `KEY_ALIAS`, `KEY_PASSWORD`.
4. **Delete the downloaded artifact** (it holds your keystore + passwords).
5. Re-run **Build Global Pulse APK** → download `global-pulse-release-apk`.

Without those secrets the release APK still builds, just debug-signed.
Keep your keystore safe — you need the *same* one to publish future updates.

## Installing the APK on a phone
1. Copy `app-debug.apk` to your Android device.
2. Open it; allow **Install unknown apps** for your file manager when prompted.
3. Launch **Global Pulse**. (It needs internet for live feeds; works offline with sample data.)

> This is an unsigned **debug** APK for testing. A Play Store release needs a signed release build.

## Updating the app after editing the prototype
The APK bundles a copy of `index.html`. After editing the root `index.html`, re-copy it:
```bash
cp index.html android/app/src/main/assets/index.html
```
(The GitHub Actions workflow does this automatically on every build.)
