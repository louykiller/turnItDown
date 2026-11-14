# Level 11 · Rewind the Crime

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | System date/time (and time zone) |
| **Permission** | None (Android and iOS) |
| **Support** | Android ✅ · iOS ✅ (the time setting is locked on some managed phones) |
| **Access difficulty: Android** | 🟢 Easy (🟡 for the revert check) |
| **Access difficulty: iOS** | 🟢 Easy (🟡 for the revert check) |
| **Setting to revert** | Automatic time back on (clock back to the correct time) |
| **Reference** | [research.md §4.10](../research.md#410-system-datetime) |

## Scene

A grand old library in a mansion, at night, in a cartoon detective style. Tall bookshelves, a green banker's lamp, a grandfather clock, and an open window with the curtains blowing.

A detective in a trench coat and hat bursts through the door, but too late. A cartoon victim lies on the rug with comic "X" eyes (no gore), a vase lies smashed, and the culprit's shadow vanishes through the window. The detective pulls out a brass pocket watch, the **ChronoWatch**: "This watch follows your phone's clock. If only we'd arrived a few minutes earlier…"

The ChronoWatch is shown large on screen, and its hands always match the phone's real clock.

**Outcomes when the player returns to the game after changing the clock:**

| Clock offset | What happens |
|---|---|
| **1 to 10 minutes back** ✅ | The room rewinds (VHS rewind effect). The detective appears just before the crime, tackles the culprit as they raise the weapon, and the "victim" turns around safe and amazed. Handcuffs click. Success! |
| **More than 10 minutes back** | An empty, peaceful library with the clock ticking. Nobody is here yet. "Too early! Nothing has happened yet." The player can adjust the clock again. |
| **Moved forward** | Police tape across the door, a chalk outline, a red stain on the rug, a detective drinking coffee: "You missed it. The body's already gone." |
| **Time zone changed instead** | The detective, dizzy, in a Hawaiian shirt: "That's jet lag, not time travel!" |

## Goal

Set the phone's **clock back by 1–10 minutes**.

**Success condition:**
- Offset = `DateTime.now() − (wall clock at level start + Stopwatch.elapsed)`.
- −10 min ≤ offset ≤ −1 min → success.
- Evaluate when the app resumes (the player comes back from Settings) and on clock-change events.
- If the time-zone offset changed but the actual time didn't move, show the jet-lag tease.

**How it works:** the stopwatch uses a clock the user can't change (monotonic), so the game knows exactly how many minutes the user moved the wall clock. This needs no internet.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟢 Built into Dart (`DateTime`, `Stopwatch`). Revert check of "set time automatically": 🟡 a small native read of `AUTO_TIME` | 🟢 Built into Dart. Revert check: 🟡 the setting can't be read, so query a time server (`ntp`) and compare (needs internet) |
| **Overall** | **🟢 Easy (🟡 for the revert check)** | **🟢 Easy (🟡 for the revert check)** |

## Technical documentation

**Reading the time (both platforms)**
- Wall clock: `DateTime.now()`. Time zone: `DateTime.now().timeZoneOffset` and `timeZoneName`.
- Monotonic clock (not affected by the user): Dart `Stopwatch`; natively, Android `SystemClock.elapsedRealtime()` and iOS `ProcessInfo.systemUptime`.

**Change events**
- Android: `Intent.ACTION_TIME_CHANGED` (`android.intent.action.TIME_SET`) and `Intent.ACTION_TIMEZONE_CHANGED`.
- iOS: `UIApplication.significantTimeChangeNotification`, `.NSSystemClockDidChange` and `.NSSystemTimeZoneDidChange`.
- The player changes the clock in the Settings app, so the game is in the background. **Always re-check when the app resumes.**

**"Set time automatically"**
- Android: `Settings.Global.AUTO_TIME` and `AUTO_TIME_ZONE` (1 = on). No permission is needed.
- iOS: can't be read. Query an NTP server (`ntp` package) or read the `Date` header of an HTTPS response, then compare with the phone's clock (±30 s).

**Gotchas**
- The iPhone and Android time pickers set whole minutes, so the seconds jump to 0. Allow for that in the window.
- **Some phones lock this setting** (work or school devices, or Screen Time restrictions).
- A wrong clock can make some websites and apps fail (security certificates depend on the date). Remind the player to revert.
- The hint timer must use the monotonic clock too, otherwise this level breaks it.

## Clues

1. **Vague** (the detective): "If only we'd got here a few minutes earlier…"
2. **Closer:** "This watch follows your phone's clock. Change the clock, change the past."
3. **Specific:** "In your phone's settings, turn off automatic time and set the clock back 5 minutes."

**(?) Solution after 2 minutes:**
- **Android (Pixel):** Settings > System > Date & time > turn off **Set time automatically** > **Time** > set it 5 minutes earlier.
- **Android (Samsung):** Settings > General management > Date and time > turn off **Automatic date and time** > set the time 5 minutes earlier.
- **iPhone:** Settings > General > Date & Time > turn off **Set Automatically** > tap the time and scroll the minutes back 5.

Then come back to the game.

## After the level

**Card:** "Time zone: Europe/Lisbon. Your clock was set automatically. No permission needed to know that."

**What you learned:** apps see your clock, your time zone, and the moment you change either of them. Android apps can even see whether "set automatically" is on.

**Trivia:**
- Some games catch players who move the clock forward to skip waiting times, using exactly this trick.
- Your time zone narrows down where you live. Together with language and other settings, it's part of your device's "fingerprint".
- Secure websites check the date. With a clock far enough off, some of them refuse to load.

**Final report entry:** "Time zone: Europe/Lisbon. Automatic time: on."

## If the device can't do it

- **The time setting is locked** (work/school phone, Screen Time): mark the level as not supported, with the card "Your phone's manager locked the clock. Apps can see that too."
- **The player isn't comfortable changing the clock:** skip, and show the card anyway.
- **No internet for the revert check on iPhone:** use the monotonic offset. The clock should be back to within ±1 minute of the expected time.

## Reverting

The detective tips his hat: "Back to the present, partner." A portal swirls.
- **Player action:** turn automatic time back on (same menu as the solution).
- **Check:** offset back within ±1 minute. Android: `AUTO_TIME == 1`. iOS: the clock matches the time server within ±30 s.
- **Why it matters:** a wrong clock breaks secure websites and app logins.
