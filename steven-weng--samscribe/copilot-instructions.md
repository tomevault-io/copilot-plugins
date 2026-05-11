## samscribe

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SamScribe is an open-source macOS transcription application that captures and transcribes audio from both the microphone and running applications (e.g., Zoom, Chrome, Teams) in real-time using FluidAudio for ASR (Automatic Speech Recognition) and speaker diarization. Features cross-recording speaker recognition using persistent voice embeddings.

## Build Commands

```bash
# Build the project
xcodebuild -project SamScribe.xcodeproj -scheme SamScribe -configuration Debug

# Build for release
xcodebuild -project SamScribe.xcodeproj -scheme SamScribe -configuration Release

# Clean build
xcodebuild clean -project SamScribe.xcodeproj -scheme SamScribe

# Open in Xcode
open SamScribe.xcodeproj
```

## Key Architecture

### Audio Pipeline Flow

1. **AudioManager** (MainActor): Orchestrates the entire audio capture and transcription pipeline
   - Manages microphone input via `AudioInput`
   - Manages per-process application audio via `ApplicationAudio`
   - Monitors audio processes via `AudioMonitor`
   - Routes audio buffers to `Transcriber`

2. **AudioInput** (Actor): Captures microphone audio using AVAudioEngine
   - Converts audio to 16kHz mono Float32 (FluidAudio format)
   - Implements mute control
   - Handles device assignment with retry logic

3. **ApplicationAudio** (Actor): Captures per-process audio using ScreenCaptureKit
   - Creates individual SCStream per audio process
   - Converts 48kHz stereo to 16kHz mono
   - Manages stream lifecycle with grace periods

4. **AudioMonitor** (@MainActor): Monitors system audio processes
   - Polls CoreAudio every 2 seconds for active audio processes
   - Implements 3-second start delay (prevents short audio spikes)
   - Implements 5-second grace period (prevents stream churn)
   - Filters for audio apps only (browsers, meeting apps, etc.)

5. **Transcriber** (Actor): Performs ASR and diarization using FluidAudio
   - Accumulates 10-second audio chunks per source
   - Processes chunks independently for each audio source (microphone vs app audio)
   - Performs ASR transcription with confidence scores
   - Performs speaker diarization and extracts speaker embeddings (256-dimensional)
   - Configured with tuned parameters: clusteringThreshold: 0.5, minSpeechDuration: 0.5s, minSilenceGap: 0.2s

6. **SpeakerManager** (@MainActor): Manages global speaker recognition
   - Matches voice embeddings using Accelerate-optimized cosine similarity
   - Uses FluidAudio's AssignmentConfig.macOS (0.65 distance threshold)
   - Creates and persists Speaker entities in SwiftData
   - Enables cross-recording speaker identification

### Data Flow

```
AudioInput (mic) ──┐
                   ├──> Transcriber ──> TranscriptionResult ──┐
ApplicationAudio ──┘    (10s chunks)    (with embeddings)    │
(per-process)                                                 │
                                                              ▼
                                                   TranscriptionsStore
                                                              │
                                                              ├──> SpeakerManager (match/create Speaker)
                                                              │
                                                              ▼
                                                          SwiftData
                                                (Recording, TranscriptionSegment, Speaker)
```

### Audio Format Conversions

- **Input formats**: Variable (44.1kHz-48kHz, mono/stereo)
- **Target format**: 16kHz mono Float32 (FluidAudio requirement)
- **Conversion**: AudioConverter (resampler using vDSP/Accelerate)
- **Chunk size**: 10 seconds = 160,000 samples @ 16kHz

### Permission Requirements

1. **Microphone**: Required for AudioInput (NSMicrophoneUsageDescription)
   - Automatically requested on app launch if not determined
   - Uses AVCaptureDevice.requestAccess(for: .audio)

2. **Screen Recording**: Required for ScreenCaptureKit to capture app audio (NSScreenCaptureUsageDescription)
   - Automatically requested when user starts recording
   - Uses CGPreflightScreenCaptureAccess() and CGRequestScreenCaptureAccess()
   - Alert guides user to System Settings if denied

3. **Permission Flow**: Managed in TranscriptionView
   - Checks both permissions on app launch
   - Button disabled until permissions checked
   - Shows appropriate alerts with "Open Settings" option

4. **Sandbox entitlements**: See SamScribe.entitlements

### Process Lifecycle Management

