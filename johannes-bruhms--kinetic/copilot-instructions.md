## kinetic

> Kinetic is a professional, high-performance iOS application designed for contemporary composers, performers, and multimedia artists. It transforms an iPhone into a trainable IMU (Inertial Measurement Unit) instrument, streaming real-time motion data and custom gesture recognition events over OSC to laptop-side environments like Max for Live, Pure Data, or SuperCollider.

# Kinetic: iOS Gesture Controller for Live Music

Kinetic is a professional, high-performance iOS application designed for contemporary composers, performers, and multimedia artists. It transforms an iPhone into a trainable IMU (Inertial Measurement Unit) instrument, streaming real-time motion data and custom gesture recognition events over OSC to laptop-side environments like Max for Live, Pure Data, or SuperCollider.

## Project Overview

- **Core Purpose**: Real-time IMU data streaming and gesture classification for artistic performance.
- **Main Technologies**:
  - **Swift & SwiftUI**: Modern iOS development with Swift 6 concurrency.
  - **CoreMotion**: High-frequency sensor data (100–200 Hz).
  - **DTW (Dynamic Time Warping)**: Primary gesture classification — works immediately after recording samples.
  - **Core ML**: Optional upgrade path for neural network-based classification.
  - **NWConnection (Network Framework)**: Low-latency OSC over UDP (no external dependencies).
- **Architecture**: MVVM with `@MainActor` default isolation.
- **Unique Features**: Energy-based auto-segmentation for gesture training, Bonjour auto-discovery with endpoint resolution, haptic feedback on gesture triggers, and a "glove-friendly" dark-mode UI.

## Project Structure

- `Kinetic/`: Main application source.
  - `App/`: Entry point (`KineticApp.swift`).
  - `Models/`: Data structures (`GestureModel.swift`, `OSCConfiguration.swift`).
  - `Services/`: Core logic providers.
    - `SensorManager.swift`: Handles CoreMotion updates at 100-200 Hz.
    - `OSCSender.swift`: Manual OSC encoding and UDP transmission.
    - `GestureClassifier.swift`: Hybrid classifier — DTW primary, Core ML optional.
    - `DTWClassifier.swift`: Template-based gesture matching via Dynamic Time Warping.
    - `GestureSegmenter.swift`: Energy-based motion segmentation with hysteresis.
    - `GestureLibrary.swift`: Persistence, recording management, and data export.
    - `BonjourBrowser.swift`: Network service discovery with IP/port resolution.
  - `Views/`: SwiftUI interface components.
    - `PerformanceView.swift`: Main dashboard with stream toggle, waveform, gesture bars.
    - `TrainingView.swift`: Gesture recording, auto-segmentation, segment review.
    - `TestModeView.swift`: Full-screen rehearsal mode with haptic feedback.
    - `GestureLibraryView.swift`: Gesture CRUD with rename (swipe-left) and delete (swipe-right).
    - `SettingsView.swift`: OSC config, Bonjour discovery, sample rate, data export.
    - `IMUWaveformView.swift`: Canvas-based real-time 3-axis waveform.
    - `GestureProbabilityBarsView.swift`: Color-coded probability display.
  - `Resources/`: Assets (AppIcon, AccentColor).
- `KineticTests/`: Unit tests for DTW classifier, segmenter, and OSC encoding.

## Building and Running

### Prerequisites
- **macOS 14+** (Sonoma) and **Xcode 15+**.
- **iPhone** running **iOS 17+** (Real device required for IMU features).

### Setup Instructions
1. **Clone the Repo**: `git clone <repo-url>`
2. **Open in Xcode**: Open `Kinetic.xcodeproj` directly.
3. **Signing**: Select your team under Signing & Capabilities.
4. **Run**: Connect your iPhone, enable Developer Mode, and build/run (Cmd+R).
5. **Test**: Cmd+U to run unit tests (works in simulator).

## Development Conventions

- **Performance First**: Sensor processing and OSC encoding happen off the main thread.
- **Native APIs**: Pure Apple frameworks — CoreMotion, Network, CoreML. No external dependencies.
- **Swift 6 Concurrency**: Project uses `@MainActor` default isolation. Types used from background queues must be marked `nonisolated` (e.g., `DTWClassifier`, `GestureSegmenter`).
- **UI/UX**: Dark mode only. Large, accessible controls for live performance contexts.
- **Data Privacy**: No tracking or analytics. Motion data never leaves the device except via user-configured OSC stream.
- **Testing**: DTW, segmentation, and OSC encoding logic verified in `KineticTests`.

## How Gesture Recognition Works

1. **Training**: User creates a gesture name, records motion samples, app auto-segments via energy thresholding.
2. **Template Storage**: Segmented samples are saved as JSON recordings in the gesture library.
3. **Classification**: During performance, the `GestureClassifier` loads saved recordings as DTW templates. A sliding window (50 samples, stride 10) is compared against all templates using Dynamic Time Warping distance.
4. **Events**: When probability exceeds 0.8, an OSC gesture event fires. At 0.9+, a trigger event fires with haptic feedback.

## Key Commands
- **Run App**: `Cmd + R` (in Xcode)
- **Run Tests**: `Cmd + U` (in Xcode)
- **Clean Build**: `Cmd + Shift + K` (in Xcode)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/johannes-bruhms) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
