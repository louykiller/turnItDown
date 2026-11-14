# Level 12 · Searchlight

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Flashlight (torch) state |
| **Permission** | None documented on Android **(verify)**. None on iOS |
| **Support** | Android ✅ · iOS ⚠️ **unverified**: it's unclear whether apps can see a flashlight switched on from Control Center |
| **Access difficulty: Android** | 🟡 Moderate |
| **Access difficulty: iOS** | 🔴 Very hard (unverified) |
| **Setting to revert** | Flashlight off |
| **Reference** | [research.md §4.11](../research.md#411-flashlight) |

## Scene

A prison yard at midnight, almost pitch black. You can just make out the high stone walls topped with barbed wire, a watchtower, and a big searchlight that sputters and won't turn on. Two cartoon guards with tiny torches bump into each other: "Is that you, Gary?" "No, YOU are Gary!"

Somewhere in the darkness, a prisoner in a striped uniform is escaping. All you can see are his two white eyes, blinking. You hear a spoon scraping on stone, then a rope made of knotted bedsheets being lowered over the wall.

- **Live feedback:** as time passes, the eyes move closer to the top of the wall. The scraping and rustling get louder.
- **Success:** the instant the player's flashlight turns on, the tower's searchlight blasts on with a *CLUNK* and sweeps across the yard. It catches the prisoner frozen halfway up the sheet-rope, one leg over the wall. "BUSTED!" He waves sheepishly, the guards cheer, and a dog barks.

## Goal

**Turn on the phone's flashlight.**

**Success condition:** the torch reports as on.
- Android: `onTorchModeChanged(cameraId, enabled = true)`.
- iOS: `isTorchActive == true`, if it can be observed (see below).

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing (no permission is documented; verify on a device) | 🟢 Nothing |
| **Code to get the value** | 🟡 About 20 lines of native code with a torch callback. No package observes the flashlight | 🔴 It may be impossible to observe a flashlight switched on from Control Center (unverified). The fallback, where the game switches it on itself, is 🟢 |
| **Overall** | **🟡 Moderate** | **🔴 Very hard (unverified)** |

## Technical documentation

**Android**
- `CameraManager.registerTorchCallback(callback, handler)` → `TorchCallback.onTorchModeChanged(cameraId, enabled)`.
- It fires when the torch is switched from Quick Settings too (that's how the Quick Settings tile itself stays in sync), and once immediately after registering.
- The documentation doesn't mention needing the `CAMERA` permission **(verify on a device)**.

**iOS**
- `AVCaptureDevice.default(for: .video)?.isTorchActive` (observable with KVO), `torchMode` and `torchLevel`.
- **Reports conflict:** one developer says KVO only fires after the app takes the camera in an `AVCaptureSession`, another says it works without one. **Test this before building the level's art for iOS** (see [research.md §9](../research.md#9-things-to-test-on-real-devices-first)).
- Using the camera isn't a way around it: while an app uses the camera, the Control Center flashlight is unavailable.

**Flutter**
- `torch_light` can only *control* the torch. Observing it needs the in-house plugin.

## Clues

1. **Vague** (a guard): "It's too dark! I can't see a thing out there!"
2. **Closer:** "Your phone has its own little searchlight."
3. **Specific:** "Turn on your phone's flashlight."

**(?) Solution after 2 minutes:**
- **Android:** swipe down from the top of the screen and tap **Flashlight** (or **Torch**).
- **iPhone:** open Control Center and tap the **flashlight** icon.
- **Either:** ask your voice assistant to "turn on the flashlight".

## After the level

**Card (Android):** "This game knew the moment your flashlight came on, without asking."

**What you learned:** apps can tell when your flashlight is on, and they can switch it on themselves. The flashlight is actually the camera's flash LED.

**Trivia:**
- In 2013, the US Federal Trade Commission took action against the popular "Brightest Flashlight Free" app. It was secretly sharing users' locations and device IDs with advertisers. A flashlight app!
- On iPhone, the flashlight is unavailable while an app is using the camera, because they share the same LED.

**Final report entry:** "Flashlight: you turned it on at 21:14."

## If the device can't do it

- **No flash LED** (many iPads, some tablets): mark the level as not supported.
- **iOS can't observe the flashlight** (if the device test fails): switch to an iOS variant where the player taps a switch in the game and **the game turns on the real flashlight itself** (`AVCaptureDevice.lockForConfiguration()` + `torchMode = .on`). The lesson becomes: "Apps can switch on your flashlight without asking."
- **The flashlight is already on at the start:** the start-of-game checks should catch this. If not, ask the player to turn it off first ("the guards need it dark to set the trap").

## Reverting

The prisoner is marched back to his cell, and the guard says: "Thanks! You can switch your light off now. Saves battery."
- **Player action:** turn the flashlight off.
- **Check:** the torch reports off (Android callback). On iOS, whatever detection method works.
