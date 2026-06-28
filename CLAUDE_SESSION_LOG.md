# WC Live — Claude Session Log

This file is maintained so any future AI session can pick up context without needing prior conversation history.

---

## Project Overview

**App name:** WC Live  
**Type:** Native Android Java/XML app  
**Purpose:** IPTV player for Android TV, phone, and tablet  
**Package:** `com.wclive.tv` (changed from `com.wcfix.tvapp` in v1.6.5)  
**Single activity:** `com.wclive.tv.MainActivity`  
**Player:** Media3 ExoPlayer (HLS, MPEG-TS)  
**Channel source:** Private GitHub repo (token-authenticated), M3U playlist, cached in SharedPreferences, bundled fallback — see section "Private Repo + AppConfig (v1.6.8)" below  

---

## Version History

| Version | versionCode | Build date | Summary |
|---|---|---|---|
| 1.4 | 5 | before 2026-06-21 | Base working build |
| 1.5 | 6 | before 2026-06-21 | Polish pass |
| 1.6 | 7 | 2026-06-21 | Round icon fix, TV dimen reduction, now-playing title bug fix, wordmark in empty state |
| 1.6.5 | 8 | 2026-06-23 | Remove "Reload defaults", in-app APK updater, channel availability checker (green dots), package rename to com.wclive.tv |
| 1.6.6 | 9 | 2026-06-23 | Sidebar divider opacity increased (#2A353534 → #4DBDB8B7), prepared for self-update testing |
| 1.6.7 | 11 | 2026-06-23 | Fix Android TV startup freeze: remove overlay.bringToFront(), add isModalDialogVisible gate, stagger startup timings, TV availability concurrency 2 threads |
| 1.6.7 | 12 | 2026-06-23 | Fix About/Update/Download modal D-pad trapping (return true for all keys), add NUMPAD_ENTER support, fix tv_banner.xml viewport to 864x486 (16:9), UPDATE_CHECK_DELAY_MS 5000→30000 |
| 1.6.7 | 13 | 2026-06-23 | Full D-pad navigation inside About/Update dialogs: scroll area gets focus first, D-pad Down scrolls then moves to buttons, D-pad Up reverses; subtle focus highlight added |
| 1.6.8 | 14 | 2026-06-25 | Migrated to private GitHub repo + token auth (AppConfig.java), switched to signed/minified release builds (was unsigned debug builds the whole time before this) — see section 21 |
| 1.6.9 | 15 | 2026-06-25 | Audit-driven perf/reliability pass: removed unused Leanback dependency, bounded logo cache LRU, row-level channel-list updates instead of full rebuild on channel selection, global uncaught-exception logger, logging added to several previously-silent catches |

---

## Architecture Notes

- **Single-activity** architecture. All UI is programmatic Java (no ViewBinding, no RecyclerView/DiffUtil — uses LinearLayout with dynamic addView).
- **Channel list** is a `LinearLayout` inside a `ScrollView`. Views are rebuilt on filter/category change.
- **Status dots** are stored in `channelStatusDots` (parallel list to `channelViews`) and updated via batched 100ms runnable.
- **Executors**: logoExecutor (3 threads), syncExecutor (1), searchExecutor (1), updateExecutor (1), availabilityExecutor (3, restartable).
- **SharedPreferences**:
  - `wc_live_channels` — channel cache (JSON), last sync timestamp, count, error
  - `wc_live_status` — channel availability cache (URL → {status, timestamp} JSON)
- **Immersive mode** applied on onCreate, onResume, onWindowFocusChanged.
- **Back chain priority**: search → download dialog → update dialog → about modal → overlay → double-back-exit.

---

## Key Constants (MainActivity.java)

```java
// Playlist and update.json URLs + token now live in AppConfig.java (private repo, see section 21)
CHANNEL_SYNC_INTERVAL_MS = 1 hour
UPDATE_CHECK_DELAY_MS    = 30 seconds (after launch)
AVAILABILITY_START_DELAY_MS = 8 seconds (after launch)
STATUS_TTL_ONLINE_MS  = 45 minutes
STATUS_TTL_OFFLINE_MS = 15 minutes
STATUS_CHECK_CONCURRENCY = 3 threads (phone) / 2 threads TV (STATUS_CHECK_CONCURRENCY_TV)
STATUS_CHECK_TIMEOUT_MS  = 8 seconds
```

---

## Permissions (AndroidManifest.xml)

```
INTERNET                    — channel sync, logo loading, update check
REQUEST_INSTALL_PACKAGES    — in-app APK install flow
WRITE_EXTERNAL_STORAGE (maxSdkVersion=28) — APK download on API 21–28
```

FileProvider authority: `com.wclive.tv.provider`  
FileProvider paths: `res/xml/file_paths.xml` (external-cache-path + cache-path)

---

## Design System (Midnight Cinema)

| Token | Value |
|---|---|
| bg_dark | #0B0B0C |
| surface | #131313 |
| surface_low | #1C1B1B |
| action_red | #E50914 |
| text_primary | #E5E2E1 |
| text_secondary | #BDB8B7 |
| pitch_green | #22d480 (online dot) |

Drawables: bg_channel_item, bg_channel_item_active, bg_category_item, bg_category_item_active, bg_overlay, bg_panel, bg_error_panel, bg_status_pill, bg_watching_badge, bg_header_icon, bg_logo_tile, bg_search_box, bg_app_root

Logo assets: ic_launcher.xml, ic_launcher_round.xml, tv_banner.xml, wc_wordmark.xml — all match design-reference/ SVGs.

---

## Dimension Variants

| Qualifier | Target |
|---|---|
| values/ | Phones |
| values-sw600dp/ | Tablets |
| values-television/ | Android TV |
| values-sw600dp-television/ | Large TV |

---

## 21. Private Repo + AppConfig (v1.6.8) — READ BEFORE TOUCHING NETWORKING CODE

As of versionCode 14, channel sync and update checks no longer hit the public
`wc-live` repo unauthenticated. They hit a private repo with a token.

**Two repos, two purposes — do not collapse this distinction:**
- `Asif-Ahsan-Sunny/wc-live` (PUBLIC) — README, GitHub Releases, manual-install
  downloads via the Downloader app. No token. Keep this alive for first installs.
- `Asif-Ahsan-Sunny/wc-live-private` (PRIVATE) — the app's actual in-app source of
  truth for `update.json` and the channel playlist, fetched with the embedded token.

**`AppConfig.java`** (`app/src/main/java/com/wclive/tv/AppConfig.java`) holds:
- `token()` — the GitHub PAT, split into fragments, joined at runtime via
  `StringBuilder` (this is deliberate — concatenating `static final String` fields
  with `+` would let javac fold them back into one literal at compile time, which
  defeats the whole point)
- `updateUrl()` → `raw.githubusercontent.com/.../wc-live-private/main/updates/update.json`
- `channelsUrl()` → `raw.githubusercontent.com/.../wc-live-private/main/wc-channels/channels.txt`

**Never hardcode the token or these URLs anywhere else.** Always call `AppConfig.*()`.

**Every GitHub fetch must send these headers** (already wired into `downloadText()`
and the APK download block in `MainActivity.java` — do not remove if refactoring):
```
Authorization: token <AppConfig.token()>
Accept: application/vnd.github.v3.raw
```
This was verified live (curl) to work against `raw.githubusercontent.com` for a
private repo — do not assume it doesn't work and switch to `api.github.com` without
testing first; both work, but the URLs in `AppConfig.java` are the raw CDN ones.

**Never add the GitHub token to:** the stream-availability checker
(`checkStreamAvailability`) or the logo loader — those hit arbitrary third-party
URLs from the M3U playlist, not GitHub. Sending the token there would leak it to
unrelated servers.

**The PAT currently embedded is READ-ONLY** on `wc-live-private` (confirmed via a
live 403 on a write attempt). If you ever need to push `update.json` or a new APK
to that repo, you need separate write credentials — the embedded token cannot do it,
by design, and that's a good thing (limits blast radius if it's ever extracted from
the APK).

