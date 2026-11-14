# Level 03 · The Overflowing Sink

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Gyroscope (rotation), plus the accelerometer to check the phone lies flat |
| **Permission** | None (Android and iOS) |
| **Support** | Android ✅ (no gyroscope on some budget phones) · iOS ✅ |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟢 Easy |
| **Setting to revert** | None |
| **Reference** | [research.md §4.3](../research.md#43-gyroscope-accelerometer-orientation) |

## Scene

A cartoon kitchen seen from above. An old brass tap is gushing water into a sink that's already full, and water spills over the edge onto the tiled floor. A rubber duck rides the overflow, a sponge floats away, and the cat sits on the counter looking horrified as the puddle spreads towards it.

The tap has a big cross-shaped handle seen from directly above, and it fills most of the screen. A water-level gauge on the side rises slowly, adding pressure.

- **Start instruction:** "Put your phone down flat on a table, screen up."
- **Live feedback:** the handle on screen turns *with* the phone. Each quarter turn clockwise visibly slows the water. Squeaky metal sounds play while it turns.
- **Wrong direction:** turning counter-clockwise makes the water gush harder, and a speech bubble from the cat says: "Lefty-loosey!"
- **Lifting the phone:** the handle "slips" and the progress resets, with a "Keep it on the table!" message.
- **Success:** after a full turn, the water stops with a last *drip… drip… plop*. The duck lands in the sink and the cat breathes out.

## Goal

With the phone **flat on a table**, rotate it **one full turn (360°) clockwise**.

**Success condition:**
- The phone stays flat the whole time: the size of gravity's z component is greater than 0.9 g.
- The rotation around the z axis, added up over time, reaches −2π (a full clockwise turn).
- Small counter-clockwise movements subtract from the total; they don't reset it.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟢 `sensors_plus` (gyroscope + accelerometer) | 🟢 `sensors_plus` (gyroscope + accelerometer) |
| **Overall** | **🟢 Easy** | **🟢 Easy** |

Adding up the rotation over time is level logic, so it isn't counted. Some budget Android phones have no gyroscope: that's missing hardware, not an access problem.

## Technical documentation

**Android**
- `Sensor.TYPE_GYROSCOPE` gives the rotation speed in rad/s. Add up `z × dt` over time to get the total turn.
- `TYPE_GRAVITY` (or the accelerometer with a low-pass filter) tells you the phone is flat: flat and face up, z ≈ +9.81 m/s².
- Alternative: `TYPE_ROTATION_VECTOR` or `TYPE_GAME_ROTATION_VECTOR`. Convert with `getRotationMatrixFromVector` and `getOrientation` for the yaw angle, and track it across the ±180° jump.

**iOS**
- `CMMotionManager.startDeviceMotionUpdates()` gives `CMDeviceMotion.rotationRate.z` (rad/s) and `.gravity`. Flat and face up, `gravity.z` ≈ −1.
- No permission is needed for `CMMotionManager`.

**Flutter**
- `sensors_plus`: `gyroscopeEventStream()` and `accelerometerEventStream()`. Check the sign conventions on both platforms with the debug overlay.

**Notes**
- Direction: by the right-hand rule, turning clockwise when seen from above gives a *negative* z rotation rate **(verify on both platforms)**.
- Adding up the gyroscope drifts only slightly over the few seconds this takes, so no correction is needed.

## Clues

1. **Vague** (the cat): "Taps don't close themselves… someone has to turn it."
2. **Closer:** "What if your phone *was* the tap handle? Put it down."
3. **Specific:** "Put your phone flat on the table, screen up, and turn it one full circle clockwise."

**(?) Solution after 2 minutes:** lay the phone flat on a table, screen up. Turn it clockwise, the same way the hands of a clock move, like turning a big dial, until it has gone all the way around. You can do it in several small turns; just don't lift it.

## After the level

**Card:** "Your phone has a gyroscope, an accelerometer and a magnetometer. Apps can use all of them without asking."

**What you learned:** apps can tell exactly how your phone moves and turns, many times per second, with no permission prompt on Android or iOS.

**Trivia:**
- In 2014, Stanford researchers (the "Gyrophone" study) showed a phone's gyroscope picks up enough vibration from nearby speech to recognise some spoken words, without microphone permission.
- Partly because of attacks like that, Android 12 limits apps to 200 readings per second unless they declare a special permission.
- On an iPhone, *websites* must ask before reading the motion sensors. *Apps* don't have to.

**Final report entry:** "Motion sensors on your phone: gyroscope, accelerometer, magnetometer, barometer…". On Android, `SensorManager.getSensorList(Sensor.TYPE_ALL)` even gives each sensor's model and manufacturer, which is a fingerprinting detail on its own.

## If the device can't do it

- **No gyroscope** (some budget Android phones): use the compass heading (magnetometer) to measure the turn, with smoothing.
- **No gyroscope and no usable compass:** the player drags the handle around with a finger. The level still passes, and the card explains which sensor was missing.
- **No table nearby:** holding the phone flat in the hand works too; the flatness check allows some tilt.

## Reverting

Nothing to revert.
