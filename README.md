# GameSpace — Android Gaming Assistant (Phase 1)

An original, independent gaming-utility app inspired by the *category* of OEM game
assistants (game library, floating sidebar, performance readout, screen recording,
per-game profiles). No iQOO/vivo branding, logos, or private APIs are used anywhere.

**Read `FEATURE_CAPABILITY_MATRIX.md` first** — it lists every feature from the spec and
states plainly whether Android lets a normal app do it, and what this build actually does.

## What's real in this build (Phase 1)

- Installed-app scanning + manual Gaming Library (add/remove/favorite/search, profile per game)
- 4 built-in Gaming Profiles (Balanced/Performance/Competitive/Battery) — real, app-controllable settings, no fake toggles
- Live system monitor: battery %, charging state, battery temperature, RAM used/total, thermal status (API 29+), network type, Wi-Fi link speed — all from public Android APIs, nothing simulated
- Floating Gaming Sidebar via `SYSTEM_ALERT_WINDOW`, requested only when you tap "Start sidebar"
- Real screen recording via `MediaProjection` + `MediaRecorder`, saved to the app's Movies folder
- Permission system that explains each permission in plain language and requests it only when you open that feature
- Dark premium UI in the exact color tokens you specified

## What's intentionally NOT faked

CPU usage %, raw device temperature in °C, and true in-game FPS are not exposed to a
normal Android app on modern versions of the OS. Rather than inventing numbers, the UI
says so explicitly. Same for CPU/GPU overclocking, governor control, kernel/firmware
changes, and hardware touch-sampling control — these require root or OEM privileges this
app does not (and, per your instructions, will not) use.

## What's scaffolded but not yet wired (Phase 2+)

Automatic launch/exit detection (via Usage Access), touch-protection overlay, display
color filter overlay, haptic pattern editor, volume/audio controls, network latency
test, notification-listener-based call filtering, and floating utility windows
(calculator/notes/stopwatch). The data models and permission plumbing for these already
exist (`model/Models.kt`, `util/PermissionHelper.kt`) so they're additive, not a rewrite.

## Project structure

```
app/src/main/java/com/gamespace/assistant/
  MainActivity.kt              — Compose host + MediaProjection consent flow
  GameSpaceApp.kt               — Application class
  model/Models.kt               — Game, GamingProfile, GameSession, SystemSnapshot
  data/GamePreferences.kt       — DataStore persistence for the library
  data/GameRepository.kt        — installed-app scanning + library operations
  monitor/SystemMonitor.kt      — battery/RAM/thermal/network via public APIs only
  overlay/OverlayService.kt     — foreground service hosting the floating sidebar
  overlay/GamingSidebarOverlay.kt — sidebar Compose UI (bubble + expanded panel)
  recording/ScreenRecordService.kt — MediaProjection screen recording
  util/PermissionHelper.kt      — permission checks + user-facing explanations
  ui/GameSpaceViewModel.kt      — MVVM view model
  ui/navigation/NavGraph.kt     — bottom nav: Home/Games/Stats/Tools/Settings
  ui/screens/*.kt               — the five screens
  ui/theme/*.kt                 — Color/Type/Theme tokens
```

## Build instructions

1. Install **Android Studio** (Koala or newer recommended).
2. `File > Open` this project's root folder (the one containing `settings.gradle.kts`).
3. Let Gradle sync — it will download the Compose BOM, Navigation, DataStore, and
   Material3 dependencies listed in `app/build.gradle.kts`.
4. Connect a device/emulator running **API 30 (Android 11) or newer**.
5. Run the `app` configuration (▶ button) or `./gradlew installDebug` from a terminal.

### Generating a release APK/AAB

```
./gradlew assembleRelease   # APK at app/build/outputs/apk/release/
./gradlew bundleRelease     # AAB at app/build/outputs/bundle/release/
```

You'll need to configure signing (a keystore) in `app/build.gradle.kts` under
`signingConfigs` before a release build is installable outside debug — Android Studio's
`Build > Generate Signed Bundle/APK` wizard will do this for you interactively.

### Debugging

- `adb logcat -s GameSpace` once you add a consistent log tag, or just use Android
  Studio's Logcat panel filtered to `com.gamespace.assistant`.
- The overlay and recording services each show a persistent notification while active —
  if the sidebar or recording seems "stuck," check Settings > Apps > GameSpace >
  Notifications first; Android requires that visible notification by policy.

## Permissions this app requests, and when

| Permission | Requested when | Why |
|---|---|---|
| `SYSTEM_ALERT_WINDOW` | You tap "Start sidebar" | Draw the floating overlay |
| `WRITE_SETTINGS` | You open a system-wide display control in Settings | Brightness/rotation/timeout |
| `ACCESS_NOTIFICATION_POLICY` | You enable Gaming Focus | Do Not Disturb access |
| `PACKAGE_USAGE_STATS` | You enable auto game-launch detection (Phase 2) | See which app is foregrounded, no content read |
| `POST_NOTIFICATIONS` | First foreground-service start | Required by Android 13+ for any visible service notification |
| MediaProjection consent | You tap "Start recording" | One-time-per-session system dialog, cannot be skipped |

None of these are requested at first launch — only when you open the matching feature,
per your spec.

## Testing checklist carried into Phase 2

Game detection, launch, profile load/restore, overlay permission flow, sidebar
lifecycle, screen recording start/stop/cancel, DND toggle, monitor accuracy across
devices, session tracking, game removal/uninstall handling, permission denial paths,
rotation, and process death — this Phase 1 delivery is structured (ViewModel +
repository separated from UI) specifically so these can be added as JVM/instrumented
tests without restructuring the app.
