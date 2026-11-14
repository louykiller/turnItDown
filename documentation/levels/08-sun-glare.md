# Level 08 · Sun in Your Eyes

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Ambient light sensor (Android). Front camera brightness estimate (iOS) |
| **Permission** | Android: none. iOS: camera prompt (`NSCameraUsageDescription`), asked during setup, needed only because of the workaround |
| **Support** | Android ✅ · iOS ⚠️ (no light-sensor API, so the camera is used instead) |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟠 Hard |
| **Setting to revert** | None |
| **Reference** | [research.md §4.7](../research.md#47-ambient-light) |

## Scene

Inside a little cartoon car driving along a coastal road at sunset. The view is from the driver's seat: the dashboard with a wobbly bobblehead, the steering wheel, and fluffy dice hanging from the mirror. The sun sits low, right in the middle of the windscreen, enormous and blinding. Lens flares streak across the screen and the road ahead fades into white.

The driver squints with tears in their eyes, and the car swerves a little. The sun visor is broken and hangs uselessly by one hinge.

- **Precondition:** the room must be reasonably bright (about 100 lux or more). Otherwise there's no light to block.
- **Live feedback:** the glare follows the real light level compared to the start. As the player's hand covers the top of the phone, the driver's cartoon hand rises in front of the sun and the glare fades.
- **Hidden danger:** as the glare fades, the road appears, and there's a cow standing right in the middle of it!
- **Success:** with the sun blocked, the driver spots the cow, brakes with a screech and stops just in time. The cow chews calmly, says "Moo", and wanders off. The driver wipes their forehead.

## Goal

**Cover the top edge of the phone** with a hand, so the light sensor (or the front camera on iPhone) sees much less light.

**Success condition** (hold for 1 s):
- Android: measured light below 20% of the starting value.
- iOS: camera brightness drops by at least 3 EV (an exposure value step; each step halves the light).

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟡 Camera prompt, only because iOS gives apps no light sensor |
| **Code to get the value** | 🟢 A light-sensor package, or about 20 lines of native code | 🟠 Native camera code that reads each frame's exposure data. No Flutter package does this |
| **Overall** | **🟢 Easy** | **🟠 Hard** |

## Technical documentation

**Android**
- `Sensor.TYPE_LIGHT`, in lux. No permission is needed.
- Typical values: dark room under 10, living room 50–200, office 300–500, daylight 10,000+.
- The sensor is near the top of the phone, sometimes under the screen.
- Flutter: `light` or `light_sensor` (check that they're still maintained), or the in-house plugin.

**iOS**
- **There's no public API for the light sensor.** SensorKit has ambient light, but only for research studies Apple approves, and its data arrives 24 hours late.
- **Workaround:** run the front camera in an `AVCaptureSession` with `AVCaptureVideoDataOutput`. For each frame, read the EXIF brightness value from the sample buffer: `CMCopyDictionaryOfAttachments` → `kCGImagePropertyExifDictionary` → `kCGImagePropertyExifBrightnessValue`. It's in APEX units, so −1 means half the light.
- Alternative: estimate from `AVCaptureDevice.exposureDuration`, `iso` and the lens aperture.
- Needs `NSCameraUsageDescription`.
- The Flutter `camera` package doesn't give per-frame EXIF data, so this goes in the in-house plugin.
- Covering the top of the iPhone also covers the front camera, so the level works the same way.

**Gotchas**
- With auto-brightness on, covering the sensor also dims the screen. That's harmless, but warn the player so they aren't surprised.
- Measure against the level-start baseline, never a fixed number of lux.

## Clues

1. **Vague** (the driver, squinting): "I can't see a thing! If only something could block that sun…"
2. **Closer:** "Your phone has a little eye at the top that measures light."
3. **Specific:** "Cover the top of your phone, above the screen, with your hand."

**(?) Solution after 2 minutes:** put your palm over the top edge of the phone's screen, where the front camera is, and keep it there for a second, like shading your eyes from the sun.

## After the level

**Android card:** "Light around you: 320 lux, about what an office has. No permission needed."

**iPhone card:** "iPhones don't let apps read the light sensor, so this game used your front camera to estimate the light (with your permission)."

**What you learned:** Android apps can measure the light around you at any time, without asking. Light reveals a lot: whether you're outdoors in the sun, in an office, or in bed in the dark.

**Trivia:**
- In 2024, MIT researchers showed that a screen's ambient light sensor can be used to capture rough images of hand gestures in front of the screen.
- Web browsers restricted their version of the light sensor because of privacy attacks like this.
- On iPhone, the green dot at the top of the screen appeared because the camera was on. Did you notice it?

**Final report entry:** "Light around you: 320 lux (typical office)." On iOS: "Estimated from the camera."

## If the device can't do it

- **iPhone camera permission refused:** use the proximity sensor instead. Covering the top of the phone also covers it. On iPhone the screen briefly turns off while it's covered, so show the success when the hand comes away.
- **Android phone without a light sensor** (rare): use the front camera like iOS, or the proximity sensor.
- **Dark room at the start** (under about 100 lux): ask the player to turn on a light, or switch straight to the proximity version.

## Reverting

Nothing to revert. Auto-brightness returns to normal by itself when the hand comes away.
