# Level 10 · Ready for Take-off

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Airplane mode |
| **Permission** | None (Android and iOS) |
| **Support** | Android ✅ · iOS ⚠️ (no API; inferred from the mobile network disappearing) |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟠 Hard |
| **Setting to revert** | Airplane mode off |
| **Reference** | [research.md §4.9](../research.md#49-airplane-mode) |

## Scene

The cabin of a small cartoon plane on the runway. Rows of passengers are buckled in, the seatbelt signs glow, and through the window you can see the runway and a little control tower.

The player's character sits in a window seat holding a phone. In the aisle, a flight attendant with a perfect bun and a big smile has finished the safety demonstration and **will not stop talking**. Speech bubbles pile up: "…and the oxygen mask goes over YOUR face first, and the life vest is under your seat, and did I mention the exits…"

- **Escalation:** the captain announces over the speaker: "Ladies and gentlemen, we're still waiting for *one* passenger to get their phone ready for the flight…" Passengers turn round one by one and glare at the player's character. A baby starts crying. The attendant starts the whole safety demo again from the beginning.
- **Live feedback:** a mobile signal icon floats above the character's phone, with full bars.
- **Success:** as soon as airplane mode is on, the signal bars drop, a little airplane icon appears above the phone, and the attendant beams, finally stops talking, and sits down. The engines roar, the plane takes off, and clouds rush past the window.

## Goal

**Turn on airplane mode.**

**Success condition:**
- Android: `Settings.Global.AIRPLANE_MODE_ON == 1`.
- iOS (heuristic, as there's no direct API): no SIM reports a mobile network type any more, **and** the network monitor shows no mobile connection.

**iOS precondition:** the phone has mobile service when the level starts. If it doesn't, see the workarounds.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟢 `airplane_mode_checker`, or 2 lines of native code | 🟠 No API. Native code combines two network signals into a guess that can't tell airplane mode from "no signal" |
| **Overall** | **🟢 Easy** | **🟠 Hard** |

## Technical documentation

**Android**
- `Settings.Global.getInt(contentResolver, Settings.Global.AIRPLANE_MODE_ON)`.
- Changes arrive through the `Intent.ACTION_AIRPLANE_MODE_CHANGED` broadcast.
- No permission is needed.

**iOS**
- **There's no public API.** The heuristic:
  - `CTTelephonyNetworkInfo().serviceCurrentRadioAccessTechnology` is nil or empty for every SIM.
  - `NWPathMonitor` shows no `.cellular` interface.
- Limits:
  - It can't tell airplane mode from "no signal" (a basement) or "no SIM".
  - Wi-Fi can stay on in airplane mode, because iOS remembers if the user turned it back on.
  - Wi-Fi-only iPads have no mobile radio to check.

**Flutter**
- `airplane_mode_checker` (Android is reliable; its own docs say iOS isn't).
- `connectivity_plus` for the network side.

**Gotchas**
- The player opens Control Center or Quick Settings to do this, so the game stays in the foreground. Still, re-check when the app resumes.
- Some work-managed phones block airplane mode.

## Clues

1. **Vague** (the captain): "We cannot take off until every device is ready for the flight."
2. **Closer:** "Every phone has a special mode just for flying."
3. **Specific:** "Turn on airplane mode."

**(?) Solution after 2 minutes:**
- **Android:** swipe down from the top of the screen and tap **Airplane mode**.
- **iPhone:** open Control Center and tap the **airplane** icon. On iPhones with Face ID, swipe down from the top-right corner. On iPhones with a Home button, swipe up from the bottom.

## After the level

**Card:** "You were connected through Wi-Fi and mobile data. No permission needed to know that."

**What you learned:** apps always know whether you're online and how (Wi-Fi or mobile data). Android apps can see airplane mode directly. iPhone apps can't, but they can work it out when the mobile network vanishes.

**Trivia:**
- Airplane mode doesn't switch off location. GPS only *receives* signals, so it keeps working in airplane mode.
- You can turn Wi-Fi and Bluetooth back on while in airplane mode, and iPhones remember that choice next time.

**Final report entry:** "Network when you started: Wi-Fi + mobile data." On iPhone, also the mobile network type (4G/5G): `serviceCurrentRadioAccessTechnology` needs no permission.

## If the device can't do it

- **Wi-Fi-only device or no SIM:** there's no mobile network to lose. Change the instruction to "Turn on airplane mode and make sure Wi-Fi is off", and succeed when the network monitor shows *no connection at all*.
- **Already in airplane mode at the start:** the start-of-game checklist prevents this. If it happens anyway, ask the player to turn it off first ("we're still at the gate").
- **Airplane mode blocked by a work profile:** mark the level as not supported.

## Reverting

The plane lands with a bump, and passengers clap. The attendant says: "Welcome to your destination! You may now switch off airplane mode."
- **Player action:** turn airplane mode off.
- **Check:** Android `AIRPLANE_MODE_ON == 0`. iOS: the mobile network is back, or there's an internet connection.
