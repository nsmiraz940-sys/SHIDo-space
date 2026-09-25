# Feature Capability Matrix

Honest classification of every feature requested, before any code was written.
`IMPLEMENTATION` says what this app actually does for that row.

| Feature | Android mechanism | Available to a normal 3rd-party app? | Special permission | OEM/root required? | Implementation in this app |
|---|---|---|---|---|---|
| Detect installed launchable apps | `PackageManager.queryIntentActivities` | Yes | `QUERY_ALL_PACKAGES` not needed (uses `<queries>` intent filter) | No | ✅ Implemented (Phase 1) |
| Game library (add/remove/favorite/search/sort) | Local app data + DataStore | Yes | None | No | ✅ Implemented (Phase 1) |
| Launch a game | `Intent` via `PackageManager.getLaunchIntentForPackage` | Yes | None | No | ✅ Implemented (Phase 1) |
| Per-game playtime / session stats | App-side tracking, foreground detection | Yes (approximate) | `PACKAGE_USAGE_STATS` (Usage Access, user-granted in Settings) for automatic launch/exit detection | No | ✅ Session data model implemented; ⏳ automatic launch/exit detection is Phase 2 |
| Gaming profiles (Balanced/Performance/Competitive/Battery) | App-side config object applied to app-controllable settings only | Yes | None | No | ✅ Implemented (Phase 1) |
| Floating gaming sidebar | `WindowManager` + `TYPE_APPLICATION_OVERLAY` | Yes | `SYSTEM_ALERT_WINDOW` ("Display over other apps", user-granted in Settings) | No | ✅ Implemented (Phase 1, basic) |
| Battery %, charging state, battery temperature | `BatteryManager`, `ACTION_BATTERY_CHANGED` | Yes | None | No | ✅ Implemented |
| Battery voltage/current | `BatteryManager` extras (device-dependent, often absent) | Yes, when device reports it | None | No | ✅ Implemented with "unavailable" fallback |
| RAM used/available/total | `ActivityManager.getMemoryInfo` | Yes (system-wide numbers, not per-app) | None | No | ✅ Implemented |
| CPU usage % | No public per-core/system CPU usage API since Android 8 (`/proc/stat` restricted) | **No** — not reliably available to a normal app on modern Android | N/A | Effectively yes (blocked without root) | ❌ Not implemented as a real metric — UI shows "Not available on this Android version" instead of a fake number |
| Thermal status | `PowerManager.getCurrentThermalStatus()` (API 29+) | Yes | None | No | ✅ Implemented (API 29+, hidden below that) |
| Raw device/CPU temperature (°C) | No general public API | **No** | N/A | Yes (OEM sysfs, not exposed) | ❌ Not implemented — thermal *status* (NONE/LIGHT/MODERATE/SEVERE/CRITICAL) shown instead, clearly labeled |
| Screen refresh rate (read) | `Display.getRefreshRate()` / `Display.getSupportedModes()` | Yes | None | No | ✅ Implemented |
| Preferred refresh rate (request) | `Window.attributes.preferredDisplayModeId` (own window only) | Yes, for this app's own window only — **cannot force another app's window to a refresh rate** | None | No | ✅ Implemented for in-app UI; clearly labeled as not controlling other apps |
| True in-game FPS | No public API exposes another app's render/frame timing | **No** | N/A | Yes (OEM/GPU driver) | ❌ Not implemented/faked — UI explicitly states this limitation, per your instruction |
| Network type / Wi-Fi vs mobile | `ConnectivityManager`, `NetworkCapabilities` | Yes | `ACCESS_NETWORK_STATE` | No | ✅ Implemented |
| Wi-Fi link speed / signal | `WifiManager.getConnectionInfo()` (link speed public; RSSI restricted since Android 12 without location permission) | Partial | `ACCESS_FINE_LOCATION` needed for signal strength on API 31+ | No | ✅ Link speed implemented; RSSI gated behind optional location permission, explained in-app |
| Ping / latency test | App-initiated ICMP-unavailable on most devices → use TCP socket connect timing or HTTP HEAD timing | Yes (approximation) | `INTERNET` | No | ⏳ Phase 2 (data model ready, test UI not yet wired) |
| Packet loss | No reliable unprivileged API | Partial/approximate only | `INTERNET` | No | ⏳ Phase 2, labeled as an estimate |
| Screenshot | No direct "capture any screen" API for normal apps | **No direct capture** — must hand off to system mechanism | User must use system Power+Volume shortcut, or app can capture **its own** window only | No | ✅ Implemented: quick shortcut that explains/launches the system screenshot gesture; app cannot silently screenshot another app, by Android design |
| Screen recording | `MediaProjection` | Yes | User must approve the system MediaProjection consent dialog **every session** (cannot be bypassed) | No | ✅ Implemented (Phase 1) |
| Brightness control | `Settings.System` (needs `WRITE_SETTINGS`) or per-window `LayoutParams.screenBrightness` | Per-window: yes, no permission. System-wide: yes, with permission | `WRITE_SETTINGS` for system-wide | No | ✅ Implemented via `WRITE_SETTINGS` (user-granted) |
| Screen timeout | `Settings.System.SCREEN_OFF_TIMEOUT` | Yes | `WRITE_SETTINGS` | No | ✅ Implemented |
| Rotation lock | `Settings.System.ACCELEROMETER_ROTATION` (system-wide) or per-Activity `requestedOrientation` | Per-activity: yes, no permission. System-wide: needs `WRITE_SETTINGS` | `WRITE_SETTINGS` for system-wide | No | ✅ Implemented (system-wide, gated on permission) |
| Do Not Disturb / notification suppression | `NotificationManager` + Notification Policy Access | Yes | `ACCESS_NOTIFICATION_POLICY` (user-granted) | No | ✅ Implemented (Phase 1, toggle + permission flow) |
| Reading/filtering other apps' notifications | `NotificationListenerService` | Yes, but sensitive | Notification Listener access (user-granted) | No | ⏳ Phase 2 — deliberately deferred since it's a sensitive permission and only needed for "favorite contacts only" call filtering |
| Call interruption handling (screen calls) | `CallScreeningService` / `TelecomManager` | Yes | Default Call Screening App role (user-granted) | No | ⏳ Phase 2 |
| Touch edge protection overlay | `WindowManager` overlay that consumes edge touch events | Yes, at the **overlay** level only | `SYSTEM_ALERT_WINDOW` | No | ⏳ Phase 2 |
| True touch sampling rate control | Hardware digitizer driver | **No** | N/A | Yes (OEM firmware) | ❌ Not implemented — explicitly shown as "Controlled by device manufacturer, not adjustable by any app" |
| Display color filters (blue light, grayscale, saturation) | Overlay `View` with color matrix / alpha blend | Yes | `SYSTEM_ALERT_WINDOW` | No | ⏳ Phase 2 (data model + presets ready) |
| Modifying another app's actual rendering/color pipeline | N/A | **No** | N/A | Yes | ❌ Not claimed anywhere in UI copy |
| Haptics/vibration patterns | `VibratorManager` / `VibrationEffect` | Yes | `VIBRATE` | No | ⏳ Phase 2 |
| Volume controls (media/ring/alarm/notification) | `AudioManager` | Yes | None (some require `ACCESS_NOTIFICATION_POLICY` for DND-adjacent ring changes) | No | ⏳ Phase 2 |
| CPU overclock / GPU overclock | Kernel/driver | **No** | N/A | Yes (root) | ❌ Not implemented, not offered |
| CPU/GPU governor change | Kernel `sysfs` | **No** | N/A | Yes (root) | ❌ Not implemented, not offered |
| Kernel parameter / touch firmware modification | Kernel/firmware | **No** | N/A | Yes (root) | ❌ Not implemented, not offered |
| Hardware frame interpolation | GPU driver/OEM firmware | **No** | N/A | Yes (OEM) | ❌ Not implemented, not offered |
| Force a specific thermal policy | Kernel thermal daemon | **No** | N/A | Yes (root) | ❌ Not implemented, not offered |
| Floating utility tools (stopwatch, notes, calculator) | `WindowManager` overlay windows | Yes | `SYSTEM_ALERT_WINDOW` | No | ⏳ Phase 2 |
| Forcing arbitrary 3rd-party apps into small-window mode | No public API for this | **No** | N/A | No (some OEM skins do it privately) | ❌ Not implemented — app launches those apps normally instead |
| Router-level QoS/ping reduction | Would require router integration | **No**, not from the phone alone | N/A | No (needs a real router API integration, out of scope) | ❌ Not claimed |

**Legend:** ✅ = real, working, non-fake implementation in this delivery · ⏳ = designed/scaffolded, wired up in a follow-up phase · ❌ = will never be faked; UI shows an honest "not supported" message instead.
