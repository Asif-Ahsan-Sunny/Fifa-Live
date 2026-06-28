# WC Live — AI Project Guide

This file is the authoritative reference for any AI session working on this project.
Read it fully before making any changes. It covers architecture, build rules, release workflow, design system, and known pitfalls.

---

## 1. Project Identity

| Field | Value |
|---|---|
| App name | WC Live |
| Package name | `com.wclive.tv` |
| Type | Native Android Java — IPTV player |
| Target devices | Android TV (primary), phone, tablet |
| Player | Media3 ExoPlayer 1.3.1 (HLS + MPEG-TS) |
| Architecture | Single Activity (`MainActivity.java`) |
| UI system | Programmatic Java — no RecyclerView, no ViewBinding, no Fragments |
| GitHub repo | https://github.com/Asif-Ahsan-Sunny/wc-live |
| Channel playlist | `wc live.txt` in the GitHub repo (M3U format) |
| Update manifest | `updates/update.json` in the GitHub repo |
| Design system | **Midnight Cinema** (dark premium, action red accent) |

---

## 2. File Structure (what matters)

```
WC-Live/
├── CLAUDE.md                        ← this file (read first)
├── CLAUDE_SESSION_LOG.md            ← version history and session notes
├── BUILD_INSTRUCTIONS.md            ← Gradle build commands
├── RELEASE.md                       ← release notes template
├── app/
│   ├── build.gradle                 ← version management HERE
│   └── src/main/
│       ├── AndroidManifest.xml      ← permissions, provider
│       ├── assets/channels.json     ← bundled fallback channel list
│       └── java/com/wclive/tv/
│           └── MainActivity.java    ← entire app logic (~2100 lines)
│       └── res/
│           ├── layout/activity_main.xml   ← full UI layout
│           ├── drawable/            ← all drawables (bg_*, ic_cat_*)
│           ├── values/dimens.xml    ← phone dimensions
│           ├── values-television/dimens.xml   ← Android TV dimensions
│           ├── values-sw600dp/dimens.xml      ← tablet
│           └── values-sw600dp-television/     ← large TV
├── updates/
│   ├── update.json                  ← local copy — push to GitHub on release
│   └── wc-live-vX.X.X.apk          ← local copy — push to GitHub on release
├── claude-output/
│   ├── WC Live Claude X.X.X.apk    ← build artifacts (keep all, never delete)
│   └── CLAUDE_BUILD_REPORT_X.X.X.txt
├── releases/                        ← older milestone APKs
└── design-reference/                ← SVG assets and design tokens (DESIGN.md)
```

---

## 3. Version Management Rules

**Always follow this exactly:**

1. `versionName` uses semantic format: `1.6.7`, `1.7.0`, etc.
2. `versionCode` is an integer that **always increments by 1** from the current value — never reset, never skip.
3. The version is set in **one place only**: `app/build.gradle`.
4. After bumping, the same `versionCode` must be reflected in `updates/update.json`.

**Current version as of last session:**

| versionName | versionCode | Date |
|---|---|---|
| 1.6.9 | 15 | 2026-06-25 |

**Next version will be:** versionCode 16, versionName TBD by user.

**Never:**
- Use the same versionCode twice
- Set versionCode lower than the current value
- Change versionName without also bumping versionCode
- Hard-code the version anywhere other than `app/build.gradle`

---

## 4. Build Process

### Standard debug build

```bash
cd ~/Documents/Projects/WC-Live
./gradlew assembleDebug
```

Output: `app/build/outputs/apk/debug/WC Live {versionName}.apk`

### After every build — copy to both output locations

```bash
# Replace X.X.X with the actual versionName
cp "app/build/outputs/apk/debug/WC Live X.X.X.apk" "claude-output/WC Live Claude X.X.X.apk"
cp "app/build/outputs/apk/debug/WC Live X.X.X.apk" "updates/wc-live-vX.X.X.apk"
```

