## loupe

> An iOS app that gives users a hands-on tour of the iOS fingerprinting surface. It reads real values from public iOS APIs—the same ones any third-party app can call—and displays them raw to show what the device exposes and why each reading makes the user identifiable.

# Loupe

An iOS app that gives users a hands-on tour of the iOS fingerprinting surface. It reads real values from public iOS APIs—the same ones any third-party app can call—and displays them raw to show what the device exposes and why each reading makes the user identifiable.

Signals are organized into three tiers reflecting the cost of access: **Passive** (no user consent needed), **Needs Permission** (triggers an iOS prompt), and **Advanced** (clever side-channel uses of public APIs — e.g. URL-scheme probing via `canOpenURL`, canvas/WebGL extraction in a hidden `WKWebView`, Keychain persistence across reinstalls).

Nothing collected leaves the device unless the user explicitly exports it. The app deliberately shows raw values without aggregation or hashing.

## Build System

The project uses Xcode's **buildable folders** (folder references), so new Swift files are automatically included in the build. There is no need to manually add files to the Xcode project's build sources.

## SwiftUI — modal Done actions

For sheets and similar modal chrome, use `.confirmationAction` with a titled checkmark button rather than plain `.topBarTrailing`:

```swift
ToolbarItem(placement: .confirmationAction) {
    Button("Done", systemImage: "checkmark") {
        dismiss()
    }
}
```

Keep `.topBarTrailing` for actions that are not the primary dismiss control (for example export or refresh).

## Writing style — user-facing copy

Loupe explains a technical subject (device fingerprinting) to non-technical users. All user-facing strings — onboarding pages, summary sheets, narrative cards, basis captions, signal descriptions — should sound like one consistent, calm voice.

### Voice principles

- **Plain English.** No jargon. Prefer "tracker" over "ad-tech vendor", "settings" over "configuration", "your iPhone" over "the device".
- **Second person, present tense.** Talk to the user about what *their* `\(PlatformDevice.localizedModel)` reveals. Avoid passive constructions like "a device can be identified".
- **Quietly.** Apps "quietly read" or "quietly check" things — this single adverb captures the whole privacy thesis. Use it where it fits, but don't overuse.
- **Short, declarative sentences.** Two short sentences beat one long one. Trim subordinate clauses.
- **No marketing language.** No exclamation marks, no "powerful", no "amazing", no rhetorical hype. The values themselves are persuasive enough.
- **No emojis** in any user-facing string.
- **Match the device noun.** Use `PlatformDevice.localizedModel` (e.g. "iPhone") when referring to the user's own device, and `PlatformDevice.marketingName` (e.g. "iPhone 15 Pro") when contrasting it with other units of the same model.
- **Contractions are fine** ("don't", "isn't", "you've") — they read more conversational. Just be consistent within a paragraph.
- **Avoid repeated words within a sentence or short paragraph.** If you find yourself writing "permission … permission" or "track you … track you", rephrase one of them.

### Recurring phrases (use these)

- "Any app can quietly read / quietly check …"
- "Trackers don't need your name, email, or location to recognize you online."
- "Each one isn't necessarily unique on its own, but together …"
- "form a fingerprint that follows you online"
- "When the same combination … shows up again across apps and websites, it stands out."
- "Nothing is uploaded, synced, or shared unless you choose to export."
- For narrative-card `basis` strings, prefer the pattern **"Read from \<source\>."** (occasionally "Inferred from …" or "Comparing …" when literally appropriate).

### Examples

**Onboarding / explainer copy**

- DO: "Apps can quietly check which other apps you have installed. That mix hints at your work, travel, finances, hobbies, and habits."
- DON'T: "Did you know?! Apps are able to detect a comprehensive list of all applications installed on your device, which can subsequently be utilized to derive insights about your lifestyle."

- DO: "Some readings are passively visible to apps with no prompt at all, while others require your permission."
- DON'T: "Some readings are passively visible to apps without permission or a prompt, while others require your permission." *(repeats "permission")*

**Narrative card headlines**

- DO: "Your time zone hints that you might be visiting or traveling in Germany."
- DON'T: "Your timezone suggests you may be in or traveling to Germany." *(awkward "in or traveling to"; "timezone" should be "time zone")*

- DO: "You have accessibility settings turned on: larger text and bold text."
- DON'T: "Accessibility settings are active: larger text and bold text." *(passive, less direct)*

- DO: "Lockdown Mode is turned on for this iPhone."
- DON'T: "Lockdown Mode is enabled on this iPhone." *("turned on" is plainer than "enabled")*

**Narrative card `basis` lines**

- DO: "Read from your iPhone's region setting. A VPN does not change this."
- DON'T: "Taken from your iPhone's region setting. Using a VPN does not change this."

- DO: "Read from the clipboard's change counter, a shared number any app can read."
- DON'T: "Read from the clipboard change counter, a shared counter accessible to all apps." *(repeats "counter")*

**Summary / closing copy**

- DO: "None of these readings are a name or an account. But together, they can be distinctive enough to recognize your iPhone again."
- DON'T: "None of these readings constitute personally identifiable information. However, in aggregate they may be sufficient to re-identify the device." *(legalistic, passive)*

- DO: "Loupe reads all of this on your iPhone and keeps it here. Nothing is uploaded, synced, or shared unless you choose to export."
- DON'T: "Loupe does not transmit any data off-device. All values remain local unless explicitly exported."

### When in doubt

Read [`OnboardingPage.swift`](code/Loupe/Views/Onboarding/OnboardingPage.swift) and the headlines in [`FingerprintNarrative.swift`](code/Loupe/Support/FingerprintNarrative.swift) — those are the canonical voice. Match their cadence before introducing a new one.

### Localization

User-facing strings are wrapped in `String(localized: …)` and exported to `Localizable.xcstrings`. When you change English source strings, the existing translations become stale — flag this in the change summary so the user can re-extract.

---
> Source: [mysk-research/loupe](https://github.com/mysk-research/loupe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
