# Level 20 · Goodnight, Rooster

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Light/dark mode (the system appearance) |
| **Permission** | None (Android and iOS) |
| **Support** | Android 10+ ✅ · iOS 13+ ✅ |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟢 Easy |
| **Setting to revert** | Back to light mode |
| **Reference** | [research.md §4.18](../research.md#418-lightdark-mode) |

## Scene

A sunny countryside farm. Rolling green hills, a red barn, a windmill turning lazily, and a big smiling sun with sunglasses in a bright blue sky. In the farmhouse window, the player's character lies in bed trying to take a nap, with an eye mask pushed up on their forehead.

On the wooden fence right outside, a proud, scruffy rooster puffs out its chest and crows non-stop: "COCK-A-DOODLE-DOOOO!" Each crow is louder than the last, and feathers fly.

- **Live feedback:** none until the switch. The rooster just keeps crowing.
- **The game's own look:** the game should not change its interface colours when dark mode turns on. Only this scene reacts, so the switch feels like part of the story.
- **Success:** the moment dark mode turns on, the sun drops behind the hills in a quick sunset, the sky turns deep blue and stars twinkle on. A sleepy moon in a nightcap rises, presses a finger to its lips and says "Shhhh!" The rooster freezes mid-crow, tucks its head under its wing and falls asleep. The character pulls down the eye mask and snores happily.

## Goal

**Switch the phone to dark mode.**

**Success condition:** the platform brightness is `Brightness.dark`.

**Precondition:** light mode at the start. This is also a start-of-game check.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟢 Built into Flutter | 🟢 Built into Flutter |
| **Overall** | **🟢 Easy** | **🟢 Easy** |

## Technical documentation

**Flutter**
- `MediaQuery.platformBrightnessOf(context)`, or `WidgetsBinding.instance.platformDispatcher.platformBrightness`.
- Get notified with `WidgetsBindingObserver.didChangePlatformBrightness()`, or by rebuilding from `MediaQuery`.

**Android**
- `resources.configuration.uiMode and Configuration.UI_MODE_NIGHT_MASK == UI_MODE_NIGHT_YES`.
- Flutter's default Android setup already handles the `uiMode` change without restarting the activity.

**iOS**
- `UITraitCollection.current.userInterfaceStyle == .dark`. Changes arrive through `traitCollectionDidChange`, or `registerForTraitChanges` on iOS 17+.

**Gotchas**
- **Pixel's Battery Saver turns on dark mode.** If the player is in Battery Saver, the start check fails. Also, [level 21](21-sparking-power-strip.md) can switch this on by itself.
- Some phones switch to dark mode on a schedule (at sunset or bedtime). If the level runs at that moment, the change happens without the player doing anything. That's rare, and acceptable.

## Clues

1. **Vague** (the character, from bed): "That rooster only shuts up when the sun goes down…"
2. **Closer:** "You can't move the sun… but can you make it night on your phone?"
3. **Specific:** "Switch your phone to dark mode."

**(?) Solution after 2 minutes:**
- **Android:** swipe down from the top and tap **Dark theme** (or **Dark mode**), or go to Settings > Display > Dark theme.
- **iPhone:** open Control Center, touch and hold the brightness slider, then tap **Dark Mode**. Or go to Settings > Display & Brightness > **Dark**.

## After the level

**Card:** "You use light mode. No permission needed to know that, or any of your display preferences."

**What you learned:** apps can see your light/dark mode, your text size, your language and many other display settings, silently. Each one alone says little. Together, they help "fingerprint" your phone.

**Trivia:**
- On phones with OLED screens, dark pixels use less power. Google showed at a 2018 developer conference that dark mode can noticeably cut screen power use.
- On Pixel phones, Battery Saver turns on dark mode automatically.
- Websites can read your dark mode setting too.

**Final report entry:** "Appearance: light mode."

## If the device can't do it

- **Android 9 or older** (no system dark mode): mark the level as not supported.
- **Dark mode already on and stuck** (for example, forced by Battery Saver): ask the player to turn Battery Saver off first, or skip.
- **The player uses dark mode for accessibility** (light sensitivity): the start checklist asks for light mode, so offer a skip that keeps dark mode, with a card explaining the level.

## Reverting

The sun pops back up, refreshed, with a coffee mug: "Good morning!" The rooster stays asleep this time.
- **Player action:** switch back to light mode.
- **Check:** platform brightness is `light`.
