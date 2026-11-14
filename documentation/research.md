# turnItDown — Technical Research

Reference for building the game in [plan.md](plan.md): which framework to use, which API reads each input on Android and iOS, which package wraps it, and what to watch out for.

Research date: 2026-11-13, updated the same day for levels 22–29 and the new ghost-hunter level 6. Package names were current when checked. Before adding any package, check pub.dev for its maintenance status and latest version. Anything marked **(verify)** could not be confirmed from documentation and needs a test on a real device.

---

## 0. Scope: Android first

**Decision (2026-11-14): the first version is Android only.** The iOS research in this document and in the level files stays, for a later iOS version. Where something says "iOS", it isn't needed now.

### Framework: keep Flutter + Flame + Rive

- Building for Android only costs nothing extra in Flutter, and a later iOS version reuses the whole game: scenes, levels, setup and cards. Only the native parts would need an iOS version.
- Write only the **Kotlin** side of the in-house plugin now. Keep Flutter's generated `ios/` folder, but don't build or maintain it yet.
- If you ever decide iOS won't happen, plain Kotlin + Jetpack Compose is the alternative: no plugin layer at all, but nothing to reuse for iOS.

### Minimum Android version: Android 10 (API 29) recommended

| Feature | Available from |
|---|---|
| `LocationManager.isLocationEnabled()` (level 22, setup) | Android 9 (API 28) |
| System dark theme (level 20, setup) | Android 10 (API 29) |
| `ACTIVITY_RECOGNITION` permission for steps (level 25) | Android 10 (API 29) |
| Background apps can't read the clipboard (level 23's lesson) | Android 10 (API 29) |
| Microphone/camera privacy toggles (setup validation) and the clipboard-read toast (level 23) | Android 12 (API 31) |
| Clipboard cleared automatically (level 23's trivia) | Android 13 (API 33) |
| **Screenshot detection (level 6)** | **Android 14 (API 34).** On older versions the level is locked |
| Non-linear font scaling (level 24) | Android 14 (API 34). Handled by Flutter automatically |

- Android 10+ covers the large majority of phones in use. Check the current numbers in Android Studio's API level distribution chart before deciding.
- **Target API level:** the latest one Google Play requires. Check the Play Console, because the requirement rises every year.

### What gets simpler without iOS

| Where | What disappears |
|---|---|
| Level 8 · Sun glare | The camera workaround. The light sensor is read directly, so no camera permission is needed for it |
| Level 10 · Airplane mode | The guess. The setting is read directly |
| Level 11 · Time travel | "Set time automatically" can be read, so the time-server check becomes optional |
| Level 12 · Flashlight | The unverified iOS detection problem. The torch callback works |
| Level 27 · Screen lock | The guess. The screen-off event is reliable |
| Level 28 · Biometrics | The Face ID prompt. Android biometrics need no permission prompt |
| Setup | Step 1 asks for 4 permissions (microphone, camera, physical activity, location). Steps 6, 8, 11 and 14 are verified directly, with no manual confirmations |

### What stays hard on Android

- **Level 18 · Tongue out:** no ready-made tongue detector, and the easy path (ARKit) was iOS. The decision in [levels/18](levels/18-tongue-out-kid.md) now decides the whole level. Recommended: a different expression through MediaPipe.
- **Missing sensors on budget phones:** barometer (level 13), sometimes the gyroscope (levels 3, 16) or step counter (level 25). Each level file has a fallback.
- **Proximity only during calls** on some Samsung/Xiaomi phones (level 4). Fallback: face down alone.
- **Level 6 needs Android 14+.**
- **Brand differences:** brightness range and curve (level 1), number of volume steps (level 2), power savers (level 21), the 3-finger screenshot gesture (level 17), and the settings menu paths used in every "How do I do this?" text (Pixel, Samsung, Xiaomi).

### Kotlin side of the in-house plugin

All small pieces (roughly 20–50 lines each), used where no maintained package fits:

| Piece | Used by |
|---|---|
| Brightness: slider-curve conversion + the device's maximum | Level 1, setup |
| Volume key presses (optional) | Level 2 |
| Proximity sensor | Level 4 |
| Light sensor | Level 8, setup |
| Airplane mode and `AUTO_TIME` | Levels 10, 11, setup |
| Torch callback | Level 12, setup |
| Rotation-vector heading | Level 16 |
| MediaPipe Face Landmarker | Level 18 |
| Clipboard change listener | Level 23 |
| Screen-off receiver | Level 27 |
| SIM state | Setup step 7 |
| Screenshot callback (if the chosen package doesn't fit) | Level 6 |

### Android test phones

| Phone | Why |
|---|---|
| A recent Pixel | Stock Android: the reference behaviour (for example, Battery Saver turning on dark theme) |
| A recent Samsung Galaxy | One UI menus, power saving, proximity only during calls, brightness range |
| A Xiaomi/Redmi (or another brand with a 3-finger screenshot gesture) | HyperOS menus, its power saver, the gesture conflict in level 17 |
| A budget phone without a barometer (ideally also without a gyroscope) | The fallbacks in levels 3, 13 and 16 |
| A phone on Android 10–13 | Level 6's locked path and older behaviour |

### Test first (Android)

Items 3, 4, 5, 6, 8 and 11 in [§9](#9-things-to-test-on-real-devices-first), plus:
- Does Android 12+ show the clipboard toast when the game reads text it copied itself? (Level 23.)
- Can the device name be read without permission? (Level 13's optional twist.)

---

## 1. Verdict

**The game can be built.** Every input works on Android. On iOS, 22 of the 27 inputs work directly. The other five need a workaround or a test first:

| Input | iOS problem | Workaround |
|---|---|---|
| Ambient light sensor | No public API (SensorKit is limited to Apple-approved research apps) | Estimate scene brightness from the front camera's exposure metadata |
| Airplane mode | No public API | Heuristic: cellular radio disappears and network path changes |
| Flashlight (turned on from Control Center) | Developer reports conflict on whether `isTorchActive` shows it | **Test on a device first.** If it fails, redesign the level for iOS |
| Date/time "Set Automatically" | The setting can't be read | Compare the device clock to an NTP server |
| Screen lock (level 27) | No API says the phone was locked; the app only learns it went to the background | Heuristic: app left the foreground + protected data became unavailable (needs a passcode) |

Android has its own problems, but they come from the hardware, not the APIs: many budget phones have no barometer, some Samsung and Xiaomi phones only turn on the proximity sensor during calls, screenshot detection needs Android 14 or later, and no off-the-shelf model detects a tongue sticking out.
---

## 2. Framework choice

### Comparison

| | **Flutter + Flame** | React Native + Expo | Unity | Kotlin Multiplatform + Compose MP | Native (Kotlin + Swift) |
|---|---|---|---|---|---|
| Inputs covered by existing packages | Most of them | Many (expo-sensors, expo-battery, expo-brightness, expo-screen-capture, expo-camera, expo-audio) | Motion sensors only. In the Input System, light, pressure and proximity sensors are Android-only | Few packages, but you can call iOS frameworks directly from Kotlin | All |
| Touch radius and pressure (level 14) | **Built in**: `PointerEvent.radiusMajor`, `.pressure`, `.size` | Only `force` on iOS. No touch radius | `Touch.radius`, `Touch.pressure` | Pressure only | All |
| Writing your own native code | Platform channels with Pigeon (code generator). Simple | Expo Modules API (Swift/Kotlin). Simple | Native plugins. Clunkier | expect/actual. Simplest | Not needed |
| 2D cartoon rendering | Flame engine + Rive/Lottie | Skia + Reanimated + Rive | Excellent, but heavy for simple scenes | Compose Canvas. Few game tools | Separate on each platform |
| Codebases | 1 | 1 | 1 | 1 (UI) | 2 |

### Recommendation: Flutter + Flame + Rive, plus one in-house native plugin

- **Flutter** has well-maintained packages for most of the list (the "plus" family: sensors_plus, battery_plus, connectivity_plus). Its pointer events already carry touch radius and pressure on both platforms (confirmed in the engine source; see §4.13).
- **Flame** is the 2D game engine for Flutter. It handles the game loop, sprites, effects and audio. The levels are simple single-screen scenes, so plain Flutter widgets plus Flame components are enough.
- **Rive** is a tool for cartoon animations with state machines. A state machine fits these levels well: for example, `baby: crying → calming → asleep`, driven by a number input such as brightness. The official bridge to Flame is `flame_rive`. **Lottie** (`lottie` package) is a simpler alternative if you only need animations that play in a loop.
- **An in-house plugin** (`packages/device_signals`, written in Kotlin and Swift with Pigeon) covers what no reliable package does: watching the torch state, airplane mode, the iOS camera-based light estimate, ARKit face blendshapes, the Android brightness slider conversion, Android auto-time, and intercepting the volume keys on Android. Each is roughly 20–80 lines of native code.

**When to pick Expo instead:** if you are much more comfortable with TypeScript/React than with Dart. The coverage is similar. You would write the same gaps as Expo native modules, and also touch radius.

### Recommended packages (Flutter)

| Purpose | Package | Notes |
|---|---|---|
| Game engine | `flame`, `flame_audio` | |
| Animation | `rive` + `flame_rive`, or `lottie` | |
| Accelerometer, gyroscope, magnetometer, barometer | `sensors_plus` | Barometer support added in recent major versions. Not available on web. iOS has a sampling-rate limit for the barometer. Needs `NSMotionUsageDescription` on iOS |
| Battery level, charging state, power saving | `battery_plus` | `batteryState`, `onBatteryStateChanged`, `isInBatterySaveMode` |
| Screen brightness | `screen_brightness` | Reads system brightness and has a change stream for it (2.1.0+) |
| Volume | `flutter_volume_controller` (or `volume_controller`) | Listener with `emitOnStart`. Doesn't work in the iOS simulator |
| Audio route (Bluetooth headset) | `audio_session` | `getDevices()`, `devicesChangedEventStream`. `AudioDeviceType` includes `bluetoothA2dp`, `bluetoothSco`, `bluetoothLe` |
| Screenshot (and screen recording) events | `screen_capture_event_plus`, `detect_screenshot` or `flutter_screenshot_detector` | All use Android 14 `ScreenCaptureCallback` and the iOS screenshot notification |
| Microphone level | `record` (amplitude stream in dBFS) or `noise_meter` | |
| Compass | `flutter_compass`, or the in-house fused orientation (§4.14) | |
| Proximity | `proximity_sensor` **(verify maintenance)**, or in-house | |
| Ambient light (Android) | `light` / `light_sensor` **(verify maintenance)**, or in-house | Android only |
| Airplane mode | `airplane_mode_checker` | Its own docs say iOS detection isn't reliable |
| Network type | `connectivity_plus` | |
| Camera | `camera` | |
| Face detection (Android) | `google_mlkit_face_detection` | Gives smile and eyes-open probabilities. No tongue |
| Permissions | `permission_handler` | |
| Open system settings pages | `app_settings` | Android can open specific pages. iOS can only open your app's own page |
| Checking the clock is correct | `ntp` | |
| Device model (for the end-of-game report) | `device_info_plus` | |
| Location switch, permission, precise vs approximate | `geolocator` (or `location`, whose `requestService()` shows Android's "turn on location" dialog) | |
| Biometrics | `local_auth` | Android: `MainActivity` must extend `FlutterFragmentActivity` |
| Steps | `pedometer` | `stepCountStream`, `pedestrianStatusStream` |
| Keep the screen on during levels | `wakelock_plus` | |
| Clipboard, text size | Built into Flutter (`Clipboard`, `MediaQuery.textScalerOf`) | |

---

## 3. Feasibility matrix

✅ works ⚠️ works with caveats or a workaround ❌ no API · "Prompt" means the user sees a runtime permission dialog.

| # | Input | Android | iOS | Android permission | iOS permission | Used in level |
|---|---|---|---|---|---|---|
| 1 | Display brightness | ✅ (slider is non-linear) | ✅ | none to read | none | 1, start check |
| 2 | Volume | ✅ | ✅ (when the app's audio session is active) | none | none | 2, 5, start check |
| 3 | Gyroscope / orientation | ✅ | ✅ | none | none (CoreMotion device motion) | 3, 4, 7, 15 |
| 4 | Proximity | ⚠️ some OEMs only enable it during calls | ✅ iPhone only (not iPad) | none | none | 4 |
| 5 | Microphone | ✅ | ✅ | `RECORD_AUDIO` (prompt) | `NSMicrophoneUsageDescription` (prompt) | 5 |
| 6 | Screenshot event | ⚠️ Android 14+ only (without restricted permissions) | ✅ | `DETECT_SCREEN_CAPTURE` (install-time, no prompt) | none | 6 |
| 7 | Ambient light | ✅ | ❌ → camera estimate | none | camera (prompt) for the workaround | 8, start check |
| 8 | Battery and charging | ✅ | ✅ | none | none | 9 |
| 9 | Airplane mode | ✅ | ⚠️ heuristic | none | none | 10, start check |
| 10 | System date/time | ✅ (auto-time setting readable) | ⚠️ auto-time setting not readable | none | none | 11, start check |
| 11 | Flashlight state | ✅ `TorchCallback` | ⚠️ **(verify)** | none documented **(verify)** | none | 12 |
| 12 | Barometer | ⚠️ missing on many budget phones | ✅ | none | `NSMotionUsageDescription` (prompt) | 13 |
| 13 | Touch size / pressure | ⚠️ pressure is often faked | ⚠️ radius ✅; pressure = 0 on every iPhone since the XR/11 | none | none | 14 |
| 14 | Compass | ✅ | ✅ | none | none (via CoreMotion) | 16 |
| 15 | Multi-touch | ✅ (usually 10 points) | ✅ (iPhone: max 5, widely reported) | none | none | 17 |
| 16 | Face expression (tongue) | ⚠️ no ready-made tongue detector | ✅ ARKit `tongueOut` (A12 chip or newer) | `CAMERA` (prompt) | `NSCameraUsageDescription` (prompt) | 18 |
| 17 | Bluetooth audio connection | ✅ via audio route | ✅ via audio route | none | none | 19, start check |
| 18 | Light/dark mode | ✅ | ✅ | none | none | 20, start check |
| 19 | Power saving mode | ✅ | ✅ | none | none | 21, start check |
| 20 | Location (switch + position) | ✅ | ✅ | location (prompt) for a position; none to read the switch | location (prompt) for a position; none to read the switch | 22, start check |
| 21 | Clipboard | ✅ (Android 12+ shows a toast on reads) | ✅ (iOS 16+ asks "Allow Paste" for content from other apps) | none | none (system prompt for other apps' content) | 23 |
| 22 | Text size | ✅ | ✅ | none | none | 24 |
| 23 | Step counter | ⚠️ sensor missing on some cheap phones | ✅ | `ACTIVITY_RECOGNITION` (prompt) | Motion & Fitness (prompt) | 25 |
| 24 | Shake (accelerometer) | ✅ | ✅ | none | none | 26 (accelerometer also in 7, 15) |
| 25 | Screen lock | ✅ screen-off broadcast | ⚠️ no API; heuristic | none | none | 27 |
| 26 | Biometrics | ✅ | ✅ | `USE_BIOMETRIC` (install-time, no prompt) | Face ID: `NSFaceIDUsageDescription` (prompt); Touch ID: none | 28 |
| 27 | Wi-Fi vs mobile data | ✅ | ✅ | `ACCESS_NETWORK_STATE` (install-time, no prompt) | none | 29 |

**A point the game can make:** Android shows a permission prompt for only 4 of these inputs (microphone, camera, location, physical activity). iOS shows one for 5 (microphone, camera, Motion & Fitness, location, Face ID), and also asks before pasting content copied in another app. **The other ~20 are read silently.** This is the strongest message you can put in front of players.

### Access difficulty

This rates how hard it is to *get* each input, using two measures:
- **What the player has to accept:** permission prompts.
- **How much code it takes to obtain a usable value.**

Level logic (thresholds, animations) isn't counted. **The overall rating is the worse of the two measures.**

| Rating | What the player has to accept | Code to obtain the value |
|---|---|---|
| 🟢 Easy | Nothing. Install-time permissions count as nothing, because the player never sees them | Flutter itself, or one maintained package: a few lines |
| 🟡 Moderate | One permission prompt | A package plus a conversion or filter, or a small piece of native code (roughly under 50 lines per platform) |
| 🟠 Hard | Several prompts for one input | Substantial native code, or a heuristic because there is no direct API |
| 🔴 Very hard | A permission that store policy restricts | No public API and no reliable workaround, or a custom ML model |

#### Android

| Input | Player has to accept | Code to get the value | Overall |
|---|---|---|---|
| Display brightness | 🟢 Nothing | 🟡 `screen_brightness` + slider-curve conversion | **🟡 Moderate** |
| Volume | 🟢 Nothing | 🟢 `flutter_volume_controller` (🟡 to catch button presses) | **🟢 Easy (🟡 with button presses)** |
| Gyroscope | 🟢 Nothing | 🟢 `sensors_plus` | **🟢 Easy** |
| Accelerometer (tilt, shake, face down) | 🟢 Nothing | 🟢 `sensors_plus` | **🟢 Easy** |
| Proximity | 🟢 Nothing | 🟡 Small native code. Unreliable on some phones | **🟡 Moderate** |
| Microphone | 🟡 Microphone prompt | 🟢 `record` / `noise_meter` | **🟡 Moderate** |
| Screenshot event | 🟢 Nothing (Android 14+) · 🔴 photo access restricted by Google Play (older) | 🟢 `screen_capture_event_plus` or similar | **🟢 Easy (14+) · 🔴 Very hard (older)** |
| Ambient light | 🟢 Nothing | 🟢 Package or tiny native code | **🟢 Easy** |
| Battery and charging | 🟢 Nothing | 🟢 `battery_plus` | **🟢 Easy** |
| Airplane mode | 🟢 Nothing | 🟢 `airplane_mode_checker` | **🟢 Easy** |
| Clock and time zone | 🟢 Nothing | 🟢 Dart `DateTime` / `Stopwatch` | **🟢 Easy** |
| "Set time automatically" | 🟢 Nothing | 🟡 Small native read of `AUTO_TIME` | **🟡 Moderate** |
| Flashlight state | 🟢 Nothing (verify) | 🟡 Small native code (`TorchCallback`) | **🟡 Moderate** |
| Barometer | 🟢 Nothing | 🟢 `sensors_plus` (hardware often missing) | **🟢 Easy** |
| Touch size and pressure | 🟢 Nothing | 🟢 Built into Flutter | **🟢 Easy** |
| Compass (smooth heading) | 🟢 Nothing | 🟡 Small native code (raw `flutter_compass` heading: 🟢) | **🟡 Moderate** |
| Multi-touch | 🟢 Nothing | 🟢 Built into Flutter | **🟢 Easy** |
| Face expression | 🟡 Camera prompt | 🔴 Tongue: custom ML model · 🟠 other expressions: MediaPipe in native code | **🔴 Very hard (tongue) · 🟠 Hard (other)** |
| Bluetooth audio connection | 🟢 Nothing | 🟢 `audio_session` | **🟢 Easy** |
| Light/dark mode | 🟢 Nothing | 🟢 Built into Flutter | **🟢 Easy** |
| Power saving mode | 🟢 Nothing | 🟢 `battery_plus` | **🟢 Easy** |
| Location switch (on/off) | 🟢 Nothing | 🟢 `geolocator` | **🟢 Easy** |
| Location position | 🟡 Location prompt (during setup) | 🟢 `geolocator` | **🟡 Moderate** |
| Clipboard | 🟢 Nothing | 🟢 Built into Flutter | **🟢 Easy** |
| Text size | 🟢 Nothing | 🟢 Built into Flutter | **🟢 Easy** |
| Step counter | 🟡 Physical activity prompt | 🟢 `pedometer` | **🟡 Moderate** |
| Screen lock | 🟢 Nothing | 🟡 Small native receiver | **🟡 Moderate** |
| Biometrics | 🟢 Nothing (install-time) | 🟢 `local_auth` | **🟢 Easy** |
| Wi-Fi vs mobile data | 🟢 Nothing (install-time) | 🟢 `connectivity_plus` | **🟢 Easy** |

#### iOS

| Input | Player has to accept | Code to get the value | Overall |
|---|---|---|---|
| Display brightness | 🟢 Nothing | 🟢 `screen_brightness` | **🟢 Easy** |
| Volume | 🟢 Nothing | 🟢 `flutter_volume_controller` | **🟢 Easy** |
| Gyroscope | 🟢 Nothing | 🟢 `sensors_plus` | **🟢 Easy** |
| Accelerometer (tilt, shake, face down) | 🟢 Nothing | 🟢 `sensors_plus` | **🟢 Easy** |
| Proximity | 🟢 Nothing | 🟡 Small native code. The screen turns off while covered | **🟡 Moderate** |
| Microphone | 🟡 Microphone prompt | 🟢 `record` / `noise_meter` | **🟡 Moderate** |
| Screenshot event | 🟢 Nothing | 🟢 `screen_capture_event_plus` or similar | **🟢 Easy** |
| Ambient light | 🟡 Camera prompt (workaround) | 🟠 Native camera code reading exposure data | **🟠 Hard** |
| Battery and charging | 🟢 Nothing | 🟢 `battery_plus` | **🟢 Easy** |
| Airplane mode | 🟢 Nothing | 🟠 Heuristic from network signals | **🟠 Hard** |
| Clock and time zone | 🟢 Nothing | 🟢 Dart `DateTime` / `Stopwatch` | **🟢 Easy** |
| "Set time automatically" | 🟢 Nothing | 🟡 Query an NTP server (`ntp`) and compare | **🟡 Moderate** |
| Flashlight state | 🟢 Nothing | 🔴 May be impossible to observe (unverified) | **🔴 Very hard (unverified)** |
| Barometer | 🟡 Motion & Fitness prompt | 🟡 `sensors_plus` + small native `CMAltimeter` wrapper | **🟡 Moderate** |
| Touch size and pressure | 🟢 Nothing | 🟢 Built into Flutter (size only, no pressure) | **🟢 Easy** |
| Compass (smooth heading) | 🟢 Nothing | 🟡 Small native code (raw `flutter_compass` heading: 🟢) | **🟡 Moderate** |
| Multi-touch | 🟢 Nothing | 🟢 Built into Flutter (max 5 touches) | **🟢 Easy** |
| Face expression | 🟡 Camera prompt | 🟠 Native ARKit code | **🟠 Hard** |
| Bluetooth audio connection | 🟢 Nothing | 🟢 `audio_session` | **🟢 Easy** |
| Light/dark mode | 🟢 Nothing | 🟢 Built into Flutter | **🟢 Easy** |
| Power saving mode | 🟢 Nothing | 🟢 `battery_plus` | **🟢 Easy** |
| Location switch (on/off) | 🟢 Nothing | 🟢 `geolocator` | **🟢 Easy** |
| Location position | 🟡 Location prompt (during setup) | 🟢 `geolocator` | **🟡 Moderate** |
| Clipboard | 🟢 Nothing for in-game text (🟡 "Allow Paste?" for other apps' content) | 🟡 Small native `changeCount` check | **🟡 Moderate** |
| Text size | 🟢 Nothing | 🟢 Built into Flutter | **🟢 Easy** |
| Step counter | 🟡 Motion & Fitness prompt | 🟢 `pedometer` | **🟡 Moderate** |
| Screen lock | 🟢 Nothing | 🟠 Heuristic in native code | **🟠 Hard** |
| Biometrics | 🟡 Face ID prompt, during setup (🟢 none with Touch ID) | 🟢 `local_auth` | **🟡 Moderate (Face ID) · 🟢 Easy (Touch ID)** |
| Wi-Fi vs mobile data | 🟢 Nothing | 🟢 `connectivity_plus` | **🟢 Easy** |

---

## 4. Per-input reference

### 4.1 Display brightness
- **Android:** read `Settings.System.getInt(contentResolver, Settings.System.SCREEN_BRIGHTNESS)`. Auto-brightness is `Settings.System.SCREEN_BRIGHTNESS_MODE`. To watch for changes, register a `ContentObserver` on `Settings.System.getUriFor(SCREEN_BRIGHTNESS)`. Reading needs no permission. Writing needs the `WRITE_SETTINGS` special access, which the user grants on a separate settings screen (`ACTION_MANAGE_WRITE_SETTINGS`).
  - **The slider is not linear.** Since Android 9 the slider uses a gamma curve (Hybrid Log Gamma, in AOSP `BrightnessUtils`), but `SCREEN_BRIGHTNESS` stores the linear value. On stock AOSP, a slider at 50% is roughly 8% of the linear range (worked out from the AOSP formula). Phone makers change this. To show the user a "slider %", convert with the same formula as AOSP's `convertLinearToGamma`.
  - The maximum is not always 255. It can be read from the hidden system resource `config_screenBrightnessSettingMaximum` with `Resources.getSystem().getIdentifier(...)`.
- **iOS:** `UIScreen.brightness` (0.0–1.0, via `windowScene.screen` since iOS 16) and `UIScreen.brightnessDidChangeNotification`. No permission. The app can also *set* brightness.
- **Flutter:** `screen_brightness`: system brightness plus a stream of system brightness changes.
- **Gotchas:** with auto-brightness on, the system changes brightness on its own (level 8 will dim the screen). Compare against ranges, never exact values.

### 4.2 Volume
- **Android:** `AudioManager.getStreamVolume(STREAM_MUSIC)` and `getStreamMaxVolume`. The number of steps varies (15 on Pixel, more on some OEMs). Watch with a `ContentObserver` on `Settings.System.CONTENT_URI`, or the unofficial `android.media.VOLUME_CHANGED_ACTION` broadcast. **While the app is in the foreground you can catch the button press itself** with `KEYCODE_VOLUME_DOWN` in `Activity.dispatchKeyEvent`, and even consume it. `setStreamVolume` needs no permission.
- **iOS:** `AVAudioSession.sharedInstance().outputVolume` (0–1, in 16 steps of 0.0625), observed with KVO. The volume buttons only change *media* volume while the app has an active audio session or is playing audio. Otherwise they change the ringer. So keep an active `.playback` session with background music. There is no public API to intercept the buttons, read the ring/silent switch, or set the volume (the hidden-`MPVolumeView` hack is discouraged).
  - KVO doesn't fire once the volume is already at 0 or 100%.
  - Apple's developer forums have a report that `outputVolume` misbehaves on iOS 18+ **(verify on current iOS)**.
- **Flutter:** `flutter_volume_controller` (`getVolume`, `addListener(emitOnStart: …)`, `setVolume`, choice of Android stream). Catching the Android key press needs the in-house plugin.

### 4.3 Gyroscope, accelerometer, orientation
- **Android:** `TYPE_GYROSCOPE` (rad/s), `TYPE_ACCELEROMETER`, `TYPE_GRAVITY`, `TYPE_ROTATION_VECTOR` (fused with the magnetometer, so its yaw is relative to north), `TYPE_GAME_ROTATION_VECTOR` (fused, no magnetometer, smoother). Convert with `SensorManager.getRotationMatrixFromVector` and then `getOrientation`.
- **iOS:** `CMMotionManager.startDeviceMotionUpdates(using: .xArbitraryZVertical | .xMagneticNorthZVertical)` gives `CMDeviceMotion` with `.attitude` (quaternion, roll/pitch/yaw), `.gravity`, `.rotationRate` and `.userAcceleration`. No permission.
- **Flutter:** `sensors_plus` gives raw streams (accelerometer, user accelerometer, gyroscope, magnetometer) but **no fused attitude**. Either compute tilt from gravity and integrate the gyroscope yourself, or have the in-house plugin forward the platform's fused orientation (recommended, about 30 lines per platform).
- **Use the right sensor:** for an *absolute tilt* (levels 7 and 15), use the gravity vector. For *rotation amount* (level 3), integrate the gyroscope's z axis. For *heading* (level 16), use the fused rotation vector.
- **Axis conventions:** flat and face up, Android reports accelerometer z ≈ +9.81, while iOS reports `gravity.z` ≈ −1. Check what `sensors_plus` reports on both platforms with the debug overlay (§7). Gyroscope sign follows the right-hand rule: clockwise as seen from above the screen is negative z **(verify)**.

### 4.4 Proximity
- **Android:** `Sensor.TYPE_PROXIMITY` (cm). It is often binary: 0 or the maximum range. On some Samsung (and reportedly Xiaomi) phones it only reports during calls (anecdotal reports from users, not documented by the makers).
- **iOS:** set `UIDevice.current.isProximityMonitoringEnabled = true`, then read `proximityState` and listen for `UIDevice.proximityStateDidChangeNotification`. **The screen turns off while the sensor is covered.** iPads don't have one. Turn monitoring off as soon as the level ends.
- **Fallback:** face-down detection from gravity alone.

### 4.5 Microphone
- **Android:** `AudioRecord` with an RMS → dBFS calculation, or `MediaRecorder.getMaxAmplitude()`. Permission: `RECORD_AUDIO`. To cancel the game's own audio picked up by the mic, use the `VOICE_COMMUNICATION` audio source and `AcousticEchoCanceler`.
- **iOS:** an `AVAudioEngine` tap on the input node with an RMS calculation, or `AVAudioRecorder` with metering (`averagePower`). Session category `.playAndRecord`. For echo cancellation, `inputNode.setVoiceProcessingEnabled(true)`. Permission: `NSMicrophoneUsageDescription`.
- **Flutter:** `record` (amplitude stream) or `noise_meter`. Speech recognition, if you want to check for the word "Stop", is `speech_to_text`. It adds a prompt on iOS (`NSSpeechRecognitionUsageDescription`), so a loudness threshold is simpler.
- **Awareness note:** both platforms show a privacy dot while the mic is in use (orange on iOS, green on Android 12+). Point it out to the player.

### 4.6 Screenshot event
- **iOS:** `UIApplication.userDidTakeScreenshotNotification`, fired after the capture. No permission. Screen recording: `UIScreen.isCaptured` with `capturedDidChangeNotification`, or `sceneCaptureState` on iOS 17+.
- **Android 14+ (API 34):** `Activity.registerScreenCaptureCallback(executor, callback)` in `onStart`, unregistered in `onStop`. Install-time permission `DETECT_SCREEN_CAPTURE` (no prompt). It fires only while the activity is visible. Google's documentation says it only detects screenshots taken **with the hardware button combination**. **The system shows a toast, "<App> detected this screenshot"**, which is a free awareness moment.
- **Android < 14:** the only way is a `ContentObserver` on MediaStore, which needs `READ_MEDIA_IMAGES`. **Google Play restricts that permission** to apps whose core purpose needs broad photo access. **Recommendation:** on Android < 14, mark this level as unavailable.
- **Android 15+:** screen recording can be detected with `WindowManager.addScreenRecordingCallback` and the `DETECT_SCREEN_RECORDING` permission.
- **Flutter:** `screen_capture_event_plus`, `detect_screenshot`, `flutter_screenshot_detector`. Check that the one you pick doesn't add `READ_MEDIA_IMAGES` to your manifest. If it does, remove it with `tools:node="remove"`.

### 4.7 Ambient light
- **Android:** `Sensor.TYPE_LIGHT` (lux). Typical values: dark room < 10, living room 50–200, office 300–500, daylight 10,000+. The sensor sits near the top of the screen, sometimes under the display.
- **iOS:** no public API. SensorKit has ambient light, but only for research studies Apple approves, and its data arrives with a 24-hour delay. **Workaround:** run the front camera in an `AVCaptureSession` and, for each frame, read the EXIF BrightnessValue from the sample buffer (`kCGImagePropertyExifDictionary` → `kCGImagePropertyExifBrightnessValue`, in APEX units: −1 means half the light). Alternatively, estimate from `exposureDuration`, `iso` and aperture. This needs camera permission. In level 8, the hand covering the top of the phone also covers the front camera, so the level still works. The Flutter `camera` package doesn't expose per-frame EXIF data, so this goes in the in-house plugin.

### 4.8 Battery and charging
- **Android:** the sticky `ACTION_BATTERY_CHANGED` intent has `EXTRA_LEVEL`, `EXTRA_SCALE`, `EXTRA_STATUS` and `EXTRA_PLUGGED` (AC, USB or wireless). Also `BatteryManager.isCharging()` and the `ACTION_POWER_CONNECTED` / `ACTION_POWER_DISCONNECTED` broadcasts.
- **iOS:** set `UIDevice.current.isBatteryMonitoringEnabled = true`, then read `batteryState` (`.unplugged`, `.charging`, `.full`) and `batteryLevel`, and listen for `batteryStateDidChangeNotification`. Treat `.full` as plugged in.
- **Flutter:** `battery_plus`.

### 4.9 Airplane mode
- **Android:** `Settings.Global.getInt(cr, Settings.Global.AIRPLANE_MODE_ON)` and the `Intent.ACTION_AIRPLANE_MODE_CHANGED` broadcast. No permission.
- **iOS:** no public API. Heuristic: `CTTelephonyNetworkInfo().serviceCurrentRadioAccessTechnology` is empty or nil for every SIM, **and** `NWPathMonitor` shows no `.cellular` interface. Limits:
  - It can't tell airplane mode apart from no signal or no SIM.
  - Wi-Fi can stay on in airplane mode (iOS remembers if the user turned it back on).
  - Wi-Fi-only iPads have no cellular radio to check.
  
  Check at level start that the phone *has* cellular service. If it doesn't, change the instructions or skip the level.
- **Flutter:** `airplane_mode_checker` (Android is reliable; its own docs say iOS isn't) plus `connectivity_plus`.

### 4.10 System date/time
- **Wall clock:** `DateTime.now()`. **Monotonic clock** (not affected by the user changing the time): Dart's `Stopwatch`, Android `SystemClock.elapsedRealtime()`, iOS `ProcessInfo.systemUptime`.
- **Change events:** Android `Intent.ACTION_TIME_CHANGED` and `ACTION_TIMEZONE_CHANGED`. iOS `UIApplication.significantTimeChangeNotification` and `.NSSystemClockDidChange`. The user goes to the Settings app to change the time, so also re-check every time the app resumes.
- **"Is the time correct?":** Android can read `Settings.Global.AUTO_TIME` and `AUTO_TIME_ZONE`. iOS can't read the setting, so query NTP (`ntp` package) or the `Date` header of an HTTPS response, and allow about ±30 s.
- **Clock offset (level 11):** `offset = DateTime.now() − (wallAtLevelStart + stopwatch.elapsed)`. This needs no network and gives the exact number of minutes the user moved the clock.

### 4.11 Flashlight
- **Android:** `CameraManager.registerTorchCallback(callback, handler)` calls `onTorchModeChanged(cameraId, enabled)`, including when the torch is switched from Quick Settings. It is called once immediately after registering. The documentation doesn't mention the CAMERA permission for this **(verify)**.
- **iOS:** `AVCaptureDevice.default(for: .video)?.isTorchActive`, which is KVO-observable, along with `torchMode` and `torchLevel`. **Developer reports conflict** on whether these reflect a torch switched on from Control Center when the app has no capture session. One report says KVO only fires after the app takes the device into an `AVCaptureSession`. **Test this first.** Fallback if it doesn't work: on iOS, the app turns the torch on itself through an in-game switch. That still teaches something ("apps can switch on your flashlight"), but it's a weaker level.
- **Flutter:** `torch_light` can only *control* the torch, not observe it. Use the in-house plugin.

### 4.12 Barometer
- **Android:** `Sensor.TYPE_PRESSURE` (hPa). `SensorManager.getAltitude(PRESSURE_STANDARD_ATMOSPHERE, p)`. `getDefaultSensor` returns `null` on phones without one, which includes many budget Samsung A-series, Xiaomi and Motorola models.
- **iOS:** `CMAltimeter.isRelativeAltitudeAvailable()`, then `startRelativeAltitudeUpdates` for `relativeAltitude` (m) and `pressure` (kPa). Permission: `NSMotionUsageDescription` (Motion & Fitness).
- **Physics:** near sea level the pressure drops about 0.12 hPa per metre. Standing up and raising the phone is about 0.8–1.2 m, so roughly 0.1–0.15 hPa. That is close to the sensor's noise, so smooth over about 1 s and expect 1–2 s of delay. Air conditioning and doors closing also move the pressure.
- **Fallback** (no barometer): integrate the vertical component of user acceleration (an upward peak followed by a deceleration), combined with the phone ending up raised (pitch close to upright).
- **Flutter:** `sensors_plus` `barometerEventStream()` (hPa). It has an iOS sampling-rate limit. For relative altitude, use the in-house `CMAltimeter`.

### 4.13 Touch size and pressure
- **Flutter `PointerEvent`** has `pressure`, `pressureMin/Max`, `size`, `radiusMajor`, `radiusMinor` and `radiusMin/Max`. From the engine source:
  - **iOS:** `pressure = UITouch.force`, `pressure_max = maximumPossibleForce`, `radius_major = UITouch.majorRadius`, `radius_min/max = majorRadius ∓ majorRadiusTolerance`. `force` is 0 on devices without 3D Touch (every iPhone since the XR/11). **Only the contact radius works on modern iPhones.** `majorRadius` comes in coarse steps.
  - **Android:** `pressure = MotionEvent.getPressure()`, `size = getSize()`, `radius_major = getToolMajor()`, `radius_minor = getToolMinor()`. Many devices report a constant or made-up pressure, and touch sizes vary a lot between devices.
- **Design consequence:** measure the player's own index-finger tap first, compare the thumb against it, and also require holding for a few seconds.

### 4.14 Compass
- **Android:** the azimuth from `TYPE_ROTATION_VECTOR`, or `TYPE_MAGNETIC_FIELD` + `TYPE_ACCELEROMETER` with `getRotationMatrix`. Watch `onAccuracyChanged`: when accuracy is low, ask the player to move the phone in a figure-8.
- **iOS:** `CMMotionManager` with `.xMagneticNorthZVertical` gives yaw relative to magnetic north and needs no permission. The alternative is `CLLocationManager.startUpdatingHeading()` (`magneticHeading`). Its `trueHeading` needs location.
- **Flutter:** `flutter_compass` (heading stream), or `sensors_plus` magnetometer (raw µT). A raw magnetometer heading jitters, so prefer the fused orientation from §4.3.

### 4.15 Multi-touch
- **Android:** `MotionEvent.getPointerCount()`. Usually 10 points, fewer on some cheap screens.
- **iOS:** an iPhone tracks at most **5** simultaneous touches (widely reported, not documented by Apple). iPads track more.
- **Flutter:** multi-touch is on by default. Track pointer IDs in a `Listener`, or use `ScaleUpdateDetails.pointerCount`.
- **Gotchas:** Xiaomi, OnePlus, Oppo, Realme and Vivo phones take a screenshot on a 3-finger swipe down, which can fire during the pinch. iOS accessibility Zoom uses a 3-finger double tap.

### 4.16 Face expression (tongue out)
- **iOS:** `ARFaceTrackingConfiguration` (`isSupported`). Since iOS 14 it works on any device with a front camera and an A12 chip or newer. Before iOS 14 it needed a TrueDepth camera. `ARFaceAnchor.blendShapes[.tongueOut]` is a value from 0 to 1. Permission: camera. Flutter: `arkit_plugin` (community) or the in-house plugin.
- **Android:**
  - **ML Kit Face Detection** (`google_mlkit_face_detection`): `smilingProbability`, `left/rightEyeOpenProbability`, face contours. No tongue.
  - **MediaPipe Face Landmarker:** 52 blendshape categories named like ARKit's. **`tongueOut` is NOT one of them.** Index 0 is `_neutral` (confirmed in `face_blendshapes_graph.cc`). There is no official Flutter plugin, so you would call it natively.
  - To detect a tongue on Android you'd need a custom classifier: for example, a small TFLite model on the mouth area cut out using the Face Landmarker, trained with Teachable Machine or MediaPipe Model Maker. A rougher alternative is `jawOpen` plus a red/pink pixel check below the lower lip.
- **Expressions both platforms detect easily:** `jawOpen` (mouth wide open), `cheekPuff`, `eyeBlinkLeft` vs `eyeBlinkRight` (wink), `mouthSmile*`, `browInnerUp` (raised eyebrows), `mouthPucker` (kiss face).

### 4.17 Bluetooth connection
- **Recommended: detect it through the audio route. No permission needed on either platform.**
  - **iOS:** check whether `AVAudioSession.sharedInstance().currentRoute.outputs` has a `portType` in `.bluetoothA2DP`, `.bluetoothHFP` or `.bluetoothLE`. Listen for `AVAudioSession.routeChangeNotification`.
  - **Android:** check whether `AudioManager.getDevices(GET_DEVICES_OUTPUTS)` has a type in `TYPE_BLUETOOTH_A2DP`, `TYPE_BLUETOOTH_SCO`, `TYPE_BLE_HEADSET`, `TYPE_BLE_SPEAKER` or `TYPE_HEARING_AID`. Listen with `AudioManager.registerAudioDeviceCallback`.
  - **Flutter:** `audio_session`, using `getDevices()` and `devicesChangedEventStream`. The types are `bluetoothA2dp`, `bluetoothSco` and `bluetoothLe`.
- **"Any Bluetooth connection"** (watch, car, keyboard) is **not possible on iOS**: CoreBluetooth only sees BLE devices whose service UUIDs you already know, through `retrieveConnectedPeripherals(withServices:)`. On Android it needs `BLUETOOTH_CONNECT` (a runtime prompt on Android 12+). The Bluetooth radio on/off state on iOS (`CBCentralManager.state`) needs a Bluetooth permission prompt. Also, the Control Center Bluetooth toggle only disconnects devices; it doesn't turn the radio off.

### 4.18 Light/dark mode
- **Flutter:** `MediaQuery.platformBrightnessOf(context)`, `WidgetsBindingObserver.didChangePlatformBrightness`, `PlatformDispatcher.platformBrightness`.
- **Native:** Android `Configuration.uiMode & UI_MODE_NIGHT_MASK`. iOS `UITraitCollection.userInterfaceStyle`.
- **Gotcha:** on Pixel phones, **Battery Saver turns on Dark theme**. That conflicts with level 21 and with the light-mode baseline.

### 4.19 Power saving mode
- **Android:** `PowerManager.isPowerSaveMode()` and the `ACTION_POWER_SAVE_MODE_CHANGED` broadcast. OEM battery savers (Samsung, Xiaomi, Huawei) can behave differently, so test them. Samsung's power saving can also lower brightness.
- **iOS:** `ProcessInfo.processInfo.isLowPowerModeEnabled` and `.NSProcessInfoPowerStateDidChange`. **Low Power Mode turns itself off once the phone charges to 80% or more.**
- **Flutter:** `battery_plus` `isInBatterySaveMode`.

### 4.20 Location
- **Reading the switch (no permission):**
  - Android: `LocationManager.isLocationEnabled()` (API 28+) and the `LocationManager.MODE_CHANGED_ACTION` broadcast.
  - iOS: `CLLocationManager.locationServicesEnabled()`. Apple warns against calling it on the main thread. Instead, react to `locationManagerDidChangeAuthorization`, which also fires when the global switch changes. While the switch is off, iOS reports the app's status as `.denied`.
- **Getting a position (prompt):**
  - Android: `ACCESS_COARSE_LOCATION` + `ACCESS_FINE_LOCATION`. Since Android 12 the user can pick "Approximate".
  - iOS: `requestWhenInUseAuthorization()` with `NSLocationWhenInUseUsageDescription`. Since iOS 14 the user can turn "Precise" off, and `accuracyAuthorization` becomes `.reducedAccuracy` (accurate to a few kilometres).
- **Switching location on from inside the app (Android only):** Google Play Services `SettingsClient.checkLocationSettings`, then `ResolvableApiException.startResolutionForResult`, shows the system "Turn on device location?" dialog. The `location` package wraps this as `requestService()`. iOS has nothing similar: the user goes to Settings > Privacy & Security > Location Services.
- **Flutter:** `geolocator`: `isLocationServiceEnabled()`, `checkPermission()`, `requestPermission()`, `getLocationAccuracy()`, `getCurrentPosition()`. Re-check the switch when the app resumes.
- **Trivia for the card:**
  - Even with location off, every server the phone connects to sees its IP address, which reveals the city.
  - Reading the Wi-Fi network name needs location permission on both platforms, because Wi-Fi names reveal where you are.

### 4.21 Clipboard
- **Flutter:** `Clipboard.getData(Clipboard.kTextPlain)`, `Clipboard.setData(...)`, `Clipboard.hasStrings()`. There is no change listener, so the in-house plugin should report changes (below) and the game reads the text only when something changed.
- **Android:** `ClipboardManager.getPrimaryClip()` and `addPrimaryClipChangedListener`.
  - Since Android 10, only the focused app (or the keyboard) can read the clipboard.
  - Since Android 12, the system shows a toast when an app reads clip data. A third-party guide says there's no toast when an app reads a clip it wrote itself **(verify)**.
  - Since Android 13, the system clears the clipboard automatically after a while.
  - Copied passwords can be flagged with `ClipDescription.EXTRA_IS_SENSITIVE` so they're hidden in the preview.
- **iOS:** `UIPasteboard.general.string`, `changeCount` and `UIPasteboard.changedNotification`.
  - Firefox for iOS reads `changeCount` because it doesn't trigger a prompt.
  - Since iOS 16, reading content that *another app* put on the clipboard shows "Allow Paste?". Since iOS 16.1, each app has a "Paste from Other Apps" setting.
  - Reading text copied inside the game itself is expected not to prompt **(verify)**.
  - `hasStrings` was reported to trigger the prompt on some iOS 16 versions, so prefer `changeCount`.

### 4.22 Text size
- **Flutter:** `MediaQuery.textScalerOf(context)`. Android 14+ scales text non-linearly (big text grows less than small text), so measure the factor at the menu's real font size: `textScaler.scale(12) / 12`. Flutter rebuilds automatically when the setting changes.
- **Android:** `Configuration.fontScale` (default 1.0; Android 14 allows up to 2.0) and `onConfigurationChanged`. "Display size" is a different setting (screen density).
- **iOS:** `UIApplication.shared.preferredContentSizeCategory` (default `.large`) and `UIContentSizeCategory.didChangeNotification`. The player can change it in Settings > Display & Brightness > Text Size, in Accessibility > Larger Text, or with the Text Size control in Control Center.
- No permission on either platform.

### 4.23 Speech recognition (not used)

Level 24 has the player *type* the dish name, so the game doesn't use speech recognition. These notes are kept in case a future level needs it.

- **Flutter:** `speech_to_text`.
- **iOS:** `SFSpeechRecognizer` with `SFSpeechAudioBufferRecognitionRequest`. Two prompts: microphone and `NSSpeechRecognitionUsageDescription`. **The audio may be sent to Apple's servers by default.** Set `requiresOnDeviceRecognition = true` (when `supportsOnDeviceRecognition` is true for the language) to keep it on the phone.
- **Android:** `SpeechRecognizer` (needs a recognition service, normally the Google app) with `RECORD_AUDIO`. To stay on the device, use `SpeechRecognizer.createOnDeviceSpeechRecognizer()` (Android 12+) or `RecognizerIntent.EXTRA_PREFER_OFFLINE`.
- **Matching:** ignore case and punctuation, and accept a close match (for example, edit-distance similarity ≥ 0.8). Use dish names made of common words in the recogniser's language.

### 4.24 Step counter
- **Android:** `TYPE_STEP_DETECTOR` (one event per step, short delay) or `TYPE_STEP_COUNTER` (total since reboot, can arrive in batches). Runtime permission `ACTIVITY_RECOGNITION` (Android 10+). Some cheap phones have neither sensor.
- **iOS:**
  - `CMPedometer.isStepCountingAvailable()`, then `startUpdates(from: Date())`, which reports `numberOfSteps`. Updates arrive in batches every few seconds. Permission: Motion & Fitness (`NSMotionUsageDescription`).
  - `queryPedometerData(from:to:)` returns past steps, reportedly the last ~7 days **(verify)**.
  - `CMMotionActivityManager` tells whether the user is walking, running, cycling or in a car.
- **Flutter:** `pedometer`. `stepCountStream` gives a running total, so subtract the value at level start. `pedestrianStatusStream` gives walking/stopped.
- **Fallback, and a lesson:** counting acceleration peaks with `sensors_plus` works with **no permission at all**. Use it for instant feedback while the official count catches up.

### 4.25 Shake
- **Flutter:** `sensors_plus` `userAccelerometerEventStream()` (acceleration with gravity removed). A shake is a magnitude above ~12–15 m/s² followed by a change of direction. Count the peaks inside a short window.
- **iOS alternative:** UIKit's `motionEnded(.motionShake)` system event. Simpler, but you can't tune it.
- **Gotcha:** if a text field has focus, iOS may offer "Undo Typing" when the phone is shaken.
- No permission.

### 4.26 Screen lock
- **Android:** a `BroadcastReceiver` registered at runtime for `Intent.ACTION_SCREEN_OFF` and `ACTION_USER_PRESENT` (unlocked). These can't be declared in the manifest. You can also read `KeyguardManager.isKeyguardLocked()` and `PowerManager.isInteractive()`. No permission.
- **iOS:** **there is no public "device locked" event.** The app is only told it went to the background, which also happens when the player goes to the home screen or switches app. Signals you can combine:
  - `UIApplication.protectedDataWillBecomeUnavailableNotification`, and `isProtectedDataAvailable == false` after returning. These only happen if the phone has a passcode, and developers report the timing isn't guaranteed.
  - How long the app spent in the background.
  
  The Darwin notification `com.apple.springboard.lockcomplete` is private API and gets apps rejected.
- **Flutter:** `AppLifecycleListener` for the lifecycle, plus the in-house plugin for the native signals.
- **Lesson for the card:** Android tells apps when your screen turns off. iOS only tells them you left.

### 4.27 Biometrics
- **Flutter:** `local_auth`: `isDeviceSupported()`, `getAvailableBiometrics()`, `authenticate(...)` with a biometric-only option. The API changed in recent major versions, so follow the current docs.
- **Android:** `androidx.biometric.BiometricPrompt` and `BiometricManager.canAuthenticate(BIOMETRIC_STRONG)`. Permission `USE_BIOMETRIC` (install-time). With `local_auth`, `MainActivity` must extend `FlutterFragmentActivity`.
- **iOS:** `LAContext.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics)`, `evaluatePolicy` and `biometryType` (`.faceID`, `.touchID`, `.opticID`). `NSFaceIDUsageDescription` is required, and the first Face ID use shows a prompt.
- **What the app learns:** success or failure, which biometric type exists, and whether biometrics are set up. On iOS, `evaluatedPolicyDomainState` (newer code reads `domainState` on iOS 18+) changes when a face or fingerprint is added or removed. Banking apps use this to force a new login. It doesn't say what changed.
- **Fallback:** if no biometrics are set up, allow the device PIN (`deviceOwnerAuthentication`). The lesson ("the app only gets yes or no") still holds.

### 4.28 Wi-Fi vs mobile data
- **Flutter:** `connectivity_plus`. `checkConnectivity()` and `onConnectivityChanged` return a list of `ConnectivityResult` values (`wifi`, `mobile`, `vpn`, …).
- **Android:** `ConnectivityManager.registerDefaultNetworkCallback` and `NetworkCapabilities.hasTransport(TRANSPORT_WIFI | TRANSPORT_CELLULAR)`. Permission `ACCESS_NETWORK_STATE` (install-time). Apps can read the carrier name without permission (`TelephonyManager.getNetworkOperatorName()`).
- **iOS:** `NWPathMonitor` with `path.usesInterfaceType(.wifi / .cellular)` and `path.isExpensive`. No permission. iOS no longer gives apps the carrier name (`CTCarrier` is deprecated).
- **Gotchas:**
  - Wi-Fi Assist (iOS) and "switch to mobile data automatically" (Android) can use mobile data while Wi-Fi is still connected. Require Wi-Fi to be gone, not just mobile data to be in use.
  - Players without a SIM or a data plan can't do this level.

---

## 5. Per-level detection spec

The thresholds are starting points for tuning. "Hold" means the condition must stay true for that long, which removes noise.

| Lvl | Signals | Success condition | Fallback / notes |
|---|---|---|---|
| 1 Baby | brightness | slider-% ≤ 5% (iOS `brightness ≤ 0.05`; Android linear → gamma converted), hold 1 s | Link the in-game lamp's intensity to brightness live |
| 2 Speaker | volume (+ Android key events) | volume ≤ 1 step, or ≥ N volume-down presses (Android) | iOS can't tell a button press from the Control Center slider |
| 3 Tap | gravity, gyroscope z | phone flat (abs(gravity z) > 0.9 g) and the integrated z rotation reaches −2π (clockwise) | Rotate the tap handle with the integrated angle. Reset if the phone is lifted off the table |
| 4 Phone | gravity, proximity | face down (gravity z sign flips) **and** proximity near, hold 1.5 s | If proximity is missing or call-only, accept face down alone. iOS turns the screen off, so play a "click" sound |
| 5 Dog | mic RMS, volume | baseline noise measured for 1 s, then RMS > baseline + 20 dB (or > −20 dBFS) for ≥ 400 ms | Turn on echo cancellation, or leave silent gaps in the barking and only listen in the gaps. A volume decrease shows the tease |
| 6 Ghost hunter | screenshot event + a ~1 s history of which ghosts were in the room | a ghost counts as photographed if it was in the room in the ~700 ms before the event; win when all 3 are caught within 6 photos | Android < 14: locked, with an explanation card. Show the button combination. Ignore 1-step volume changes (the combination uses a volume button). Pause ghost timers if the player opens the screenshot preview |
| 7 TV remote | gravity | calibrate the "pointing at ceiling" pose, then the angle between the current and starting "up" direction ≥ 70°, hold 1 s | This is tilt, not gyroscope |
| 8 Car sun | lux (Android) / camera EV (iOS) | below 20% of the level-start baseline (iOS: drop of ≥ 3 EV), hold 1 s | Precondition: baseline ≥ about 100 lux |
| 9 Hospital | charging state | charging, full, or plugged | Show the real battery % on the machine |
| 10 Plane | airplane mode | Android setting = 1; iOS heuristic (§4.9) | iOS precondition: cellular service at level start |
| 11 Crime | clock offset (§4.10) | −10 min ≤ offset ≤ −1 min → success; below −10 min → "nothing has happened yet"; ≥ +1 min → police tape | Re-check when the app resumes. iOS pickers have minute precision. A time-zone change only shifts local time: detect it and tease the player |
| 12 Prison | torch state | torch on | iOS: see §4.11 |
| 13 Teacher | relative altitude / pressure, user acceleration | +0.6 m (iOS) or −0.07 hPa (smoothed) compared to the seated baseline, hold 1 s | No barometer: acceleration pattern plus phone held up |
| 14 Elbow | `radiusMajor`, `size`, `pressure` | after an index-finger calibration tap: radius ≥ 1.3× the calibration, held 3 s with little movement | If radius never changes on the device, fall back to holding only |
| 15 Painting | gravity roll = atan2(gx, gy) | painting starts 15–30° off; abs(roll − target) < 2°, hold 1.5 s | Smooth with a low-pass filter |
| 16 Fly | fused yaw (north-referenced) | fly has a world bearing; screen x = (fly bearing − device bearing) mapped to the field of view; tap hits it | Prompt a figure-8 when compass accuracy is low |
| 17 Mouth | pointer count and spread | ≥ 5 pointers (allow 4), average distance to centroid shrinks ≥ 50% | iPhone max is 5 touches |
| 18 Kid | ARKit blendshapes / Android model | `tongueOut ≥ 0.5`, hold 0.5 s | Android: see §4.16 |
| 19 Bus | audio route | a Bluetooth output is present | Allow wired headphones, or skip |
| 20 Rooster | platform brightness | dark | |
| 21 Power strip | power save | true | Allow the side effects (Pixel dark theme, Samsung dimming) while this level runs |
| 22 Delivery | location switch, permission, accuracy, position | switch on + permission granted + a position received | If the switch is already on, start from the permission step. Possible twist: with "Approximate", the courier goes to the wrong house. Android: offer the in-app "turn on location" dialog |
| 23 Alarm code | clipboard change + text | clipboard text equals the code | Show the code as selectable text. Afterwards the game clears the clipboard itself ("don't leave the code lying around") |
| 24 Restaurant | text scale + typed answer | scale at the menu's font size ≥ 1.3× the level-start value, **and** the typed name matches the starred dish (ignoring case, accents and extra spaces) | Measure against the player's own starting size |
| 25 Doctor | step detector/counter (+ acceleration for instant feedback) | ≥ 20 steps since level start | No sensor or permission denied: count steps from acceleration |
| 26 Ketchup | user acceleration | 6 shake peaks within 3 s; a bit more ketchup comes out with each | Show a "hold your phone tight!" warning |
| 27 Lock | Android screen-off event; iOS background time + protected data | Android: screen-off event, then the player returns. iOS: app left for ≥ 2 s (plus protected data unavailable, when there's a passcode), then the player returns | Keep the screen awake during every level so auto-lock can't complete this one by accident |
| 28 Bank | biometric result | `authenticate` returns true | No biometrics set up: device PIN |
| 29 Video | network type | Wi-Fi gone and mobile data present | If the player starts on mobile data, flip the story ("switch to Wi-Fi to save your data"). No SIM or data plan: skip |

---

## 6. Start checklist, baseline snapshot and reverting

### Can each start check be verified?
| Check | Android | iOS | Comment |
|---|---|---|---|
| Accept all permissions | ✅ | ✅ | Asked during setup; the player may refuse (§8) |
| Light mode | ✅ | ✅ | |
| Volume 50% | ⚠️ step counts differ (7/15 = 47%, 8/15 = 53%) | ✅ (8/16) | Use a range such as 30–70% |
| Brightness 50% | ⚠️ non-linear slider, OEM ranges | ✅ | Use a range such as 25–90% of slider %, or just "not at minimum" |
| Airplane mode off | ✅ | ⚠️ heuristic (connectivity) | |
| Date/time correct | ✅ `AUTO_TIME` | ⚠️ NTP comparison (needs network) | |
| No Bluetooth connections | ⚠️ needs a permission | ❌ for non-audio devices | Change to "no Bluetooth **audio** device" (audio route, both platforms) |
| Not power saving | ✅ | ✅ | |
| Well-lit room | ✅ lux | ⚠️ needs camera permission | Move to level 8's precondition |
| Not charging | ✅ | ✅ | Needed for level 9 |
| Location off | ✅ `isLocationEnabled()` | ✅ `locationServicesEnabled()` | Many people rely on location (maps, Find My). Check it only as level 22's precondition |
| *Suggested:* on Wi-Fi, with mobile data available | ✅ | ✅ | Needed for level 29 (or flip the story) |
| *Text size* | — | — | Don't set a target. Record the starting value: players who use large text need it |
| *Suggested:* torch off | ✅ | ⚠️ | Needed for level 12 |
| *Suggested:* battery ≥ 25% | ✅ | ✅ | Prevents automatic power saving turning on mid-game |

### Baseline snapshot ("StateGuard")
1. When the game starts, take a snapshot of every tracked setting: brightness, volume, dark mode, airplane mode, power saving, Bluetooth audio, charging, torch, clock offset, location switch, Wi-Fi/mobile data, text size.
2. Each level declares which settings it changes (`touchedSettings`) and its preconditions.
3. After the level succeeds, compare the current values with the snapshot, using **ranges, not exact equality**. Show a checklist that ticks itself off live as the player puts things back.
4. Re-read every value on `AppLifecycleState.resumed`, because listeners can miss changes made while the player was in the Settings app.
5. **Shortcuts to the right setting.** Android can open specific pages: `Settings.ACTION_DISPLAY_SETTINGS`, `Settings.Panel.ACTION_VOLUME`, `ACTION_AIRPLANE_MODE_SETTINGS`, `ACTION_DATE_SETTINGS`, `ACTION_BLUETOOTH_SETTINGS`, `ACTION_BATTERY_SAVER_SETTINGS`, `ACTION_LOCATION_SOURCE_SETTINGS`, `Settings.Panel.ACTION_INTERNET_CONNECTIVITY` (all available through `app_settings`). iOS can only open the app's own page (`UIApplication.openSettingsURLString`); `App-Prefs:` URLs are private API and risk App Store rejection. On iOS, show text instructions and suggest Control Center.
6. **The app can revert some settings itself:** volume on Android, brightness on iOS. Offering a "fix it for me" button is also a lesson: apps can do this without asking.

---

## 7. Suggested project structure

```
lib/
  signals/
    signal_hub.dart          # one normalized Stream per input
    providers/real/...       # backed by packages + device_signals plugin
    providers/fake/...       # debug sliders for emulator/desktop
  levels/
    level.dart               # requiredSignals, preconditions, touchedSettings,
                             # isSupported(device), success stream, fallback,
                             # hints (with the time each appears), solution
    l01_baby.dart … l29_video.dart
  state_guard/               # baseline snapshot, diff, revert checklist
  awareness/                 # per-level "what the app learned" cards + final report
  ui/                        # Flame scenes, Rive assets
packages/
  device_signals/            # in-house plugin (Pigeon): Kotlin + Swift
```

- **Debug overlay:** a hidden panel that shows every signal's live value. It is essential for tuning thresholds on real devices.
- **Fake providers:** let you build levels in the emulator or on desktop, where most of these inputs don't exist.
- **Lock the app to portrait,** so tilt levels don't rotate the UI.
- **Keep the screen awake during levels** with `wakelock_plus`. Auto-lock can be as short as 30 s. It would interrupt levels 3, 4, 13 and 25, and it would complete level 27 by accident.
- **Hint timer:** show hints at about 30, 60 and 90 s and the (?) solution at 120 s. Time it with `Stopwatch` (a monotonic clock), so level 11's clock change can't break it. Keep it running while the player is in the Settings app.
- **Device test matrix:** at least one Pixel, one Samsung, one budget Android without a barometer, one iPhone with Face ID, and one iPhone SE.

---

## 8. Permissions and store policy

**AndroidManifest** (level numbers in brackets):
- With a prompt: `RECORD_AUDIO` (5), `CAMERA` (18), `ACCESS_COARSE_LOCATION` + `ACCESS_FINE_LOCATION` (22), `ACTIVITY_RECOGNITION` (25).
- Install-time, no prompt: `DETECT_SCREEN_CAPTURE` (6), `USE_BIOMETRIC` (28), `ACCESS_NETWORK_STATE` (29). Optional: `DETECT_SCREEN_RECORDING`.
- Remove any `READ_MEDIA_IMAGES` that a plugin brings in.

**iOS Info.plist:** `NSMicrophoneUsageDescription` (5), `NSCameraUsageDescription` (18, plus level 8's light estimate), `NSMotionUsageDescription` (13, 25), `NSLocationWhenInUseUsageDescription` (22), `NSFaceIDUsageDescription` (28).

**App Store guideline 5.1.1(iv):** apps must not "manipulate, trick, or force people to consent to unnecessary data access". Refusing to start the game until *every* permission is granted is a rejection risk. Ask for all permissions during setup, explaining each one, and let the player continue if they decline: the matching levels become skippable. Asking everything up front also stops a mid-game prompt from giving away which level uses it.

**Google Play:** fill in the Data safety form. The photo/video permission policy is why screenshot detection is limited to Android 14+.

**Privacy policy:** both stores require one when you use the camera or microphone. Keep all processing on the device with no analytics, and say so clearly. For a game about privacy, that is what makes it credible.

---

## 9. Things to test on real devices first

Test these before investing in the art for the matching levels:

1. *(iOS, later)* **iOS torch state** after switching it from Control Center (`isTorchActive` KVO with no capture session). Decides level 12 on iOS.
2. *(iOS, later)* **iOS `outputVolume`** on current iOS (reported regression on iOS 18+). Decides levels 2 and 5.
3. **Android `TorchCallback`** without the CAMERA permission.
4. **Proximity on Samsung** outside a call.
5. **Touch radius range** on your test devices: how many distinct values does a thumb give compared with an index finger?
6. **`CMAltimeter` / `TYPE_PRESSURE`** response to standing up (how much it changes and how fast).
7. *(iOS, later)* **iOS camera EV** values for a lit room compared with a hand covering the camera.
8. **Android screenshot callback** with methods other than the buttons (Quick Settings tile, palm swipe, 3-finger swipe).
9. *(iOS, later)* **iOS screen-lock heuristic:** how fast `protectedDataWillBecomeUnavailable` fires with "Require Passcode: Immediately", and what happens on a phone with no passcode. Decides level 27 on iOS.
10. *(iOS, later)* **iOS paste prompt:** does reading text the player copied *inside the game* show "Allow Paste"? (Expected: no.)
11. **Step counting delay:** Android `TYPE_STEP_COUNTER` batching vs `TYPE_STEP_DETECTOR`, and how often `CMPedometer` updates.
---

## 10. Inputs not in the plan yet (API summary)

Most of the earlier suggestions are now levels 22–29. These are left:

| Input | Android | iOS | Permission |
|---|---|---|---|
| Magnetic field strength (metal detector) | `TYPE_MAGNETIC_FIELD` | `CMMagnetometerData` | none |
| Ringer / silent | `AudioManager.getRingerMode()` | ❌ no API | none |
| Do Not Disturb / Focus | `NotificationManager.getCurrentInterruptionFilter()` | `INFocusStatusCenter` (needs an authorization prompt and the Communication Notifications capability) | iOS: prompt |
| Screen recording | `addScreenRecordingCallback` (Android 15+) | `UIScreen.isCaptured` | Android: install-time |
| Notifications | `POST_NOTIFICATIONS` (Android 13+) | `UNUserNotificationCenter` | prompt on both |

---

## Sources

- Android 14 screenshot detection: https://developer.android.com/about/versions/14/features/screenshot-detection
- Android 15 screen recording detection: https://proandroiddev.com/screen-recording-detection-android-15-26ee709b66b4
- Expo screen capture: https://docs.expo.dev/versions/latest/sdk/screen-capture
- Google Play photo/video permission policy: https://support.google.com/googleplay/android-developer/answer/14115180
- Apple user privacy and data use: https://developer.apple.com/app-store/user-privacy-and-data-use/
- MediaPipe Face Landmarker: https://developers.google.com/edge/mediapipe/solutions/vision/face_landmarker
- MediaPipe blendshape names (source): https://github.com/google-ai-edge/mediapipe/blob/master/mediapipe/tasks/cc/vision/face_landmarker/face_blendshapes_graph.cc
- ARKit face tracking: https://developer.apple.com/documentation/arkit/arfacetrackingconfiguration and https://developer.apple.com/forums/thread/703607
- SensorKit (research only): https://developer.apple.com/documentation/sensorkit/configuring-your-project-for-sensor-reading
- iOS airplane mode (no public API): https://pub.dev/documentation/airplane_mode_checker/latest and https://developer.apple.com/forums/thread/843544
- iOS torch: https://developer.apple.com/documentation/avfoundation/capture-device-lighting
- Android TorchCallback: https://developer.android.com/reference/android/hardware/camera2/CameraManager.TorchCallback
- iOS outputVolume iOS 18 report: https://developer.apple.com/forums/thread/799104
- Samsung proximity during calls only (anecdotal): https://groups.google.com/g/tasker/c/4-HnBtGVkAc
- Android brightness slider curve: https://stkent.github.io/2018/11/12/more-on-android-pies-brightness-control-changes.html
- Pixel Battery Saver turns on Dark theme: https://support.google.com/googlepixeltablet/answer/13554746 and https://www.androidpolice.com/2018/11/13/november-security-patch-forces-night-mode-battery-saver-enabled-whether-want-not/
- iOS Low Power Mode turns off at 80%: https://support.apple.com/101604
- Flutter engine touch mapping: `FlutterViewController.mm` and `AndroidTouchProcessor.java` in https://github.com/flutter/flutter (engine/src/flutter/shell/platform/…)
- sensors_plus: https://pub.dev/packages/sensors_plus/changelog
- screen_brightness: https://pub.dev/packages/screen_brightness
- flutter_volume_controller: https://pub.dev/packages/flutter_volume_controller
- audio_session AudioDeviceType: https://pub.dev/documentation/audio_session/latest/audio_session/AudioDeviceType.html
- Screenshot packages: https://pub.dev/packages/screen_capture_event_plus, https://pub.dev/packages/detect_screenshot, https://pub.dev/packages/flutter_screenshot_detector
- Flame + Rive: https://docs.flame-engine.org/main/bridge_packages/flame_rive/rive.html and https://rive.app/docs/runtimes/flutter/state-machines
- Android clipboard (toast on reads, auto-clear, sensitive flag): https://developer.android.com/privacy-and-security/risks/secure-clipboard-handling
- iOS clipboard `changeCount` without a prompt (Firefox for iOS): https://github.com/mozilla-mobile/firefox-ios/pull/20200
- iOS 16.1 "Paste from Other Apps" setting: https://www.macrumors.com/2022/10/17/ios-16-1-paste-from-other-apps-settings/
- iOS lock detection (forum reports): https://developer.apple.com/forums/thread/69333 and https://developer.apple.com/forums/thread/744170
- Biometric enrollment change detection: https://swiftrocks.com/detecting-touchid-fingerprint-changes
- CMPedometer: https://developer.apple.com/documentation/coremotion/cmpedometer