**Never delete older APKs.** Both directories accumulate all builds.

### Get the real APK size in MB (needed for update.json)

```bash
APK_BYTES=$(stat -f%z "updates/wc-live-vX.X.X.apk")
echo "scale=1; $APK_BYTES / 1048576" | bc
```

---

## 5. The update.json Format

This file is read by the app on startup to check for updates. The local copy lives at `updates/update.json`. After building, this file must be updated and pushed to GitHub.

```json
{
  "appName": "WC Live",
  "packageName": "com.wclive.tv",
  "versionCode": 12,
  "versionName": "1.7.0",
  "minimumSupportedVersion": 1,
  "forceUpdate": false,
  "updateAvailable": true,
  "releaseDate": "YYYY-MM-DD",
  "apkSizeMB": 6.9,
  "apkUrl": "https://raw.githubusercontent.com/Asif-Ahsan-Sunny/wc-live-private/main/wc-live-apk/wc-live-v1.7.0.apk",
  "releaseNotes": [
    "What changed in this version",
    "Another change"
  ]
}
```

**Rules:**
- `packageName` must always be `com.wclive.tv` — any other value is silently ignored by the app
- `versionCode` must be higher than the installed app's versionCode for the update dialog to show
- `apkUrl` uses the GitHub blob URL format — the app internally converts it to raw.githubusercontent.com
- `forceUpdate: true` — user cannot dismiss the dialog; Back shows a toast and re-focuses the install button
- `forceUpdate: false` — user can dismiss with Back or tap "Later"
- `minimumSupportedVersion` — if the installed versionCode is below this, force update is activated even if `forceUpdate` is false. Set to `1` unless deliberately restricting old versions.
- Use the real APK size from the stat command above — do not guess or use a round number

---

## 6. GitHub Workflow (every release)

The GitHub repo is `Asif-Ahsan-Sunny/wc-live`. The `gh` CLI is authenticated and has push access.

**The local clone for pushes lives at `/tmp/wc-live`. Pull before every session.**

### Step-by-step release workflow

```bash
# 1. Pull the latest from GitHub
cd /tmp/wc-live
git pull

# 2. Copy the new APK and update.json into the clone
cp "~/Documents/Projects/WC-Live/updates/wc-live-vX.X.X.apk" updates/
cp "~/Documents/Projects/WC-Live/updates/update.json" updates/

# 3. Update the README changelog (see README section below)
# Edit /tmp/wc-live/README.md — add the new version under "## 📋 Changelog"

# 4. Stage, commit, push
git add updates/wc-live-vX.X.X.apk updates/update.json README.md
git commit -m "Release vX.X.X — short description of what changed"
git push

# 5. Create (or recreate) the GitHub release
# If a release for this tag already exists, delete it first:
gh release delete vX.X.X --repo Asif-Ahsan-Sunny/wc-live --yes

gh release create vX.X.X \
  "~/Documents/Projects/WC-Live/updates/wc-live-vX.X.X.apk#wc-live.apk" \
  --repo Asif-Ahsan-Sunny/wc-live \
  --title "WC Live vX.X.X — Brief Title" \
  --notes "$(cat <<'EOF'
Release notes here (markdown supported)
EOF
)" \
  --latest
```

**The asset is always named `wc-live.apk` in the release** (via the `#wc-live.apk` alias). This is the filename users see when downloading.

### What lives on GitHub

```
wc live.txt              ← M3U channel playlist
updates/
  update.json            ← version manifest the app polls
  wc-live-vX.X.X.apk    ← all versioned APKs (keep them all)
README.md                ← project page
```

---

## 7. Build Report (required every release)

Create `claude-output/CLAUDE_BUILD_REPORT_X.X.X.txt`. It must include:

- Final versionName and versionCode
- APK output paths (all three)
- update.json path and key fields
- Root cause or change description
- Files modified (list every file)
- What was NOT changed (confirms no regressions)
- Known limitations
- Testing checklist (mark what was verified vs what needs device testing)
- GitHub actions completed

