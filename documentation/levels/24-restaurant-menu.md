# Level 24 · The Tiny Menu

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Text size (system font scale), plus a typed answer |
| **Permission** | None (Android and iOS) |
| **Support** | Android ✅ · iOS ✅ |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟢 Easy |
| **Setting to revert** | Text size back to what it was |
| **Reference** | [research.md §4.22](../research.md#422-text-size) |

## Scene

A fancy candle-lit restaurant with white tablecloths, a violinist in the corner and a fountain gurgling. The player's character sits at a table holding a huge leather menu, squinting. The text inside is microscopic: rows of grey specks.

A snooty waiter with a curled moustache stands over the table, tapping his pen on a notepad and his foot on the floor: "We don't have all night, *monsieur/madame*." Other diners turn to stare, and a queue forms at the door.

On the waiter's notepad is a text field, **"Your order:"**. The player must type the name of **the dish marked with a star ★**, which is the chef's special. The starred dish is picked at random each time, so it can't be guessed.

**The menu text follows the system text size, starting tiny.** It's drawn at a size that's unreadable at the player's *current* setting and readable once they increase it by about one or two steps. That way it works for players who already use large text.

Example dishes (plain words, easy to type):
- Grilled Salmon with Lemon Butter
- Wild Mushroom Risotto
- Spaghetti Carbonara
- Roast Chicken with Garlic Potatoes ★
- Chocolate Lava Cake

What happens:
- **Live feedback:** as the player increases the text size, the menu's specks grow into readable words. The waiter's foot-tapping speeds up the longer it takes.
- **Wrong order:** if the player types a wrong dish, the waiter scoffs "That is not on our menu" or "That is not the special".
- **Success:** the waiter bows: "Excellent choice." A plate arrives under a silver dome, the dome lifts in a cloud of steam, and the character tucks in happily.

## Goal

**Make the phone's text size bigger** so the menu becomes readable, then **type the starred dish name** exactly.

**Success condition:**
- The text scale at the menu's font size is at least 1.3× the value at the start of the level.
- **And** the typed text matches the starred dish, ignoring capitals, accents, punctuation and extra spaces.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing | 🟢 Nothing |
| **Code to get the value** | 🟢 Built into Flutter (`MediaQuery.textScalerOf`) | 🟢 Built into Flutter (`MediaQuery.textScalerOf`) |
| **Overall** | **🟢 Easy** | **🟢 Easy** |

## Technical documentation

**Flutter**
- `MediaQuery.textScalerOf(context)`.
- **Android 14+ scales text non-linearly** (large text grows less than small text). Measure the factor at the menu's real font size: `textScaler.scale(12) / 12`.
- Flutter rebuilds when the setting changes, so the menu grows by itself.
- Record the value when the level starts as the baseline.

**Android**
- `Configuration.fontScale` (default 1.0; Android 14 allows up to 2.0) and `onConfigurationChanged`.
- "Display size" is a different setting (screen density). Ignore it.

**iOS**
- `UIApplication.shared.preferredContentSizeCategory` (default `.large`) and `UIContentSizeCategory.didChangeNotification`.

**Gotchas**
- Only the menu should start tiny. **The rest of the game must still look right with large text,** because it will be large during this level, and for players who always use large text.
- Players who already use large text must never be asked to make it smaller. The goal is always *bigger than where you started*.
- The typing field also follows the text size, which helps.

## Clues

1. **Vague** (the character, squinting): "I can't read a single word on this menu…"
2. **Closer:** "Maybe the menu isn't too small. Maybe your phone's *letters* are."
3. **Specific:** "Make your phone's text size bigger in Settings, then read the dish with the star."

**(?) Solution after 2 minutes:**
- **Android (Pixel):** Settings > Display > Display size and text > **Font size**, drag right.
- **Android (Samsung):** Settings > Display > **Font size and style**, drag right.
- **iPhone:** Settings > Display & Brightness > **Text Size**, drag right. Or use the Text Size control in Control Center, if you've added it.

Then come back and type the dish with the ★.

## After the level

**Card:** "Text size: 100% when you started. No permission needed to see it, or your other accessibility settings."

**What you learned:** apps can see your text size and other accessibility settings without asking, so they can adapt to you.

**Trivia:**
- Apps can also check whether a screen reader (VoiceOver or TalkBack) is running. That's useful so they can adapt, but it can also reveal a disability, which is very sensitive information.
- Android 14 can make text up to twice as big. iPhones have even bigger "Accessibility sizes".
- Your text size, language, dark mode and time zone together help "fingerprint" your phone.

**Final report entry:** "Text size: 100% (default)."

## If the device can't do it

- **The player's text size is already the largest possible:** they can't increase it, so the menu starts readable. The level becomes "type the starred dish", and the card explains why.
- **The player used screen zoom/magnification instead** (they typed the right dish without changing the text size): the waiter says "How did you read that? Oh, a magnifying glass! Clever." Pass, with a card about text size.
- **Typing is difficult** (accessibility): accept a close match (one or two typos), or let the player tap the dish on the menu after the third clue.

## Reverting

The character finishes the meal and finds their glasses in a pocket: "Oh, *that's* where they were!"
- **Player action:** put the text size back where it was.
- **Check:** text scale within ±0.05 of the start of the level.
