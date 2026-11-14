# Level 28 · Forgot My PIN

> **Scope:** the first version is Android only. The iOS notes in this file are kept for a later iOS version.

| | |
|---|---|
| **Input** | Biometrics (fingerprint / Face ID / face unlock) |
| **Permission** | Android: `USE_BIOMETRIC` (install-time, no prompt). iOS: Face ID prompt (`NSFaceIDUsageDescription`), triggered during setup by a quick Face ID check; Touch ID needs none |
| **Support** | Android ✅ · iOS ✅ |
| **Access difficulty: Android** | 🟢 Easy |
| **Access difficulty: iOS** | 🟡 Moderate (Face ID) · 🟢 Easy (Touch ID) |
| **Setting to revert** | None |
| **Reference** | [research.md §4.27](../research.md#427-biometrics) |

## Scene

A cartoon banking app fills the screen, from a made-up bank, **"PiggyBank"**, with a smiling pig logo (no real bank's branding). The player's character needs to pay for concert tickets that are about to sell out: a countdown shows **"2 tickets left!"** and a queue of eager fans grows in the corner.

The app asks for a **6-digit PIN**. The character scratches their head, sweating: "I *never* remember this PIN!"

- **PIN pad:** the player can try PINs, and each wrong one shakes the pad with a tease: `123456` → "Really?", `000000` → "Nope", the player's birthday year → "Too obvious!" After 3 tries: "⚠️ 1 attempt left" (fake pressure, nothing is really locked).
- **The way out:** below the pad, a small button reads **"Use fingerprint"** or **"Use Face ID"**, its label matching what the phone has. It glows gently after the second clue.
- **Success:** after a successful scan, the pig logo winks, a vault door swings open, coins spill out, and the tickets are bought ("You're going to the concert! 🎸"). The character dances.

## Goal

**Unlock the app with biometrics:** a fingerprint or face scan.

**Success condition:** the authentication call returns `true`.

## Access difficulty

Scale: [research.md › Access difficulty](../research.md#access-difficulty). Each platform's overall rating is the worse of its two rows.

| | Android | iOS |
|---|---|---|
| **Player has to accept** | 🟢 Nothing (the permission is install-time and never shown) | 🟡 Face ID prompt (shown during setup) · 🟢 nothing with Touch ID |
| **Code to get the value** | 🟢 `local_auth`, plus a one-line change to the main activity | 🟢 `local_auth` |
| **Overall** | **🟢 Easy** | **🟡 Moderate (Face ID) · 🟢 Easy (Touch ID)** |

The fingerprint or face scan itself is the level's action, not a permission.

## Technical documentation

**Flutter**
- `local_auth`: `isDeviceSupported()`, `canCheckBiometrics`, `getAvailableBiometrics()` (fingerprint / face / strong / weak), and `authenticate(localizedReason: …)` with a biometric-only option.
- The API changed in recent major versions, so follow the current docs.

**Android**
- `androidx.biometric.BiometricPrompt` and `BiometricManager.canAuthenticate(BIOMETRIC_STRONG | BIOMETRIC_WEAK)`.
- Permission `USE_BIOMETRIC` (install-time).
- `MainActivity` must extend `FlutterFragmentActivity`.

**iOS**
- `LAContext.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error:)` and `evaluatePolicy(…)`.
- `LAContext.biometryType` is `.faceID`, `.touchID` or `.opticID`.
- `NSFaceIDUsageDescription` is required. The first Face ID use shows a system prompt. The game triggers it during setup, so it doesn't give this level away.

**What the app actually learns**
- Success or failure, which biometric type exists, and whether biometrics are set up.
- On iOS, `evaluatedPolicyDomainState` (newer code reads `domainState` on iOS 18+) changes whenever a fingerprint or face is **added or removed**. It doesn't say what changed.
- Never the fingerprint or face itself.

## Clues

1. **Vague** (the character): "Who even remembers PINs these days?"
2. **Closer:** "Your phone already knows who you are." (The biometric button starts glowing.)
3. **Specific:** "Tap 'Use fingerprint / Face ID' below the keypad and scan your finger or face."

**(?) Solution after 2 minutes:** tap the button under the PIN pad, then touch the fingerprint sensor or look at your phone, the same way you unlock it.

## After the level

**Card:** "The game only received 'yes, it's you'. Your fingerprint and face never left your phone's secure chip."

**What you learned:** with biometrics, the app only gets a yes or no. It never receives your fingerprint or your face.

**Trivia:**
- Your fingerprint or face data is stored in a separate secure chip (iPhone's "Secure Enclave", Android's secure hardware). Not even Apple, Google or the app can read it.
- What an app *can* tell is when a new fingerprint or face is **added** to the phone. Banking apps use this to ask you to log in again, in case someone else added theirs.
- Apps can also see which kind of biometrics your phone has.

**Final report entry:** "Biometrics: Face ID set up."

## If the device can't do it

- **No biometrics set up, but a screen lock exists:** allow the phone's PIN/passcode instead (`biometricOnly: false`; on iOS the policy `.deviceOwnerAuthentication`). The lesson stays the same: the app only gets yes or no.
- **No screen lock at all:** mark the level as not supported, with a card: "Your phone has no screen lock. Anyone who picks it up can open it. Consider setting one!"
- **The player refused Face ID during setup:** fall back to the device passcode.
- **No biometric hardware:** use the device passcode as above.

## Reverting

Nothing to revert.
