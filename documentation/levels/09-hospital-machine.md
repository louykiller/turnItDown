# Level 09 · Plug In the Patient

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Battery charging status (and battery level) |
| **Permission** | None (Android and iOS) |
| **Support** | Android ✅ · iOS ✅ |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟢 Easy |
| **Setting to revert** | Unplug the charger |
| **Reference** | [research.md §4.8](../research.md#48-battery-and-charging) |

## Scene

A cheerful cartoon hospital room, kept light and comic rather than grim. A patient with one leg in a cast hanging from a pulley is fast asleep and snoring. Wires run from him to a beeping heart monitor on a trolley.

The monitor's plug lies on the floor, a metre from the wall socket. A banana peel next to it suggests a nurse slipped and yanked it out. The monitor now runs on its own battery, shown as a big battery icon on its screen. **That icon shows the player's real battery percentage.**

- **Live feedback:** a slow drain animation on the monitor's battery adds tension, and the heartbeat line gets lazier.
- **Comedy:** a doctor runs in circles in the background with a clipboard, shouting "Somebody find a plug!" The patient keeps snoring.
- **Success:** the moment the phone starts charging, the plug zips across the floor into the socket by itself with a *zap*. The monitor shows the charging lightning bolt and the heartbeat line becomes strong and happy. The patient wakes up, stretches, and gives a thumbs up. The doctor faints with relief.

## Goal

**Connect the phone to a charger**, using a cable or a wireless charger.

**Success condition:**
- iOS: `batteryState` is `.charging` or `.full`.
- Android: `EXTRA_PLUGGED` is not 0 (AC, USB or wireless), or `BatteryManager.isCharging()` is true.

**Precondition:** not charging when the level starts. This is also a start-of-game check.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟢 `battery_plus` | 🟢 `battery_plus` |
| **Overall** | **🟢 Easy** | **🟢 Easy** |

## Technical documentation

**Android**
- Register for the sticky `Intent.ACTION_BATTERY_CHANGED` broadcast (`registerReceiver(null, IntentFilter(ACTION_BATTERY_CHANGED))`). It returns immediately with:
  - `EXTRA_LEVEL` / `EXTRA_SCALE` (percentage)
  - `EXTRA_STATUS` (charging, discharging, full, …)
  - `EXTRA_PLUGGED` (`BATTERY_PLUGGED_AC`, `_USB`, `_WIRELESS`)
- `BatteryManager.isCharging()` (API 23+).
- `ACTION_POWER_CONNECTED` / `ACTION_POWER_DISCONNECTED` broadcasts.
- No permission is needed.

**iOS**
- Set `UIDevice.current.isBatteryMonitoringEnabled = true`.
- `UIDevice.current.batteryState` (`.unplugged`, `.charging`, `.full`) and `batteryLevel` (0–1).
- `UIDevice.batteryStateDidChangeNotification` and `batteryLevelDidChangeNotification`.
- No permission is needed.

**Flutter**
- `battery_plus`: `batteryLevel`, `batteryState`, `onBatteryStateChanged`.

**Gotchas**
- At 100%, a plugged-in phone reports *full*, not *charging*. Count both.
- Battery-protection features (iOS Optimized Charging, Android "protect battery") may pause charging at 80%. The phone still reports that it's plugged in, so count that too.

## Clues

1. **Vague** (the doctor): "The machine is running out of power!"
2. **Closer:** "That machine needs power… and so does a certain phone I know."
3. **Specific:** "Plug your phone into a charger."

**(?) Solution after 2 minutes:** connect your phone to a charging cable or put it on a wireless charger. As soon as it starts charging, the machine is saved.

## After the level

**Card:** "Battery: 47%, charging via USB. No permission needed." The charging type is only shown on Android.

**What you learned:** any app can read your battery level and see when you plug in or unplug, and on Android exactly *how* you charge (USB, wall charger or wireless).

**Trivia:**
- In 2015, privacy researchers showed websites could use your battery level and how fast it drained to recognise you across different sites ("The Leaking Battery"). Firefox later removed this feature for websites.
- Charging patterns reveal your routine. Plugging in every night at 23:30 is basically your bedtime.

**Final report entry:** "Battery: 47%. You charged your phone during the game."

## If the device can't do it

- Every phone supports this level.
- **No charger nearby:** offer "Skip for now", and keep the level marked to come back to later.
- **The phone is already at 100%:** *full* counts as success.

## Reverting

The patient walks out of the hospital, cured. The nurse plugs the machine into the wall properly: "You can have your charger back now."
- **Player action:** unplug the phone.
- **Check:** unplugged (Android `EXTRA_PLUGGED == 0`; iOS `.unplugged`).
- **Why it matters:** on iPhone, Low Power Mode switches itself off when the phone charges past 80%, which would break [level 21](21-sparking-power-strip.md).
