# WC Live — Project Migration & Release Package Report

**Date:** 2026-06-29  
**Session type:** Project path migration, documentation update, redistributable package preparation  
**App version:** 1.6.9 (versionCode 15)

---

## 1. New project path confirmed

```
~/Documents/Projects/WC-Live
```

The project was previously at `/Users/abc/Desktop/android-tv-apk`. All documentation has been updated. The old path is referenced only in historical build reports inside `claude-output/` — those files are kept as-is because they are timestamped records of past sessions.

---

## 2. Old path references found and replaced

| File | Old reference | Action |
|---|---|---|
| `CLAUDE.md` | `android-tv-apk/` (file structure root label) | Replaced with `WC-Live/` |
| `CLAUDE.md` | `cd /Users/abc/Desktop/android-tv-apk` (build command) | Replaced with `cd ~/Documents/Projects/WC-Live` |
| `CLAUDE.md` | `"/Users/abc/Desktop/android-tv-apk/updates/..."` (GitHub workflow cp) | Replaced with relative `~/Documents/Projects/WC-Live/updates/...` |
| `CLAUDE.md` | `"/Users/abc/Desktop/android-tv-apk/updates/..."` (gh release asset) | Replaced with `~/Documents/Projects/WC-Live/updates/...` |
| `BUILD_INSTRUCTIONS.md` | `cd /Users/abc/Desktop/android-tv-apk` | Replaced with `cd ~/Documents/Projects/WC-Live` |
| `claude-output/CLAUDE_BUILD_REPORT_1.6.txt` | Multiple old Desktop paths | Left unchanged — historical record |
| `claude-output/SECURITY_AND_RELEASE_SETUP_v1.6.8.txt` | Old Desktop paths | Left unchanged — historical record |
| `Web-Project/WEB_BUILD_REPORT.txt` | Old Desktop/android-tv-apk path | Left unchanged — historical record |

---

## 3. Documentation files updated or created

| File | Action |
|---|---|
| `CLAUDE.md` | Updated: file structure root, build command, GitHub workflow paths, version table (was v1.6.7/code 11, now v1.6.9/code 15), update.json example URL corrected to private repo |
| `BUILD_INSTRUCTIONS.md` | Updated: `cd` command to new project path |
| `keystore.properties.example` | Updated: corrected storeFile relative path and added explanatory comment |
| `PROJECT_HANDOFF.md` | Created: full AI/developer handoff document |
| `CLAUDE_PROJECT_UPDATE_REPORT.md` | Created: this file |

---

## 4. Android build status

Both APKs exist from the 2026-06-25 session:

| Build type | File | Size |
|---|---|---|
| Release (signed, minified) | `app/build/outputs/apk/release/WC Live 1.6.9-release.apk` | 1.88 MB |
| Debug (testing only) | `app/build/outputs/apk/debug/WC Live 1.6.9.apk` | 6.45 MB |

No new build was triggered in this session — the existing 1.6.9 builds are current and no code changes were made.

---

## 5. Release signing status

**Signing is fully configured and working.**

| Item | Status |
|---|---|
| Keystore file | `app/wc-live-release.jks` — present, gitignored |
| `keystore.properties` | Present, gitignored |
| `build.gradle` signing config | Configured — loads from keystore.properties |
| Release builds debuggable | No (`debuggable false`) |
| Minification (R8) | Enabled (`minifyEnabled true`) |
| Signed APK verified | Yes — verified with jarsigner in v1.6.8 session |

**Do not regenerate the keystore.** Existing installs depend on the same signing key for update compatibility.

---

## 6. Redistributable package created

Location: `./redistributable/`

| File | Description |
|---|---|
| `WC Live 1.6.9-release.apk` | Release-signed APK (1.88 MB) |
| `SHA256SUMS.txt` | SHA-256 checksum for the release APK |
| `INSTALL_GUIDE.md` | User install instructions (USB, ADB, enable unknown sources) |
| `INSTALL_WARNING_GUIDE.md` | Full explanation of Play Protect warnings and what can/cannot be fixed |
| `RELEASE_NOTES_1.6.9.md` | What changed in v1.6.9 |
| `update.json` | Copy of the in-app update manifest |
| `updates/wc-live-v1.6.9.apk` | Versioned APK copy for in-app updater reference |

SHA-256 of release APK:
```
fcc621cc05c777e8e8d1bbb69d978489c688dd9c48d46c700718ecf236ade525  WC Live 1.6.9-release.apk
```

No AAB was created in this session. To create one: `./gradlew bundleRelease`

---

## 7. Permissions audit

| Permission | Status | Reason |
|---|---|---|
| `INTERNET` | Kept | Channel sync, logo loading, update check |
| `REQUEST_INSTALL_PACKAGES` | Kept | In-app APK update flow — required |
| `WRITE_EXTERNAL_STORAGE` (maxSdk 28) | Kept | APK download on API 21–28 only |
| All other permissions | Not present | No camera, mic, location, contacts, etc. |

Permissions are minimal. No removals recommended.

---

## 8. Signing and install-warning audit results

