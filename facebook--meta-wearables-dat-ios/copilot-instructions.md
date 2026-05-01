## dat-conventions

> DAT SDK conventions for ios development



# DAT SDK Conventions (iOS)

## Architecture

The SDK is organized into three modules:
- **MWDATCore**: Device discovery, registration, permissions, device selectors
- **MWDATCamera**: StreamSession, VideoFrame, photo capture
- **MWDATMockDevice**: MockDeviceKit for testing without hardware

## Swift Patterns

- Use `async/await` for all SDK operations — the SDK is fully async
- Use `AsyncSequence` / publisher `.listen {}` for observing streams
- Annotate UI-updating code with `@MainActor`
- Never block the main thread with frame processing
- Handle errors with do/catch — the SDK throws typed errors

## Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Entry point | `Wearables.shared` | `Wearables.shared.startRegistration()` |
| Sessions | `*Session` | `StreamSession`, `DeviceStateSession` |
| Selectors | `*DeviceSelector` | `AutoDeviceSelector`, `SpecificDeviceSelector` |
| Config | `*Config` | `StreamSessionConfig` |
| Publishers | `*Publisher` | `statePublisher`, `videoFramePublisher` |

## Imports

```swift
import MWDATCore    // Registration, devices, permissions
import MWDATCamera  // StreamSession, VideoFrame, photo capture
```

For testing:
```swift
import MWDATMockDevice  // MockDeviceKit, MockRaybanMeta, MockCameraKit
```

## Key Types

- `Wearables` — SDK entry point. Call `Wearables.configure()` at launch, then use `Wearables.shared`
- `StreamSession` — Camera streaming session. Create with config + device selector
- `VideoFrame` — Individual video frame with `.makeUIImage()` convenience
- `AutoDeviceSelector` — Automatically selects the best available device
- `SpecificDeviceSelector` — Selects a specific device by identifier
- `StreamSessionConfig` — Configure video codec, resolution, frame rate
- `MockDeviceKit` — Factory for creating simulated devices in tests

## Error Handling

```swift
do {
    try Wearables.configure()
} catch {
    // Handle configuration error
}

do {
    try Wearables.shared.startRegistration()
} catch {
    // Handle registration error
}
```

## Links

- [iOS API Reference](https://wearables.developer.meta.com/docs/reference/ios_swift/dat/0.6)
- [Developer Documentation](https://wearables.developer.meta.com/docs/develop/)
- [GitHub Repository](https://github.com/facebook/meta-wearables-dat-ios)

---
> Source: [facebook/meta-wearables-dat-ios](https://github.com/facebook/meta-wearables-dat-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
