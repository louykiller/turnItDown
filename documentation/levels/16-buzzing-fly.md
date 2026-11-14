# Level 16 · Swat the Fly

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Compass (magnetometer) combined with the gyroscope, for a smooth heading |
| **Permission** | None (Android and iOS) |
| **Support** | Android ✅ · iOS ✅ |
| **Access difficulty: Android** | 🟡 Moderate |
| **Access difficulty: iOS** | 🟡 Moderate |
| **Setting to revert** | None |
| **Reference** | [research.md §4.14](../research.md#414-compass) and [§4.3](../research.md#43-gyroscope-accelerometer-orientation) |

## Scene

A sunny kitchen seen in first person, drawn as a wide 360° panorama around the player: a fridge covered in magnets, a window with curtains, a fruit bowl, a hanging lamp, a door, a cooker. The phone acts as a window into this room. As the player turns their body, the view pans to show what's in that direction.

A fat, cheeky fly zooms around the room with a loud *BZZZZZ*. It lands for a moment on the banana, the window or the lamp, rubs its hands together, then takes off again. It often flies out of view.

- **Live feedback:** the buzz is stereo. It's louder in the left or right ear depending on where the fly is compared to where the phone points, so players can follow it by ear. A rolled-up newspaper waits at the bottom of the screen.
- **Hint arrow:** after the second clue, a small arrow at the edge of the screen points towards the fly.
- **Missed swat:** the newspaper slaps the wall, the fly blows a raspberry and zooms away.
- **Success:** tapping the fly while it's visible (best while it has landed) lands the newspaper with a *THWACK*. The fly spins with cartoon stars around its head, then staggers out of the open window. Silence, apart from a bird singing outside.

**Compass twist (optional):** the fly always escapes towards a compass direction ("It flew NORTH!"). An on-screen compass helps players use real directions.

## Goal

**Turn your body (and the phone) to find the fly, then tap it.**

**Success condition:**
- The fly has a world direction (bearing) that changes over time.
- Its screen position: x = (fly bearing − phone heading) mapped across the view, with a field of view of about 60°.
- A tap within the fly's hitbox while it's on screen succeeds.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟡 About 30 lines of native code forwarding the rotation vector (a smooth heading). A raw heading from `flutter_compass` is 🟢 but too jittery here | 🟡 About 30 lines of native code forwarding device motion referenced to north. A raw heading from `flutter_compass` is 🟢 but too jittery here |
| **Overall** | **🟡 Moderate** | **🟡 Moderate** |

## Technical documentation

**Android**
- `Sensor.TYPE_ROTATION_VECTOR` combines the gyroscope, accelerometer and magnetometer, so its direction is measured from north. Convert with `SensorManager.getRotationMatrixFromVector` and `getOrientation` → azimuth.
- Watch `onAccuracyChanged`. When accuracy is low, ask the player to wave the phone in a figure-8 to calibrate the compass.
- No permission is needed.

**iOS**
- `CMMotionManager.startDeviceMotionUpdates(using: .xMagneticNorthZVertical)` gives a smooth direction relative to magnetic north from `CMDeviceMotion.attitude`. No permission is needed.
- Alternative: `CLLocationManager.startUpdatingHeading()` → `magneticHeading`. Its `trueHeading` needs location permission.

**Flutter**
- `flutter_compass` for a simple heading stream.
- `sensors_plus` `magnetometerEventStream()` for raw µT values.
- For the smooth panorama, use the in-house plugin forwarding the combined orientation.

**Gotchas**
- A raw magnetometer heading jitters and is disturbed by metal and magnets (magnetic phone cases, fridge magnets, laptop speakers). The gyroscope smooths this out.
- The panorama has to wrap around at 360°.

## Clues

1. **Vague:** "Bzzzz… where did it go?"
2. **Closer:** "Listen: the buzz is louder on the side where the fly is. Turn towards it." (The arrow hint appears.)
3. **Specific:** "Hold your phone up in front of you like a camera and slowly turn around until the fly is on screen, then tap it."

**(?) Solution after 2 minutes:** hold the phone upright in front of you and turn your whole body slowly in a circle. Follow the arrow until the fly appears, wait for it to land, and tap it.

## After the level

**Card:** "You were facing north-west when this level started. Your phone knew, without location permission."

**What you learned:** the compass and motion sensors tell apps which way you're facing, at every moment, without asking.

**Trivia:**
- Combining direction (compass) with steps (motion sensors) lets apps estimate the path you walk indoors, without GPS. It's called "pedestrian dead reckoning".
- If your compass seems wrong, a magnetic phone case or a nearby speaker could be the culprit. Waving the phone in a figure-8 helps it recalibrate.

**Final report entry:** "Direction you faced at the start: north-west."

## If the device can't do it

- **No magnetometer, or strong interference:** use gyroscope-only orientation (Android `TYPE_GAME_ROTATION_VECTOR`; iOS `.xArbitraryZVertical`). Directions are then relative to where the player started instead of north. The level still works; only the north twist is lost.
- **No gyroscope** (some budget Android phones): use the compass alone with stronger smoothing.
- **Neither works well, or the player can't turn around** (seated, limited mobility): let them swipe to look around the room.

## Reverting

Nothing to revert.
