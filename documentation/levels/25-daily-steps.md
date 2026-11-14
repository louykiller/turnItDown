# Level 25 · Doctor's Orders

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Step counter (with the accelerometer for instant feedback) |
| **Permission** | Android: "Physical activity" prompt (`ACTIVITY_RECOGNITION`, Android 10+). iOS: "Motion & Fitness" prompt (`NSMotionUsageDescription`). Both asked during setup |
| **Support** | Android ✅ (some cheap phones have no step sensor) · iOS ✅ |
| **Access difficulty: Android** | 🟡 Moderate |
| **Access difficulty: iOS** | 🟡 Moderate |
| **Setting to revert** | None |
| **Reference** | [research.md §4.24](../research.md#424-step-counter) |

## Scene

A bright, friendly doctor's office: an eye chart, a skeleton model wearing a party hat, and a jar of lollipops. A cheerful doctor with a stethoscope and an oversized fitness band on her wrist looks at a big screen: **"9,980 / 10,000 steps"**.

"You're **20 steps** short of your daily goal! Off you go, walk it off!" She points dramatically at the door.

The screen shows a park path with the player's character standing at the start, a big circular step dial, and a finish ribbon twenty steps away.

- **Live feedback:** each real step makes the character take a step along the path, with a little *tap* sound and a pop on the dial. The accelerometer detects steps instantly, so the animation feels responsive; the official step counter confirms them, sometimes a few seconds later.
- **Encouragement:** ducks waddle alongside, and joggers overtake the character.
- **Success:** at 20 steps, the character breaks the finish ribbon. Confetti, the doctor cheers "10,000!", a medal drops around the character's neck, and the skeleton waves its party hat.

## Goal

**Walk 20 steps** holding the phone, or with it in a pocket.

**Success condition:** at least 20 steps counted by the step counter since the level started. If there's no step counter or the permission is refused, count steps from the accelerometer instead.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟡 "Physical activity" prompt | 🟡 "Motion & Fitness" prompt |
| **Code to get the value** | 🟢 `pedometer` | 🟢 `pedometer` |
| **Overall** | **🟡 Moderate** | **🟡 Moderate** |

The accelerometer fallback needs no permission at all on either platform (🟢). That's the lesson of this level.

## Technical documentation

**Android**
- `Sensor.TYPE_STEP_DETECTOR`: one event per step, with a short delay. Best for live feedback.
- `Sensor.TYPE_STEP_COUNTER`: total steps since the phone last restarted. It can arrive in batches, several seconds late.
- Runtime permission `ACTIVITY_RECOGNITION` (Android 10+), shown as "Physical activity".
- Some cheap phones have neither sensor: `getDefaultSensor` returns `null`.

**iOS**
- `CMPedometer.isStepCountingAvailable()`, then `startUpdates(from: Date())` → `CMPedometerData.numberOfSteps`. Updates arrive in batches every few seconds.
- `queryPedometerData(from:to:)` returns *past* steps, reportedly for about the last 7 days **(verify)**.
- `CMMotionActivityManager` tells whether the user is walking, running, cycling or in a car.
- Permission: `NSMotionUsageDescription` (Motion & Fitness).

**Flutter**
- `pedometer`: `stepCountStream` gives a running total, so subtract the value at the start of the level. `pedestrianStatusStream` reports walking or stopped.
- Fallback: `sensors_plus` `userAccelerometerEventStream()` with peak detection (a step is a peak of about 1.2–2 m/s² with at least 250 ms between steps).

## Clues

1. **Vague** (the doctor): "Doctor's orders: MOVE!"
2. **Closer:** "Your phone counts something every time your foot hits the ground."
3. **Specific:** "Walk 20 steps while holding your phone."

**(?) Solution after 2 minutes:** hold the phone (or put it in your pocket) and walk 20 normal steps, for example across the room and back. Walking on the spot counts too.

## After the level

**Card:** "Physical activity: allowed. You walked 20 steps. Today you've walked 4,312 in total." The total is iPhone only.

**What you learned:** official step counting needs a permission. But the accelerometer, which needs **no** permission, can count your steps almost as well, and any app can do that silently.

**Trivia:**
- After you grant Motion & Fitness, an iPhone app can read your steps from the past days too, not just from now on.
- Your phone can also tell whether you're walking, running, cycling or driving ("activity recognition").
- Some health insurers offer discounts in exchange for your step data.

**Final report entry:** "Steps today: 4,312." On Android: "Steps during the game: 20."

## If the device can't do it

- **No step sensor, or the permission is refused:** count steps from the accelerometer (no permission needed). The card then explains: "We counted your steps *without* asking, using the accelerometer."
- **The player can't walk** (wheelchair users, injury): offer "Swing your arm 20 times" as an alternative, counted the same way from the accelerometer, or a skip.
- **A tiny room:** walking on the spot counts.

## Reverting

Nothing to revert. Optionally, the card shows how to remove the Physical activity / Motion & Fitness permission.
