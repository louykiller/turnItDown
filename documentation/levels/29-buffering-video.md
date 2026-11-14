# Level 29 · Endless Buffering

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Network type (Wi-Fi vs mobile data) |
| **Permission** | Android: `ACCESS_NETWORK_STATE` (install-time, no prompt). iOS: none |
| **Support** | Android ✅ · iOS ✅ (both need a SIM with mobile data) |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟢 Easy |
| **Setting to revert** | Wi-Fi back on |
| **Reference** | [research.md §4.28](../research.md#428-wi-fi-vs-mobile-data) |

> Branding note: use a made-up video site. Don't show YouTube's name, logo or red play button. Those are trademarks.

## Scene

A cosy home office at night. On the desk, a computer monitor plays a video on a made-up video site: a cat on a kitchen counter, about to attempt an epic jump onto the fridge. Just as it crouches, the video **freezes**. The spinning loading circle appears, and the progress bar crawls. When it plays again it's in blurry 144p: the cat is a smudge of three orange squares.

The player's character pulls at their hair. In the corner, the Wi-Fi router has a sad face and a blinking red light. Through a little window in its side, an exhausted hamster stumbles on a wheel.

- **Escalation:** "Buffering… 12%… 11%…" (it goes *backwards*). The hamster lies down. The character starts typing an angry complaint.
- **Success:** the moment the phone is on mobile data with Wi-Fi off, a mobile tower outside the window lights up with "5G" sparkles. The video bursts into crisp HD, and the cat leaps, flips and lands perfectly on the fridge. The character jumps up cheering, and the hamster gets a well-earned nap.

**Flipped version (automatic):** if the player starts the level *on mobile data* (with no Wi-Fi), the story reverses. The router is fine, but a "data plan almost used up!" warning flashes, and the goal becomes "switch to Wi-Fi".

## Goal

**Turn Wi-Fi off** so the phone uses **mobile data**.

**Success condition:** the current connections include mobile data and **not** Wi-Fi.
- Android: the default network has `TRANSPORT_CELLULAR` and no `TRANSPORT_WIFI`.
- iOS: `path.usesInterfaceType(.cellular)` and not `.wifi`.

**Precondition:** connected to Wi-Fi at the start, with a SIM and mobile data available. Otherwise use the flipped version or skip.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing (the permission is install-time and never shown) | 🟢 Nothing |
| **Code to get the value** | 🟢 `connectivity_plus` | 🟢 `connectivity_plus` |
| **Overall** | **🟢 Easy** | **🟢 Easy** |

## Technical documentation

**Flutter**
- `connectivity_plus`: `checkConnectivity()` and `onConnectivityChanged`, which return a **list** of `ConnectivityResult` values (`wifi`, `mobile`, `ethernet`, `vpn`, `none`, …).
- Success = the list contains `mobile` and not `wifi`.

**Android**
- `ConnectivityManager.registerDefaultNetworkCallback(…)`, then `NetworkCapabilities.hasTransport(TRANSPORT_WIFI)` / `hasTransport(TRANSPORT_CELLULAR)`.
- Permission `ACCESS_NETWORK_STATE` (install-time).
- The carrier name can be read without permission: `TelephonyManager.getNetworkOperatorName()`.

**iOS**
- `NWPathMonitor`: `path.usesInterfaceType(.wifi)` / `.cellular`, and `path.isExpensive` (true on mobile data or a personal hotspot). No permission is needed.
- iOS no longer gives apps the carrier name (`CTCarrier` is deprecated).

**Gotchas**
- **Wi-Fi Assist** (iOS) and **"switch to mobile data automatically"** (Android) can use mobile data while Wi-Fi is still connected. Require Wi-Fi to be *gone*, not just mobile data to be in use.
- On iPhone, the Control Center Wi-Fi button only disconnects from the current network. That's fine for this level, because the connection list no longer contains Wi-Fi.
- The game doesn't stream any real video, so the level uses almost no mobile data. Say so, to reassure players worried about their data plan.

## Clues

1. **Vague** (the character): "This Wi-Fi is HOPELESS…"
2. **Closer:** "The router is broken. Is there another way your phone can get online?"
3. **Specific:** "Turn off Wi-Fi so your phone uses mobile data."

**(?) Solution after 2 minutes:**
- **Android:** swipe down from the top. Tap **Internet** and switch **Wi-Fi** off (on Samsung, tap the **Wi-Fi** tile). Make sure **Mobile data** is on.
- **iPhone:** Settings > **Wi-Fi** > off (or tap the Wi-Fi button in Control Center). Make sure **Mobile Data** is on.

## After the level

**Card:** "You were on Wi-Fi. No permission needed to know that, or to notice the moment you switched."

**What you learned:** every app can see what kind of connection you're using and every time it changes, silently. On Android, apps can even read your mobile carrier's name.

**Trivia:**
- Apps need **location** permission to read your Wi-Fi network's *name*, because Wi-Fi names reveal where you are. Your home network is basically your address.
- Every server you connect to sees your IP address. On mobile data it usually points to your carrier and region; on Wi-Fi, often to your home's internet provider and city.
- Apps can check whether you're on an "expensive" connection (mobile data or hotspot) and wait for Wi-Fi before downloading big updates.

**Final report entry:** "Connection when you started: Wi-Fi." On Android, also: "Carrier: <name>".

## If the device can't do it

- **No SIM, no data plan, or mobile data disabled by policy:** mark the level as not supported ("Your phone has no mobile data to switch to").
- **Abroad, with roaming:** warn about possible roaming costs, offer the skip, and reassure the player that the game uses almost no data.
- **Wi-Fi-only tablet:** mark the level as not supported.
- **No Wi-Fi at the start:** use the flipped version ("switch to Wi-Fi"), if a Wi-Fi network is available. Otherwise skip.

## Reverting

The video ends, the cat bows, and the router's hamster wakes up refreshed: "I'm back! Switch me on to save your data."
- **Player action:** turn Wi-Fi back on.
- **Check:** the connection list contains `wifi` again (or matches the start state).
