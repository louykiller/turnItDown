# Level 21 · Overloaded

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Power saving mode (Android Battery Saver / iPhone Low Power Mode) |
| **Permission** | None (Android and iOS) |
| **Support** | Android ✅ (some brands' power savers behave differently) · iOS ✅ (iPad: iPadOS 15+) |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟢 Easy |
| **Setting to revert** | Power saving off |
| **Reference** | [research.md §4.19](../research.md#419-power-saving-mode) |

## Scene

A messy teenager's bedroom. On the floor, one 10-socket power strip has **ten** things plugged in and all running at once: a gaming PC with rainbow lights, a space heater glowing orange, a mini fridge, a toaster, a hair dryer blowing a poster off the wall, a TV, a lava lamp, a desk fan, a microwave spinning a burrito, and a disco ball.

The power strip glows red-hot and buzzes. Sparks fly and small puffs of smoke rise. The smoke alarm on the ceiling starts chirping, and the house cat stands frozen in the doorway with its fur on end.

- **Escalation:** the longer it goes, the more the sparks fly. The lights flicker, and the lava lamp's blobs move faster and faster.
- **Success:** the moment power saving is on, a green leaf icon appears on each device and they dim and slow down one by one: the PC lights go white, the heater cools, the disco ball stops. The strip cools from red to normal with a hiss of steam, the smoke alarm stops, and the cat's fur settles as it curls up on the warm PC.

## Goal

**Turn on power saving mode** (Battery Saver / Power saving on Android, Low Power Mode on iPhone).

**Success condition:**
- Android: `PowerManager.isPowerSaveMode() == true`.
- iOS: `ProcessInfo.processInfo.isLowPowerModeEnabled == true`.

**Precondition:** power saving off and the phone **not charging**. iPhone Low Power Mode switches itself off when the phone charges past 80%.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟢 `battery_plus` (`isInBatterySaveMode`) | 🟢 `battery_plus` (`isInBatterySaveMode`) |
| **Overall** | **🟢 Easy** | **🟢 Easy** |

Some Android brands' power savers behave differently, so test each brand you support.

## Technical documentation

**Android**
- `PowerManager.isPowerSaveMode()`.
- Changes arrive through the `PowerManager.ACTION_POWER_SAVE_MODE_CHANGED` broadcast.
- No permission is needed.
- Some brands' power savers (Samsung, Xiaomi, Huawei) can behave differently. Test on each brand you support.

**iOS**
- `ProcessInfo.processInfo.isLowPowerModeEnabled`.
- Changes arrive through the `.NSProcessInfoPowerStateDidChange` notification.
- No permission is needed.

**Flutter**
- `battery_plus`: `isInBatterySaveMode`. Re-check when the app resumes, and/or poll every second during the level.

**Side effects to allow while this level runs**
- **Pixel:** Battery Saver turns on **dark mode**, so the light-mode check must not complain during this level.
- **Samsung:** power saving can lower the screen brightness.
- **iPhone:** Low Power Mode caps some animations, and the battery icon turns yellow.

## Clues

1. **Vague** (the cat, hissing): "Way too much power! Something has to save energy!"
2. **Closer:** "Your phone has a special mode for saving energy too."
3. **Specific:** "Turn on Battery Saver (Android) or Low Power Mode (iPhone)."

**(?) Solution after 2 minutes:**
- **Android:** swipe down from the top and tap **Battery Saver** (Samsung: **Power saving**), or go to Settings > Battery.
- **iPhone:** Settings > Battery > turn on **Low Power Mode**. If you've added the battery button to Control Center, tap it there.

## After the level

**Card:** "Power saving was off when you started. Battery: 47%. No permission needed to see either."

**What you learned:** apps know when your phone is in power saving mode, and many change how they behave because of it.

**Trivia:**
- In 2016, Uber's head of economic research said riders with almost-dead batteries are more likely to accept higher "surge" prices. He also said Uber didn't use this to set prices. Still, your battery state says something about you.
- On iPhone, Low Power Mode turns itself off when the phone charges past 80%.
- On Pixel phones, Battery Saver switches the phone to dark mode.

**Final report entry:** "Power saving: off (battery 47%)."

## If the device can't do it

- **A brand whose power saver the API doesn't report:** if the setting never changes while the player is clearly in the battery settings, mark the level as not supported on that phone after the third clue.
- **The phone is charging:** ask the player to unplug first. On iPhone it may otherwise turn itself off at 80%.
- **Battery already low and power saving switched on automatically:** the start-of-game check (battery at 25% or more) prevents this. If it happens anyway, the level passes immediately, with a card explaining that the phone did it by itself.

## Reverting

An electrician arrives, unplugs half the devices and gives the strip a thumbs-up: "Back to normal. You can turn power saving off."
- **Player action:** turn power saving off.
- **Check:** power saving is false. On Pixel, dark mode turns off by itself when Battery Saver does. Re-check the light-mode baseline after reverting.