**Release builds are now signed and minified.** Use `./gradlew assembleRelease`,
not `assembleDebug`, for anything you intend to distribute. See section 4 below
("Build Process") — it still describes the debug build for local dev iteration,
but the actual release artifact must come from `assembleRelease`. The signing
keystore (`app/wc-live-release.jks`) and `keystore.properties` are gitignored,
already exist, and must never be regenerated — doing so breaks update compatibility
for every existing install. If they ever go missing, STOP and ask the user before
generating new ones.

Full incident writeup and what's still pending manual upload:
`claude-output/SECURITY_AND_RELEASE_SETUP_v1.6.8.txt`

---

## Update JSON Format

As of versionCode 14 (v1.6.8), the app reads `update.json` from `AppConfig.updateUrl()`
(the PRIVATE repo `wc-live-private`), authenticated with `AppConfig.token()` — see
section 21 for the full architecture. The format itself is unchanged:

```json
{
  "appName": "WC Live",
  "packageName": "com.wclive.tv",
  "versionCode": 15,
  "versionName": "1.7.0",
  "minimumSupportedVersion": 1,
  "forceUpdate": false,
  "updateAvailable": true,
  "releaseDate": "2026-07-01",
  "apkSizeMB": 2.3,
  "apkUrl": "https://raw.githubusercontent.com/Asif-Ahsan-Sunny/wc-live-private/main/wc-live-apk/wc-live-v1.7.0.apk",
  "releaseNotes": ["What changed", "Another note"]
}
```

