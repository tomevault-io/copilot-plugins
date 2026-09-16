## airpoise

> macOS 14+ menu-bar utility. Reads AirPods IMU via `CMHeadphoneMotionManager`.

# AirPoise

macOS 14+ menu-bar utility. Reads AirPods IMU via `CMHeadphoneMotionManager`.

## Build

```
./scripts/build-app.sh
open dist/AirPoise.app
```

Always run the `.app` (Info.plist carries `NSMotionUsageDescription`).

```
swift test --package-path .
swift build -c release --product AirPoise
```

## Layout

- `Sources/AirPoiseCore` — math, pose, gesture engine, posture coach, settings store. No AppKit.
- `Sources/AirPoise` — menu bar, overlays, action runner, SwiftUI settings.
- `Tests/AirPoiseTests` — math / posture / gesture unit tests.

## Motion rules

- Do not invent a second AHRS. Consume Apple's fused `CMDeviceMotion`.
- **Headphone body frame: x = right ear, y = nose, z = crown** (not the iPhone frame). Euler convention lives in `Quat.headphoneEuler`; gyro: turn = ωz, nod = ωx, tilt = ωy.
- Posture = gravity vs calibrated upright. Gestures = excursions from a slow rest pose.
- Resubscribe on wake / silent stream. Don't stack `startDeviceMotionUpdates` handlers.

---
> Source: [jaskirat1616/AirPoise](https://github.com/jaskirat1616/AirPoise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