- **3-second start delay**: Waits 3s after detecting audio before creating SCStream (prevents spurious stream creation)
- **5-second grace period**: Waits 5s after process stops before destroying SCStream (handles temporary audio pauses)
- **Helper process mapping**: Chrome Helper → Chrome, Safari Renderer → Safari, etc.

### Key Files by Function

**Audio Capture**:
- `AudioInput.swift`: Microphone capture with AVAudioEngine
- `ApplicationAudio.swift`: Per-process app audio with ScreenCaptureKit
- `AudioMonitor.swift`: System audio process monitoring
- `AudioProcess.swift`: Audio process identification and filtering

**Transcription & Speaker Recognition**:
- `Transcriber.swift`: FluidAudio integration for ASR + diarization with embedding extraction
- `SpeakerManager.swift`: Embedding-based speaker matching and clustering
- `TranscriptionResult`: Data model for transcription output with speaker embeddings

**Data Models (SwiftData)**:
- `Recording.swift`: Recording session model
- `TranscriptionSegment.swift`: Transcription segment with speaker relationship and embedding storage
- `Speaker.swift`: Global speaker model with persistent embeddings
- `RecordingViewModel.swift`: View model for recordings list
- `TranscriptionSegmentViewModel.swift`: View model for transcript segments

**UI Components**:
- `TranscriptionView.swift`: Main UI entry point with permission management
- `SidebarView.swift`: Recordings list with real-time elapsed timer
- `RecordingDetailView.swift`: Transcript detail view with segment editing
- `TranscriptSegmentRow.swift`: Individual segment row with speaker editing
- `EditSpeakerSheet.swift`: Modal sheet for renaming speakers
- `RecordingControlsView.swift`: Recording controls (start/stop)

**Business Logic**:
- `TranscriptionsStore.swift`: Business logic layer integrating SpeakerManager with SwiftData

**Utilities**:
- `Logging.swift`: Structured logging with OSLog
- `AudioConverter` (FluidAudio): Audio resampling and format conversion

## Development Notes

### FluidAudio Integration