- `packageName` must be `com.wclive.tv` or the update will be silently ignored
- `apkUrl` must point at `wc-live-private` (raw.githubusercontent.com) — the app sends
  the same `Authorization`/`Accept` headers when downloading the APK as it does for
  `update.json` itself. A public-repo blob URL will fail since it's unauthenticated
  and no longer fetched by the app's update flow.
- Set `versionCode` higher than the currently installed value for the update to show

---

## Known Outstanding Items

1. **wc-live-private write access** — the embedded PAT is read-only; pushing new
   `update.json`/APK releases requires separate write credentials (see section 21).
2. **Gradient launcher icon** — requires API 24+; current build approximates with flat #171615 (minSdk 21 constraint).
3. **Adaptive icon** (API 26+) — not yet implemented; would need PNG density variants in mipmap-* folders.
4. **False-negative availability checks** — streams that reject HEAD/range requests will show OFFLINE even if playable.
5. **Force update + no internet** — re-shows update dialog after failed download, no retry delay.

---

## Build Commands

```bash
# Normal debug build
./gradlew assembleDebug

# Bump minor version first (optional)
./scripts/bump_minor_version.sh

# Copy to releases/
./scripts/copy_versioned_apk.sh

# Copy to claude-output/ (manual after build)
cp "app/build/outputs/apk/debug/WC Live X.X.X.apk" "claude-output/WC Live Claude X.X.X.apk"
```

Output: `app/build/outputs/apk/debug/WC Live {versionName}.apk`

---

## Session Notes

**Session 2026-06-23:**
- User confirmed package name: `com.wclive.tv` (was `com.wcfix.tvapp`)
- Removed "Reload defaults" from sidebar (clearCacheAction)
- Added in-app APK update system with optional/forced update dialogs
- Added channel availability checker with green status dots
- Version bumped to 1.6.5 (versionCode 8) as incremental update
- Build: SUCCESS, APK 7.0 MB
- Output: `claude-output/WC Live Claude 1.6.5.apk`

