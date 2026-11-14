# Level 14 · Apply Pressure

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Touch size (contact radius), plus pressure on devices that report it |
| **Permission** | None (Android and iOS) |
| **Support** | Android ✅ (sizes vary between devices) · iOS ✅ (size only; iPhones since the XR/11 have no pressure sensing) |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟢 Easy |
| **Setting to revert** | None |
| **Reference** | [research.md §4.13](../research.md#413-touch-size-and-pressure) |

## Scene

A sunny park path. A kid in a helmet sits on the grass after falling off a bike. The bike lies on its side with its front wheel still spinning, and an ice cream lies upside down on the path. The kid holds out a scraped elbow with cartoon tears in their eyes. The wound is bleeding, kept mild and cartoonish: a few red drops dripping slowly onto the grass.

A friend runs over with a first-aid kit and fumbles with a roll of plasters.

The camera zooms in on the elbow until it fills most of the screen.

- **Step 1 — the check-up tap (calibration):** the friend says "Let me see… tap where it hurts!" The player taps with an index finger, and the game records that finger's contact size.
- **Step 2 — pressing:** a ring appears around the player's finger showing how big the contact is. It's red when too small and green when big enough. While a big enough press is held, the drops slow down and a 3-second ring fills up.
- **Moving or lifting** resets the timer, and the kid says "Ow! Keep still!"
- **Success:** after 3 seconds of steady pressure the bleeding stops. The friend slaps on a plaster with a smiley face, the kid stops crying, picks up the bike and wobbles off happily.

## Goal

**Press the wound with a big, flat fingertip (ideally a thumb) and hold it still for 3 seconds.**

**Success condition:**
- Contact radius ≥ 1.3× the index-finger calibration tap.
- Held for 3 s, moving less than about 10 dp.
- If pressure is available (`pressureMax > 1`, or a real range), a firm press adds to the score but is never required.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟢 Built into Flutter: every touch event carries size and pressure | 🟢 Built into Flutter. Size only: modern iPhones report no pressure |
| **Overall** | **🟢 Easy** | **🟢 Easy** |

The quality of the values varies a lot between devices, which is why the level calibrates first.

## Technical documentation

**Flutter**
- Use a `Listener` with `onPointerDown`, `onPointerMove` and `onPointerUp`. `PointerEvent` has `radiusMajor`, `radiusMinor`, `radiusMin`, `radiusMax`, `size`, `pressure`, `pressureMin` and `pressureMax`.
- From the Flutter engine source:
  - **iOS:** `radiusMajor = UITouch.majorRadius` (in points, in coarse steps). `radiusMin` and `radiusMax` are that radius ∓ `majorRadiusTolerance`. `pressure = UITouch.force`, which is **always 0 on iPhones without 3D Touch** (every model since the XR/11).
  - **Android:** `radiusMajor = MotionEvent.getToolMajor()`, `radiusMinor = getToolMinor()`, `size = getSize()` (0–1), `pressure = getPressure()`. Many devices report a constant or made-up pressure.

**Native equivalents**
- Android: `MotionEvent.getTouchMajor()`, `getToolMajor()`, `getSize()` and `getPressure()`.
- iOS: `UITouch.majorRadius`, `majorRadiusTolerance`, `force` and `maximumPossibleForce`.

**Why calibrate:** touch sizes differ hugely between screens. Comparing the thumb with the same player's index finger on the same phone is what makes the check reliable.

## Clues

1. **Vague** (the friend): "It won't stop bleeding! What do you do with a cut?"
2. **Closer:** "You have to *press* on it… and a fingertip is way too small."
3. **Specific:** "Press your thumb flat on the wound and hold it still for 3 seconds."

**(?) Solution after 2 minutes:** put the flat pad of your thumb (not the tip) on the red scrape so the ring turns green, then keep it completely still until the 3-second circle fills.

## After the level

**Card:** "On your screen, your thumb is 1.6× wider than your index finger. Every app receives this with every tap."

**What you learned:** your touchscreen tells apps how big each touch is, and on some phones how hard you press. Every app gets this with every tap and swipe, without asking.

**Trivia:**
- Banks and anti-fraud companies use "behavioural biometrics": how big your touches are, how hard you press, how you swipe and type. It helps tell you apart from a fraudster using your phone.
- iPhones 6s to XS had "3D Touch", which measured real pressure. Apple removed it, so modern iPhones only measure touch size.
- Real first aid: firm, direct pressure on a wound is the right first step to stop bleeding.

**Final report entry:** "Your thumb on screen: 1.6× your index finger."

## If the device can't do it

- **The device reports the same touch size for every finger:** detect this when the thumb's value equals the calibration value. Switch to "hold still for 3 seconds" with any finger, and have the card explain that the screen doesn't report touch size.
- **Players who can't use a thumb** (accessibility): after the third clue, accept any finger with a lower size ratio (1.1×) or hold-only.
- **Phone with a screen protector or a glove:** sizes may come out odd. The calibration handles most of this.

## Reverting

Nothing to revert.
