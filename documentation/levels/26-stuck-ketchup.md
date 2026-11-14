# Level 26 · Stubborn Ketchup

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Accelerometer (shake detection) |
| **Permission** | None (Android and iOS) |
| **Support** | Android ✅ · iOS ✅ |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟢 Easy |
| **Setting to revert** | None |
| **Reference** | [research.md §4.25](../research.md#425-shake) |

## Scene

A retro 1950s diner: red leather booths, a chequered floor, a jukebox playing rock 'n' roll, and a waitress on roller skates. On the table, a giant juicy burger with melted cheese sits on a plate. It's missing one thing.

The player's character holds a glass ketchup bottle upside down over the burger. Nothing comes out. They slap the bottom, tap the side with a knife, and peer into the neck. Their stomach rumbles and they're drooling.

**The phone is the bottle.** A big "HOLD TIGHT!" warning appears with a cartoon hand gripping a phone, because people really do throw phones while shaking them.

- **Live feedback:** each shake makes the bottle jolt on screen, and a little more ketchup creeps towards the neck. Sloshing sounds play, and a progress meter shaped like the bottle fills up.
- **Comedy:** after a few shakes, a single drop lands. "That's it?!" The waitress skates past, giggling.
- **Success:** after six good shakes in a row, **SPLAT!** Way too much ketchup erupts all over the burger, the plate and the character's face. The character licks their lips and gives a ketchup-covered thumbs up. The waitress hands over a napkin.

## Goal

**Shake the phone** firmly.

**Success condition:**
- 6 shake peaks within 3 s.
- A peak is the acceleration size (with gravity removed) above about 12–15 m/s², followed by a change of direction.
- Progress builds with each peak and fades slowly if the player stops.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟢 `sensors_plus` (acceleration with gravity removed) | 🟢 `sensors_plus` (acceleration with gravity removed) |
| **Overall** | **🟢 Easy** | **🟢 Easy** |

Counting the shake peaks is level logic, so it isn't counted.

## Technical documentation

**Android**
- `Sensor.TYPE_LINEAR_ACCELERATION` (gravity removed), or `TYPE_ACCELEROMETER` minus a low-pass gravity estimate.
- No permission is needed.

**iOS**
- `CMMotionManager.startDeviceMotionUpdates()` → `CMDeviceMotion.userAcceleration` (in g, gravity removed). No permission is needed.
- Simpler alternative: UIKit's system shake event `motionEnded(.motionShake, with:)`. You can't tune it, though.

**Flutter**
- `sensors_plus` `userAccelerometerEventStream()` (m/s², gravity removed).

**Gotchas**
- If a text field has focus, iOS may offer "Undo Typing" when the phone is shaken. Make sure no text field is focused during this level.
- Test thresholds on several phones; sensors and hand strength vary.

## Clues

1. **Vague** (the character): "This ketchup is STUCK. What do you do with a stuck bottle?"
2. **Closer:** "Your phone *is* the bottle."
3. **Specific:** "Hold your phone tightly and shake it up and down, fast."

**(?) Solution after 2 minutes:** grip the phone firmly with your whole hand and shake it quickly up and down about six times in a row.

## After the level

**Card:** "Your strongest shake: 2.4 g. Every app can feel how you move your phone, without asking."

**What you learned:** the accelerometer reports every movement, from a gentle tilt to a hard shake, to any app, silently. Movement patterns reveal what you're doing: walking, running, riding in a car, even falling.

**Trivia:**
- iPhone 14 and later have "Crash Detection", which uses a special high-range accelerometer and other sensors to notice a car crash and call for help.
- On iPhones, shaking can trigger "Shake to Undo" when you're typing.

**Final report entry:** "Strongest shake: 2.4 g."

## If the device can't do it

- Every phone has an accelerometer.
- **Players who can't shake** (limited mobility, tremor, or a phone they're worried about dropping): after 60 s, lower the threshold, or accept many gentle shakes over a longer time. Always offer the skip.
- **A heavy case or a phone held loosely:** lower the threshold slightly after the second clue.

## Reverting

Nothing to revert.
