## bluetooth-tracker-ios

> When creating new Swift files that use `@Published`, `ObservableObject`, or any Combine types, **always include**:

# bluetooth-tracker

## iOS Swift — Import Reminders

When creating new Swift files that use `@Published`, `ObservableObject`, or any Combine types, **always include**:

```swift
import Combine
```

Without it, the compiler will error with:
- `Type '...' does not conform to protocol 'ObservableObject'`
- `Initializer 'init(wrappedValue:)' is not available due to missing import of defining module 'Combine'`

### Other commonly needed imports for this project

| Feature | Import |
|---|---|
| `@Published`, `ObservableObject`, `Timer.publish`, `.autoconnect()` | `import Combine` |
| `CGPoint`, `CGRect`, `CGFloat` | `import CoreGraphics` (or `import SwiftUI`) |
| `CLLocationManager`, heading | `import CoreLocation` |
| `CMMotionManager`, accelerometer/gyro | `import CoreMotion` (wrap in `#if os(iOS)`) |
| `CBCentralManager`, BLE | `import CoreBluetooth` |
| `AVAudioEngine` | `import AVFoundation` |

### CoreMotion guard pattern

`CMMotionManager` APIs are unavailable on macOS. Always guard with:

```swift
#if canImport(CoreMotion)
import CoreMotion
#endif

// Property declaration:
#if os(iOS)
private let motionManager = CMMotionManager()
#endif

// Usage:
#if os(iOS)
// ... motionManager calls ...
#endif
```

---
> Source: [glgmainstream12/bluetooth-tracker-ios](https://github.com/glgmainstream12/bluetooth-tracker-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
