# turnItDown — Game Notes

General notes for the whole game: what it is, how a session flows, the **step-by-step setup** that puts the phone in the right starting state, the rules every level follows, and the open decisions.

> **Scope:** the first version is **Android only** (see [research.md §0](research.md#0-scope-android-first)). The iOS details here and in the level files are kept for a later iOS version. In the setup steps, follow the Android parts.

Related documents:
- [plan.md](plan.md): the original idea and the level list.
- [research.md](research.md): APIs, packages, feasibility and access difficulty.
- [levels/](levels/): one detailed file per level.

---

## 1. The game in one paragraph

turnItDown is a cartoon mobile game made of 29 short levels, for Android first (iOS later). Each level is a silly everyday problem (a crying baby, a barking dog, a lost food courier) that can only be solved by using one of the phone's sensors or settings in the real world: dimming the screen, shaking the phone, turning on airplane mode, sticking your tongue out at the camera. After each level, the game shows what it just learned about the player and whether it needed permission to learn it. The answer is usually *no*. The game ends with a report: **"Here's what this game knows about you."**

**The message:** apps know far more about your phone, your surroundings and your habits than you think, and most of it needs no permission at all.

### Principles

| Principle | What it means in practice |
|---|---|
| **Privacy first** | Everything stays on the phone. No analytics, no ads, no accounts, no servers. The only network request is the time check during setup, which sends no personal data. A "Delete everything" button wipes all stored values. |
| **Show, don't lecture** | The player *does* something real, the game *reacts*, and only then does a short card explain. |
| **Short levels** | Every level should be solvable in under 2 minutes. Hints appear over time, and the solution after 2 minutes. |
| **Never stuck** | Every level has a Skip button and a "not supported on this device" path. No level requires a specific physical ability. |
| **Leave the phone as it was** | Every setting a level changes is put back afterwards, as part of the story. At the end, the player is guided back to their *own* original settings. |
| **Simple cartoon style** | Flat, colourful, friendly. Comic, never gory or mean. |

### Terminology (from the plan)

- **User:** the real person holding the phone.
- **Character / player:** the in-game version of the user.

---

## 2. Session flow

```
Install & open
  │
  ├─ Step 0 · Welcome ............ silently records the player's ORIGINAL settings
  ├─ Setup steps 1–15 ............ each step: do → verify → validate → next
  │
  ├─ Level loop (× 29)
  │    preconditions → play → hints at 30 / 60 / 90 s → (?) solution at 120 s
  │    → success → "What you learned" card → revert step (verified) → next level
  │
  ├─ Final report ................ "Here's what this game knows about you"
  └─ Restore your phone .......... back to the ORIGINAL settings from Step 0
```

**Returning players:** when the app opens again later, show a **check-up** screen: one list with every setup setting, its current value and a ✓ or ✗. The player sees all of them again (a reminder of what the game watches), fixes anything marked ✗ (for example, the phone charged overnight, or dark mode came on by schedule), and taps **Continue** once everything is ✓.

---

## 3. Initial setup (step by step)

This turns the plan's start checklist into a guided setup. **The player can't move to the next step until the current one passes.**

### 3.1 Verification and validation

Every step has up to two checks:
- **Verify:** the game *reads* the setting automatically and compares it with the target. This is live: the check mark changes as soon as the player changes the setting.
- **Validate:** a quick test that the result *actually works* for the game, or that it's stable. Examples: the player hears a test sound, the microphone meter moves, the value stays correct for 2 seconds.

**A step passes only when both checks pass.**

### 3.2 Setup rules

1. **One step per screen.** There's a progress bar ("Step 5 of 15"), the setting's name and its current value, and a **How do I do this?** button with instructions for the player's platform and brand (Pixel/Samsung/iPhone, detected with `device_info_plus`).
2. **Next stays disabled** until the step passes.
3. **Every step is shown, even when it's already fine.** The player always sees the setting, its current value and the ✓, and taps **Next** themselves. **This is on purpose: the setup doubles as a hint.** By the end, the player has seen every setting the game watches, which helps when they're stuck in a level later. The in-game text names the setting and says it will matter ("Remember this one!"), but never says which level uses it or how.
4. **Live checking:** use change listeners, re-check when the app comes back from Settings, and also poll every second as a safety net.
5. **Stable for 2 seconds:** a value must stay correct for 2 s before it counts, so passing over the target while dragging a slider doesn't count.
6. **Regression watch:** if a step that already passed breaks during a later step (for example, the player plugs the charger back in), show a banner, "Step 3 is no longer OK", and disable Next until it's fixed.
7. **Skipping:** a step can be skipped only where its card says so: when the device can't do it, or for permissions, which the player must be free to refuse. Skipping always shows which levels become unavailable.
8. **Don't rely on colour alone:** show ✓ / ✗ icons and text, not only green and red.
9. **Keep the screen awake** during setup (`wakelock_plus`), and save progress, so closing the app resumes at the same step.

### 3.3 The steps at a glance

| Step | Target | Main levels that need it | Skippable? |
|---|---|---|---|
| 0 | Welcome; record the original settings | Restoring at the end | — |
| 1 | All permissions: microphone, camera, motion, location, Face ID (iPhone) | 5, 8 (iPhone), 13, 18, 22, 25, 28 | Yes, per permission (see Step 1) |
| 2 | Battery at least 25% | 21 (and the light-mode baseline) | No (charge first) |
| 3 | Charger unplugged | 9, 21 | No |
| 4 | Power saving off | 21 | No |
| 5 | Light mode | 20 | Yes (accessibility; level 20 disabled) |
| 6 | Airplane mode off | 10 | No |
| 7 | Connected to Wi-Fi (and mobile network checked) | 29, Step 8's time check | Yes (level 29 adapts) |
| 8 | Automatic date & time | 11 | Only if locked by the device's manager |
| 9 | Location off | 22 | No |
| 10 | No headphones connected | 19, 2, 5 | No |
| 11 | Flashlight off | 12 | No |
| 12 | Volume about half | 2, 5 | No (sound test can be skipped by players who can't hear) |
| 13 | Brightness about half | 1, 8 | No |
| 14 | Well-lit room | 8, 18 | No (manual confirmation if it can't be measured) |
| 15 | Final check and baseline snapshot | All | — |

**Why this order:**
- Battery comes before unplugging, because a low phone has to charge first.
- Power saving comes before light mode, because Pixel's Battery Saver forces dark mode.
- Airplane mode and Wi-Fi come before the time check, which needs internet.
- The Settings-app steps (time, location) are grouped together.
- Sound and screen come near the end, because players fiddle with them.
- The final check catches anything that changed along the way.

**About "Accept all permissions" in the plan:** Step 1 asks for **every** permission the game will ever need, including location and Face ID. That way no prompt appears in the middle of the game and gives away which level uses it. The player can still continue if they refuse one: Apple's App Store rule 5.1.1(iv) forbids forcing people to accept data access. Refusing only affects the matching levels (see Step 1).

### 3.4 Step cards

The **Why** rows are for the team. Don't show them in the game: they would give the levels away.

#### Step 0 · Welcome

| | |
|---|---|
| **Player does** | Reads a short welcome: what the game is, that it will read the phone's settings, and that nothing leaves the phone. Taps **Let's go**. |
| **Game does (silently)** | Records the player's **original state** before asking them to change anything: brightness, auto-brightness (Android), volume, light/dark mode, airplane mode, Wi-Fi, location, power saving, charging, Bluetooth audio, text size, time zone. These are used at the end to restore the phone. |
| **Verify / Validate** | — |
| **Awareness hook** | The final report later reveals: "Before you touched anything, we had already read all of this." |

#### Step 1 · All permissions

| | |
|---|---|
| **Player does** | Answers every system prompt the game will ever need, one at a time. Each comes with a neutral one-line reason that doesn't point to a specific level: **Microphone** ("so the game can hear you"), **Camera** ("so the game can see you"), **Motion & Fitness** on iPhone / **Physical activity** on Android ("so the game can feel how you move"), **Location** ("so the game can know where you are"), and, on iPhones with Face ID, **Face ID** ("so the game can recognise you"). |
| **Why all here** | Asking everything up front means no prompt appears mid-game and gives a level away. For example, a location prompt in level 22 would reveal that the level needs location. |
| **Why** | Microphone → level 5. Camera → level 18, plus level 8 and Step 14 on iPhone (light estimate). Motion → levels 13 (iPhone) and 25. Location → level 22 (its ending depends on precise / approximate / refused). Face ID → level 28. |
| **Verify** | `permission_handler` reports the status of each one. For location, also record *precise* or *approximate* (`geolocator` `getLocationAccuracy()`); level 22 uses it. |
| **Validate** | **Microphone:** a meter moves when the player says "hello". **Camera:** the front camera finds a face. These two catch Android 12+'s Quick Settings privacy toggles, which can block the microphone or camera even when the permission is granted. **Motion:** the authorization status reads *authorized*. **Face ID:** iOS only shows its prompt during a real Face ID check, so the step runs one quick scan ("Look at your phone"). |
| **No prompt needed** | **Fingerprint** (Android, and iPhones with Touch ID) needs no permission. Show it in the list anyway, marked "no permission needed", so the player sees it too. |
| **Platform note** | On iPhone, if Location Services are off, iOS may not show the location prompt **(verify)**. If that happens, level 22 asks right after the player turns location on. Android shows the prompt even with location off. |
| **If refused** | Show what changes. Microphone or camera refused → those levels become skippable. Motion refused → steps and height are measured with the accelerometer instead. Location refused → level 22 still works, with its "refused" ending. Face ID refused → level 28 uses the phone's passcode. Offer an **Open settings** button. The player may continue. |
| **Skippable** | Yes, per permission. |

#### Step 2 · Battery at least 25%

| | |
|---|---|
| **Player does** | Nothing if the battery is at 25% or more. Otherwise, charges until it is. |
| **Why** | Near 20%, phones start switching on power saving by themselves (iPhone suggests it; many Androids do it automatically). That would break level 21 and, on Pixel, the light-mode baseline. |
| **Verify** | `battery_plus` `batteryLevel ≥ 25`. |
| **Validate** | — |
| **Skippable** | No. While charging, the step shows the percentage climbing, and the next step asks to unplug. |

#### Step 3 · Charger unplugged

| | |
|---|---|
| **Player does** | Unplugs the charger, or lifts the phone off a wireless charger. |
| **Why** | Level 9 asks the player to plug in. On iPhone, Low Power Mode (level 21) switches itself off while charging past 80%. |
| **Verify** | Android: `EXTRA_PLUGGED == 0`. iOS: `batteryState == .unplugged` (both through `battery_plus`). |
| **Validate** | Stays unplugged for 2 s. |
| **Skippable** | No. |

#### Step 4 · Power saving off

| | |
|---|---|
| **Player does** | Turns off Battery Saver / Power saving (Android) or Low Power Mode (iPhone). |
| **How** | Android: Quick Settings tile **Battery Saver** / **Power saving**, or Settings > Battery. iPhone: Settings > Battery > **Low Power Mode**, or the Control Center battery button. |
| **Why** | Level 21. On Pixel, turning Battery Saver off also turns forced dark mode off, which Step 5 needs. |
| **Verify** | `battery_plus` `isInBatterySaveMode == false`. |
| **Validate** | Stays off for 2 s. |
| **Skippable** | No. Some brands' power savers aren't reported. If the player says it's off and the game still reads *on* after the third hint, mark level 21 as not supported and continue. |

#### Step 5 · Light mode

| | |
|---|---|
| **Player does** | Switches the phone to light mode. If dark mode runs on a schedule, sets it to manual for the duration of the game. |
| **How** | Android: Quick Settings **Dark theme** tile, or Settings > Display > Dark theme (schedule: *None*). iPhone: Settings > Display & Brightness > **Light**, with **Automatic** off. |
| **Why** | Level 20 asks for dark mode. |
| **Verify** | `MediaQuery.platformBrightnessOf(context) == Brightness.light`. |
| **Validate** | Stays light for 2 s. |
| **Skippable** | Yes, for players who rely on dark mode (light sensitivity). Level 20 is then disabled. |

#### Step 6 · Airplane mode off

| | |
|---|---|
| **Player does** | Makes sure airplane mode is off. |
| **Why** | Level 10 asks for airplane mode. Step 8 needs internet. |
| **Verify** | Android: `Settings.Global.AIRPLANE_MODE_ON == 0`. iOS (no API): a mobile network is present (`serviceCurrentRadioAccessTechnology` not empty), or there's an internet connection. |
| **Validate** | Stays off for 2 s. |
| **If it can't be verified** | Wi-Fi-only iPad: an internet connection is enough. |
| **Skippable** | No. |

#### Step 7 · Connected to Wi-Fi

| | |
|---|---|
| **Player does** | Connects to a Wi-Fi network. |
| **Why** | Level 29 asks the player to switch from Wi-Fi to mobile data. Step 8's time check needs internet. |
| **Verify** | `connectivity_plus` result contains `wifi`. |
| **Also checked (information only)** | Is there a mobile network to switch to? Android: `TelephonyManager.getSimState() == SIM_STATE_READY`. iOS: `serviceCurrentRadioAccessTechnology` not empty. If not, the card says level 29 will be unavailable. |
| **Validate** | Stays connected for 2 s. |
| **Skippable** | Yes. No Wi-Fi nearby → level 29 uses its flipped version ("switch to Wi-Fi") or is disabled. |

#### Step 8 · Automatic date & time

| | |
|---|---|
| **Player does** | Makes sure the clock is set automatically. |
| **How** | Android (Pixel): Settings > System > Date & time > **Set time automatically**. Android (Samsung): Settings > General management > Date and time > **Automatic date and time**. iPhone: Settings > General > Date & Time > **Set Automatically**. |
| **Why** | Level 11 asks the player to move the clock back. The game must know the clock was correct to begin with. |
| **Verify** | Android: `Settings.Global.AUTO_TIME == 1`. iOS: the setting can't be read; validation covers it. |
| **Validate** | The phone's clock is within ±30 s of an internet time server (`ntp` package), on both platforms. The game also records the time zone. |
| **If it can't be validated** | Offline: on Android, the setting alone is enough. On iPhone, the player confirms "Set Automatically is on" with a button. |
| **Skippable** | Only if the clock is locked by device management (work/school phone). Level 11 is then disabled. |

#### Step 9 · Location off

| | |
|---|---|
| **Player does** | Turns location (Location Services on iPhone) off. |
| **How** | Android: Quick Settings **Location** tile. iPhone: Settings > Privacy & Security > **Location Services**. |
| **Why** | Level 22 asks the player to turn it on. |
| **Verify** | Android: `LocationManager.isLocationEnabled() == false`. iOS: `CLLocationManager.locationServicesEnabled() == false`. No permission is needed for either. |
| **Validate** | Stays off for 2 s. |
| **Note** | Tell the player it's only for the game, and the restore step at the end will remind them to turn it back on. |
| **Skippable** | No. If location is locked by device management, level 22 starts directly at the permission part. |

#### Step 10 · No headphones connected

| | |
|---|---|
| **Player does** | Disconnects Bluetooth headphones and unplugs wired ones, so sound comes from the phone itself. |
| **Why** | Level 19 asks the player to connect headphones. Levels 2 and 5 need the phone's own speaker and microphone. |
| **Verify** | The current audio output is the built-in speaker: no Bluetooth, wired or USB audio device (`audio_session`). |
| **Validate** | Covered by Step 12's sound test. |
| **Note** | Watches, cars and other non-audio Bluetooth devices don't matter. iPhones can't see them anyway, and the plan's "no Bluetooth connections" really means "no Bluetooth *audio*". |
| **Skippable** | No. |

#### Step 11 · Flashlight off

| | |
|---|---|
| **Player does** | Turns the flashlight off. |
| **Why** | Level 12 asks the player to turn it on. |
| **Verify** | Android: torch callback reports *off*. iPhone: `isTorchActive == false`, *if* it turns out iOS reports the Control Center flashlight (see [research.md §9](research.md#9-things-to-test-on-real-devices-first)). |
| **If it can't be verified** | The player confirms "My flashlight is off". |
| **Skippable** | No. |

#### Step 12 · Volume about half

| | |
|---|---|
| **Player does** | Uses the volume buttons to set the media volume to roughly half. |
| **Why** | Level 2 turns the volume down, and level 5 notices volume changes. Starting at half gives room in both directions. |
| **Verify** | Media volume between 40% and 60%. That's 6–9 of 15 steps on a Pixel, or 7–9 of 16 on an iPhone. |
| **Validate** | The game plays a short sound (for example a cartoon *boing*), and the player taps which of three pictures they heard (boing / bark / bell). This proves the sound is audible and comes from the phone. |
| **Platform note** | On iPhone, the volume buttons only change the media volume while sound is playing, so this step plays quiet background music. |
| **Skippable** | No. Players who can't hear can skip the sound test only. |

#### Step 13 · Brightness about half

| | |
|---|---|
| **Player does** | Sets the screen brightness to roughly half. Recommended: turn auto-brightness off. |
| **How** | Android: swipe down for the brightness slider (auto-brightness: Settings > Display > **Adaptive brightness**). iPhone: Control Center brightness bar (auto-brightness: Settings > Accessibility > Display & Text Size > **Auto-Brightness**). |
| **Why** | Level 1 asks for minimum brightness. With auto-brightness on, level 8 (covering the light sensor) also dims the screen. |
| **Verify** | Slider position between 40% and 60%. On Android this is converted from the stored value, because the slider is a curve ([research.md §4.1](research.md#41-display-brightness)). On iOS, `UIScreen.brightness` 0.4–0.6. |
| **Validate** | Stays in range for 2 s. Auto-brightness on (readable on Android only) shows a non-blocking warning. |
| **Skippable** | No. |

#### Step 14 · Well-lit room

| | |
|---|---|
| **Player does** | Moves somewhere bright, or turns on a light. |
| **Why** | Level 8 needs light to block. Level 18's face tracking needs a visible face. |
| **Verify** | Android: light sensor ≥ 100 lux. iPhone: front-camera brightness estimate above the tuned threshold (uses the camera permission from Step 1). |
| **Validate** | Stays bright for 3 s, so a passing lamp flash doesn't count. |
| **If it can't be verified** | No light sensor and no camera permission: the player confirms "I'm in a bright room". |
| **Skippable** | No. |

#### Step 15 · Final check and baseline

| | |
|---|---|
| **Game does** | Re-runs Steps 2–14 together. Everything must be green at the same time for 3 s. |
| **Then** | Saves the **baseline snapshot** that every revert is compared against (§4.3), and shows a summary. |
| **Awareness teaser** | Under the summary: "We also noted these, without asking: text size 100%, time zone Europe/Lisbon, language English, phone model Pixel 8, Android 16." This comes from `MediaQuery`, `DateTime`, `Platform.localeName` and `device_info_plus`. |
| **Player does** | Taps **Start the game**. |

---

## 4. Rules for every level

### 4.1 Level loop

1. **Device support check:** if the hardware or OS can't do the level, show the "not supported" card and offer Next.
2. **Permission check:** if a needed permission was refused, show a skip card, with an "Open settings" button to reconsider.
3. **Preconditions:** level-specific checks (for example, level 2 needs the volume at 30% or more). If one isn't met, a character asks the player to fix it in the story ("the party starts quietly… turn it up!").
4. **Play:** live feedback from the sensor or setting.
5. **Hints:** clue 1 at 30 s, clue 2 at 60 s, clue 3 at 90 s, and the (?) solution at 120 s. Each level file has the text. Time this with a monotonic `Stopwatch`, so level 11's clock change can't break it. Keep counting while the player is in the Settings app.
6. **Skip:** always available.
7. **Success:** the success animation, then the **"What you learned"** card (§5.1).
8. **Revert:** if the level changed a setting, a story scene asks the player to put it back. A live checklist ticks as they do (§4.3). On Android, a button opens the exact settings page. On iPhone, text instructions are shown, because iOS can't open specific settings pages.
9. **Next level.**

### 4.2 Global technical rules

- **Portrait only,** so tilt levels don't rotate the screen.
- **Keep the screen awake** during levels (`wakelock_plus`). A 30 s auto-lock would interrupt levels 3, 4, 13 and 25, and complete level 27 by accident.
- **Re-check everything when the app resumes.** Listeners can miss changes made while the player was in Settings.
- **Turn hardware off when a level ends:** the camera (so the privacy dot disappears), the microphone, and iOS proximity monitoring (otherwise the screen keeps going black).
- **Debug overlay and fake sensors** for development and threshold tuning (see [research.md §7](research.md#7-suggested-project-structure)).

### 4.3 Baseline snapshot and reverting ("StateGuard")

The snapshot from Step 15 is the reference. After each level, the tracked settings are compared with it, using **ranges, not exact values**.

| Setting | Baseline (from setup) | Accepted when reverting | Changed by level |
|---|---|---|---|
| Brightness | 40–60% | 25–90%, or within 10% of the baseline | 1 |
| Volume | 40–60% | 30–70%, or within 1 step of the baseline | 2, 5 (tease only) |
| Charging | unplugged | unplugged | 9 |
| Airplane mode | off | off | 10 |
| Clock | automatic, ±30 s | within ±1 min of expected time. Android: `AUTO_TIME == 1` | 11 |
| Flashlight | off | off | 12 |
| Bluetooth audio | none | none | 19 |
| Light/dark mode | light | light | 20 |
| Power saving | off | off | 21 |
| Location | off | off | 22 |
| Clipboard | — | cleared by the game itself | 23 |
| Text size | recorded | within ±0.05 of the baseline | 24 |
| Wi-Fi | connected | connected | 29 |

**Changes to ignore while a specific level is running** (side effects, not mistakes):

| Level | Ignore | Because |
|---|---|---|
| 6 | Volume ±1 step | The screenshot button combination includes a volume button |
| 8 | Brightness | Auto-brightness dims the screen when the sensor is covered |
| 9 | Battery level | It's charging |
| 10 | Wi-Fi, Bluetooth audio, network | Airplane mode switches radios off |
| 17 | Screenshot event | Some brands take a screenshot on a 3-finger swipe |
| 19 | Volume | Headphones keep their own volume level |
| 21 | Dark mode, brightness | Pixel's Battery Saver forces dark mode; Samsung's power saving dims the screen |
| 29 | Network | That's the level |

After a revert, re-check **every** tracked setting, not just the one the level changed. Some side effects only clear when the original setting is reverted (for example, Pixel dark mode turns off with Battery Saver).

### 4.4 Interactions between settings to remember

| Interaction | Consequence |
|---|---|
| Pixel Battery Saver → dark mode | Level 21 and the light-mode baseline; Step 4 comes before Step 5 |
| iPhone Low Power Mode turns off when charging past 80% | Level 9's revert (unplug) must come before level 21 |
| Auto-brightness + covering the light sensor | Level 8 dims the screen; recommended off in Step 13 |
| Screenshot combination uses a volume button | Level 6 may move the volume a step |
| 3-finger screenshot gesture (Xiaomi, OnePlus, Oppo, Realme, Vivo) | Level 17's pinch may take a screenshot |
| Auto-lock | Interrupts physical levels and fakes level 27, hence the wakelock |
| Changing the clock | Breaks timers that use the wall clock, so use `Stopwatch` |
| Headphones connected | The game's sound and microphone move to the headset (levels 2, 5); Step 10 and level 19's revert |
| Airplane mode | Wi-Fi and Bluetooth go off too; the revert brings them back (check in level 10's revert) |

---

## 5. Learning content

### 5.1 "What you learned" card (after every level)

Same layout every time:
1. **The value read**, with the player's real data: "Battery: 47%".
2. **Permission needed?** A big **No** (green) or **Yes, [permission name]** (amber).
3. **What it means:** one sentence.
4. **Trivia:** one or two facts, from the level file.
5. **Added to your report:** the line it adds to the final report.

### 5.2 Final report: "Here's what this game knows about you"

- One screen, grouped into **Without asking** and **With your permission**, built from each level's "Final report entry" (see the level files) plus what was recorded at Step 0 and Step 15.
- A headline number, for example: "We learned **23 things** about you. **19** of them needed no permission."
- **Sharing:** the player can screenshot it (the game will notice, of course). Before showing location or headphone names, ask: "This includes your approximate area. Show it?"
- **Delete everything** button at the bottom.

### 5.3 Restore your phone (end of game)

A final checklist that compares the current settings with the **original state recorded at Step 0**. For example: "Location was ON before we started. Turn it back on?", "Dark mode was ON…", "Auto-brightness was ON…", "Your headphones were connected…". It uses the same live checks as setup, but nothing blocks: the player can tick off or ignore each item.

---

## 6. Permissions summary

| Permission | When it's asked | Levels |
|---|---|---|
| Microphone | Setup Step 1 | 5 |
| Camera | Setup Step 1 | 18 (and 8 + Step 14 on iPhone) |
| Motion & Fitness (iPhone) / Physical activity (Android) | Setup Step 1 | 13 (iPhone), 25 |
| Location | Setup Step 1 | 22 |
| Face ID (iPhones with Face ID) | Setup Step 1 (a quick Face ID check) | 28 |
| Screenshot detection, biometrics, network state (Android) | Never shown (install-time) | 6, 28, 29 |

Every prompt happens during setup, so none appears mid-game and gives a level away.

Full lists for the manifest and Info.plist: [research.md §8](research.md#8-permissions-and-store-policy).

---

## 7. Level index

Ratings are the access difficulty from each level file (🟢 Easy · 🟡 Moderate · 🟠 Hard · 🔴 Very hard).

| # | Level | Uses | Android | iOS | Revert |
|---|---|---|---|---|---|
| 1 | [Lights Out for the Baby](levels/01-crying-baby.md) | Brightness | 🟡 | 🟢 | Brightness up |
| 2 | [The Neighbour's Nightmare](levels/02-loud-speaker.md) | Volume | 🟢 | 🟢 | Volume up |
| 3 | [The Overflowing Sink](levels/03-overflowing-sink.md) | Gyroscope | 🟢 | 🟢 | — |
| 4 | [Hang Up That Phone](levels/04-ringing-landline.md) | Proximity + accelerometer | 🟡 | 🟡 | — |
| 5 | [The Dog That Won't Stop Barking](levels/05-barking-dog.md) | Microphone | 🟡 | 🟡 | Volume (if teased) |
| 6 | [Spirit Snapshot](levels/06-ghost-hunter.md) | Screenshot | 🟢 (14+) / 🔴 | 🟢 | — |
| 7 | [Point the Remote](levels/07-tv-remote.md) | Accelerometer | 🟢 | 🟢 | — |
| 8 | [Sun in Your Eyes](levels/08-sun-glare.md) | Light sensor / camera | 🟢 | 🟠 | — |
| 9 | [Plug In the Patient](levels/09-hospital-machine.md) | Charging | 🟢 | 🟢 | Unplug |
| 10 | [Ready for Take-off](levels/10-flight-attendant.md) | Airplane mode | 🟢 | 🟠 | Airplane off |
| 11 | [Rewind the Crime](levels/11-time-travel.md) | Date/time | 🟢 | 🟢 | Automatic time on |
| 12 | [Searchlight](levels/12-prison-escape.md) | Flashlight | 🟡 | 🔴 (unverified) | Flashlight off |
| 13 | [Present!](levels/13-attendance-check.md) | Barometer | 🟢 | 🟡 | — |
| 14 | [Apply Pressure](levels/14-bleeding-elbow.md) | Touch size | 🟢 | 🟢 | — |
| 15 | [Hang It Straight](levels/15-crooked-painting.md) | Accelerometer | 🟢 | 🟢 | — |
| 16 | [Swat the Fly](levels/16-buzzing-fly.md) | Compass + gyroscope | 🟡 | 🟡 | — |
| 17 | [Shut the Big Mouth](levels/17-big-mouth.md) | Multi-touch | 🟢 | 🟢 | — |
| 18 | [Copycat](levels/18-tongue-out-kid.md) | Camera + face expressions | 🔴 / 🟠 | 🟠 | — |
| 19 | [Peace and Quiet on the Bus](levels/19-loud-bus.md) | Bluetooth audio | 🟢 | 🟢 | Disconnect headphones |
| 20 | [Goodnight, Rooster](levels/20-crowing-rooster.md) | Dark mode | 🟢 | 🟢 | Light mode |
| 21 | [Overloaded](levels/21-sparking-power-strip.md) | Power saving | 🟢 | 🟢 | Power saving off |
| 22 | [Where's My Food?](levels/22-lost-delivery.md) | Location | 🟢 / 🟡 | 🟢 / 🟡 | Location off |
| 23 | [Silence the Alarm](levels/23-house-alarm.md) | Clipboard | 🟢 | 🟡 | Game clears clipboard |
| 24 | [The Tiny Menu](levels/24-restaurant-menu.md) | Text size | 🟢 | 🟢 | Text size back |
| 25 | [Doctor's Orders](levels/25-daily-steps.md) | Step counter | 🟡 | 🟡 | — |
| 26 | [Stubborn Ketchup](levels/26-stuck-ketchup.md) | Shake | 🟢 | 🟢 | — |
| 27 | [3 A.M. Gamer](levels/27-caught-gaming.md) | Screen lock | 🟡 | 🟠 | — |
| 28 | [Forgot My PIN](levels/28-bank-vault.md) | Biometrics | 🟢 | 🟡 / 🟢 | — |
| 29 | [Endless Buffering](levels/29-buffering-video.md) | Wi-Fi vs mobile data | 🟢 | 🟢 | Wi-Fi on |

**Pacing:** alternate levels that send the player to the Settings app (10, 11, 20, 21, 22, 24, 29) with quick physical ones (3, 7, 15, 17, 26). Spread out the levels that rely on permissions (5, 13, 18, 22, 25, 28), so players don't link them to the permissions screen too easily. Place the riskiest levels (12 on iPhone, 18 on Android) where a skip doesn't hurt the flow.

---

## 8. Content and branding guidelines

- **Original everything.** No real brands, logos or famous characters: no "Ghostbusters" (level 6), no YouTube (level 29), no real banks (level 28) or delivery apps (level 22). App stores reject apps that use other companies' trademarks.
- **Cartoon tone.** Level 11's crime scene uses comic "X" eyes and police tape, never gore. Injuries (level 14) stay mild.
- **Be kind.** Jokes come from situations, not from groups of people. Characters are varied in age, gender and look.
- **Age rating:** aim for a rating that schools can use. Avoid suggestive content (this is why level 27 became gaming at 3 a.m.).
- **Accessibility:** every physical level has an alternative or a skip, all text respects the system text size (except the menu in level 24, on purpose), and check marks use icons, not only colour.

---

## 9. Store and legal checklist

**Google Play (now):**
- [ ] Privacy policy (required for camera and microphone use). Say clearly that nothing leaves the phone.
- [ ] Data safety form: no data collected or shared.
- [ ] Target the API level Google Play currently requires.
- [ ] Permissions are never forced: every refusal leads to a skip, not a dead end.
- [ ] Remove `READ_MEDIA_IMAGES` if a plugin adds it (Google Play photo/video policy).
- [ ] Android 14 screenshot detection: tell the player in context before level 6 that screenshots will be detected. Google recommends this, because the system shows a notice.
- [ ] Age rating questionnaire (IARC, through the Play Console).

**App Store (iOS, later):**
- [ ] App Store privacy label: *Data not collected*.
- [ ] Purpose strings that explain each permission in plain words.
- [ ] App Store rule 5.1.1(iv): permissions are never forced.
- [ ] No private APIs (for example, no `App-Prefs:` links, no private lock notification).

---

## 10. Open decisions

| # | Decision | Where |
|---|---|---|
| 1 | Level 18 on Android: different expression (recommended), a custom tongue model, or a heuristic | [levels/18](levels/18-tongue-out-kid.md) |
| 2 | Minimum Android version (recommended: Android 10, API 29) | [research.md §0](research.md#0-scope-android-first) |
| 3 | Level 13: use the Android device name for the roll call? (verify it's readable) | [levels/13](levels/13-attendance-check.md) |
| 4 | Level order and chapters (see the pacing note) | §7 |
| 5 | Languages: English only, or also Portuguese (and others)? This affects level 24's dish names and all card text | — |
| 6 | Game name and art direction (palette, character style) | — |
| 7 | *(iOS, later)* Level 12 on iPhone: depends on whether iOS reports the Control Center flashlight (device test). Fallback: the game switches the flashlight on itself | [levels/12](levels/12-prison-escape.md) |
| 8 | *(iOS, later)* Level 27 on iPhone: test how reliable the lock guess is | [levels/27](levels/27-caught-gaming.md) |
