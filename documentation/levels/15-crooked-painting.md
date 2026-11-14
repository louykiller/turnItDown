# Level 15 · Hang It Straight

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Accelerometer (gravity direction, i.e. tilt / roll) |
| **Permission** | None (Android and iOS) |
| **Support** | Android ✅ · iOS ✅ |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟢 Easy |
| **Setting to revert** | None |
| **Reference** | [research.md §4.3](../research.md#43-gyroscope-accelerometer-orientation) |

## Scene

A freshly painted living room. A man balances on a wobbly stepladder, holding up a huge gold-framed painting: a very serious portrait of a dog in a royal robe. His wife stands behind him with her arms crossed, giving directions: "A bit to the left… no, the *other* left… now it's crooked the other way…"

A spirit level sits on top of the frame, with its little bubble far off to one side. The painting starts tilted by a random 15–30°.

- **Live feedback:** the painting's angle follows the phone's roll in real time. Tilting the phone tilts the painting, and the bubble slides along the level. Each time the painting passes through level, the wife says "Close…!" and the man's ladder wobbles.
- **When nearly level** (within 5°): the dog in the portrait starts to look pleased.
- **Success:** held level within 2° for 1.5 s, the bubble locks into the centre with a *ding*. The man hammers the nail, the wife says "PERFECT!" and applauds, and the dog in the painting winks.

## Goal

**Tilt the phone sideways** (like a steering wheel) until the painting is level, and **hold it still** for 1.5 s.

**Success condition:**
- roll = `atan2(gx, gy)` from the gravity vector, with the phone held upright facing the player.
- `|roll − target| < 2°`, held for 1.5 s. The target cancels out the random starting tilt.
- Low-pass filter the readings so the bubble doesn't jitter.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟢 `sensors_plus` | 🟢 `sensors_plus` |
| **Overall** | **🟢 Easy** | **🟢 Easy** |

The angle (one `atan2` plus smoothing) is level logic, so it isn't counted.

## Technical documentation

**Android**
- `Sensor.TYPE_GRAVITY`, or `TYPE_ACCELEROMETER` with a low-pass filter. No permission is needed.

**iOS**
- `CMMotionManager.startDeviceMotionUpdates()` → `CMDeviceMotion.gravity`. No permission is needed.

**Flutter**
- `sensors_plus` `accelerometerEventStream()`. Apply a low-pass filter (for example `g = 0.85·g + 0.15·raw`), then compute the roll.

**Gotchas**
- This is tilt, so it uses the accelerometer, not the gyroscope.
- Lock the app to portrait, or the user interface will rotate while the player tilts.
- Near-flat phones give unstable roll values. Ask the player to hold the phone upright, facing them.

## Clues

1. **Vague** (the wife): "It's STILL crooked!"
2. **Closer:** "The painting moves when your phone moves."
3. **Specific:** "Slowly tilt your phone sideways, like a steering wheel, until the bubble is in the middle, then hold still."

**(?) Solution after 2 minutes:** hold the phone upright in front of you. Slowly rotate it left or right, as if turning a steering wheel, until the bubble sits in the centre of the level. Keep it perfectly still for 2 seconds.

## After the level

**Card:** "Your phone knew its angle to within a fraction of a degree. No permission needed."

**What you learned:** the accelerometer tells apps your phone's exact angle at any moment. That's how spirit-level apps work, and it's available to every app silently.

**Trivia:**
- In 2019, researchers found that the tiny factory calibration errors of each phone's motion sensors form a unique "fingerprint". Apps and websites could recognise a device from them ("SensorID"). Apple fixed it in iOS 12.2.
- On an iPhone, websites must ask before reading the motion sensors. Apps don't.

**Final report entry:** "Steadiest hand: you held the painting within ±0.8°."

## If the device can't do it

- Every phone has an accelerometer.
- **Shaky hands or a tremor** (accessibility): after 60 s, widen the tolerance to 4° and shorten the hold to 1 s.
- **The player is lying down** (the phone isn't upright): detect it and show "Sit up and hold your phone in front of you".

## Reverting

Nothing to revert.