| Check | Result |
|---|---|
| Built as debug or release? | Release (since v1.6.8) |
| Release signing configured? | Yes |
| Package name stable? | Yes — `com.wclive.tv` since v1.6.5 |
| versionName/versionCode valid? | Yes — 1.6.9 / 15 |
| Launcher icon/banner assets valid? | Yes — ic_launcher, ic_launcher_round, tv_banner all present |
| Permissions minimal? | Yes |
| `REQUEST_INSTALL_PACKAGES` justified? | Yes — in-app updater only |
| WebView used? | No |
| Proxy/test files packaged? | No — R8 strips unused code |
| Debug flags in release? | No (`debuggable false`) |
| `android:debuggable` false in release? | Yes |
| Minify/R8 configured? | Yes (`minifyEnabled true`) |
| Stable signing key for future updates? | Yes — same keystore since v1.6.8 |

---

## 9. Install warning explanation

Android shows warnings when installing APKs outside Google Play. This is a platform behavior — it is not caused by anything wrong with the app. The release-signed APK in this package reduces friction compared to a debug APK, but some Play Protect prompts will still appear for sideloaded apps.

**What cannot be eliminated without Google Play:**
- "Install unknown apps" one-time permission prompt
- Play Protect "App not recognized" warning

**Best recommendation for easiest user install:**  
Enroll in Google Play Console ($25 one-time), build an AAB (`./gradlew bundleRelease`), and publish to an Internal Testing or Closed Testing track. Users who install through a Play Console testing link get a normal Play Store experience with no sideload warnings.

See `redistributable/INSTALL_WARNING_GUIDE.md` for the full breakdown.

---

## 10. .gitignore audit

Current `.gitignore` already covers:
- `*.jks` — keystore files
- `*.keystore` — keystore files
- `keystore.properties` — signing credentials
- `build/`, `app/build/`, `.gradle/`, `.idea/`, `local.properties` — IDE and build artifacts

No additions needed.

---

## 11. Files modified this session

- `CLAUDE.md` — 5 old path references updated, version table updated, update.json example URL corrected
- `BUILD_INSTRUCTIONS.md` — 1 old path reference updated
- `keystore.properties.example` — updated storeFile path and added explanatory comment

### Files created this session

- `PROJECT_HANDOFF.md` — comprehensive handoff document
- `CLAUDE_PROJECT_UPDATE_REPORT.md` — this file
- `redistributable/WC Live 1.6.9-release.apk` — release APK copy
- `redistributable/SHA256SUMS.txt` — checksums
- `redistributable/INSTALL_GUIDE.md` — user install guide
- `redistributable/INSTALL_WARNING_GUIDE.md` — Play Protect warning explanation
- `redistributable/RELEASE_NOTES_1.6.9.md` — release notes
- `redistributable/update.json` — update manifest copy
- `redistributable/updates/wc-live-v1.6.9.apk` — versioned APK copy

---

## 12. Commands run this session

```bash
# Structural inspection
find ~/Documents/Projects/WC-Live -maxdepth 3 ...

# Checksum generation
shasum -a 256 "app/build/outputs/apk/release/WC Live 1.6.9-release.apk"

# Redistributable folder setup
mkdir -p redistributable/updates
cp "app/build/outputs/apk/release/WC Live 1.6.9-release.apk" redistributable/
cp updates/update.json redistributable/
cp updates/wc-live-v1.6.9.apk redistributable/updates/
```

No new Gradle builds were run — existing 1.6.9 APKs are current.

---

## 13. Testing checklist

### Documentation
- [x] No old absolute path remains in active documentation (old paths in historical `claude-output/` reports are expected)
- [x] `PROJECT_HANDOFF.md` exists and covers overview, folder map, build steps, signing, release flow, architecture constraints, security rules
- [x] `BUILD_INSTRUCTIONS.md` uses new path
- [x] `CLAUDE.md` uses new path in all commands and version table is current
- [x] Future AI/developer can understand the project from `PROJECT_HANDOFF.md` + `CLAUDE.md` alone

### Android
- [x] App builds successfully (confirmed — release APK exists from 2026-06-25)
- [x] Version info correct (1.6.9 / versionCode 15)
- [ ] APK install on test device — requires physical device (not verified in this session)
- [ ] App launches and playback works — requires physical device
- [x] Permissions minimal
- [x] Release build is not debuggable

### Redistributable
- [x] `redistributable/` exists
- [x] Release APK present and labeled clearly
- [x] SHA256SUMS.txt exists
- [x] INSTALL_GUIDE.md exists
- [x] INSTALL_WARNING_GUIDE.md exists
- [x] Release notes exist
- [x] update.json copy exists
- [x] updates/ APK copy exists

### Security
- [x] Keystore not committed (gitignored, only `keystore.properties.example` committed)
- [x] No passwords/tokens in committed files
- [x] No raw private repo links in committed documentation
- [x] No Play Protect bypass logic added
- [x] No DRM/geo/paid stream bypass added

---

## 14. Remaining steps (not done in this session)

1. **Device test**: Install `redistributable/WC Live 1.6.9-release.apk` on a real Android TV or phone, confirm app launches, channels load, playback works, in-app update prompt appears when a newer versionCode is in update.json.

2. **Google Play Console**: For zero-warning distribution, enroll at play.google.com/console, build an AAB (`./gradlew bundleRelease`), and upload to a testing track.

3. **Private repo write access**: Pushing new `update.json` and versioned APKs to `wc-live-private` requires a PAT with write scope (the embedded app PAT is read-only by design). Coordinate with the repo owner.

4. **Media3 version bump (deferred)**: ExoPlayer 1.3.1 is behind the current stable release. Update `media3-exoplayer*` in `app/build.gradle` and regression-test playback on real hardware before distributing.
