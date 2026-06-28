# WC Live — Project Handoff

**For any AI assistant or developer taking over this project.**  
Read this file and `CLAUDE.md` before touching anything.

---

## Project overview

| Field | Value |
|---|---|
| App name | WC Live |
| Package name | `com.wclive.tv` |
| Type | Native Android Java — IPTV player |
| Target devices | Android TV (primary), Android phone, tablet |
| Player | Media3 ExoPlayer 1.3.1 (HLS + MPEG-TS) |
| Architecture | Single Activity — `MainActivity.java` (~2,300 lines) |
| UI | Programmatic Java, no RecyclerView, no ViewBinding, no Fragments |
| Design system | **Midnight Cinema** (dark, `#0B0B0C` background, `#E50914` red accent) |
| Current version | 1.6.9 (versionCode 15) |
| Project location | `~/Documents/Projects/WC-Live` |
| GitHub (public) | `Asif-Ahsan-Sunny/wc-live` |
| GitHub (private) | `Asif-Ahsan-Sunny/wc-live-private` (channel playlist + update.json source of truth) |

---

## Folder map

```
WC-Live/                              ← project root
├── CLAUDE.md                         ← AUTHORITATIVE AI guide — read this first
├── PROJECT_HANDOFF.md                ← this file
├── CLAUDE_SESSION_LOG.md             ← full session history, version table
├── CLAUDE_PROJECT_UPDATE_REPORT.md   ← latest migration/cleanup report
├── BUILD_INSTRUCTIONS.md             ← build command reference
├── RELEASE.md                        ← version bump and release workflow
├── app/
│   ├── build.gradle                  ← versionName, versionCode, signing config
│   ├── wc-live-release.jks           ← release keystore (LOCAL ONLY — never commit)
│   ├── proguard-rules.pro            ← ProGuard/R8 rules
│   └── src/main/
│       ├── AndroidManifest.xml       ← permissions, launcher, FileProvider
│       ├── assets/channels.json      ← bundled fallback channel list
│       └── java/com/wclive/tv/
│           ├── MainActivity.java     ← entire app logic
│           └── AppConfig.java        ← private-repo URLs + split-string token
│       └── res/
│           ├── layout/activity_main.xml
│           ├── drawable/             ← all drawables
│           ├── values/               ← phone dimensions
│           ├── values-television/    ← Android TV dimensions
│           ├── values-sw600dp/       ← tablet
│           └── values-sw600dp-television/
├── updates/
│   ├── update.json                   ← local copy — push to wc-live-private on release
│   └── wc-live-v1.6.9.apk           ← local APK copies (keep all, never delete)
├── claude-output/
│   ├── WC Live Claude 1.6.9.apk     ← build artifacts (keep all, never delete)
│   └── CLAUDE_BUILD_REPORT_*.txt    ← session build reports
├── redistributable/                  ← release package for distribution
│   ├── WC Live 1.6.9-release.apk    ← release-signed APK
│   ├── SHA256SUMS.txt               ← integrity checksums
│   ├── INSTALL_GUIDE.md             ← user install instructions
│   ├── INSTALL_WARNING_GUIDE.md     ← explains Play Protect warnings
│   ├── RELEASE_NOTES_1.6.9.md      ← what changed
│   ├── update.json                  ← update manifest copy
│   └── updates/wc-live-v1.6.9.apk  ← APK for in-app updater reference
├── releases/                         ← older milestone APKs (1.4, 1.5)
├── design-reference/                 ← SVG brand assets + DESIGN.md
├── Web-Project/                      ← companion web app (separate Node.js project)
│   ├── backend/                      ← Express.js API server
│   └── frontend/                     ← Vite/React frontend
├── scripts/
│   ├── bump_minor_version.sh         ← increments versionName + versionCode
│   └── copy_versioned_apk.sh        ← copies built APK to releases/
├── gradle/                           ← Gradle wrapper
├── keystore.properties               ← LOCAL ONLY — never commit (gitignored)
├── keystore.properties.example       ← commit-safe template with placeholders
└── .gitignore                        ← covers *.jks, keystore.properties, build/
```

---

## Build instructions

### Prerequisites

- Android Studio or JDK 8+ and the Android SDK
- Gradle wrapper included (`./gradlew`)
- `keystore.properties` must exist at the project root (see Signing section below)

### Debug build (testing only)

```bash
cd ~/Documents/Projects/WC-Live
./gradlew assembleDebug
```

Output: `app/build/outputs/apk/debug/WC Live 1.6.9.apk`

Debug APKs are signed with the Android debug certificate. They are `debuggable=true`, not minified, and should never be distributed to users.

### Release build (for distribution)

```bash
cd ~/Documents/Projects/WC-Live
./gradlew assembleRelease
```

Output: `app/build/outputs/apk/release/WC Live 1.6.9-release.apk`

Release APKs are signed with the production keystore, `debuggable=false`, and R8-minified. **Always use release builds for distribution.**

### AAB (for Google Play)

```bash
cd ~/Documents/Projects/WC-Live
./gradlew bundleRelease
```

Output: `app/build/outputs/bundle/release/app-release.aab`

Google Play Console requires AAB, not APK. Use this for Play Store submissions.

### After every build

