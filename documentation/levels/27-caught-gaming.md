# Level 27 · 3 A.M. Gamer

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Screen lock (screen turned off) |
| **Permission** | None (Android and iOS) |
| **Support** | Android ✅ · iOS ⚠️ (no "device locked" event; inferred from the app going to the background) |
| **Access difficulty: Android** | 🟡 Moderate |
| **Access difficulty: iOS** | 🟠 Hard |
| **Setting to revert** | None |
| **Reference** | [research.md §4.26](../research.md#426-screen-lock) |

## Scene

A dark bedroom. A digital clock on the bedside table glows **03:00**. Under the duvet there's a lump, and a blue glow: the player's character, wide awake, playing a game on a tablet. Their face is lit blue, surrounded by empty crisp packets and an energy drink. Tiny game sounds play: *pew pew, level up!*

Suddenly the hallway light switches on, and a strip of light appears under the bedroom door. Footsteps: *thump… thump… thump.* The door handle starts to turn. **Mum is coming!** A 10-second countdown is shown as the door creeping open, millimetre by millimetre.

- **Escalation:** the footsteps get louder, a shadow appears in the door gap, and the character's eyes go wide.
- **Too slow:** if the countdown runs out, Mum flicks on the light: "**GROUNDED!**" The character's soul leaves their body (cartoon ghost), and the level restarts.
- **Success:** the moment the player locks their phone, the in-game tablet goes black. When the player unlocks and comes back, they see what happened: Mum peeks in, sees a peacefully "sleeping" child fake-snoring with a suspicious smile, whispers "Aww, sleeping like an angel", and closes the door softly. Under the duvet, a thumbs-up.

## Goal

**Lock the phone** (press the side/power button), then unlock it and come back to the game.

**Success condition:**
- Android: the screen-off event arrives during the countdown, then the player returns.
- iOS (heuristic): the app left the foreground during the countdown and stayed away for at least 2 s. If the phone has a passcode, the "protected data became unavailable" signal confirms it was a lock and not a trip to the home screen. Then the player returns.

**Important:** the game keeps the screen awake during all levels (`wakelock_plus`). Otherwise a short auto-lock (30 s) could lock the phone and complete this level by accident.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟡 A small native receiver for the screen-off event | 🟠 No lock event. Native code combines several signals into a guess that can't always tell "locked" from "went to the home screen" |
| **Overall** | **🟡 Moderate** | **🟠 Hard** |

## Technical documentation

**Android**
- A `BroadcastReceiver` registered **at runtime** (it can't be declared in the manifest) for `Intent.ACTION_SCREEN_OFF`, plus `Intent.ACTION_USER_PRESENT` for the unlock.
- Also available: `KeyguardManager.isKeyguardLocked()`, `isDeviceLocked()` and `PowerManager.isInteractive()`.
- No permission is needed.

**iOS**
- **There's no public "device locked" event.** The app is only told it went to the background, which also happens when the player goes to the home screen or switches app.
- Signals to combine:
  - `UIApplication.protectedDataWillBecomeUnavailableNotification`, and `UIApplication.shared.isProtectedDataAvailable == false` on return. These only happen if the phone has a passcode, and developers report the timing isn't guaranteed.
  - How long the app spent in the background.
- The Darwin notification `com.apple.springboard.lockcomplete` is private API and gets apps rejected. **Don't use it.**

**Flutter**
- `AppLifecycleListener` (`onHide`, `onPause`, `onResume`) for the lifecycle.
- The in-house plugin for Android's screen-off event and iOS's protected-data signal.
- `wakelock_plus` to keep the screen on during levels.

**Test first:** [research.md §9](../research.md#9-things-to-test-on-real-devices-first), item 9.

## Clues

1. **Vague** (the character, whispering in panic): "She can't see the screen glowing!"
2. **Closer:** "The tablet can't hide itself… but your phone can go dark."
3. **Specific:** "Press your phone's side (power) button to lock the screen, then unlock it."

**(?) Solution after 2 minutes:** press the side button (iPhone) or the power button (Android) once to lock the screen. Wait a second, then unlock your phone and come back to the game.

## After the level

**Card (Android):** "Your phone told this game the moment your screen turned off. No permission needed."
**Card (iPhone):** "Your iPhone only told this game that you *left*. It doesn't tell apps when you lock your phone."

**What you learned:** Android apps can know exactly when your screen turns on and off. iPhones are stricter: apps only know when you leave them.

**Trivia:**
- Research has estimated people's sleep schedules just from when their phone screen turns on and off.
- Android's Digital Wellbeing and iPhone's Screen Time count every unlock. Apps with Android's special "usage access" permission can read similar statistics.

**Final report entry:** "Times you left this game: 14." Count lifecycle pauses across the whole game.

## If the device can't do it

- **iPhone with no passcode:** the protected-data signal never fires, so rely on time in the background (≥ 2 s). The card explains the difference.
- **The player can't press the side button** (accessibility):
  - iPhone: AssistiveTouch has a **Lock Screen** action.
  - Android: the Accessibility Menu has a **Lock screen** button.
- **The player went to the home screen instead of locking:**
  - Android: no screen-off event, so Mum catches them. Retry with the third clue.
  - iPhone: it can't be told apart, so it passes.

## Reverting

Nothing to revert. The next morning, the clock shows 07:00 and the character is a zombie at breakfast. Make sure the screen-awake lock is active again for the next level.
