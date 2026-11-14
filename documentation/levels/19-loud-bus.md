# Level 19 · Peace and Quiet on the Bus

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Bluetooth audio connection (detected through the audio output route) |
| **Permission** | None (Android and iOS) when using the audio route |
| **Support** | Android ✅ · iOS ✅ |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟢 Easy |
| **Setting to revert** | Disconnect the headphones |
| **Reference** | [research.md §4.17](../research.md#417-bluetooth-connection) |

## Scene

A packed city bus in rush hour. The windows are steamy and people hang from the handrails, swaying at every turn. The player's character is squeezed into a seat in the middle of a loud group: one laughs like a hyena, one has a phone call on speaker, one plays tinny music from a phone, and one tells a very long story about their cousin's wedding.

Noise lines and "BLA BLA" bubbles fill the air, and the background audio is a wall of chatter. On the empty seat right next to the character, a pair of headphones glows softly, as if waiting.

- **Live feedback:** the character's eye twitches more as time goes on, and the noise slowly gets louder.
- **Success:** the moment Bluetooth headphones connect, the character grabs the glowing pair and puts them on. **The game's audio switches to calm lo-fi music, which the player now hears in their real headphones.** The loud group keeps flapping their mouths, but now they look like silent mimes. The character leans back, smiling, and the bus drives on into a sunset.

## Goal

**Connect Bluetooth headphones (or earbuds) to the phone.**

**Success condition:** the current audio output includes a Bluetooth device type.
- iOS: `.bluetoothA2DP`, `.bluetoothHFP` or `.bluetoothLE`.
- Android: `TYPE_BLUETOOTH_A2DP`, `TYPE_BLUETOOTH_SCO`, `TYPE_BLE_HEADSET`, `TYPE_BLE_SPEAKER` or `TYPE_HEARING_AID`.

**Precondition:** no Bluetooth audio connected at the start.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟢 `audio_session` | 🟢 `audio_session` |
| **Overall** | **🟢 Easy** | **🟢 Easy** |

Checking the audio output needs no Bluetooth permission. Asking Bluetooth directly would need a "Nearby devices" prompt on Android 12+, and iOS can't see headphones that way at all.

## Technical documentation

**iOS**
- `AVAudioSession.sharedInstance().currentRoute.outputs[].portType` and `portName`.
- `AVAudioSession.routeChangeNotification` when it changes.
- No permission is needed.

**Android**
- `AudioManager.getDevices(AudioManager.GET_DEVICES_OUTPUTS)` → `AudioDeviceInfo.getType()` and `getProductName()`.
- `AudioManager.registerAudioDeviceCallback` for changes.
- No permission is needed.

**Flutter**
- `audio_session`: `getDevices()` and `devicesChangedEventStream`. `AudioDeviceType` includes `bluetoothA2dp`, `bluetoothSco` and `bluetoothLe`.

**Why not "any Bluetooth connection"**
- iOS can't list classic Bluetooth devices at all (watches, cars, keyboards). Core Bluetooth only sees low-energy devices whose service IDs you already know.
- On Android it needs the `BLUETOOTH_CONNECT` permission (a "Nearby devices" prompt on Android 12+).
- The Control Center Bluetooth toggle on iPhone only *disconnects* devices; it doesn't turn Bluetooth off.

## Clues

1. **Vague** (the character, through gritted teeth): "If only I could block out all this noise…"
2. **Closer:** "Those headphones on the seat look a lot like a pair you own."
3. **Specific:** "Connect your Bluetooth headphones or earbuds to your phone."

**(?) Solution after 2 minutes:** turn on your Bluetooth headphones, or open your earbuds' case near the phone. If they don't connect by themselves:
- **Android:** Settings > Connected devices (Samsung: Settings > Connections > Bluetooth) and tap your headphones.
- **iPhone:** Settings > Bluetooth and tap your headphones.

## After the level

**Card:** "Headphones connected: 'Ana's AirPods Pro'. No permission needed to see that, or their name."

**What you learned:** apps know when you plug in or connect headphones, and they can read the headphones' name. Bluetooth headphones are often named after their owner, so that name can reveal your first name.

**Trivia:**
- Shops have used Bluetooth "beacons" to follow customers around the store. Apps need Bluetooth or location permission for that kind of tracking.
- Apps also use the headphone signal to pause music automatically when you unplug, so it doesn't blast out loud.

**Final report entry:** "Headphones: 'Ana's AirPods Pro' (Bluetooth)."

## If the device can't do it

- **No Bluetooth headphones:** accept wired headphones too (route types `wiredHeadphones`, `wiredHeadset`, `usbAudio`). The story still works: "Any headphones will do!"
- **No headphones at all:** offer "Skip for now" (come back later). A Bluetooth speaker or car audio also counts, so it can be solved in the car too.
- **Headphones connect but the audio goes elsewhere** (iPhone switching between devices): the check uses the *current* output, so ask the player to make sure their sound is playing through the headphones.

## Reverting

Your stop! The character takes the headphones off and hops off the bus: "Ahh, that was nice."
- **Player action:** disconnect the headphones (turn them off, or put the earbuds back in their case).
- **Check:** no Bluetooth output in the current audio route.