The app depends on FluidAudio (https://github.com/FluidInference/FluidAudio) which handles:
- ASR model downloading and caching
- Streaming transcription
- Speaker diarization
- Voice Activity Detection (VAD)

Note: The `Audio/` folder in the repository is a reference folder that will be deleted soon - ignore it.

### Audio Format Requirements

FluidAudio requires:
- 16kHz sample rate
- Mono (1 channel)
- Float32 PCM format
- 10-second chunks for processing

All audio inputs are converted to this format before transcription.

### ScreenCaptureKit Quirks

1. Requires at least one window to capture audio (even minimized)
2. `onScreenWindowsOnly: false` is required to capture minimized/hidden apps
3. Permission preflight: Use `CGPreflightScreenCaptureAccess()` before `CGRequestScreenCaptureAccess()`
4. Transient failures: Implement retry logic (3 attempts with 500ms backoff)

### Concurrency Model

- **AudioManager**: @MainActor (UI integration)
- **AudioInput, ApplicationAudio, Transcriber**: Actors (isolated state)
- **AudioMonitor**: @MainActor @Observable (SwiftUI integration)
- **Audio callbacks**: @Sendable closures for cross-actor communication

### SwiftData Schema

```swift
Recording {
    id: UUID
    title: String
    startDate: Date
    endDate: Date?
    segments: [TranscriptionSegment]  // cascade delete

    // Computed
    duration: TimeInterval  // calculated from start/end dates
    segmentCount: Int  // count of segments
}

TranscriptionSegment {
    id: UUID
    text: String  // Editable transcript text
    timestamp: Date
    confidence: Float
    startTime: TimeInterval
    endTime: TimeInterval
    isPartial: Bool
    audioSourceType: String  // "microphone" or "appAudio"
    embeddingData: Data?  // 256 floats × 4 bytes = 1024 bytes

    // Relationships
    recording: Recording?  // inverse relationship
    speaker: Speaker?  // nullable - can be nil for "No Speaker" segments

    // Computed
    @Transient speakerLabel: String?  // returns speaker?.displayName
}

Speaker {
    id: UUID
    customName: String?  // User-provided custom name (optional)
    speakerNumber: Int  // Auto-assigned: 1, 2, 3... (0 for unknown)
    embeddingData: Data  // 256 floats = 1024 bytes
    createdAt: Date
    updatedAt: Date

    // Relationships
    segments: [TranscriptionSegment]  // nullify on delete

    // Computed
    displayName: String {
        // Returns customName if set, otherwise "Speaker N"
        // speakerNumber == 0 → "Unknown Speaker"
    }
}
```

### Speaker Recognition Flow

1. **Transcription**: Transcriber extracts 256-dimensional embedding from diarization
2. **Matching**: SpeakerManager compares embedding to existing speakers using cosine similarity
   - Uses FluidAudio's `SpeakerUtilities.cosineDistance()` (vDSP-optimized)
   - Threshold: 0.65 from `AssignmentConfig.macOS`
3. **Assignment**:
   - If match found (distance < 0.65): Assign existing Speaker
   - If no match: Create new Speaker with next available number
   - If no embedding (nil): Leave speaker as nil → displays "No Speaker"
4. **Persistence**: Speaker saved to SwiftData, linked to TranscriptionSegment
5. **Cross-Recording**: Future segments automatically match to existing speakers

### UI Features

**Recording Management**:
- NavigationSplitView with sidebar (recordings list) and detail (transcript view)
- Real-time elapsed timer in sidebar for active recording
- Recordings grouped by date: Today, Yesterday, This Week, This Month, Older
- Context menu for rename/delete on recordings

**Transcript Editing**:
- Click segment to edit transcript text inline
- TextEditor with Save/Cancel buttons
- Edits persist immediately to SwiftData

**Speaker Management**:
- Speaker name displayed next to each segment (e.g., "Speaker 1", "Bob")
- Pencil icon next to speaker name for renaming
- EditSpeakerSheet modal for entering custom name
- Renaming affects ALL segments with that speaker across all recordings
- "No Speaker" segments (nil speaker) show gray text without edit icon
- Segments without embeddings cannot be assigned speakers

**Permission Handling**:
- Automatic permission checks on app launch
- "Start Transcribing" button disabled until permissions checked
- Alert dialogs with "Open Settings" button for denied permissions
- Graceful handling of missing permissions

### Diarization Configuration

Configured in Transcriber.swift (line 68-72):
```swift
DiarizerConfig(
    clusteringThreshold: 0.5,  // Higher = merges similar voices more aggressively
    minSpeechDuration: 0.5,    // Minimum speech segment length (seconds)
    minSilenceGap: 0.2,        // Minimum silence to separate segments (seconds)
    debugMode: false
)
```

**Tuning guidance**:
- `clusteringThreshold`: Lower (0.3-0.4) = more speakers, Higher (0.6-0.7) = fewer speakers
- `minSpeechDuration`: Increase to filter out very short utterances
- `minSilenceGap`: Increase to better separate rapid back-and-forth dialogue

### Common Debugging

**No transcriptions appearing**:
1. Check microphone permission granted
2. Check screen recording permission granted
3. Verify audio processes detected (check AudioMonitor logs)
4. Verify audio not silent (check amplitude logs in Transcriber)
5. Check FluidAudio model download succeeded

**SCStream creation failures**:
1. Verify screen recording permission
2. Check process has at least one window
3. Check helper process mapping worked
4. Review retry logs in ApplicationAudio

**Speaker recognition issues**:
1. Check embedding extraction logs in Transcriber (should see "📍 Longest speaker")
2. Verify SpeakerManager matching logs (distance values, threshold comparisons)
3. Check SwiftData saves succeeded (no save errors in TranscriptionsStore)
4. Verify Speaker entities created (query SwiftData directly)
5. Ensure segments have non-nil speaker relationship

**Audio quality issues**:
1. Check input format conversion logs
2. Verify 16kHz resampling working
3. Check for buffer underruns/overruns
4. Review silence detection thresholds

### Logging

The app uses OSLog for structured logging. View logs via:

**Option 1: Console.app** (recommended for standalone app)
1. Open Console.app (Applications/Utilities)
2. Select your Mac in sidebar
3. Filter by process: `process:SamScribe`
4. Enable "Include Info Messages" and "Include Debug Messages"

**Option 2: Xcode Console** (when running from Xcode)
- Logs appear in bottom console panel
- Use filter icon to enable Debug/Info levels

**Key log patterns**:
- `[ℹ️ INFO]` - Important events (transcription start/stop, speaker matching)
- `[🔍 DEBUG]` - Detailed debugging (only in DEBUG builds)
- `[❌ ERROR]` - Errors that need attention

---
> Source: [Steven-Weng/SamScribe](https://github.com/Steven-Weng/SamScribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
