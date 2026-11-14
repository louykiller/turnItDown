# Level 02 · The Neighbour's Nightmare

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Volume level (and, on Android, the volume button presses themselves) |
| **Permission** | None (Android and iOS) |
| **Support** | Android ✅ · iOS ✅ |
| **Access difficulty: Android** | 🟢 Easy (🟡 with button presses) |
| **Access difficulty: iOS** | 🟢 Easy |
| **Setting to revert** | Volume (back up from mute) |
| **Reference** | [research.md §4.2](../research.md#42-volume) |

## Scene

A small apartment living room. On a shelf sits an enormous retro boombox with two speaker cones pumping in and out, blasting music. The game really plays this music through the phone's speaker, so the in-game song *is* the real sound.

Everything in the room shakes to the beat: picture frames rattle, a plant bounces in its pot, and the cat hides under the sofa with its ears flat. Every few seconds the grumpy neighbour bangs on the wall with a broom handle, and dust falls from the ceiling.

- **Live feedback:** the size of the speaker cones, the floating music notes and the shaking of the room all follow the real volume. Each press of the volume button visibly shrinks the party.
- **Wrong direction:** pressing volume *up* makes the boombox jump, the bass drops, and the neighbour's broom pokes a hole through the wall.
- **Success:** at zero volume the boombox powers down with a "click" and its lights fade. The cat comes out, the broom pulls back, and a little sign appears in the hole in the wall: "THANK YOU".

## Goal

Press the phone's **volume-down button** until the music is off.

**Success condition:**
- Volume at 0, or at most 1 step, held for 1 s.
- Android extra: the game counts the physical volume-down key presses, so it can congratulate the player for using the button rather than the on-screen slider.

**Precondition:** the volume must be at least 30% when the level starts. If it's lower, the party "starts quietly" and the game asks the player to turn it up first.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟢 `flutter_volume_controller` reads the volume and every change. Catching the button press itself: 🟡 a few lines of native code | 🟢 `flutter_volume_controller`. If the iOS 18 bug is confirmed, add a small polling fallback |
| **Overall** | **🟢 Easy (🟡 with button presses)** | **🟢 Easy** |

## Technical documentation

**Android**
- Read `AudioManager.getStreamVolume(STREAM_MUSIC)` and `getStreamMaxVolume(STREAM_MUSIC)`. The number of steps varies: 15 on Pixel phones, more on some other brands.
- Watch for changes with a `ContentObserver` on `Settings.System.CONTENT_URI`, or the unofficial `android.media.VOLUME_CHANGED_ACTION` broadcast.
- **While the game is in the foreground, it receives the button presses themselves:** `KEYCODE_VOLUME_DOWN` and `KEYCODE_VOLUME_UP` in `Activity.dispatchKeyEvent`. It could even consume them so the volume doesn't change.
- No permission is needed. `setStreamVolume` also needs none.

**iOS**
- `AVAudioSession.sharedInstance().outputVolume` (0–1, in 16 steps of 0.0625), observed with KVO.
- The volume buttons only change the *media* volume while the app has an active audio session (category `.playback`) or is playing sound. Otherwise they change the ringer volume. This level always plays music, so that's covered.
- There is no public API to catch the button press itself. Pressing the button and dragging the Control Center slider look the same to the app.
- KVO doesn't fire if the volume is already at 0 or 100%.
- Apple's forums have a report that `outputVolume` misbehaves on iOS 18+. **Test this on current iOS** (see [research.md §9](../research.md#9-things-to-test-on-real-devices-first)).

**Flutter**
- `flutter_volume_controller`: `getVolume()`, `addListener(..., emitOnStart: true)`, and a choice of Android audio stream. Volume doesn't work in the iOS simulator.
- Catching the Android key presses needs the in-house plugin.

## Clues

1. **Vague** (the neighbour, through the wall): "Some of us have work in the morning!"
2. **Closer:** "There's no off switch on that speaker. Look at the sides of your phone."
3. **Specific:** "Press your phone's volume-down button until the music stops."

**(?) Solution after 2 minutes:** press the volume-down button on the side of the phone repeatedly until the volume bar is empty.
- **Android:** it's usually the lower of the two buttons on the right side.
- **iPhone:** it's the lower of the two buttons on the left side.

## After the level

**Card:** "Media volume: 50% (8 of 16 steps) when you started. No permission needed."

**What you learned:** apps can read your volume level and see every change you make, without asking. Android apps can even see the button press itself, and can stop it from changing the volume (camera apps use this to take photos with the volume button). iPhone apps only see the result.

**Trivia:**
- Video ad tools report the device volume to advertisers, so they know whether their ad could actually be heard.
- An iPhone has 16 volume steps. Android phones usually have 15 for media, but some brands have more.
- On an iPhone, when nothing is playing, the volume buttons change the *ringer* volume, not the media volume.

**Final report entry:** "Media volume: 50%".

## If the device can't do it

- Every phone supports this level.
- **iOS `outputVolume` stops reporting changes** (the iOS 18 regression): poll `outputVolume` every 200 ms instead of relying on KVO.
- **The player can't press the hardware buttons** (accessibility): also accept the volume slider in Control Center or Quick Settings. iOS can't tell the difference anyway.
- **Do Not Disturb or silent mode:** the media volume still works, so no change is needed.

## Reverting

The neighbour knocks politely: "Thanks! But you'll want to hear the next level…"
- **Player action:** turn the volume back up to about half.
- **Check:** volume between 30% and 70%, or within one step of the value at the start.

The next few levels use sound, and level 5 relies on volume changes, so this revert matters.