```bash
# Copy to build archive (adjust versionName)
cp "app/build/outputs/apk/release/WC Live 1.6.9-release.apk" "claude-output/WC Live Claude 1.6.9.apk"

# Copy to updates folder (used by in-app updater)
cp "app/build/outputs/apk/release/WC Live 1.6.9-release.apk" "updates/wc-live-v1.6.9.apk"

# Measure real APK size (for update.json apkSizeMB)
APK_BYTES=$(stat -f%z "updates/wc-live-v1.6.9.apk")
echo "scale=1; $APK_BYTES / 1048576" | bc
```

### Version management

- `versionCode` and `versionName` are set in **one place only**: `app/build.gradle`
- `versionCode` always increments by 1 — never reset, never reuse
- `versionName` uses semantic format: `1.6.9`, `1.7.0`, etc.
- After bumping, update `versionCode` and `versionName` in `updates/update.json`
- Auto-bump script: `./scripts/bump_minor_version.sh`

---

## Release signing

The release keystore is:
- **Location:** `app/wc-live-release.jks` (inside the module directory)
- **Config:** `keystore.properties` at the project root

`keystore.properties` format (do not commit this file):
```properties
storeFile=wc-live-release.jks
storePassword=<password>
keyAlias=wclive
keyPassword=<password>
```

**Critical:** Never regenerate the keystore. If you do, existing installs can no longer receive silent over-the-air updates because Android enforces signature continuity. If the keystore is lost, stop and ask the project owner before proceeding.

To generate a new keystore (only if starting fresh — not for this project):
```bash
keytool -genkeypair -v \
  -keystore wc-live-release.jks \
  -alias wclive \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000
```

---

## Release/distribution flow

### For sideload distribution

1. Build release APK: `./gradlew assembleRelease`
2. Copy to `updates/wc-live-vX.X.X.apk`
3. Update `updates/update.json` (versionCode, versionName, apkSizeMB, apkUrl, releaseNotes)
4. Push APK + update.json to `wc-live-private` repo (requires write access to private repo)
5. For public installs, also push to `wc-live` public repo (GitHub Release with asset named `wc-live.apk`)
6. Copy release APK to `redistributable/` and update SHA256SUMS.txt, RELEASE_NOTES

### For Google Play distribution (recommended for zero-warning installs)

1. Build AAB: `./gradlew bundleRelease`
2. Upload to Google Play Console → Production or Closed Testing track
3. Users install through Play Store — no sideload warnings

---

## Web project

The web companion lives at `./Web-Project/`. It has:
- `backend/` — Express.js API server (serves protected channel/APK routes)
- `frontend/` — Vite/React frontend

Web project README: `./Web-Project/README.md`

**Important:** The web project does not expose raw stream URLs. Channel data access is protected server-side.

```bash
# Install dependencies (from Web-Project/)
cd Web-Project && npm install

# Dev server
npm run dev

# Production build
cd frontend && npm run build
```

---

## Two-repo architecture (IMPORTANT)

As of v1.6.8, the app uses two GitHub repos:

| Repo | Visibility | Purpose |
|---|---|---|
| `Asif-Ahsan-Sunny/wc-live` | Public | README, GitHub Releases, first-install APK downloads |
| `Asif-Ahsan-Sunny/wc-live-private` | Private | In-app channel playlist, update.json, authenticated APK source |

The app's embedded PAT (`AppConfig.java`) is **read-only** on the private repo. Pushing new releases to `wc-live-private` requires separate write credentials.

Never hardcode private repo URLs or the token anywhere except `AppConfig.java`.

---

## Security rules

- **Never commit**: `keystore.properties`, `*.jks`, `*.keystore`, `.env`, tokens, passwords
- **Never expose**: private Git links, raw stream URLs, the embedded GitHub PAT
- **Never implement**: DRM bypass, geo-block bypass, paid stream bypass
- **Never attempt to bypass**: Play Protect, Android install prompts
- **Keep permissions minimal**: only INTERNET, REQUEST_INSTALL_PACKAGES, WRITE_EXTERNAL_STORAGE (≤API 28)
- **Sanitize logs**: do not log tokens, URLs containing credentials, or stream keys

---

## Architecture constraints (do not violate)

The following are **deliberate owner decisions**, not oversights. Do not introduce them even if they seem like improvements:

- No RecyclerView or DiffUtil — UI uses LinearLayout + addView()
- No Fragments
- No ViewModel or LiveData
- No Jetpack Compose
- No WebView
- No third-party HTTP clients — uses plain HttpURLConnection
- No image loading libraries — manual Bitmap loading

See `CLAUDE.md` sections 9 and 16 for the full constraint list and the reasoning behind each.

---

## Known issues / next steps

| Issue | Status |
|---|---|
| No retry button on playback error | Open |
| Auto-reconnect on transient network errors | Open |
| False-negative availability dots (streams rejecting HEAD) | Open |
| Force-update + no internet = immediate retry loop | Open |
| Adaptive/gradient launcher icon (needs API 26+) | Open |
| Play Protect warnings on sideloaded APK | Inherent to sideloading — use Play Console to eliminate |
| Private repo write access for CI/CD release push | Requires separate PAT with write scope |

**Recommended next step for easiest user distribution:** enroll in Google Play Console, build an AAB, and publish to a Closed Testing or Internal Testing track.

---

## Contacts

- Asif Ahsan Sunny
- Kazi Hasan Shariare
