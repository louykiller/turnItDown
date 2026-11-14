# Level 07 · Point the Remote

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Accelerometer (gravity direction, i.e. tilt) |
| **Permission** | None (Android and iOS) |
| **Support** | Android ✅ · iOS ✅ |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟢 Easy |
| **Setting to revert** | None |
| **Reference** | [research.md §4.3](../research.md#43-gyroscope-accelerometer-orientation) |

## Scene

A cosy living room at night, seen in first person from the sofa. Snack wrappers and a bowl of popcorn lie on the coffee table, and a grandpa character snores in the armchair next to you. Across the room an old TV is blasting a shouty infomercial at full volume ("BUT WAIT, THERE'S MORE!"), its picture flickering.

The player's hand holds a chunky TV remote at the bottom of the screen. Your phone is the remote, and right now it's pointing at the ceiling. A little red infrared dot shows where the remote points: on the ceiling, among the cobwebs and the lamp.

- **Calibration (start of level):** "Hold your phone like a remote, pointing up at the ceiling." The game records this pose as its starting point.
- **Live feedback:** as the player tips the top of the phone forward, the red dot slides down from the ceiling, across the wall and the picture frames, towards the TV. A faint beam shows the path.
- **Wrong direction:** tipping backwards sends the dot over the player's own head; the grandpa mumbles "Wrong way, kiddo" in his sleep.
- **Success:** when the dot sits on the TV for 1 s, the remote's power button clicks. The TV switches off with the old tube effect (the picture shrinks to a white dot and fades). Silence. Grandpa snores peacefully.

## Goal

Tip the phone **about 90° forward**, from pointing at the ceiling to pointing at the TV in front of you.

**Success condition:**
- At calibration, record the gravity vector g₀.
- Success when the angle between the current gravity vector g and g₀ is at least 70°, the direction is forward (the top edge goes down towards the TV), and it holds for 1 s.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟢 `sensors_plus` | 🟢 `sensors_plus` |
| **Overall** | **🟢 Easy** | **🟢 Easy** |

The angle maths (a dot product) is level logic, so it isn't counted.

## Technical documentation

**Android**
- `Sensor.TYPE_GRAVITY`, which is already filtered. Alternatively, `TYPE_ACCELEROMETER` with a low-pass filter (for example `g = 0.9·g + 0.1·raw`).
- When the phone is still, gravity tells you exactly which way it is tilted.

**iOS**
- `CMMotionManager.startDeviceMotionUpdates()` → `CMDeviceMotion.gravity`. No permission is needed.

**Flutter**
- `sensors_plus` `accelerometerEventStream()`, with a low-pass filter to get gravity.

**Maths**
- `angle = acos( (g · g₀) / (|g| · |g₀|) )`.
- Use the sign of the change in the phone's y/z axes to tell forward from backward.

**Gotchas**
- This is tilt, so it uses the accelerometer, not the gyroscope.
- Calibrate every time, because people hold "pointing at the ceiling" differently.

## Clues

1. **Vague** (grandpa, half asleep): "Turn that racket off, would you…"
2. **Closer:** "You're pointing the remote at the ceiling. The TV isn't up there."
3. **Specific:** "Tip the top of your phone forward until it points straight ahead at the TV."

**(?) Solution after 2 minutes:** hold the phone upright with its top pointing at the ceiling. Then slowly tip the top forward and down until the phone points straight ahead, about a quarter turn, as if aiming the remote at the TV. Hold it there for a second.

## After the level

**Card:** "Your phone knew its exact angle the whole time. No permission needed."

**What you learned:** the accelerometer tells every app how you're holding your phone: upright, flat, tilted, or upside down. It updates many times per second.

**Trivia:**
- Researchers have shown that the tiny tilts your phone makes as you tap can help guess the digits of a PIN, from the motion sensors alone.
- Some Android phones (several Xiaomi models, for example) have a real infrared blaster that *can* switch off a real TV.
- Your phone's screen rotation uses this same sensor.

**Final report entry:** "You usually hold your phone tilted at about 35°." Compute an average angle over the whole game.

## If the device can't do it

- Every phone has an accelerometer, so no fallback is needed.
- **Calibration went wrong** (for example, the player was holding the phone flat): add a "Recalibrate" button after 30 s.
- **Limited arm movement** (accessibility): after 60 s, lower the required angle from 70° to 45°.

## Reverting

Nothing to revert.
