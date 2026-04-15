## p2tomxf

> Enables users to queue multiple independent conversion tasks and execute them sequentially. Decouples "setup" from "execution" - users can configure jobs while others run.

# P2toMXF Project

## Project Location
`/Users/sim/XcodeProjects/1-macOS/P2toMXF`

## Status: WORKING ✓
Successfully merges P2 card clips into single MXF/MOV files using BMX + FFmpeg pipeline.

---

## Architecture

### App Files
- `P2toMXF.xcodeproj/` - Xcode project
- `P2toMXF/P2toMXFApp.swift` - App entry point
- `P2toMXF/ContentView.swift` - Main UI with clip list, console output panel
- `P2toMXF/ConversionViewModel.swift` - State management, merge orchestration
- `P2toMXF/Models/P2Clip.swift` - Data models for clips, cards, settings
- `P2toMXF/Services/P2CardParser.swift` - Parses P2 XML metadata
- `P2toMXF/Services/FFmpegWrapper.swift` - FFmpeg + BMX integration
- `P2toMXF/Services/BMXWrapper.swift` - BMX toolkit wrapper

### Bundled Tools (in Resources/)
```
Resources/
├── ffmpeg           (59MB) - FFmpeg 8.0.1 arm64
├── bmxtranswrap     (268KB) - BMX rewrap tool
├── mxf2raw          (252KB) - BMX inspection tool
└── lib/
    ├── libbmx.1.dylib       (2.2MB)
    ├── libMXF++.1.dylib     (634KB)
    ├── libMXF.1.dylib       (805KB)
    ├── libexpat.1.dylib     (163KB)
    └── liburiparser.1.dylib (145KB)
```

### Build Settings
- **Sandbox**: DISABLED (required for subprocess execution)
- **Hardened Runtime**: ENABLED (required for distribution/notarization)
- **ENABLE_USER_SCRIPT_SANDBOXING**: NO (required for Run Script build phase)
- **Development Team**: FDMSRXXN73
- **Deployment Target**: macOS 14.0

### Entitlements (P2toMXF.entitlements)
```xml
com.apple.security.cs.disable-library-validation    <!-- Load bundled dylibs -->
com.apple.security.cs.allow-unsigned-executable-memory  <!-- FFmpeg codecs -->
com.apple.security.cs.allow-jit                     <!-- JIT compilation -->
```

### Bundled Binary Signing
All bundled executables and dylibs must be signed with Hardened Runtime before distribution.
Run `./sign-bundled-binaries.sh` to sign with your development certificate.
For notarization, use: `./sign-bundled-binaries.sh "Developer ID Application: Your Name (TEAMID)"`

### Build Phases
- **Run Script** phase copies `lib/` folder to bundle:
  ```bash
  ditto "${SRCROOT}/P2toMXF/Resources/lib" "${BUILT_PRODUCTS_DIR}/${CONTENTS_FOLDER_PATH}/Resources/lib"
  ```
- The `lib` folder is NOT in Xcode project navigator (removed reference to prevent linking issues)

---

## P2 Card Structure (Panasonic)

```
CONTENTS/
├── CLIP/           # XML metadata: {ClipName}.XML
├── VIDEO/          # Video MXF: {ClipName}.MXF (AVC-Intra, OPAtom)
├── AUDIO/          # Audio MXF: {ClipName}00-03.MXF (4 mono channels)
├── ICON/           # Thumbnails
├── PROXY/          # Low-res MP4 proxies
└── AVCLIP/         # (typically empty)
```

### File Relationships
For clip `0234LZ`:
- XML: `CLIP/0234LZ.XML`
- Video: `VIDEO/0234LZ.MXF` (1 file, AVC-Intra)
- Audio: `AUDIO/0234LZ00.MXF` through `AUDIO/0234LZ03.MXF` (4 files)

### P2 MXF "Reference Matrix" Structure
Each MXF has 5 stream slots but only ONE contains actual data:

| File | Stream 0 | Stream 1 | Stream 2 | Stream 3 | Stream 4 |
|------|----------|----------|----------|----------|----------|
| VIDEO/*.MXF | **H.264** | placeholder | placeholder | placeholder | placeholder |
| AUDIO/*00.MXF | placeholder | **PCM Ch1** | placeholder | placeholder | placeholder |
| AUDIO/*01.MXF | placeholder | placeholder | **PCM Ch2** | placeholder | placeholder |
| AUDIO/*02.MXF | placeholder | placeholder | placeholder | **PCM Ch3** | placeholder |
| AUDIO/*03.MXF | placeholder | placeholder | placeholder | placeholder | **PCM Ch4** |

### Key XML Fields
```xml
<Video ValidAudioFlag="false">  <!-- Audio in video MXF is NOT usable -->
  <Codec>AVC-I_1080/25p</Codec>
  <StartTimecode>18:09:34:24</StartTimecode>
</Video>
<Audio>...</Audio>  <!-- 4 elements = 4 channels in separate files -->
```

---

## The Core Problem & Solution

### Why FFmpeg Alone Fails
FFmpeg's MXF muxer fails with P2 AVC-Intra:
```
[mxf @ 0x...] track 0: frame size does not match index unit size, 568320 != 568832
```

**Root cause**: MXF uses Index Tables for random access, requiring exact frame size prediction:
- FFmpeg calculates: 568,832 bytes/frame (from AVC-Intra spec)
- P2 actual frames: 568,320 bytes/frame
- Difference: 512 bytes (Panasonic's padding scheme)

FFmpeg uses generic calculation; P2 cameras use proprietary padding.

### Why BMX Works
BMX (BBC MXF toolkit) has manufacturer-specific lookup tables for padding schemes, including Panasonic P2.

### The Working Pipeline
```
P2 Card (OPAtom)     BMX                    FFmpeg
┌─────────────┐   ┌─────────────┐      ┌─────────────┐
│ VIDEO/*.MXF │─┬─│bmxtranswrap │─────▶│ concat      │───▶ Final
│ AUDIO/*.MXF │─┘ │ -t op1a     │ OP1a │ demuxer     │     MXF/MOV
└─────────────┘   └─────────────┘ MXF  └─────────────┘
```

1. **BMX rewraps** each P2 clip (5 files) → single OP1a MXF
2. **FFmpeg concatenates** the OP1a MXFs → final output (stream copy, fast)

---

## Working Commands

### Single Clip Rewrap (BMX)
```bash
bmxtranswrap -t op1a -o output.mxf \
  VIDEO/0234LZ.MXF \
  AUDIO/0234LZ00.MXF AUDIO/0234LZ01.MXF \
  AUDIO/0234LZ02.MXF AUDIO/0234LZ03.MXF
```

### Single Clip to MOV (FFmpeg - also works)
```bash
ffmpeg -i VIDEO/0234LZ.MXF \
  -i AUDIO/0234LZ00.MXF -i AUDIO/0234LZ01.MXF \
  -i AUDIO/0234LZ02.MXF -i AUDIO/0234LZ03.MXF \
  -map 0:v:0 -map 1:a:0 -map 2:a:0 -map 3:a:0 -map 4:a:0 \
  -c:v copy -c:a copy -timecode 18:09:34:24 \
  -f mov -y output.mov
```

### Concatenate Multiple Clips
```bash
# After BMX rewrap of each clip:
echo "file 'clip1.mxf'" > concat.txt
echo "file 'clip2.mxf'" >> concat.txt

ffmpeg -f concat -safe 0 -i concat.txt \
  -c copy -map 0:v -map 0:a \
  -timecode 18:09:34:24 -f mxf -y output.mxf
```

---

## Building BMX from Source

```bash
# Install dependencies
brew install cmake uriparser expat

# Clone and build
git clone https://github.com/ebu/bmx.git
cd bmx && mkdir -p out/build && cd out/build
export CMAKE_PREFIX_PATH="/opt/homebrew/opt/expat;/opt/homebrew/opt/uriparser"
cmake ../../ -DCMAKE_BUILD_TYPE=Release
cmake --build . -j$(sysctl -n hw.ncpu)
```

### Bundling BMX for App Distribution
After building, fix library paths:
```bash
# Copy binaries and libs
cp bmxtranswrap mxf2raw /path/to/Resources/
mkdir /path/to/Resources/lib
cp *.dylib /path/to/Resources/lib/

# Fix paths to look in Resources/lib/
install_name_tool -change @rpath/libbmx.1.dylib @executable_path/lib/libbmx.1.dylib bmxtranswrap
install_name_tool -change @rpath/libMXF++.1.dylib @executable_path/lib/libMXF++.1.dylib bmxtranswrap
install_name_tool -change @rpath/libMXF.1.dylib @executable_path/lib/libMXF.1.dylib bmxtranswrap
# (repeat for mxf2raw and all library cross-references)

# Re-sign after modifying
codesign --force --sign - bmxtranswrap mxf2raw lib/*.dylib
```

---

## Xcode Setup Notes

### Adding lib folder to bundle (what worked)
1. Do NOT add `lib/` folder to Xcode project navigator (causes linking issues)
2. Use **Run Script** build phase:
   ```bash
   ditto "${SRCROOT}/P2toMXF/Resources/lib" "${BUILT_PRODUCTS_DIR}/${CONTENTS_FOLDER_PATH}/Resources/lib"
   ```
3. Set `ENABLE_USER_SCRIPT_SANDBOXING = NO` in Build Settings

### Common Issues
- **SIGABRT at launch**: lib folder was added to project, causing Xcode to link dylibs. Fix: remove lib folder reference from project navigator.
- **Sandbox denial in Run Script**: Set `ENABLE_USER_SCRIPT_SANDBOXING = NO`
- **dylib version warnings**: Can be ignored (built on newer macOS, runs fine on older)

---

## Performance
- **BMX rewrap**: ~30x realtime (stream copy)
- **FFmpeg concat**: ~60x realtime (stream copy)
- **Single 2-min clip**: ~4-5 seconds
- **16 clips concatenated**: ~1-2 minutes

---

## Test P2 Card
- **Location**: `/Volumes/1TB-exFAT/Silvesterkonzert 2025 P2 K1 3v4`
- **Clips**: 16, AVC-I_1080/25p @ 50fps, 4 audio channels each
- **Camera**: Panasonic AJ-PX5000G
- **Content**: Continuous recording split across multiple files (every ~X GB)

---

## Future Enhancements
1. ~~**Batch rewrap mode**: Rewrap each clip to self-contained file (no concatenation)~~ ✓ Done
2. ~~**Output format selection**: MXF vs MOV option in UI~~ ✓ Done
3. **XML metadata copy**: Option to copy P2 XML alongside output
4. ~~**Per-clip export**: For NLE workflows needing original timecodes~~ ✓ Done (Individual mode)
5. **Progress parsing**: Parse BMX output for better progress reporting
6. **Large binary handling**: Consider download script instead of storing ffmpeg (59MB) in repo
   - Option A: `download-deps.sh` script fetching from https://evermeet.cx/ffmpeg/
   - Option B: Xcode build phase that downloads if missing
   - Option C: Git LFS for large file storage
7. ~~**Multi-job queueing**: Queue multiple conversion tasks and execute sequentially~~ ✓ Done
8. ~~**Output verification**: Verify converted files decode correctly~~ ✓ Done
9. ~~**Multi-card loading**: Load multiple P2 cards from a parent folder at once~~ ✓ Done

---

## Session Log: 2026-01-01 (Initial Build)

### What We Discovered
1. P2 uses OPAtom MXF with "reference matrix" - 5 stream slots per file, only 1 has data
2. `ValidAudioFlag="false"` in XML means video MXF audio is NOT usable
3. FFmpeg MXF muxer fails due to frame size padding mismatch (568320 vs 568832 bytes)
4. MOV container works with FFmpeg (records sizes after encoding, not predictive)
5. BMX toolkit has Panasonic-specific lookup tables that handle padding correctly

### What We Built
1. Built BMX from source with Homebrew dependencies
2. Created BMXWrapper.swift service
3. Updated FFmpegWrapper.swift with BMX + FFmpeg pipeline
4. Bundled BMX binaries + dylibs in app Resources
5. Fixed library paths with install_name_tool for @executable_path/lib/
6. Added Run Script build phase for copying lib folder
7. Disabled user script sandboxing to allow file copying

### Key Files Modified
- `Services/BMXWrapper.swift` (new)
- `Services/FFmpegWrapper.swift` (updated with BMX integration)
- `Resources/bmxtranswrap`, `mxf2raw`, `lib/*.dylib` (new)
- `CLAUDE.md` (comprehensive documentation)

---

## Session Log: 2026-01-01 (Refinements)

### New Features Added

#### 1. Custom Output Filename
- User can specify output filename for merged file
- Required field (button disabled until entered)
- Hidden in Individual mode (uses clip names)

#### 2. Container Format Selection
- Picker to choose MOV or MXF output format
- Applies to both Concatenate and Individual modes

#### 3. Processing Mode Selection
- **Merge & Concatenate**: Combines all clips into single file (original behavior)
- **Individual Files**: Rewraps each clip to its own self-contained file

#### 4. Timecode Continuity Check
- Detects gaps/overlaps between consecutive clips when concatenating
- Shows warning in UI and blocks merge if discontinuous
- User can switch to Individual mode or deselect problematic clips

#### 5. Duration Display
- Each clip shows duration in HH:MM:SS:FF format
- Properly converted from P2 edit units to timecode frames

### P2 XML Parsing Fixes

#### EditUnit Conversion (Critical Bug Fix)
```xml
<Duration>6432</Duration>
<EditUnit>1/50</EditUnit>
```
- **Problem**: Duration is in edit units (1/50 sec), not TC frames
- **Fix**: Convert `duration_tc_frames = edit_units × (tc_rate / edit_rate)`
- Example: 6432 × (25/50) = 3216 TC frames

#### Frame Rate Priority (Critical Bug Fix)
```xml
<Codec>AVC-I_1080/25p</Codec>    <!-- TC rate: 25fps -->
<FrameRate>50p</FrameRate>        <!-- Sensor rate: 50fps -->
```
- **Problem**: `<FrameRate>` element (sensor rate) was overwriting TC rate from `<Codec>`
- **Fix**: Added `frameRateFromCodec` flag - once Codec sets TC rate, FrameRate is ignored
- TC operations now correctly use 25fps from codec string

### Key Files Modified
- `Models/P2Clip.swift` - Added `OutputContainer`, `ProcessingMode` enums, `Timecode` struct
- `ContentView.swift` - Redesigned footer with mode/format/filename controls, TC warning display
- `ConversionViewModel.swift` - Added TC check logic, `canConvert`, individual conversion mode
- `Services/P2CardParser.swift` - Fixed EditUnit conversion and frame rate priority

### Architecture Notes

#### ConversionSettings (P2Clip.swift)
```swift
struct ConversionSettings {
    var outputDirectory: URL?
    var outputFilename: String = ""
    var outputContainer: OutputContainer = .mov  // .mov or .mxf
    var processingMode: ProcessingMode = .concatenate  // .individual or .concatenate
    var preserveTimecode: Bool = true
    var audioMapping: AudioMapping = .allChannels
}
```

#### Timecode Struct (P2Clip.swift)
```swift
struct Timecode {
    let hours, minutes, seconds, frames: Int
    let frameRate: Double

    init?(string: String, frameRate: Double)  // Parse "HH:MM:SS:FF"
    var totalFrames: Int                       // Convert to absolute frames
    static func frameGap(...) -> Int           // Check continuity
}
```

#### TC Continuity Check (ConversionViewModel.swift)
```swift
var timecodeIssues: [(clip1: P2Clip, clip2: P2Clip, gapFrames: Int)]
// Returns non-empty array if gaps/overlaps detected between consecutive clips
// gap > 0: missing frames, gap < 0: overlapping frames
```

---

## Session Log: 2026-01-01 (UI Polish & App Icon)

### UI Improvements

#### Footer Reorganization
- **Row 1**: Format, Mode, Audio, Preserve TC (inline labels, horizontal)
- **Row 2**: Output directory + "Use folder name" checkbox + Filename field
- **Row 3**: Action button (right-aligned)

#### Layout Fixes
- Minimum window width: 600 → 850 pixels
- All labels use consistent font size (removed `.font(.caption)`)
- Added `.fixedSize()` to prevent label text wrapping
- Filename field expands to fill available space (`maxWidth: .infinity`)
- "Use folder name" checkbox auto-fills P2 card folder name

#### Header Cleanup
- Moved "Load P2 Card" button to left side
- Removed FFmpeg version display (now in About panel)
- Kept "FFmpeg not found" warning for error feedback

### App Icon & About Panel

#### App Icon Setup
- Generated all macOS icon sizes from 1024px source (NanoBanana design)
- Sizes: 16, 32, 64, 128, 256, 512, 1024 pixels
- Added to `AppIcon.appiconset` with proper Contents.json
- Created `AboutIcon.imageset` for potential custom About view

#### Credits (About Panel)
- Created `Credits.rtf` with clickable links:
  - **FFmpeg** (ffmpeg.org) - LGPL/GPL license
  - **BMX** (github.com/bbc/bmx) - BSD 3-Clause license
  - Icon credit to NanoBanana
- Standard macOS About panel now shows icon + credits

### New Settings
- `useFolderNameAsFilename: Bool` - Auto-use P2 card folder name
- `effectiveOutputFilename` computed property in ViewModel

### Files Modified
- `ContentView.swift` - Footer layout, header cleanup
- `ConversionViewModel.swift` - Added `effectiveOutputFilename`
- `Models/P2Clip.swift` - Added `useFolderNameAsFilename` setting
- `Assets.xcassets/AppIcon.appiconset/` - All icon sizes
- `Assets.xcassets/AboutIcon.imageset/` - About view icon
- `Credits.rtf` - About panel credits with links

---

## Session Log: 2026-01-01 (Batch Queue)

### Multi-Job Queue Implementation
Enables users to queue multiple independent conversion tasks and execute them sequentially. Decouples "setup" from "execution" - users can configure jobs while others run.

### New Data Structures

#### JobStatus Enum (P2Clip.swift)
```swift
enum JobStatus: Equatable {
    case pending       // Waiting in queue
    case preparing     // Gathering files/rewrapping
    case active        // FFmpeg is processing
    case completed     // Successfully finished
    case failed(String) // Error encountered
    case cancelled     // User cancelled
}
```

#### ConversionJob Struct (P2Clip.swift)
```swift
struct ConversionJob: Identifiable {
    let id: UUID
    let cardName: String          // Source P2 card name
    let cardPath: URL             // For security-scoped access
    let clips: [P2Clip]           // Clips to process
    let settings: ConversionSettings
    let destinationURL: URL       // Final output path
    let createdAt: Date
    var status: JobStatus = .pending
    var progress: Double = 0.0    // 0.0 to 1.0
}
```

### QueueManager Service (Services/QueueManager.swift)
Central singleton service managing the job queue:
- **Queue Storage**: Array of `ConversionJob` objects
- **Sequential Execution**: Picks next pending job automatically
- **Service Orchestration**: Communicates with FFmpegWrapper
- **Security Scoped Access**: Handles `startAccessingSecurityScopedResource()` for each job
- **Status Tracking**: Updates job status and progress in real-time

### QueueListView (Views/QueueListView.swift)
Collapsible panel showing the queue:
- **Job Rows**: Card name, clip count, format, status icon, progress bar
- **Job Controls**: Remove pending, retry failed, clear finished
- **Global Controls**: Cancel current, cancel all pending

### UI Integration (ContentView.swift)
- **"Add to Queue" Button**: Next to Convert, queues current selection
- **Queue Toggle**: Header button shows/hides queue panel with badge
- **Console Integration**: Shows queue output when processing
- **Feedback Message**: Temporary "Added X jobs to queue" confirmation

### ViewModel Integration (ConversionViewModel.swift)
- `addToQueue()`: Validates settings and creates job(s)
- `addConcatenateJobsToQueue()`: Creates one job per fully selected group
- `addIndividualJobToQueue()`: Creates single job for selected clips
- `canAddToQueue`: Same validation as `canConvert`

### Files Created
- `Services/QueueManager.swift` - Queue management singleton
- `Views/QueueListView.swift` - Queue panel UI

### Files Modified
- `Models/P2Clip.swift` - Added `ConversionJob`, `JobStatus`
- `ConversionViewModel.swift` - Added queue integration methods
- `ContentView.swift` - Added queue panel, toggle button, "Add to Queue" button

---

## Session Log: 2026-01-01 (Enhanced Batch Queue)

### Overview
Major enhancements to the batch queue system: manual start control, queue persistence, filename conflict resolution, and sleep prevention. Implements the refined architecture from `UPDATED_QUEUE_PLAN.md`.

### New Features

#### 1. Queue Persistence (Codable + JSON)
Queue state survives app restarts/crashes:
- All data models now implement `Codable`: `P2Clip`, `ConversionSettings`, `JobStatus`, `ConversionJob`
- Queue saved to `~/Library/Application Support/P2toMXF/queue.json`
- Only pending and failed jobs restored on launch (completed jobs cleared)
- Active jobs reset to pending if app crashes mid-conversion

#### 2. Manual Start Control
Disabled auto-start for better user control:
- `addJob()` no longer auto-starts queue processing
- New `autoStart: Bool` parameter for immediate execution
- "Convert Now" → adds to queue AND starts (unified execution path)
- "Add to Queue" → adds without starting
- "Start Queue (N)" button appears when pending jobs exist

#### 3. Filename Conflict Resolution
Prevents overwrites when queuing duplicate filenames:
- Checks against all pending job destinations
- Checks against existing files on disk
- Auto-renames with counter: `Output.mxf` → `Output (1).mxf`
- Logs the rename action

#### 4. Sleep Prevention (IOPMAssertion)
Prevents Mac from sleeping during long conversions:
```swift
import IOKit.pwr_mgt

IOPMAssertionCreateWithName(
    kIOPMAssertionTypeNoIdleSleep as CFString,
    IOPMAssertionLevel(kIOPMAssertionLevelOn),
    "P2toMXF is converting video files" as CFString,
    &sleepAssertionID
)
```
- Enabled when queue starts processing
- Released when queue completes or stops
- User can still manually sleep the Mac

### Context-Aware UI Buttons

| Queue State | Primary Button | Action |
| :--- | :--- | :--- |
| Empty | "Convert Now" | Add to queue + start |
| Has Pending | "Start Queue (N)" | Start queue processing |
| Processing | "Stop" | Cancel current job |

### Architecture Changes

#### P2Clip.swift - Codable Support
- URL properties stored as String paths internally
- Computed properties convert to/from URL
- Custom Codable for `JobStatus` enum with associated value

#### QueueManager.swift - New Methods
```swift
// Persistence
private func saveQueue()
private func loadQueue()

// Sleep prevention
private func preventSleep()
private func allowSleep()

// Conflict resolution
private func resolveFilenameConflict(for url: URL) -> URL

// Updated addJob signature
func addJob(_ job: ConversionJob, autoStart: Bool = false)
```

### Files Modified
- `Models/P2Clip.swift` - Added `Codable` to all models, URL→String storage
- `Services/QueueManager.swift` - Persistence, sleep prevention, conflict resolution
- `ConversionViewModel.swift` - `addToQueue(autoStart:)` parameter
- `ContentView.swift` - Context-aware primary action button
- `Views/QueueListView.swift` - Added "Start" button in queue controls

---

## Session Log: 2026-01-01 (Video Verification)

### Overview
Added output file verification to ensure converted MXF/MOV files are correctly playable. Uses FFmpeg to perform container validation and full decode tests.

### Verification Modes

#### Quick Mode (~5 seconds)
- Container structure check using ffprobe
- Decode first 5 seconds
- Decode last 5 seconds
- Fast sanity check for most common issues

#### Full Mode (faster than realtime)
- Complete frame-by-frame decode test
- Uses VideoToolbox hardware acceleration when available
- Verifies frame count matches expected
- Catches bitstream errors, truncation, corruption

### New Data Structures

#### VerificationStatus Enum (P2Clip.swift)
```swift
enum VerificationStatus: Equatable, Codable {
    case unverified       // Not yet verified
    case verifying        // Currently running verification
    case verified         // Passed verification
    case failed(String)   // Failed with error message
}
```

#### VerificationResult Struct (P2Clip.swift)
```swift
struct VerificationResult: Codable {
    let fileURL: URL
    let passed: Bool
    let mode: VerificationMode
    let duration: TimeInterval
    let framesDecoded: Int?
    let totalFrames: Int?
    let decodingSpeed: String?     // e.g., "45.2x"
    let containerValid: Bool
    let errorMessage: String?
    let verifiedAt: Date
}
```

### VerificationService (Services/VerificationService.swift)
Standalone service for file verification:
- **Container Validation**: Uses ffprobe to check MXF/MOV structure
- **Decode Testing**: FFmpeg decode to null output (`-f null -`)
- **Hardware Acceleration**: VideoToolbox for faster decode
- **Progress Reporting**: Frame-by-frame progress with speed metrics
- **Cancellation Support**: Can stop verification mid-process

### QueueManager Integration
Added verification methods:
```swift
func verifyJob(_ jobId: UUID, mode: VerificationMode)
func verifyAllCompleted(mode: VerificationMode)
func cancelVerification()
```

### UI Features (QueueListView.swift)

#### Per-Job Verification
- Completed jobs show verify button (seal icon)
- Menu with Quick/Full options
- Progress bar during verification
- Status icons: ✓ Verified (green seal), ✗ Failed (red seal)
- Tooltip shows verification summary

#### Batch Verification
- "Verify All" button in queue controls
- Processes all unverified completed jobs
- Shows count badge

#### Status Indicators
- Orange highlight during verification
- Header shows "Verifying..." when active
- "Stop Verify" button to cancel

### Files Created
- `Services/VerificationService.swift` - Verification engine

### Files Modified
- `Models/P2Clip.swift` - Added `VerificationStatus`, `VerificationResult`, `VerificationMode`
- `Services/QueueManager.swift` - Verification integration, `isVerifying` state
- `Views/QueueListView.swift` - Verification UI controls and status display

---

## Session Log: 2026-01-01 (Time Estimation & Speed Tracking)

### Overview
Added conversion time estimation based on historical speed data, with slow speed warnings during conversion.

### Features

#### Time Estimation
- **Pre-conversion estimates**: Shows expected time before starting
- **Historical tracking**: Records conversion speeds for accurate predictions
- **Confidence levels**: High (recent data), Medium (historical), Low (defaults)
- **Queue totals**: Shows total estimated time for all pending jobs

#### Speed Tracking
- Persists last 50 conversion records to `~/Library/Application Support/P2toMXF/speed_records.json`
- Tracks: bytes processed, duration, realtime multiplier, format, mode
- Filters by matching settings for relevant estimates

#### Slow Speed Warning
- Detects when speed drops below 30% of expected
- Shows banner with current vs expected speed
- Estimates remaining time at current rate
- Attempts to diagnose reason (network storage, external drive, etc.)

### New Data Structures

#### ConversionEstimate (P2Clip.swift)
```swift
struct ConversionEstimate {
    let totalBytes: Int64
    let totalDurationSeconds: Double
    let clipCount: Int
    let estimatedSeconds: TimeInterval
    let speedMultiplier: Double       // e.g., 30.0 means 30x realtime
    let confidence: EstimateConfidence
}
```

#### ConversionSpeedRecord (P2Clip.swift)
```swift
struct ConversionSpeedRecord: Codable {
    let date: Date
    let bytesProcessed: Int64
    let durationSeconds: TimeInterval
    let speedMultiplier: Double
    let processingMode: ProcessingMode
    let outputFormat: OutputContainer
}
```

#### SlowSpeedWarning (P2Clip.swift)
```swift
struct SlowSpeedWarning {
    let currentSpeed: Double
    let expectedSpeed: Double
    let estimatedRemaining: TimeInterval
    let reason: SlowSpeedReason  // slowDisk, externalDrive, networkStorage, etc.
}
```

### SpeedTracker Service (Services/SpeedTracker.swift)
- **Singleton**: `SpeedTracker.shared`
- **recordConversion()**: Saves speed data after each job
- **estimateConversion()**: Predicts time based on history
- **checkSpeed()**: Detects slow speeds during conversion

### UI Components (Views/EstimateSheet.swift)

#### EstimateSheet
Pre-conversion dialog showing:
- Source info (clips, size, duration)
- Estimated time with speed multiplier
- Confidence indicator
- Start/Cancel buttons

#### SlowSpeedBanner
In-conversion warning showing:
- Current vs expected speed
- Remaining time estimate
- Dismiss button

#### QueueEstimateBadge
Compact estimate display for pending jobs in queue list

### Queue Integration
- Pending jobs show estimate badge
- Queue header shows total estimated time
- Processing jobs show current estimate
- Slow speed banner appears in queue panel

### Files Created
- `Services/SpeedTracker.swift` - Speed tracking and estimation
- `Views/EstimateSheet.swift` - Estimate dialog and warning views

### Files Modified
- `Models/P2Clip.swift` - Added estimation models, `totalFileSize` computed property
- `Services/QueueManager.swift` - Integration with SpeedTracker, estimate methods
- `Views/QueueListView.swift` - Estimate badges, slow speed banner, header estimates

---

## Session Log: 2026-01-04 (UI Polish & Bug Fixes)

### Overview
Multiple UI improvements and bug fixes for the queue panel, verification system, and clip list display.

### Commits
- `ca64750` - Improve queue panel layout and sizing
- `36bba88` - Improve job display name and footer layout
- `cb1507c` - Fix thumbnail jitter during progress updates
- `6b3d31e` - Fix verification failing on directory instead of file
- `eda3b1a` - Center queue empty state both horizontally and vertically

### Issues Fixed

#### 1. Queue Panel Too Narrow and Short
**Problem**: Queue panel only used 1/4 of available vertical space due to `maxHeight: 200` constraint.
**Fix**: 
- Removed `maxHeight: 200` from job list in QueueListView
- Expanded queue panel width from 220-350px to 280-450px
- Increased minimum window width to 1280px

#### 2. Job Names Truncated Awkwardly
**Problem**: Job names showed awkward line breaks like "Silvesterkonz-ert 2025..."
**Fix**:
- Changed `lineLimit(1)` to `lineLimit(2)` for job names
- Added `truncationMode(.middle)` to preserve start and end of names
- Removed redundant cardName from job row (already in displayName)

#### 3. Job Display Name Missing Extension
**Problem**: Jobs showed "Silvesterkonzert 2025 P2 K7 2v4" without the .mxf extension
**Fix**: Changed `displayName` from `destinationURL.deletingPathExtension().lastPathComponent` to just `destinationURL.lastPathComponent`

#### 4. Output Directory Display Too Narrow
**Problem**: Output folder name truncated to 150px fixed width
**Fix**: Changed to `minWidth: 120, maxWidth: 280` for dynamic sizing

#### 5. Thumbnail Jitter During Progress Updates
**Problem**: Thumbnails visually jittered when progress bar updated during conversion
**Root Cause**: SwiftUI re-rendered entire ClipRowView body when `status` property changed
**Fix**: Extracted into separate isolated views:
- `ClipThumbnailsView` - Only depends on clip and thumbnailManager (stable)
- `ClipStatusBadge` - Handles frequently-updating status (isolated)

#### 6. Verification Failing on Directory
**Problem**: Verification showed "Invalid container: Is a directory" error
**Root Cause**: Legacy jobs had `destinationURL` pointing to output directory instead of file
**Fix**: Added directory detection in QueueManager.performVerification():
```swift
if FileManager.default.fileExists(atPath: fileURL.path, isDirectory: &isDirectory), isDirectory.boolValue {
    let ext = job.settings.outputContainer.fileExtension
    let baseName = job.destinationURL.lastPathComponent
    let fileName = "\(baseName).\(ext)"
    fileURL = fileURL.appendingPathComponent(fileName)
}
```

#### 7. Queue Empty State Not Centered
**Problem**: Empty queue state was top-left aligned instead of centered
**Fix**: Added `maxHeight: .infinity` to empty state frame for proper centering

### Files Modified
- `P2toMXF/ContentView.swift` - Window width, queue panel frame
- `P2toMXF/Models/P2Clip.swift` - displayName includes extension
- `P2toMXF/Views/QueueListView.swift` - Removed height constraint, centered empty state
- `P2toMXF/Views/JobRowView.swift` - Better text layout, line limits
- `P2toMXF/Views/ClipRowView.swift` - Extracted thumbnail/status into isolated views
- `P2toMXF/Views/FooterControlsView.swift` - Wider output directory display
- `P2toMXF/Services/QueueManager.swift` - Directory detection for verification

### SwiftUI Performance Insight
When a view property changes frequently (like progress), extract static content into separate child views. SwiftUI only re-renders views whose inputs change - isolated views with stable inputs won't redraw even when sibling views update.

---

## Session Log: 2026-01-04 (Verification & Display Fixes)

### Overview
Fixed critical bug where job names lost their suffixes after processing started, and enhanced verification UI with re-verify capability.

### Commits
- `bc7f722` - Fix job display names and enhance verification UI

### Issues Fixed

#### 1. Job Names Losing Suffix After Completion (Critical)
**Problem**: Jobs created with suffixes like `_01`, `_02`, `_03` for multiple groups would all display the same name (just the base directory name) after processing started.
**Root Cause**: `resolveOutputBookmark()` in `ConversionJob` was storing a security-scoped bookmark for the OUTPUT DIRECTORY (for write access), but when resolving it, the method incorrectly overwrote `destinationPathString` (the full FILE path) with the directory path.
```swift
// BUG: This line overwrote file path with directory path
destinationPathString = url.path  // url was directory, not file!
```
**Fix**: Removed the line that overwrote `destinationPathString`. The method now only returns the directory URL for security-scoped access without modifying the stored file path.

#### 2. Cannot Re-verify Already Verified Jobs
**Problem**: After running Quick Verify on a job, the verify menu disappeared (replaced by "Verified" text), preventing users from running Full Verify.
**Fix**: Added re-verify menu (↻ icon) to the verified state, allowing users to run Quick or Full verification again on already-verified jobs.

#### 3. Verification Details Not Shown
**Problem**: After verification completed, only "Verified" was shown with no details.
**Fix**: Now shows verification mode (Quick/Full) and decoding speed (e.g., "15x") below the Verified label. Full details in tooltip.

### Key Code Changes

#### P2Clip.swift - resolveOutputBookmark()
```swift
// Before (BUG):
destinationPathString = url.path  // Overwrote file path with directory!
return url

// After (FIXED):
// Do NOT update destinationPathString here!
// The bookmark is for the OUTPUT DIRECTORY, not the file.
return url
```

#### JobRowView.swift - Verified State
```swift
// Added re-verify menu to verified content
Menu {
    Button { queueManager.verifyJob(job.id, mode: .quick) } 
    Button { queueManager.verifyJob(job.id, mode: .full) }
} label: {
    Image(systemName: "arrow.clockwise")
}
```

### Files Modified
- `P2toMXF/Models/P2Clip.swift` - Fixed resolveOutputBookmark() to not overwrite file path
- `P2toMXF/Views/JobRowView.swift` - Added re-verify menu, verification details display
- `P2toMXF/Views/FooterControlsView.swift` - Added Stop All button
- `P2toMXF/Services/QueueManager.swift` - Added stopAllProcessing() method

### Architecture Insight
Security-scoped bookmarks for file access require careful handling:
- Card bookmark → points to card directory (correct to update path)
- Output bookmark → points to output DIRECTORY for write access
- But `destinationPathString` should store the full FILE path (with filename)

The bug occurred because both `resolveCardBookmark()` and `resolveOutputBookmark()` were updating their respective path strings, but only the card path should be updated (since it IS a directory). The destination path must preserve the filename.

---

## Session Log: 2026-01-05 (Multi-Card Loading)

### Overview
Added ability to load multiple P2 cards from a parent folder at once, plus Cmd+O keyboard shortcut.

### Features Added

#### 1. Multi-Card Discovery
- Select a parent folder containing multiple P2 cards
- App automatically discovers all P2 cards in immediate subdirectories
- Cards are loaded in parallel for speed
- Works with single P2 card selection too (backwards compatible)

#### 2. Keyboard Shortcut
- **Cmd+O** opens the P2 card picker from anywhere in the app
- Added via SwiftUI `.commands` modifier with NotificationCenter bridge

### Key Code

#### P2CardParser.discoverP2Cards(in:)
```swift
func discoverP2Cards(in url: URL) -> [URL] {
    // First check if URL itself is a P2 card
    if validateP2Structure(at: url) { return [url] }
    
    // Otherwise scan subdirectories
    let contents = try fm.contentsOfDirectory(at: url, ...)
    return contents.filter { validateP2Structure(at: $0) }
                   .sorted { $0.lastPathComponent < $1.lastPathComponent }
}
```

#### ConversionViewModel.loadP2Cards(from:)
- Uses `withTaskGroup` for parallel parsing
- Collects results then updates UI on MainActor
- Swift 6 compliant (no captured var mutations in concurrent code)

### Files Modified
- `P2toMXF/Services/P2CardParser.swift` - Added `discoverP2Cards(in:)`
- `P2toMXF/ConversionViewModel.swift` - Added `loadP2Cards(from:)`
- `P2toMXF/ContentView.swift` - Updated fileImporter, added notification listener
- `P2toMXF/P2toMXFApp.swift` - Added File menu with Cmd+O shortcut

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Xpycode)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Xpycode)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