Use the existing reports in `claude-output/` as templates.

---

## 8. Session Log (update every session)

`CLAUDE_SESSION_LOG.md` is the version history. After every build:

1. Add a row to the version table
2. Add a session note at the bottom

This file is what future AI sessions read to understand what was done previously. Keep it accurate and concise.

---

## 9. Architecture — Critical Rules

### What this app IS

- Single Java Activity: `com.wclive.tv.MainActivity`
- All UI is built programmatically with `addView()` — no RecyclerView, no DiffUtil, no adapters
- Channel list is a `LinearLayout` inside a `ScrollView`
- Categories are built similarly
- ExoPlayer handles all video playback
- SharedPreferences stores channel cache and status cache

### What this app is NOT

- No fragments
- No ViewModel or LiveData
- No Jetpack Compose
- No WebView
- No third-party HTTP clients (plain `HttpURLConnection`)
- No image loading libraries (manual Bitmap loading in `logoExecutor`)

### Do NOT introduce any of the above.

### Thread model

| Executor | Threads | Purpose |
|---|---|---|
| `logoExecutor` | 3 | Logo image downloads |
| `syncExecutor` | 1 | Channel playlist sync |
| `searchExecutor` | 1 | Search filtering |
| `updateExecutor` | 1 | Update JSON fetch + APK download |
| `availabilityExecutor` | 2 (TV) / 3 (phone) | Channel online checks |

**All UI updates must be posted to `handler` (main thread). Never update views from executor threads.**

### The `isModalDialogVisible` flag — CRITICAL

This boolean was added in v1.6.7 to fix the Android TV freeze bug. **You must respect it in all future code.**

When any modal dialog is shown (update dialog, download progress, about), set:
```java
isModalDialogVisible = true;
```

When any modal dialog is dismissed, set:
```java
isModalDialogVisible = false;
```

The following methods check this flag and return early if it is true:
- `showOverlay()` — overlay will not appear or manipulate z-order
- `scheduleStatusDotRefresh()` — dot updates are suppressed
- `startAvailabilityChecker()` — new checks will not start
- `replaceChannels()` — focus steal and availability restart are suppressed

**If you add any new modal or dialog, follow the same pattern.**

### Z-order rules (FrameLayout in activity_main.xml)

The layout stacks views in this order (last = on top):

```
PlayerView           (background video)
videoTapLayer        (touch intercept)
emptyState           (no channel selected)
loadingView          (buffering spinner)
errorView            (playback error)
overlay              (channel list + sidebar) ← DO NOT call bringToFront() here
aboutModal           (about dialog)
updateDialogView     (update dialog)
downloadProgressView (APK download progress)
launchOverlay        (splash screen)
```

**Never call `overlay.bringToFront()`** — this was the root cause of the v1.6.7 freeze bug. The overlay is already correctly positioned above the player but below the dialogs in the XML.

`aboutModal`, `updateDialogView`, and `downloadProgressView` are allowed to call `bringToFront()` because they are modal and need to stay above everything.

---

## 10. Startup Sequence (do not change without reason)

```
t=0ms      onCreate — load cached channels, render UI
t=~500ms   launch animation fade-out
t=2500ms   background channel sync (executor, not main thread)
t=5000ms   app update JSON check (executor, not main thread)
t=8000ms   availability checker start (2 threads on TV, 3 on phone)
t=3600000ms  hourly sync repeat
```

The key rule: **render first, network second.** The app must be visually usable before any network call happens.

---

## 11. Design System — Midnight Cinema

**Do not deviate from these tokens.**

| Token | Hex | Usage |
|---|---|---|
| `bg_dark` | `#0B0B0C` | App root background |
| `surface` | `#131313` | Panel/sidebar background |
| `surface_low` | `#1C1B1B` | Card backgrounds |
| `action_red` | `#E50914` | Focus ring, live badge, active state |
| `text_primary` | `#E5E2E1` | All primary text |
| `text_secondary` | `#BDB8B7` | Meta text, timestamps |
| `pitch_green` | `#22d480` | Online status dot |
| `divider` | `#73BDB8B7` | Sidebar divider (45% opacity off-white) |

