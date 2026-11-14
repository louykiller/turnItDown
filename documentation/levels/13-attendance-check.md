# Level 13 · Present!

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Barometer (air pressure → height change), with the accelerometer as backup |
| **Permission** | Android: none. iOS: Motion & Fitness prompt (`NSMotionUsageDescription`), asked during setup |
| **Support** | Android ⚠️ (many budget phones have no barometer) · iOS ✅ |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟡 Moderate |
| **Setting to revert** | None |
| **Reference** | [research.md §4.12](../research.md#412-barometer) |

## Scene

A cartoon classroom. A chalkboard says "ATTENDANCE" in big letters, a globe sits on the teacher's desk, and paper planes are frozen mid-air. A strict teacher with half-moon glasses and a very long list calls names one by one. Each student jumps up, raises a hand and shouts "Present!"

The player's character is slumped at a desk at the back, half asleep, drooling on a textbook.

- **Start:** "Everyone sit down for the roll call!" The game asks the player to sit down too, and measures their seated height (the baseline) during the first few names.
- **The call:** the teacher reaches the player's name. If nothing happens, she repeats it louder, the other students turn round and giggle, and a paper plane hits the character's head.
- **Live feedback:** a little "height" bar beside the character rises as the player stands. Barometers react with a delay of 1–2 s, so the character starts to stir as soon as the accelerometer feels movement.
- **Success:** the character jumps up with an arm stretched high: "PRESENT!" The teacher peers over her glasses, ticks the list with a flourish, and the class applauds.

**Whose name is called:** the name the player typed at the start of the game. Optional awareness twist on Android: the teacher reads the phone's *device name* ("Ana's Galaxy S24"), which apps can read without permission **(verify)**. On iOS 16+, apps only see "iPhone".

## Goal

When your name is called, **stand up and raise your phone high above your head**.

**Success condition** (hold for 1 s):
- iOS: relative altitude ≥ +0.6 m compared to the seated baseline.
- Android: smoothed air pressure drops by ≥ 0.07 hPa compared to the seated baseline.
- Average the baseline over 3–5 s while seated, and smooth the readings over about 1 s.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟡 Motion & Fitness prompt |
| **Code to get the value** | 🟢 `sensors_plus` | 🟡 `sensors_plus` gives pressure. The better relative-altitude value needs a small native wrapper around `CMAltimeter` |
| **Overall** | **🟢 Easy** | **🟡 Moderate** |

Many budget Android phones have no barometer at all. That's missing hardware, covered by the accelerometer fallback.

## Technical documentation

**Android**
- `Sensor.TYPE_PRESSURE` (hPa). Convert with `SensorManager.getAltitude(SensorManager.PRESSURE_STANDARD_ATMOSPHERE, p)`.
- `getDefaultSensor(TYPE_PRESSURE)` returns `null` on phones without one, which includes many budget Samsung A-series, Xiaomi and Motorola models.
- No permission is needed.

**iOS**
- `CMAltimeter.isRelativeAltitudeAvailable()`, then `startRelativeAltitudeUpdates(to:withHandler:)` → `CMAltitudeData.relativeAltitude` (m) and `pressure` (kPa).
- Needs `NSMotionUsageDescription` (Motion & Fitness prompt).

**Flutter**
- `sensors_plus` `barometerEventStream()` (hPa). iOS has a sampling-rate limit.
- For relative altitude on iOS, wrap `CMAltimeter` in the in-house plugin.

**Physics and noise**
- Near sea level, pressure drops about 0.12 hPa per metre. Standing up and raising the phone moves it about 0.8–1.2 m, so roughly 0.1–0.15 hPa.
- That's close to the sensor's noise. Smooth the readings, and expect a delay of 1–2 s.
- Air conditioning and doors closing also change the pressure. **Test this on real devices.**

## Clues

1. **Vague** (the teacher, tapping her pen): "I'll call your name *once* more…"
2. **Closer:** "In this class, you don't just *say* you're here. You show it!"
3. **Specific:** "Sit down. When your name is called, stand up and raise your phone high above your head."

**(?) Solution after 2 minutes:** sit on a chair, wait for your name, then stand up and lift your phone as high as you can, arm straight up, and hold it there for a second or two.

## After the level

**Card:** "Air pressure here: 1013.2 hPa. Your phone noticed when you stood up, from the air pressure alone." (On iPhone: "…with your Motion & Fitness permission.")

**What you learned:** phones measure air pressure so precisely they can tell when you stand up, take the stairs or ride a lift. Android apps get this without asking.

**Trivia:**
- In the US, emergency rules require phones to help report *which floor* a 911 caller is on, and barometers are one of the ways they do it.
- The barometer also helps the GPS work out your altitude faster.
- Pressure readings across thousands of phones can even help forecast the weather.

**Final report entry:** "Estimated altitude from air pressure: about 40 m above sea level."

## If the device can't do it

- **No barometer** (common on budget Android phones): use the accelerometer instead. Detect the stand-up pattern (an upward push, then a slowdown) plus the phone ending up held high and upright. It's less precise, so use a slightly more generous threshold.
- **iPhone: Motion & Fitness refused:** use the accelerometer instead. `CMMotionManager` needs no permission.
- **Players who can't stand up** (wheelchair users, injury): after the first clue, offer "Raise your phone as high as you can", with a lower threshold (+0.4 m) or a lift detected by the accelerometer. Never make a level impossible for physical reasons; always offer the skip.

## Reverting

Nothing to revert.
