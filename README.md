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

[**⬇ Download APK**](https://github.com/Asif-Ahsan-Sunny/wc-live/releases/latest) &nbsp;&nbsp;·&nbsp;&nbsp; [Installation](#-installation) &nbsp;&nbsp;·&nbsp;&nbsp; [Changelog](#-changelog)

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
100+ channels across 10 categories. Playlist is hosted remotely and refreshes automatically.

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
A couple MB. No background services, no analytics, no ads.

</td>
</tr>
</table>

---

## 🌐 Web Version

WC Live also has a browser-based player — same channel lineup, same Midnight Cinema design, no app install needed.
Built with React + HLS.js on the frontend and a Node.js/Express backend that proxies streams and keeps all real stream URLs off the browser entirely.

Available for self-hosting — see the `Web-Project/` folder in the development repository.

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

## 🔧 Tech Stack

| | |
|---|---|
| **Language** | Java (Android) |
| **Min SDK** | 21 — Android 5.0 Lollipop |
| **Target SDK** | 34 — Android 14 |
| **Player** | Media3 ExoPlayer |

---

## 📋 Changelog

### v1.7.0 — 2026-07-02
- Live viewer count — see how many people are watching alongside you
- Firebase push notifications — receive alerts from the broadcast team
- Notification bell — in-app notification inbox for announcements
- Background improvements and stability fixes

### v1.6.9 — 2026-06-29
- Smaller app package — removed an unused library, reducing APK size from ~2.2 MB to ~1.9 MB
- Bounded memory usage — channel logo cache is now capped so memory stays predictable on low-RAM devices
- Faster channel switching — selecting a channel no longer triggers a full list rebuild
- Build hardening — release build cleanup and Play Protect compatibility audit

### v1.6.8 — 2026-06-25
- Smaller, more efficient app package
- Improved app update reliability
- Minor stability improvements

### v1.6.7 — 2026-06-23
- Fixed an Android TV startup freeze affecting some devices
- Full remote D-pad navigation inside the About and Update screens
- Fixed the About screen's Close button not responding on some TV remotes
- Fixed remote navigation occasionally landing on the wrong item behind a dialog
- Fixed the Android TV launcher logo appearing stretched
- Smoother update check timing for slower TV devices

### v1.6.6 — 2026-06-23
- Visual polish to the sidebar

### v1.6.5 — 2026-06-23
- Added **in-app updates** — no manual sideloading needed after the first install
- Added **live channel status indicators** — see which channels are online at a glance

### v1.6 — 2026-06-21
- Fixed round launcher icon rendering on Android 8+
- Reduced sidebar dimensions for TV layouts
- Fixed now-playing title display bug
- Added wordmark to empty channel state

### v1.5 & v1.4
- Initial stable releases

---

<div align="center">

Made with ❤️ for live TV lovers &nbsp;·&nbsp; [Report an Issue](https://github.com/Asif-Ahsan-Sunny/wc-live/issues)

</div>
