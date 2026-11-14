# Level 22 · Where's My Food?

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Location services switch, plus the location permission chosen during setup (precise, approximate or refused) |
| **Permission** | Reading the switch: none. Location permission: asked during setup, together with all the others, so the prompt doesn't give this level away |
| **Support** | Android ✅ · iOS ✅ |
| **Access difficulty: Android** | 🟢 Easy (switch) · 🟡 Moderate (position) |
| **Access difficulty: iOS** | 🟢 Easy (switch) · 🟡 Moderate (position) |
| **Setting to revert** | Location off again (and, optionally, remove the game's location permission) |
| **Reference** | [research.md §4.20](../research.md#420-location) |

## Scene

A rainy evening. Inside a cosy flat, the player's character sits in pyjamas on the sofa, stomach rumbling loudly, staring at a food-delivery app on their phone: "Your courier can't find you 😢". Use a made-up delivery app; no real brand.

Outside, seen through the window and on the in-app map, a courier on a little scooter with a huge insulated backpack drives in confused circles around a roundabout. Their map is upside down, they stop to ask a pigeon for directions, and go down the same street three times. On the in-game map, the courier's dot wanders randomly and the player's house has no blue dot.

- **Escalation:** the character's stomach growls louder, and they start eyeing the houseplant.
- **Switch on (success):** the moment location turns on, a blue dot appears on the map, the courier's route snaps into a straight line to the house, and the scooter zooms off.
- **Then the ending depends on the location permission the player chose during setup.** This is the teaching moment: the player finally sees what that early choice meant.

| Choice made during setup | Ending |
|---|---|
| **Precise** | The doorbell rings *ding-dong*. Hot pizza at the door, and a happy courier gets a tip. |
| **Approximate** | The courier rings a neighbour's door two streets away. They phone: "Are you the blue house? There are a lot of blue houses…" Eventually they find you, with a slightly cold pizza. |
| **Refused** | The courier calls and the character has to give directions by voice: "Left at the bakery…" The food arrives late. |

Every choice passes the level. The difference is the lesson.

## Goal

**Turn on location** (Location Services on iPhone).

**Success condition:** the location switch is on.
- Android: `LocationManager.isLocationEnabled()`.
- iOS: `CLLocationManager.locationServicesEnabled()`.

Then the game reads the location permission chosen during setup (and, if granted, whether it's precise or approximate) and plays the matching ending.

**Precondition:** location off at the start. If it's already on, the courier finds the house straight away and the ending plays.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing to read the switch · 🟡 location prompt for a position (shown during setup) | 🟢 Nothing to read the switch · 🟡 location prompt for a position (shown during setup) |
| **Code to get the value** | 🟢 `geolocator` | 🟢 `geolocator` |
| **Overall** | **🟢 Easy (switch) · 🟡 Moderate (position)** | **🟢 Easy (switch) · 🟡 Moderate (position)** |

## Technical documentation

**The switch (no permission)**
- **Android:** `LocationManager.isLocationEnabled()` (API 28+). Changes arrive through the `LocationManager.MODE_CHANGED_ACTION` broadcast.
- **iOS:** `CLLocationManager.locationServicesEnabled()`.
  - Apple warns against calling it on the main thread.
  - React to the `locationManagerDidChangeAuthorization` callback, which also fires when the global switch changes.
  - While the switch is off, the app's status reads `.denied`.

**The position (prompt)**
- **Android:** declare `ACCESS_COARSE_LOCATION` + `ACCESS_FINE_LOCATION`. Since Android 12 the user chooses "Precise" or "Approximate".
- **iOS:** `requestWhenInUseAuthorization()` with `NSLocationWhenInUseUsageDescription`. Since iOS 14 the user can turn "Precise" off, and `accuracyAuthorization` becomes `.reducedAccuracy` (accurate to a few kilometres).

**Turning location on from inside the app (Android only)**
- Google Play Services `SettingsClient.checkLocationSettings` → `ResolvableApiException.startResolutionForResult` shows the system dialog "Turn on device location?".
- The `location` package wraps this as `requestService()`.
- iOS has nothing similar.

**Flutter**
- `geolocator`: `isLocationServiceEnabled()`, `checkPermission()`, `requestPermission()`, `getLocationAccuracy()` and `getCurrentPosition()`.
- Re-check when the app resumes, because the iPhone player has to go to Settings.

## Clues

1. **Vague** (the courier, on the phone): "I've been round this roundabout six times… where ARE you?"
2. **Closer:** "Your phone knows exactly where you are… if you let it say so."
3. **Specific:** "Turn on Location (called Location Services on iPhone)."

**(?) Solution after 2 minutes:**
- **Android:** swipe down from the top and tap **Location**, or tap "Turn on" in the dialog the game shows.
- **iPhone:** Settings > Privacy & Security > **Location Services** > turn it on, then come back to the game.

## After the level

**Card:** "You allowed precise location. This game could see where you are to within about 12 metres. (Nothing was stored or sent.)" The text changes for approximate or denied.

**What you learned:** reading whether location is on needs no permission. Your *actual* position does, and you can choose to share only an approximate one. Approximate is enough for weather apps and plenty of others.

**Trivia:**
- A 2013 study ("Unique in the Crowd") found that just **4** location points with times were enough to identify 95% of people in a large phone dataset.
- Even with location off, every website and app server sees your **IP address**, which usually reveals your city.
- Apps need location permission to read your Wi-Fi network's *name*, because Wi-Fi names reveal where you are.
- Photos can store the GPS position where they were taken.

**Final report entry:** "Location: allowed (precise). Approximate area: Lisbon." Show the area only if granted.

## If the device can't do it

- **The location setting is locked** (managed devices): mark the level as not supported.
- **The player refused location permission during setup:** the level still passes (the switch is what counts), with the "Refused" ending.
- **Location already on at the start:** the courier finds the house straight away and the ending plays.
- **iPhone: the prompt couldn't be shown during setup** (iOS may not show it while Location Services are off; verify): ask right after the player turns location on.
- **Wi-Fi-only tablet without GPS:** turning location on still works, and the position comes from Wi-Fi.

## Reverting

The courier waves goodbye: "Enjoy! …and you might not want the whole world knowing where you live." The character draws the curtains.
- **Player action:** turn location off (start state). Optionally, remove the game's location permission in Settings. The card shows how.
- **Check:** location switch off.
