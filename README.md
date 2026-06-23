<div align="center">

<br/>

```
██╗    ██╗ ██████╗    ██╗     ██╗██╗   ██╗███████╗
██║    ██║██╔════╝    ██║     ██║██║   ██║██╔════╝
██║ █╗ ██║██║         ██║     ██║██║   ██║█████╗  
██║███╗██║██║         ██║     ██║╚██╗ ██╔╝██╔══╝  
╚███╔███╔╝╚██████╗    ███████╗██║ ╚████╔╝ ███████╗
 ╚══╝╚══╝  ╚═════╝    ╚══════╝╚═╝  ╚═══╝  ╚══════╝
```

**Premium live TV for Android TV, Tablet & Phone**

<br/>

[![Latest Release](https://img.shields.io/github/v/release/Asif-Ahsan-Sunny/wc-live?style=for-the-badge&color=E50914&label=LATEST&logo=android&logoColor=white)](https://github.com/Asif-Ahsan-Sunny/wc-live/releases/latest)
[![Platform](https://img.shields.io/badge/PLATFORM-Android%205.0%2B-3DDC84?style=for-the-badge&logo=android&logoColor=white)](https://github.com/Asif-Ahsan-Sunny/wc-live/releases/latest)
[![Downloads](https://img.shields.io/github/downloads/Asif-Ahsan-Sunny/wc-live/total?style=for-the-badge&color=22d480&label=DOWNLOADS)](https://github.com/Asif-Ahsan-Sunny/wc-live/releases)
[![License](https://img.shields.io/badge/LICENSE-Personal%20Use-555?style=for-the-badge)](https://github.com/Asif-Ahsan-Sunny/wc-live)

<br/>

[**⬇ Download APK**](https://github.com/Asif-Ahsan-Sunny/wc-live/releases/latest) &nbsp;&nbsp;·&nbsp;&nbsp; [Channel Lineup](#-channel-lineup) &nbsp;&nbsp;·&nbsp;&nbsp; [Installation](#-installation) &nbsp;&nbsp;·&nbsp;&nbsp; [Changelog](#-changelog)

<br/>

</div>

---

## Overview

**WC Live** is a native Android IPTV player engineered for the big screen. It delivers 100+ curated live channels across sports, news, entertainment, movies, kids, music, religious content, and documentary — all inside a clean, remote-navigable interface built for lean-back viewing.

Built with a **Midnight Cinema** design language: deep obsidian backgrounds, action red accents, and zero visual noise so the content takes center stage.

Works on **Android TV**, **Amazon Fire Stick**, **tablets**, and **phones**.

---

## ✨ Features

<table>
<tr>
<td width="50%">

**📺 Live Channel Catalogue**
100+ channels across 10 categories. Playlist is hosted remotely and refreshes automatically every hour.

**🟢 Channel Status Indicators**
Real-time green dots show which streams are online right now. Checked in the background every 45 minutes.

**🔄 In-App Updates**
The app detects new releases and downloads the APK for you — no sideloading needed after the first install.

**🔍 Instant Search**
Filter across all channels by name in real time.

</td>
<td width="50%">

**📂 Category Sidebar**
Browse by: Sports · News · Bangla · Movies · Kids · Music · Religious · Entertainment · Documentary

**🖥 TV-First UI**
Full immersive mode, D-pad navigation, large touch targets. No login, no account, no subscription.

**⚡ Media3 ExoPlayer**
HLS & MPEG-TS hardware-accelerated playback. Supports 4K streams where available.

**🪶 Lightweight**
~7 MB APK. No background services, no analytics, no ads.

</td>
</tr>
</table>

---

## ⬇ Installation

### Android TV / Amazon Fire Stick

1. Go to **Settings → My Fire TV → Developer Options** and enable **Apps from Unknown Sources**
2. Install the [**Downloader**](https://www.amazon.com/dp/B01N0BP507) app from the Amazon store
3. Open Downloader, go to the [**Releases page**](https://github.com/Asif-Ahsan-Sunny/wc-live/releases/latest) on a phone/PC, copy the `.apk` link, and paste it into Downloader
4. Download and install — done

> **Subsequent updates are automatic.** Once installed, WC Live checks for new versions at launch and can update itself without repeating this process.

### Android Phone / Tablet

1. Download the APK from the [**Releases page**](https://github.com/Asif-Ahsan-Sunny/wc-live/releases/latest)
2. Open the file and allow installation from unknown sources when prompted

---

## 📡 Channel Lineup

| Category | Channels |
|---|---|
| ⚽ **Fifa Live** | T Sports, Bein Sports 1 Max, D Sports, TSN 1–5 (USA & Canada), Win Sports, DAZN Direct, TVP Sports, Fussball TV |
| 🏏 **Sports** | Star Sports 1 HD, Sky Sports, Euro Sport HD, Cricbuzz HD, Cricket Gold, Flash Guys HD, Fighters |
| 📺 **Bangla** | BTV, NTV, Channel 24, Jamuna TV, Maasranga, Deepto TV, Ekattor HD, Boishakhi, Bangla Vision, News 24, ATN News + more |
| 📰 **News** | Al Jazeera, CNN (US), DW English, NDTV English & Hindi, France 24, RT News, TRT World, Ekattor HD, Independent TV |
| 🎬 **Movies** | HBO, Goldmines, Sheemaroo Bollywood, Action Hollywood, Cineedge HD, Hindi Movies, True Stories, BBC Earth Films |
| 🎵 **Music** | 9XM TV, YRF Music HD, Music Mastii, Party Universe, Deewana HD |
| 👶 **Kids** | Tom & Jerry, Doraemon, Mr Bean Animated, PBS Kids, Oggy & the Cockroaches, Motu Patlu, Gopal Bhar |
| 🕌 **Religious** | Al Quran, Peace TV Bangla & Urdu, Saudi Quran, Saudi Sunnah, Madani TV, EWTN, Live Quran TV |
| 🎭 **Entertainment** | HUM TV, SRK TV, Colors Infinity, Fashion One, Motor Vision, Luxel HD, Delicious |
| 🌍 **Documentary** | Discovery Bangla & Hindi, BBC Earth, Animal Planet, Wild Earth, Travel XP, Real Wild |

> Channel availability depends on stream uptime. The green status dot next to each channel reflects its current state.

---

## 🔧 Tech Stack

| | |
|---|---|
| **Language** | Java (Android) |
| **Min SDK** | 21 — Android 5.0 Lollipop |
| **Target SDK** | 34 — Android 14 |
| **Player** | [Media3 ExoPlayer](https://github.com/androidx/media) 1.3.1 |
| **Architecture** | Single-activity · Programmatic UI · No ViewBinding · No RecyclerView |
| **Networking** | Plain `HttpURLConnection` — no third-party HTTP clients |
| **Package** | `com.wclive.tv` |

---

## 📋 Changelog

### v1.6.7 — 2026-06-23
- Increased sidebar divider opacity to 45% for clear TV viewing distance visibility
- Added 2dp top margin to divider for cleaner visual breathing room
- **Force update** — all existing installs will be prompted to update

### v1.6.6 — 2026-06-23
- Increased sidebar divider opacity for better visual separation

### v1.6.5 — 2026-06-23
- Renamed app package to `com.wclive.tv`
- Removed *Reload Defaults* from settings sidebar
- Added **in-app APK update system** with optional and forced update dialogs
- Added **live channel availability checker** — green dots show online streams in real time

### v1.6 — 2026-06-21
- Fixed round launcher icon rendering on Android 8+
- Reduced sidebar dimensions for TV layouts
- Fixed now-playing title display bug
- Added wordmark to empty channel state

### v1.5 & v1.4
- Initial stable releases

---

## 📁 Repository Structure

```
wc-live/
├── wc live.txt          # M3U channel playlist (edit here to add/remove channels)
└── updates/
    ├── update.json      # Version manifest read by the app for update checks
    └── *.apk            # APK releases
```

---

<div align="center">

Made with ❤️ for live TV lovers &nbsp;·&nbsp; [Report an Issue](https://github.com/Asif-Ahsan-Sunny/wc-live/issues)

</div>
