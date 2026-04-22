## higlab

> HIG Lab contains 50 AI Reference documents for Apple frameworks in the `ai-reference/` directory. These documents provide comprehensive guides for generating accurate iOS/Swift code.

# GitHub Copilot Instructions for HIG Lab

## Project Overview

HIG Lab contains 50 AI Reference documents for Apple frameworks in the `ai-reference/` directory. These documents provide comprehensive guides for generating accurate iOS/Swift code.

## Instructions

1. **Always consult AI References first** — Before generating iOS/Swift code, check if a relevant reference exists in `ai-reference/`. Read it fully before writing code.
2. **Use modern APIs** — Target Swift 5.9+ and iOS 17+.
3. **Prefer SwiftUI** — Use SwiftUI over UIKit unless UIKit is specifically required.
4. **Use Swift Concurrency** — Prefer async/await and Actor over completion handlers and GCD.
5. **Handle errors properly** — Create custom error types conforming to `LocalizedError`.
6. **Support accessibility** — Add VoiceOver labels to all interactive elements.
7. **Use modern patterns** — `@Observable` over `ObservableObject`, `SwiftData` over Core Data.

## Framework Reference Files

| Framework | File |
|-----------|------|
| SwiftUI | `ai-reference/swiftui.md` |
| Observation (@Observable) | `ai-reference/swiftui-observation.md` |
| SwiftData | `ai-reference/swiftdata.md` |
| WidgetKit | `ai-reference/widgets.md` |
| ActivityKit | `ai-reference/activitykit.md` |
| App Intents | `ai-reference/appintents.md` |
| Foundation Models | `ai-reference/foundation-models.md` |
| TipKit | `ai-reference/tipkit.md` |
| StoreKit 2 | `ai-reference/storekit.md` |
| PassKit | `ai-reference/passkit.md` |
| CloudKit | `ai-reference/cloudkit.md` |
| Authentication Services | `ai-reference/authservices.md` |
| LocalAuthentication | `ai-reference/localauth.md` |
| CryptoKit | `ai-reference/cryptokit.md` |
| HealthKit | `ai-reference/healthkit.md` |
| MapKit | `ai-reference/mapkit.md` |
| Core Location | `ai-reference/corelocation.md` |
| Core ML | `ai-reference/coreml.md` |
| Vision | `ai-reference/vision.md` |
| User Notifications | `ai-reference/notifications.md` |
| SharePlay | `ai-reference/shareplay.md` |
| EventKit | `ai-reference/eventkit.md` |
| Contacts | `ai-reference/contacts.md` |
| MusicKit | `ai-reference/musickit.md` |
| WeatherKit | `ai-reference/weatherkit.md` |
| ARKit | `ai-reference/arkit.md` |
| RealityKit | `ai-reference/realitykit.md` |
| SpriteKit | `ai-reference/spritekit.md` |
| Core Image | `ai-reference/coreimage.md` |
| PencilKit | `ai-reference/pencilkit.md` |
| PDFKit | `ai-reference/pdfkit.md` |
| AVFoundation | `ai-reference/avfoundation.md` |
| AVKit | `ai-reference/avkit.md` |
| PhotosUI | `ai-reference/photosui.md` |
| Core Haptics | `ai-reference/corehaptics.md` |
| ShazamKit | `ai-reference/shazamkit.md` |
| Image Playground | `ai-reference/image-playground.md` |
| Core Bluetooth | `ai-reference/core-bluetooth.md` |
| Core NFC | `ai-reference/core-nfc.md` |
| MultipeerConnectivity | `ai-reference/multipeerconnectivity.md` |
| Network | `ai-reference/network.md` |
| CallKit | `ai-reference/callkit.md` |
| Wi-Fi Aware | `ai-reference/wifi-aware.md` |
| Visual Intelligence | `ai-reference/visual-intelligence.md` |
| AlarmKit | `ai-reference/alarmkit.md` |
| EnergyKit | `ai-reference/energykit.md` |
| PermissionKit | `ai-reference/permissionkit.md` |
| RelevanceKit | `ai-reference/relevancekit.md` |
| AccessorySetupKit | `ai-reference/accessorysetupkit.md` |
| ExtensibleImage | `ai-reference/extensibleimage.md` |

## Code Quality Checklist

- [ ] Swift Concurrency (async/await, Actor)
- [ ] SwiftUI best practices
- [ ] Error handling with custom error types
- [ ] Accessibility support (VoiceOver)
- [ ] #Preview macros
- [ ] /// documentation comments

---
> Source: [M1zz/HIGLab](https://github.com/M1zz/HIGLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
