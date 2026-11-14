# Level 23 · Silence the Alarm

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Clipboard |
| **Permission** | None for text copied inside the game. iOS 16+ asks "Allow Paste?" only for content copied in *another* app |
| **Support** | Android ✅ · iOS ✅ |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟡 Moderate |
| **Setting to revert** | None. The game clears the clipboard itself |
| **Reference** | [research.md §4.21](../research.md#421-clipboard) |

## Scene

Night. The player's character comes home, opens the front door, and the burglar alarm goes off: *WEEOO WEEOO WEEOO*. Red lights flash across the hallway, the dog howls along, and lights come on in the houses across the street.

By the door is a sleek alarm panel: **"SMART ALARM 3000 — enter your 32-character code"**. Its keypad is broken: the buttons have fallen off and lie on the floor.

On the fridge, held by a magnet, is a sticky note with the code, written as real text the player can select: `Xq7#Lm9!vR2@zK8$wN4&pT6*hY3^bJ5%`.

Under the panel, in tiny print: *"Smart Alarm 3000 automatically reads codes from your clipboard. For your convenience!"* That small print is both the hint and the lesson.

- **Escalation:** the siren gets louder, a neighbour shouts "TURN IT OFF!", and a police siren starts in the distance.
- **Partial copy:** if the player copies only part of the code, the panel beeps angrily: "INCOMPLETE CODE".
- **Success:** the instant the full code is on the clipboard, the panel reads it ("CODE ACCEPTED ✓"), the siren winds down with a sad *wooo…*, the lights go back to normal, and the dog stops howling, embarrassed.

## Goal

**Copy the code from the sticky note.** The alarm panel reads the clipboard.

**Success condition:** the clipboard text, with spaces trimmed, exactly matches the code.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing for text copied in the game (🟡 "Allow Paste?" if copied from another app) |
| **Code to get the value** | 🟢 Built into Flutter (`Clipboard`). Android can also report clipboard changes | 🟡 Reading is built in. Noticing changes without risking the prompt needs a small native `changeCount` check |
| **Overall** | **🟢 Easy** | **🟡 Moderate** |

## Technical documentation

**Flutter**
- Show the code as `SelectableText`, so long-press shows the system Copy / Select All menu.
- Read it with `Clipboard.getData(Clipboard.kTextPlain)`.
- Clear it afterwards with `Clipboard.setData(const ClipboardData(text: ''))`.
- There's no built-in change listener. Read the text only when the in-house plugin reports a change.

**Android**
- `ClipboardManager.getPrimaryClip()` and `addPrimaryClipChangedListener`.
- Since Android 10, only the focused app (or the keyboard) can read the clipboard.
- **Since Android 12, the system shows a toast when an app reads the clipboard.** A third-party guide says reading a clip the app wrote itself doesn't show it **(verify)**.
- Since Android 13, the system clears the clipboard automatically after a while.
- Sensitive text can be flagged `ClipDescription.EXTRA_IS_SENSITIVE` so it's hidden in the clipboard preview.

**iOS**
- `UIPasteboard.general.string` and `UIPasteboard.changedNotification`.
- `UIPasteboard.general.changeCount` can be read without a prompt; Firefox for iOS uses it for exactly that.
- **Since iOS 16, reading content placed by *another app* shows "Allow Paste?".** Since iOS 16.1, each app has a "Paste from Other Apps" setting.
- Text copied inside the game itself is expected not to prompt **(verify)**.
- `hasStrings` was reported to trigger the prompt on some iOS 16 versions, so prefer `changeCount`.

## Clues

1. **Vague** (the character, covering their ears): "That code is WAY too long to type… and the keypad is broken anyway!"
2. **Closer:** "Read the small print under the alarm. It's a *smart* alarm. Maybe you just need to hand it the code."
3. **Specific:** "Long-press the code on the sticky note and tap Copy."

**(?) Solution after 2 minutes:** press and hold on the code on the sticky note until it's highlighted. Tap **Select All** if not everything is selected, then tap **Copy**. The alarm reads it straight away.

## After the level

**Card:** "This alarm read your clipboard, and then cleared it for you. Any app you open can try to do the same."

**What you learned:** apps can read whatever you've copied: passwords, addresses, messages, bank details. Phones now warn you sometimes, but not always.

**Trivia:**
- In 2020, when iOS 14 started showing a banner every time an app read the clipboard, dozens of popular apps (TikTok among them) were caught reading it over and over.
- iOS 16 added the "Allow Paste?" prompt. Android 12 added a "pasted from your clipboard" message. Android 13 started clearing the clipboard automatically after a while.
- Good password managers clear the clipboard shortly after you copy a password, for exactly this reason.

**Final report entry:** "Clipboard: read once by this game, then cleared." Never show the content.

## If the device can't do it

- **iOS shows "Allow Paste?" and the player taps "Don't Allow":** show a small paste box, "Paste the code here". Pasting manually through the system menu never shows the prompt. Alternatively, use Apple's `UIPasteControl` system paste button (iOS 16+), which doesn't prompt.
- **The player can't long-press or select text** (accessibility): after the third clue, show a "Copy" button next to the code. It removes the puzzle, but keeps the lesson.
- **Clipboard managed by a work profile:** detect a failed read and offer the paste box.

## Reverting

The game clears the clipboard itself: "We don't leave alarm codes lying around. Your clipboard is empty now." The player has nothing to do. It's also a lesson: apps can *change* your clipboard too.
