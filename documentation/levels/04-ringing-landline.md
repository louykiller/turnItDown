# Level 04 · Hang Up That Phone

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Proximity sensor + accelerometer (face-down detection) |
| **Permission** | None (Android and iOS) |
| **Support** | Android ✅ (proximity only works during calls on some Samsung/Xiaomi phones) · iOS ✅ (iPhone only; iPads have no proximity sensor) |
| **Access difficulty: Android** | 🟡 Moderate |
| **Access difficulty: iOS** | 🟡 Moderate |
| **Setting to revert** | None |
| **Reference** | [research.md §4.4](../research.md#44-proximity) and [§4.3](../research.md#43-gyroscope-accelerometer-orientation) |

## Scene

A 1980s-style hallway with patterned wallpaper. On a small wooden side table sits a bright red rotary phone. Its handset is *off the hook*, lying next to the base with the curly cord tangled. The phone is ringing loudly, *RRRRING*, and the handset jumps a little with every ring.

A grandma character in slippers stands nearby with her fingers in her ears. A picture of a cat on the wall swings with each ring.

- **Live feedback:** every ring also makes the real phone vibrate (haptics), so the player feels the ringing.
- **Start:** the ringing gets louder and faster the longer it goes on.
- **Success:** once the player's phone is face down, the ringing stops with a satisfying mechanical *CLACK* as the handset drops onto its cradle. This is played as a sound, because the player can't see the screen. When they turn the phone back over, the handset is on its base, grandma takes her fingers out of her ears, and a little heart floats up.

## Goal

Put the phone **face down on a table**, like hanging up a handset.

**Success condition** (hold for 1.5 s):
- Face down: on Android, gravity z ≈ −9.81 m/s². On iOS, `gravity.z` ≈ +1.
- **And** the proximity sensor reports "near", if the device has a working one.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟡 About 20–30 lines of native code for proximity (I couldn't confirm a maintained package). `sensors_plus` for face down | 🟡 About 20 lines of native code (`isProximityMonitoringEnabled` + notification). `sensors_plus` for face down |
| **Overall** | **🟡 Moderate** | **🟡 Moderate** |

Android's proximity readings are unreliable on some Samsung/Xiaomi phones. On iOS, the screen turns off while the sensor is covered.

## Technical documentation

**Proximity: Android**
- `Sensor.TYPE_PROXIMITY` (cm). It's often binary: 0 for near, or the maximum range for far.
- On some Samsung (and reportedly Xiaomi) phones, it only reports during calls. These are user reports, not documented by the manufacturers.

**Proximity: iOS**
- Set `UIDevice.current.isProximityMonitoringEnabled = true`, then read `proximityState` and listen for `UIDevice.proximityStateDidChangeNotification`.
- **The screen turns off while the sensor is covered.** That's why the success feedback here is a sound.
- iPads have no proximity sensor.
- **Turn monitoring off as soon as the level ends.** Otherwise the screen keeps going black whenever something covers the top of the phone.

**Face down (accelerometer)**
- Android: `TYPE_GRAVITY`, or the accelerometer with a low-pass filter.
- iOS: `CMDeviceMotion.gravity`.
- Flutter: `sensors_plus` `accelerometerEventStream()`.

**Flutter (proximity):** `proximity_sensor` (check that it's still maintained), or the in-house plugin.

## Clues

1. **Vague** (grandma): "Somebody hang up that phone, I can't hear myself think!"
2. **Closer:** "How do you hang up an old phone? You put it down… the right way round."
3. **Specific:** "Put your phone screen-down on a table."

**(?) Solution after 2 minutes:** place the phone on a flat surface with the screen facing down and leave it there for 2 seconds. You'll hear a *clack* when it's hung up. Then turn it back over.

## After the level

**Card:** "Your phone has a proximity sensor. No permission needed."

**What you learned:** without asking, apps can tell whether your phone is face down on a table, in your pocket or held against your ear. That reveals whether you're actually using it.

**Trivia:**
- The proximity sensor is what turns your screen off during calls, so your cheek doesn't press buttons.
- Some Pixel phones have a "Flip to Shhh" feature that silences notifications when you put the phone face down. It uses the same signals as this level.
- Many proximity sensors only report "near" or "far", not an actual distance.

**Final report entry:** "Proximity sensor: present (reports near/far only)".

## If the device can't do it

- **No proximity sensor** (iPads, some tablets), **or one that only works during calls** (some Samsung/Xiaomi phones): accept face down alone, using the accelerometer.
- **No table nearby:** holding the phone face down in a flat palm also works. The palm covers the sensor too.

## Reverting

Nothing to revert. Make sure iOS proximity monitoring has been switched off.
