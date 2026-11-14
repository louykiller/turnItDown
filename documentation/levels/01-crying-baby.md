# Level 01 · Lights Out for the Baby

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Display brightness |
| **Permission** | None (Android and iOS) |
| **Support** | Android ✅ · iOS ✅ |
| **Access difficulty: Android** | 🟡 Moderate |
| **Access difficulty: iOS** | 🟢 Easy |
| **Setting to revert** | Brightness (back up from minimum) |
| **Reference** | [research.md §4.1](../research.md#41-display-brightness) |

## Scene

A cosy cartoon nursery at night. Pastel wallpaper with little stars, a wooden crib in the middle, a mobile of felt moons above it, and a rocking chair with a teddy bear. A huge ceiling lamp hangs over the crib and glares like a stadium floodlight, its cone of light falling straight onto the baby's face.

The baby is wide awake, squinting and wailing in a loop. A tired parent stands in the doorway in a dressing gown, holding a bottle, with dark circles under their eyes, whispering "please sleep…".

- **Live feedback:** the lamp's glow follows the real screen brightness. As the player lowers the brightness, the lamp dims in real time and the baby moves through *screaming → sobbing → yawning → asleep*. In Rive this is one state machine driven by a `brightness` number input.
- **Wrong direction:** raising the brightness makes the lamp flare. The baby cries louder and the teddy bear covers its eyes.
- **Success:** the lamp goes out and a soft night light comes on. The mobile turns to a lullaby, the baby falls asleep with a tiny "zzz" bubble, and the parent gives a thumbs-up and tiptoes out.

## Goal

Lower the phone's **display brightness to the minimum**.

**Success condition** (hold for 1 s):
- iOS: `UIScreen.brightness ≤ 0.05`.
- Android: slider position ≤ 5%, converted from the stored value as explained below. Alternatively, the stored value is at the device's minimum.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟡 `screen_brightness`, plus converting the stored value into a slider position (Android's brightness curve) | 🟢 `screen_brightness`. The 0–1 value already matches the slider |
| **Overall** | **🟡 Moderate** | **🟢 Easy** |

## Technical documentation

**Android**
- Read `Settings.System.getInt(contentResolver, Settings.System.SCREEN_BRIGHTNESS)`. No permission is needed.
- Watch for changes with a `ContentObserver` on `Settings.System.getUriFor(Settings.System.SCREEN_BRIGHTNESS)`.
- `Settings.System.SCREEN_BRIGHTNESS_MODE` tells you whether auto-brightness is on.
- **The slider is not linear.** Since Android 9 the slider follows a curve (Hybrid Log Gamma), but the setting stores the linear value. On stock Android, a slider at 50% is only about 8% of the linear range. To compare against slider positions, convert with the same formula as AOSP's `BrightnessUtils.convertLinearToGamma`.
- The maximum isn't always 255. Read the hidden system resource `config_screenBrightnessSettingMaximum` with `Resources.getSystem().getIdentifier(...)`.

**iOS**
- `UIScreen.brightness` (0.0–1.0). On iOS 16+ reach it through `view.window?.windowScene?.screen`. No permission is needed.
- `UIScreen.brightnessDidChangeNotification` fires while the player drags the slider in Control Center. The game stays on screen behind Control Center, so the lamp can dim live.

**Flutter**
- `screen_brightness` reads the system brightness and has a stream of system brightness changes (2.1.0+).

**Gotchas**
- With auto-brightness on, the system can push the brightness back up by itself.
- Compare against ranges, never exact values.

## Clues

1. **Vague** (the parent sighs): "It's so bright in here… no wonder she can't sleep."
2. **Closer:** "That lamp has no switch. But your phone's screen has one…"
3. **Specific:** "Turn your phone's screen brightness all the way down."

**(?) Solution after 2 minutes:**
- **Android:** swipe down from the top of the screen and drag the brightness slider all the way to the left.
- **iPhone:** open Control Center and drag the brightness bar all the way down. On iPhones with Face ID, swipe down from the top-right corner. On iPhones with a Home button, swipe up from the bottom.

## After the level

**Card:** "Screen brightness: 64% when you started. No permission needed."

**What you learned:** any app can read your screen brightness at any time without asking. While it's open, an app can also *change* it. That's how boarding-pass and QR-code apps make the screen fully bright.

**Trivia:**
- Brightness gives away your surroundings. Auto-brightness near the minimum usually means a dark room, for example lying in bed at night.
- Android's slider is a curve. The middle of the slider is only about 8% of the brightness value the phone actually stores.
- An Android app can change the brightness of its own window without any permission. Changing the system setting needs a special permission.

**Final report entry:** "Screen brightness: 64% (auto-brightness on)". Auto-brightness can only be read on Android.

## If the device can't do it

- Every phone and tablet supports this level, so no fallback is needed.
- **Auto-brightness keeps pushing the value up:** show a tip to turn auto-brightness off.
  - Android: Settings > Display > Adaptive brightness.
  - iPhone: Settings > Accessibility > Display & Text Size > Auto-Brightness.
- **The device reports an unusually high minimum** (some manufacturers do): also accept a value that hasn't moved for 2 s after the player dropped it by at least 80% from where it started.

## Reverting

Morning arrives. Sunlight comes through the curtains, the baby wakes up giggling, and the parent says: "Good morning! Let's turn the lights back on."
- **Player action:** raise the brightness back to around where it was.
- **Check:** the slider is between 25% and 90%, or within 10% of the value at the start.