**Focus states on Android TV:**
- Active category/channel: `bg_category_item_active` / `bg_channel_item_active` (red-tinted drawable)
- The D-pad focus ring is the background drawable — there is no separate ring drawn in code

**Typography:** The app uses system fonts. Text sizes are defined in `dimens.xml` per device class and applied with `setTextSizeFromDimen()`.

---

## 12. Dimension Variants

| Folder | Target device |
|---|---|
| `values/` | Phone |
| `values-sw600dp/` | Tablet (600dp+) |
| `values-television/` | Android TV |
| `values-sw600dp-television/` | Large TV screen |

When adding a new dimension, add it to all four files. The TV values tend to be smaller (less screen real estate per dp at 10-foot viewing distance with the overlay layout).

**Key TV dimensions:**
- `about_dialog_width`: 520dp (update and about dialogs share this)
- `category_panel_width`: 210dp
- `channel_row_height`: 48dp

---

## 13. Update System — How It Works

1. At startup (t=5s), `checkForUpdates()` fetches `update.json` from GitHub on `updateExecutor`
2. If `versionCode` in JSON > installed `versionCode`, `showUpdateDialog()` is called on the main thread
3. User taps "Update Now" → `startApkDownload()` downloads the APK to `getExternalCacheDir()`
4. On download complete → `installApk()` triggers the system installer via FileProvider
5. FileProvider authority: `com.wclive.tv.provider`

**GitHub blob URL → raw URL conversion** is handled automatically by `resolveRawGithubUrl()`:
```
https://github.com/Asif-Ahsan-Sunny/wc-live/blob/main/updates/wc-live-v1.6.7.apk
→
https://raw.githubusercontent.com/Asif-Ahsan-Sunny/wc-live/main/updates/wc-live-v1.6.7.apk
```
Always use the blob URL in `update.json` — the app converts it.

---

## 14. Channel Playlist — How It Works

1. App fetches `wc live.txt` from GitHub (M3U format) via `REMOTE_PLAYLIST_URL`
2. Parsed by `parseM3uPlaylist()` — reads `#EXTINF` lines for name/category/logo, next line for URL
3. Result is stored as JSON in SharedPreferences (`wc_live_channels`)
4. On next launch, cached JSON is loaded in `loadInitialChannels()` without network
5. Bundled `assets/channels.json` is the fallback if both cache and network fail

**To update the channel list:** edit `wc live.txt` in the GitHub repo. The app fetches it automatically within 1 hour, or immediately when the user taps "Update channels".

**M3U format required by the parser:**
```
#EXTINF:-1 group-title="Category Name" tvg-logo="https://logo.url/img.png", Channel Name
https://stream.url/live.m3u8
```

**Supported categories** (must match exactly for icons to work):
`Bangla`, `Sports`, `News`, `Movies`, `Music`, `Kids`, `Religious`, `Entertainment`, `Documentary`, `Fifa Live`

Any other category gets the default globe icon.

---

## 15. Back Key Priority Chain

This is fixed behavior — do not change the priority order:

1. **Search open** → close search
2. **Download progress visible** → cancel download (optional only)
3. **Update dialog visible** → dismiss if optional; show toast if forced
4. **About modal visible** → close about
5. **Overlay visible** → hide overlay
6. **None of the above** → first press: toast "Press back again to close the app"; second press within 2s: `finish()`

---

## 16. What to NEVER Do

