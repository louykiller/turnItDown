# Level 05 · The Dog That Won't Stop Barking

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Microphone (loudness), plus the volume level for the tease |
| **Permission** | Microphone: a runtime prompt on Android (`RECORD_AUDIO`) and iOS (`NSMicrophoneUsageDescription`), asked during setup |
| **Support** | Android ✅ · iOS ✅ |
| **Access difficulty: Android** | 🟡 Moderate |
| **Access difficulty: iOS** | 🟡 Moderate |
| **Setting to revert** | Volume, only if the player turned it down |
| **Reference** | [research.md §4.5](../research.md#45-microphone) and [§4.2](../research.md#42-volume) |

## Scene

A suburban back garden at night under a full moon. A big fluffy dog stands behind a white picket fence, barking furiously at a cat sitting smugly on top of the fence. Every bark throws a "WOOF!" bubble that bounces across the screen.

One by one, the windows of the houses behind light up as neighbours wake. The player's character leans out of a window in pyjamas, hair messy, holding a pillow over their head.

- **Barking rhythm:** the dog barks in bursts of three (*WOOF WOOF WOOF*), then pauses for about 1.5 s while panting. The game listens during those pauses, so its own barking doesn't count as the player's shout.
- **Live feedback:** a "shout-o-meter" (a speech bubble coming from the character's window) grows with the player's real voice level and fills towards a threshold line.
- **Tease:** if the player turns the volume *down* during the level, a sign pops up: *"Just because you can't hear the dog doesn't mean it stopped barking."* The dog keeps barking with its sound muted, and the neighbours keep waking up.
- **Success:** a loud shout during a pause makes the dog's ears go flat. It whimpers, lies down and covers its eyes with its paws. The cat jumps off the fence, the windows go dark one by one, and crickets chirp.

## Goal

**Shout something loud and short** at the phone ("SHHH!", "QUIET!", "STOP!") while the dog pauses.

**Success condition:**
- During the first second of the level, measure the room's background noise (baseline).
- Success when the microphone level is at least 20 dB above the baseline (or above −20 dBFS) for at least 400 ms, during a pause in the barking.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟡 Microphone prompt | 🟡 Microphone prompt |
| **Code to get the value** | 🟢 `record` or `noise_meter` | 🟢 `record` or `noise_meter` |
| **Overall** | **🟡 Moderate** | **🟡 Moderate** |

The prompt is shown during setup, together with all the others, so it doesn't give this level away.

## Technical documentation

**Android**
- `AudioRecord`, computing RMS → dBFS from the buffer, or `MediaRecorder.getMaxAmplitude()`.
- Permission `RECORD_AUDIO`, requested at runtime.
- To remove the game's own sound from the microphone: the `MediaRecorder.AudioSource.VOICE_COMMUNICATION` source, plus `AcousticEchoCanceler` if available.

**iOS**
- An `AVAudioEngine` tap on the input node, computing RMS → dBFS. Alternatively, `AVAudioRecorder` with `isMeteringEnabled` and `averagePower(forChannel:)`.
- Audio session category `.playAndRecord`, so the barking plays while the mic listens.
- Echo cancellation: `inputNode.setVoiceProcessingEnabled(true)`.
- Permission: `NSMicrophoneUsageDescription`, requested at runtime.

**Flutter**
- `record`: an amplitude stream in dBFS. Check whether its config has echo-cancel and noise-suppression options.
- `noise_meter` is a simpler alternative.
- Volume tease: `flutter_volume_controller` listener (see [level 2](02-loud-speaker.md)).

**Gotchas**
- The phone's microphone hears the game's barking. Use the barking pauses, echo cancellation, or both.
- In a noisy room, the baseline is already high. Make the threshold relative to the baseline, not a fixed number.

## Clues

1. **Vague** (a neighbour yells from across the street): "Somebody make that mutt be quiet!"
2. **Closer:** "Dogs only listen to someone louder than them. Wait for it to take a breath…"
3. **Specific:** "When the dog pauses, shout 'SHHH!' or 'QUIET!' at your phone."

**(?) Solution after 2 minutes:** wait until the dog stops to pant, then shout one short, loud word at the bottom edge of the phone, where the microphone is.

## After the level

**Card:** "Microphone access: granted. You were 32 dB louder than your room. (We didn't record anything. We only measured the loudness.)"

**What you learned:** the microphone is one of the few inputs that needs your permission. Once you say yes, the app can listen any time it's open.

**Trivia:**
- When an app uses the microphone, a coloured dot appears at the top of the screen: orange on iPhone, green on Android 12+. Look for it now!
- Android 12+ has a Quick Settings toggle that blocks the microphone for *every* app at once.
- Since Android 9, apps in the background can't use the microphone without showing a notification.
- On iPhone, Control Center shows which app used the microphone most recently.

**Final report entry:** "Microphone: allowed. Loudest shout: 32 dB above your room."

## If the device can't do it

- **The player refused the microphone permission during setup:** skip the level and show a positive card: "You said no, and that's your right. Good apps keep working without it." Offer a button to reconsider, which opens the app's settings page.
- **No microphone** (rare tablets): mark the level as not supported.
- **Very noisy room:** the relative threshold handles most cases. If the baseline is above about −30 dBFS, suggest moving somewhere quieter.
- **The player can't shout** (a library, a sleeping house, or a speech impairment): a loud clap or a tap on the phone near the microphone also counts, because it's also a short loud sound.

## Reverting

Only if the tease was triggered: "The dog's asleep. You can turn the volume back up now."
- **Check:** volume back within the start range.
