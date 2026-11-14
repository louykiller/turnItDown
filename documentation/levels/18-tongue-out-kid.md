# Level 18 · Copycat

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Front camera + face expression recognition |
| **Permission** | Camera prompt on Android (`CAMERA`) and iOS (`NSCameraUsageDescription`), asked during setup |
| **Support** | iOS ✅ (iPhone XS / XR and newer, A12 chip or later) · Android ⚠️ (no ready-made tongue detector; **decision needed**, see below) |
| **Access difficulty: Android** | 🔴 Very hard (tongue) · 🟠 Hard (other expressions) |
| **Access difficulty: iOS** | 🟠 Hard |
| **Setting to revert** | None |
| **Reference** | [research.md §4.16](../research.md#416-face-expression-tongue-out) |

## Scene

A school playground at break time, with a climbing frame, a sandpit and a skipping rope. A cheeky kid with a backwards cap stands on a bench, thumbs in his ears, fingers wiggling, tongue out, going "Nyeh nyeh nyeh!" He switches between faces to wind the player up.

In the corner of the screen is a round "mirror" showing **a cartoon avatar that copies the player's real face** live: blinking, smiling and opening its mouth when the player does, using the face-tracking data. Showing an avatar instead of the raw camera picture feels less invasive and fits the cartoon style.

- **Live feedback:** the avatar mirrors the player's expression. On iOS, the avatar's tongue slides out as the player's does, so they can see they're close.
- **Escalation:** the kid gets more annoying with time. He blows a raspberry, his friends join in, and one throws a paper ball.
- **Success:** when the player sticks their tongue out for half a second, the kid freezes in shock, his cap falls off, and he bursts into laughter. The two become friends and run off to play football together.

## Goal

**Look at the phone and stick your tongue out.**

**Success condition:** iOS: `ARFaceAnchor.blendShapes[.tongueOut] ≥ 0.5` for 0.5 s. For Android, see the decision below.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟡 Camera prompt | 🟡 Camera prompt |
| **Code to get the value** | 🔴 A tongue needs a custom ML model you train yourself · 🟠 other expressions through MediaPipe in native code | 🟠 A native ARKit session (or the community `arkit_plugin`). ARKit gives `tongueOut` directly |
| **Overall** | **🔴 Very hard (tongue) · 🟠 Hard (other expressions)** | **🟠 Hard** |

## Technical documentation

**iOS (ARKit)**
- Check `ARFaceTrackingConfiguration.isSupported`. Since iOS 14 it works on any device with a front camera and an A12 chip or newer, even without the Face ID camera. Before iOS 14, it needed the Face ID (TrueDepth) camera.
- Run an `ARSession` with this configuration. Each frame updates `ARFaceAnchor.blendShapes`: 52 values from 0 to 1, including `.tongueOut`, `.jawOpen`, `.cheekPuff`, `.eyeBlinkLeft` and more.
- Permission: `NSCameraUsageDescription`.
- Flutter: `arkit_plugin` (community), or the in-house plugin forwarding the values you need.

**Android: what's available**
- **ML Kit Face Detection** (`google_mlkit_face_detection`): `smilingProbability`, `leftEyeOpenProbability`, `rightEyeOpenProbability` and contours. **No tongue.**
- **MediaPipe Face Landmarker:** 52 blendshape values named like ARKit's, but **`tongueOut` is not one of them**. Index 0 is `_neutral` (confirmed in MediaPipe's source, `face_blendshapes_graph.cc`). There's no official Flutter plugin, so call it from native code.
- Permission: `CAMERA`.

**Android: decision needed** (pick one)
1. **Use a different expression on Android (recommended for version 1).** The kid pulls a face that MediaPipe can detect: puffed cheeks (`cheekPuff`), mouth wide open (`jawOpen`), a wink (`eyeBlinkLeft` vs `eyeBlinkRight`) or raised eyebrows (`browInnerUp`). The scene stays the same, only the expression changes.
2. **Train a small custom model:** a TFLite image classifier on the mouth area (cropped using Face Landmarker points), trained with Teachable Machine or MediaPipe Model Maker. More work, plus collecting training photos.
3. **Rough heuristic:** `jawOpen` high, plus red/pink pixels below the lower lip. Unreliable across lighting and skin tones.

## Clues

1. **Vague** (the kid): "Nyeh nyeh! You can't do it!"
2. **Closer:** "Fight fire with fire. Copy his face!"
3. **Specific:** "Look at your phone and stick your tongue out, all the way." (Android with option 1: "Puff out your cheeks like a balloon!")

**(?) Solution after 2 minutes:** hold the phone in front of your face with your face in the mirror circle, then stick your tongue out as far as you can and hold it for a second.

## After the level

**Card:** "Camera: allowed. Your face was analysed 412 times during this level. Nothing was saved or sent."

**What you learned:** once an app has the camera, modern face tracking can measure dozens of separate facial movements many times per second: your smile, your eyebrows, where you look, even your tongue.

**Trivia:**
- Apple's App Store rules forbid using face-tracking data for advertising or marketing. That rule exists because the data is so revealing.
- A green dot at the top of the screen shows the camera is on. Android 12+ also has a Quick Settings toggle that blocks the camera for every app at once.
- ARKit's 52 face values are the same ones animated emoji (Memoji) use.

**Final report entry:** "Camera: allowed. Expressions detected: smile, wink, tongue out."

## If the device can't do it

- **Camera permission refused:** skip, with a positive card: "You kept your camera private. Good instinct." Offer a button to reconsider.
- **iPhone older than the XS/XR (A11 or earlier):** use Apple's Vision framework (`VNDetectFaceLandmarksRequest`) to detect a wide-open mouth instead of a tongue.
- **Android:** see the decision above. Option 1 means it works on almost every phone.
- **No front camera** (rare): mark the level as not supported.

## Reverting

Nothing to revert. Make sure the camera session stops as soon as the level ends, so the green dot disappears.