**Session 2026-06-23 (continued):**
- Created professional README.md on GitHub (badges, features, channel table, install guide, changelog)
- Created GitHub Release v1.6.6 with APK asset
- Built v1.6.7 (versionCode 10): sidebar divider opacity 30%→45% (#4DBDB8B7→#73BDB8B7) + 2dp marginTop
- update.json set to forceUpdate=true targeting versionCode 10
- Build: SUCCESS, APK 6.9 MB
- Output: `claude-output/WC Live Claude 1.6.7.apk`
- GitHub: pushed updates/wc-live-v1.6.7.apk, updates/update.json, README changelog
- GitHub Release v1.6.7 created and marked Latest

**Session 2026-06-23 (continued, versionCode 12):**
- Fixed About modal dispatchKeyEvent: changed `return super.dispatchKeyEvent(event)` → `return true`, routes center/enter directly to `closeAboutAction.performClick()`
- Fixed Update dialog dispatchKeyEvent: added NUMPAD_ENTER, routes click to focused button explicitly (updateCancelAction or updateInstallAction)
- Fixed Download progress dispatchKeyEvent: added NUMPAD_ENTER, routes click directly to downloadCancelAction
- Fixed tv_banner.xml: expanded viewportHeight 262→486 (16:9 ratio), updated background fills, wrapped content in `<group android:translateY="112">` to center
- Changed UPDATE_CHECK_DELAY_MS: 5000 → 30000ms
- versionCode bumped: 11 → 12
- Build: SUCCESS, APK 7.0 MB
- update.json: versionCode 12, forceUpdate false, updated releaseNotes
- GitHub: pushed updates/, README changelog updated, GitHub Release v1.6.7 updated (title + notes + APK replaced)

**Session 2026-06-25 (versionCode 13):**
- Added full D-pad navigation inside About/Update modal dialogs: added aboutBodyScroll
  and updateNotesScroll fields (focusable ScrollViews), D-pad Down scrolls then moves to
  the action button, D-pad Up reverses, added isScrolledToBottom/Top helpers
- Build: SUCCESS, APK 7.0 MB (debug)
- GitHub Release v1.6.7 updated (title + notes + APK replaced)

**Session 2026-06-25 (versionCode 14) — MAJOR ARCHITECTURE CHANGE, see section 21:**
- User pasted a live GitHub PAT in plaintext in chat — flagged immediately, advised
  rotation. Token confirmed READ-ONLY on the private repo via a live 403 write test.
- Created `AppConfig.java` — split-string token + private-repo URLs, runtime-joined via
  StringBuilder (defeats javac constant folding)
- Migrated channel sync + update check from the PUBLIC repo (unauthenticated) to the
  PRIVATE repo `Asif-Ahsan-Sunny/wc-live-private` (token-authenticated via
  `Authorization: token` + `Accept: application/vnd.github.v3.raw` headers on
  raw.githubusercontent.com — verified this combination actually works with a live curl
  test before writing any code)
- Discovered ALL prior APK builds (through versionCode 13) were unsigned DEBUG builds
  (debuggable=true, minifyEnabled=false) — ProGuard/obfuscation was never actually
  active on any previously distributed APK
- Generated a new release signing keystore (`app/wc-live-release.jks` + root
  `keystore.properties`, both gitignored) and flipped the release build type to
  `debuggable false`, `minifyEnabled true`
- Verified: assembleRelease succeeds, output is correctly signed (jarsigner -verify
  passes), the raw token string does not appear as a single dex literal, AppConfig
  class name was renamed by R8
- Release APK dropped from 7.0 MB (debug) to 2.23 MB (release, minified) at the same
  feature set
- update.json updated locally (versionCode 14, apkUrl now points at wc-live-private)
  but COULD NOT BE PUSHED — the provided PAT has no write access to wc-live-private.
  New update.json + new signed APK are staged at `updates/` waiting for someone with
  write access to upload them manually.
- Full details, what's left to do, and the keystore backup warning: see
  `claude-output/SECURITY_AND_RELEASE_SETUP_v1.6.8.txt`

**Session 2026-06-25 (continued, versionCode 15, v1.6.9 — audit pass):**
- Full functionality/UI-UX/performance/security audit produced first
  (`claude-output/AUDIT_REPORT_2026-06-25.md`); user approved everything except
  the Functionality section (skipped entirely) and the status-dot text alternative
  (kept dots visual-only, no text — explicit user preference: "keep it clean")
- Removed unused `androidx.leanback:leanback:1.0.0` dependency (zero usage found anywhere)
- `logoCache` changed from unbounded `ConcurrentHashMap` to a 150-entry LRU
  (`LinkedHashMap` with access-order + `removeEldestEntry`, wrapped in `synchronizedMap`)
- `playChannel()` no longer calls full `renderChannels()` — added `ChannelRowRefs` +
  `applyChannelRowState()` + `refreshCurrentChannelHighlight()` so only the
  previously-playing and newly-playing rows update in place. Search/category-switch
  paths still do a full rebuild (left untouched, lower frequency, lower risk)
- Added `installCrashLogger()` — global `Thread.UncaughtExceptionHandler` that logs
  then chains to the platform's previous handler (does not suppress normal crash UI)
- Added `Log.w`/`Log.e` to several previously-silent `catch` blocks: cached channel
  parse failure, bundled channel load failure, status cache load/save, update check,
  update JSON parsing. Left genuinely high-frequency/best-effort catches silent
  (logo loading, cosmetic PackageInfo lookups)
- Explicitly skipped (user decision): retry button on playback error, auto-reconnect,
  forced-update retry backoff, Media3 version bump (needs real-device regression
  testing not possible in this environment — deferred, not done)
- Build: SUCCESS both variants. Release APK: 1.8 MB (down from 2.23 MB in v1.6.8)
- Pushed to **both** repos to cover the update-notification gap discovered last
  session:
  - `wc-live-private` (source of truth for versionCode 14+): updates/update.json +
    wc-live-apk/wc-live-v1.6.9.apk — verified live with the app's actual embedded token
  - `wc-live` public repo (bridge for versionCode <=13, pre-migration installs):
    updates/update.json + updates/wc-live-v1.6.9.apk — verified live unauthenticated,
    exactly as old app code would fetch it. GitHub Release v1.6.9 created, marked Latest.
- Both paths confirmed end-to-end before considering the release done — this was the
  exact gap that caused the "no update prompt" issue in the previous session.

**Session 2026-06-29 (documentation/path migration — no new build):**
- Project location confirmed moved: `/Users/abc/Desktop/android-tv-apk` → `~/Documents/Projects/WC-Live`
- Fixed all old path references in `CLAUDE.md` (5 locations) and `BUILD_INSTRUCTIONS.md` (1 location)
- Updated `CLAUDE.md` version table: corrected stale entry (was v1.6.7/versionCode 11, now reflects actual current v1.6.9/versionCode 15)
- Updated `CLAUDE.md` update.json example URL to use private repo (`wc-live-private`) instead of old public blob URL
- Updated `keystore.properties.example` with correct relative storeFile path and explanatory comment
- Created `PROJECT_HANDOFF.md` — comprehensive AI/developer handoff document covering overview, folder map, build steps, signing, release flow, web project, two-repo architecture, security rules, architecture constraints, known issues
- Created `redistributable/` package: release-signed APK (1.88 MB), SHA256SUMS.txt, INSTALL_GUIDE.md, INSTALL_WARNING_GUIDE.md, RELEASE_NOTES_1.6.9.md, update.json copy, updates/wc-live-v1.6.9.apk
- Created `CLAUDE_PROJECT_UPDATE_REPORT.md` — full migration session report with checklist
- Historical build reports in `claude-output/` left unchanged (old paths there are expected timestamped history)
- Signing audit: release signing fully configured since v1.6.8, keystore at `app/wc-live-release.jks`, no action needed
- Permissions audit: INTERNET + REQUEST_INSTALL_PACKAGES + WRITE_EXTERNAL_STORAGE(≤28) — minimal, no removals needed
- Pushed updated docs to public GitHub repo: CLAUDE.md, CLAUDE_SESSION_LOG.md, PROJECT_HANDOFF.md, CLAUDE_PROJECT_UPDATE_REPORT.md
