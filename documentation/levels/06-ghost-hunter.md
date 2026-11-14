# Level 06 · Spirit Snapshot

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Screenshot event |
| **Permission** | None on iOS. Android 14+: `DETECT_SCREEN_CAPTURE`, an install-time permission with no prompt |
| **Support** | Android 14+ ✅ (older Android: not supported) · iOS ✅ |
| **Access difficulty: Android** | 🟢 Easy (Android 14+) · 🔴 Very hard (older) |
| **Access difficulty: iOS** | 🟢 Easy |
| **Setting to revert** | None (optional reminder to delete the screenshots) |
| **Reference** | [research.md §4.6](../research.md#46-screenshot-event) |

> Naming note: keep this an original ghost-hunting story. Don't call it "Ghostbusters" or use anything from the films (the logo, proton packs, the theme song). That name is a trademark and app stores reject apps that use it.

## Scene

A dark, dusty cartoon living room in a haunted house, lit by moonlight through tall windows. There's a sofa with a sheet over it, a rocking chair, a tall bookshelf, a portrait of a stern old lady, a candle on the mantelpiece, and a creaky door.

A nervous ghost hunter in a too-big trench coat stands in the corner, clutching an antique wooden box camera with a glowing green lens: the "spirit camera". A film counter in the corner of the screen shows **6 shots**. A corkboard on the right waits for photos.

**The three ghosts.** Each one is invisible and only gives itself away through a clue:

| Ghost | Clue that it's in the room |
|---|---|
| The Rocker | The rocking chair starts rocking by itself |
| The Reader | A book floats off the shelf and flips its pages |
| The Prankster | The candle flickers and the portrait's eyes move |

Each ghost drifts in and out of the room at random intervals of 3–8 s, and stays 2–4 s. Ghostly giggles play from the left or right speaker channel, hinting which ghost is about to arrive.

- **Taking a photo:** the hunter says "My spirit camera's shutter is stuck. Use yours!" and the screen shows the screenshot button combination for the player's phone. When the player takes a screenshot, a Polaroid slides out of the camera and "develops" with a little shake animation, showing the room exactly as it was. Any ghost that was there appears in the photo, caught mid-pose. A photographed ghost shrieks and flees through the wall. The photo is pinned to the corkboard.
- **Missed shot:** a photo with no ghost gets a funny caption ("Lovely picture of a sofa.") and uses up one shot.
- **Out of film:** the hunter sighs and reloads, and the level restarts. The ghosts tease: "Missed meee!"
- **Optional finale:** for the last ghost, all three clues happen at once for one second, as a "group photo" challenge.
- **Success:** all 3 ghosts are on the corkboard. The curtains open, the house brightens, and the hunter cheers: "Finally, PROOF!"

**Hidden extra:** the ghosts are also drawn *on the real screen* at 2–3% opacity. That's invisible normally, but if the player opens their screenshot in a photo editor and raises the exposure, the ghosts appear.

## Goal

Take a **screenshot at the right moment** to photograph each ghost: all 3 ghosts within 6 photos.

**Success condition:**
- Keep about 1 s of history of which ghosts were in the room.
- When a screenshot event arrives, count a ghost as photographed if it was in the room at any point in the previous ~700 ms. The event arrives slightly *after* the capture. Tune this window on real devices.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing on Android 14+ (install-time permission, never shown) · 🔴 Android 13 and older: needs photo access that Google Play restricts | 🟢 Nothing |
| **Code to get the value** | 🟢 `screen_capture_event_plus` (or similar) | 🟢 `screen_capture_event_plus` (or similar) |
| **Overall** | **🟢 Easy (Android 14+) · 🔴 Very hard (older)** | **🟢 Easy** |

## Technical documentation

**iOS**
- `UIApplication.userDidTakeScreenshotNotification`, posted just *after* the capture. No permission is needed.
- The app doesn't receive the image.

**Android 14+ (API 34)**
- Call `Activity.registerScreenCaptureCallback(executor, callback)` in `onStart()` and `unregisterScreenCaptureCallback` in `onStop()`.
- Declare `<uses-permission android:name="android.permission.DETECT_SCREEN_CAPTURE" />`. It's install-time, so there's no prompt.
- The callback only fires while the activity is visible. **Google's documentation says it only detects the hardware button combination**, not ADB or test captures.
- **The system shows a toast: "<App> detected this screenshot".** That's a free awareness moment.
- The app doesn't receive the image.

**Android 13 and older**
- The only method is watching MediaStore for new screenshot files, which needs `READ_MEDIA_IMAGES`. Google Play restricts that permission to apps whose core purpose needs broad photo access.
- Don't do it. Lock the level instead.
- If a screenshot package adds `READ_MEDIA_IMAGES` to the manifest, remove it with `tools:node="remove"`.

**Gotchas**
- Both button combinations include a volume button. If the player presses it slightly early, the volume moves one step. Ignore one-step volume changes during this level.
- Both platforms show a screenshot thumbnail. If the player taps it, the game goes to the background. Pause the ghost timers and resume them when the player comes back.

## Clues

1. **Vague** (the hunter, trembling): "They only show up on film… and my camera's broken."
2. **Closer:** "Watch the room. When something moves by itself, a ghost is there. That's when you shoot."
3. **Specific:** "When the chair rocks, the book floats or the candle flickers, take a screenshot."

**(?) Solution after 2 minutes:** watch for a clue, then immediately press the screenshot buttons:
- **iPhone with Face ID:** side button + volume up, at the same time.
- **iPhone with a Home button:** Home + side button.
- **Android:** power + volume down.

## After the level

**Card:** "This game knew every time you took a screenshot, and it never asked for permission."

**What you learned:** apps are told when you take a screenshot. They don't get the picture itself (that would need access to your photos), but they know *when*, and they know what was on their own screen at that moment.

**Trivia:**
- Snapchat uses the same signal to tell your friends you screenshotted their message.
- On Android 14+, your phone showed a small notice ("…detected this screenshot"). On iPhone, there was no notice at all.
- Your real screenshot shows an empty room. Ghosts don't show up in normal photos. (Unless you turn the exposure right up…)

**Final report entry:** "Screenshots you took during the game: 5."

## If the device can't do it

- **Android 13 or older:** show the level as locked, with a card: "On your Android version, an app could only detect this by reading all your photos, and we chose not to ask." That card is a lesson in itself.
- **Screenshots blocked** (some work-managed phones): mark as not supported.
- **The player can't press two buttons at once** (accessibility):
  - iPhone: AssistiveTouch has a "Screenshot" action. Back Tap (double-tap the back of the phone) can also be set to take a screenshot.
  - Android: the Accessibility Menu has a "Screenshots" button.
  - On Android 14, these other methods may not trigger the callback (Google documents only the buttons), so test them.

## Reverting

Nothing to change in settings. Optional reminder: "Your ghost photos are in your gallery. Delete them if you like." The game can't check this without photo access.