- **Never call `overlay.bringToFront()`** — this is the bug that caused the v1.6.7 freeze
- **Never run network code on the main thread** — use executors
- **Never parse M3U or JSON on the main thread**
- **Never call `notifyDataSetChanged()` or rebuild the channel list while a modal is visible**
- **Never introduce RecyclerView, Fragments, ViewModel, Compose, or third-party HTTP libraries** — the owner has deliberately kept this architecture simple
- **Never bring back "Reload defaults"** — it was intentionally removed in v1.6.5
- **Never add DRM bypass, geo-block bypass, or paid stream access logic**
- **Never commit `.env` files, keystore files, or `keystore.properties`**
- **Never delete older APKs from `claude-output/` or `updates/`**

---

## 17. Per-Release Checklist

Run through this for every build before reporting done:

### Code
- [ ] `versionCode` incremented by 1 in `app/build.gradle`
- [ ] `versionName` updated correctly
- [ ] No new calls to `overlay.bringToFront()`
- [ ] Any new modal dialog sets/clears `isModalDialogVisible`
- [ ] No network calls added to the main thread
- [ ] No UI updates made directly from executor threads

### Build
- [ ] `./gradlew assembleDebug` — BUILD SUCCESSFUL
- [ ] APK copied to `claude-output/WC Live Claude X.X.X.apk`
- [ ] APK copied to `updates/wc-live-vX.X.X.apk`
- [ ] APK size measured with `stat -f%z` (not guessed)

### update.json
- [ ] `versionCode` matches `app/build.gradle`
- [ ] `packageName` is `com.wclive.tv`
- [ ] `apkUrl` points to the new APK filename
- [ ] `apkSizeMB` is the real measured size
- [ ] `forceUpdate` is set intentionally (confirm with user)
- [ ] JSON is valid (no trailing commas, correct quotes)

### GitHub
- [ ] `/tmp/wc-live` pulled before making any changes
- [ ] New APK pushed to `updates/`
- [ ] `update.json` pushed to `updates/`
- [ ] `README.md` changelog updated with the new version
- [ ] Commit message is descriptive (not just "update")
- [ ] GitHub Release created with tag `vX.X.X`
- [ ] Release marked as Latest
- [ ] APK asset named `wc-live.apk` in the release

### Reports
- [ ] `claude-output/CLAUDE_BUILD_REPORT_X.X.X.txt` created
- [ ] `CLAUDE_SESSION_LOG.md` version table updated
- [ ] `CLAUDE_SESSION_LOG.md` session notes updated
- [ ] `CLAUDE.md` updated if any architecture changed

### Verification
- [ ] App builds without errors (warnings about deprecated API are acceptable)
- [ ] Confirmed no existing feature was broken (state which ones were checked)

---

## 18. GitHub Access

The `gh` CLI is authenticated as `mromi16` with `repo` scope (push access). No additional credentials are needed.

```bash
# Verify access at any time
gh auth status

# Check repo permissions
gh api "repos/Asif-Ahsan-Sunny/wc-live" --jq '.permissions'
```

The local working clone: `/tmp/wc-live` (may not exist — clone fresh if needed)

```bash
git clone https://github.com/Asif-Ahsan-Sunny/wc-live.git /tmp/wc-live
```

---

## 19. Known Issues and History

| Issue | Status | Fixed in |
|---|---|---|
| Android TV freeze — update dialog appeared behind channel rows | Fixed | v1.6.7 |
| D-pad focus stolen by channel sync while update dialog visible | Fixed | v1.6.7 |
| Availability checker overloading main thread during startup | Fixed | v1.6.7 |
| Forced update Back did nothing (no feedback) | Fixed | v1.6.7 |
| Package name was `com.wcfix.tvapp` | Fixed (renamed) | v1.6.5 |
| "Reload defaults" showed in sidebar | Removed | v1.6.5 |
| Adaptive/gradient launcher icon | Open | — |
| False-negative availability dots (streams rejecting HEAD) | Open | — |
| Force update + no internet = repeated dialog with no delay | Open | — |

---

## 20. Contact / Credits

App prepared by:
- Asif Ahsan Sunny
- Kazi Hasan Shariare
