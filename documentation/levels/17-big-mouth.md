# Level 17 · Shut the Big Mouth

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Multi-touch (number of fingers and how they move) |
| **Permission** | None (Android and iOS) |
| **Support** | Android ✅ (usually 10 touch points) · iOS ✅ (iPhone: maximum 5 touches) |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟢 Easy |
| **Setting to revert** | None |
| **Reference** | [research.md §4.15](../research.md#415-multi-touch) |

## Scene

A giant cartoon mouth fills the whole screen: big glossy lips, a gap between the front teeth, a wiggling tongue and a dangling uvula. It never stops talking. Gibberish speech bubbles pour out ("blah blah BLAH and then I said blah…"), getting faster and louder.

With every few words it **spits**: droplets fly towards the player and splat onto the "glass" of the screen, sliding down slowly (a wet-glass overlay). After a while the screen is getting pretty soggy.

- **Live feedback:** each finger placed on the screen makes a cartoon fingertip appear on the lips. The lips react with squishy deformation. Two fingers make the mouth laugh ("Ha! Is that all you've got?"), three make it nervous, four make its eyes go wide.
- **Pinching:** when 5 fingers squeeze inwards, the lips are pushed together in real time.
- **Success:** the lips zip shut with a *ZZZIP* sound, and a cartoon zip appears across them. A muffled "Mmmph! Mmmph!" follows, the screen wipes itself dry, and blessed silence.

## Goal

**Put all five fingers of one hand on the screen, then pinch them together.**

**Success condition:**
- At least 5 touches at once. Accept 4 on screens or for players that can't manage 5.
- The average distance from each finger to their centre point shrinks by at least 50% from when the fifth finger landed.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟢 Built into Flutter (`Listener`, one ID per finger) | 🟢 Built into Flutter (`Listener`, one ID per finger). iPhones track at most 5 touches |
| **Overall** | **🟢 Easy** | **🟢 Easy** |

## Technical documentation

**Flutter**
- Multi-touch is on by default. Use a `Listener` (`onPointerDown`, `onPointerMove`, `onPointerUp`, `onPointerCancel`) and keep a map of `pointer → position`.
- Alternatively, a `ScaleGestureRecognizer`: `ScaleUpdateDetails.pointerCount` and `scale`.

**Android**
- `MotionEvent.getPointerCount()`, `getX(i)`/`getY(i)` and `getPointerId(i)`.
- Usually up to 10 touches, fewer on some cheap screens.

**iOS**
- `UIView.isMultipleTouchEnabled` (Flutter enables it). An iPhone tracks **at most 5** touches at once. This is widely reported, but Apple doesn't document it. iPads track more.

**Gotchas**
- **3-finger screenshot gesture:** several Android brands (Xiaomi, OnePlus, Oppo, Realme, Vivo) take a screenshot when three fingers swipe down. A 5-finger pinch can trigger it, which would fire [level 6](06-ghost-hunter.md)'s screenshot detection by accident. Warn players on those brands.
- iOS accessibility Zoom uses a 3-finger double tap.
- If the system cancels the touches (`onPointerCancel`), reset the gesture.

## Clues

1. **Vague:** "Somebody shut that mouth!"
2. **Closer:** "One or two fingers won't do it. This mouth is BIG."
3. **Specific:** "Put all five fingers of one hand on the screen, spread out, then pinch them together."

**(?) Solution after 2 minutes:** spread your hand and place all five fingertips on the screen around the mouth, then bring them together in one movement, as if grabbing something off the screen.

## After the level

**Card:** "Your screen detected 5 fingers at once. Every app sees each finger, its position and how it moves."

**What you learned:** apps know exactly how many fingers touch the screen, where they are and how they move, all without permission.

**Trivia:**
- iPhones track up to 5 touches at once. Many Android phones handle 10.
- The way you touch is surprisingly personal. In 2013, researchers ("Touchalytics") showed people can be recognised just from how they swipe.
- On some Android phones, a 3-finger swipe takes a screenshot. Did you trigger one by accident?

**Final report entry:** "Most fingers your screen detected at once: 5."

## If the device can't do it

- **Screens that track fewer than 5 touches:** use the highest number of touches seen during the level as the target (minimum 3).
- **Limited hand mobility** (accessibility): after 60 s, accept 3 fingers. Or let the player use two hands with two or three fingers each.
- **The 3-finger screenshot gesture fires:** detect the screenshot event and show a tip on how to turn the gesture off for this level.

## Reverting

Nothing to revert. If a screenshot was taken by accident, mention that it's in the gallery.
