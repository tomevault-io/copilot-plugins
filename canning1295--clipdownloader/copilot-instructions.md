## clipdownloader

> * **Purpose:** A simple, reliable macOS desktop app that lets me (I will be the only user ever of this app) enter a **YouTube URL + start/stop times**, pick **quality/container** options, and **save** the resulting clip locally.

# Project: ClipDownloader (macOS Desktop Only)

## 0) Purpose & Non-Goals

* **Purpose:** A simple, reliable macOS desktop app that lets me (I will be the only user ever of this app) enter a **YouTube URL + start/stop times**, pick **quality/container** options, and **save** the resulting clip locally.
* **Non-Goals:** No playlist scraping, no preview player, no background downloading service, no login/cookies. Not targeting the Mac App Store (MAS).

---

## 1) Technology & Distribution

* **Platform:** macOS 15+, desktop only, apple silicon
* **UI:** SwiftUI (App lifecycle), with AppKit bridges for folder pickers.
* **Process execution:** `Process` + `Pipe` for invoking bundled binaries.
* **Bundled tools:**

  * `yt-dlp` (standalone macOS universal2 executable).
  * `ffmpeg` (universal2, **LGPL-only build** preferred; use `h264_videotoolbox` and `aac_at` to avoid GPL encoders).
* **Packaging:** Include binaries under `ClipDownloader.app/Contents/Resources/bin/`.
* **Code signing & notarization:** Developer ID + Hardened Runtime. (Distribution outside MAS.)
* **Sandbox:** Start **without App Sandbox** to simplify invoking helper binaries (or, if sandboxed later, ensure helpers are inside the bundle and signed with same team ID).

---

## 2) UX / UI Design (Desktop-only)

**Single-window, clean layout:**

1. **Source Panel**

   * **TextField**: “YouTube URL”
   * **Time Inputs**: “Start” + “End”

     * Accept `SS`, `MM:SS`, or `HH:MM:SS`
   * **Validation labels** (inline): invalid URL, malformed time, end <= start.

2. **Options Panel**

   * **Quality (Picker):** Auto (best ≤1080p), 1080p, 720p, 480p.
   * **Container (Picker):** MP4 (default), WebM, Audio-only (M4A/Opus).
   * **Clip Accuracy (Radio):**

     * **Frame-accurate (re-encode)** — default, slower, exact.
     * **Keyframe-only (no re-encode)** — very fast, may trim to nearest keyframe.
   * **Advanced (Disclosure):**

     * “Download only the selected section (yt-dlp `--download-sections`)” (experimental).
     * Video bitrate (if re-encoding), Audio bitrate.
     * Hardware acceleration toggle (defaults on; `h264_videotoolbox`).

3. **Output Panel**

   * **Folder selector** (defaults to `~/Downloads/ClipDownloader/`).
   * **Filename field** with tokens support (live preview):

     * Tokens: `{title}`, `{id}`, `{start}`, `{end}`, `{res}`, `{container}`
     * Default: `{title}_{start}-{end}.{container}`

4. **Run Controls**

   * **Primary button:** “Download Clip”
   * **Progress view** with **two stages**:

     * Stage A: “Fetching & downloading source” (yt-dlp progress %).
     * Stage B: “Cutting & encoding” (ffmpeg progress %).
   * **Console log** area (collapsible).
   * **Cancel** button (terminates current process tree).
   * **Result toast** with “Reveal in Finder”.

5. **Help & Compliance**

   * Mini help popover: time format examples.
   * Reuse / fair-use reminder.

**Keyboard shortcuts:**

* ⌘R — Run (Download Clip)
* ⌘. — Cancel
* ⌘O — Choose output folder

**Accessibility:**

* Proper labels, VoiceOver friendly, high-contrast validation states.

---

## 3) Data Model & State

```swift
struct ClipRequest: Codable, Equatable {
    var url: String
    var startTime: String // raw user input
    var endTime: String   // raw user input
    var quality: VideoQuality   // .auto1080, .p1080, .p720, .p480
    var container: Container    // .mp4, .webm, .m4a, .opus
    var accuracy: Accuracy      // .frameAccurate, .keyframeCopy
    var useDownloadSections: Bool
    var videoBitrateMbps: Double? // re-encode only
    var audioBitrateKbps: Int?    // re-encode only
    var outputFolder: URL
    var filenameTemplate: String
}

enum JobStage { case idle, downloading, cutting, finished, failed, canceled }
```

**Preferences (UserDefaults):**

* Last used output folder
* Last selected quality/container/accuracy
* Filename template
* Show legal reminder (dismissed?)

---

## 4) Core Workflow

### 4.1 Validation

* URL must match YouTube patterns.
* Parse times into seconds (support `SS`, `MM:SS`, `HH:MM:SS`).
* Ensure `end > start`.
* If container = M4A/Opus → auto-set audio-only formats in yt-dlp.

### 4.2 Build yt-dlp command

* **Format string** (quality picker):

  * Auto ≤1080p: `bv*[height<=1080]+ba/b[height<=1080]/b`
  * 1080p: `bv*[height=1080]+ba/b[height=1080]/b`
  * 720p: `bv*[height=720]+ba/b[height=720]/b`
  * 480p: `bv*[height=480]+ba/b[height=480]/b`
* **Container merge preference:** `--merge-output-format mp4` (if MP4) or `webm` if chosen; for audio-only use `-x --audio-format m4a` or `opus`.
* **Output template:** temp dir, e.g., `/tmp/clipdownloader/<jobID>/input.%(ext)s`
* **Sections (optional):** `--download-sections "*START-END"` (formatted HH\:MM\:SS). Keep in Advanced; still keep ffmpeg stage for final trimming if accuracy requested.

**Example (auto 1080p, MP4):**

```
yt-dlp --no-playlist \
  -f 'bv*[height<=1080]+ba/b[height<=1080]/b' \
  -o '/tmp/clipdownloader/JOB/input.%(ext)s' \
  --merge-output-format mp4 \
  --newline \
  URL
```

### 4.3 Parse yt-dlp progress

* Launch with `--newline` and read `stderr` line-by-line.
* Parse lines beginning with `[download]` for percent like `(\d+\.\d+)%`.
* Update `JobStage.downloading` progress (0–1.0).

### 4.4 Cut with ffmpeg

* Determine **accurate vs keyframe copy**:

  * **Accurate (re-encode):**

    ```
    ffmpeg -y -ss START -to END -i input.mp4 \
      -c:v h264_videotoolbox -b:v {videoBitrate}M -maxrate {ceil(1.5×)}M -bufsize {2×}M \
      -c:a aac_at -b:a {audioBitrate}k \
      -movflags +faststart \
      -progress pipe:1 -nostats OUTPUT
    ```
  * **Keyframe-only (no re-encode):**

    ```
    ffmpeg -y -ss START -to END -i input.mp4 \
      -c copy \
      -movflags +faststart \
      -progress pipe:1 -nostats OUTPUT
    ```
* Read `-progress pipe:1` key=value pairs; compute % from `out_time_ms` vs total duration (`END-START`).

### 4.5 Save result

* Resolve filename from template (sanitize illegal characters).
* Write to chosen folder; if exists, append “(1)”.
* Show completion toast + “Reveal in Finder”.

### 4.6 Cancellation & Cleanup

* On Cancel, terminate child `Process` (and any spawned sub-processes).
* Always cleanup temp dir.

---

## 5) File & Module Structure

```
ClipDownloader/
  ClipDownloaderApp.swift           // @main entry
  Models/
    ClipRequest.swift
    Enums.swift                 // VideoQuality, Container, Accuracy, JobStage
    Validation.swift            // URL + time parsing
  Services/
    Toolchain.swift             // resolves paths to bundled yt-dlp/ffmpeg
    YtDlpService.swift          // builds & runs yt-dlp, progress parsing
    FFmpegService.swift         // builds & runs ffmpeg, progress parsing
    FilenameTemplating.swift    // {title}/{start}/{end} substitutions
    JobOrchestrator.swift       // state machine for a run
  Views/
    ContentView.swift           // main form
    OptionsView.swift
    OutputView.swift
    ProgressConsoleView.swift
  ViewModels/
    ClipViewModel.swift         // binds UI to JobOrchestrator
  Resources/
    bin/yt-dlp                  // executable (universal2)
    bin/ffmpeg                  // executable (universal2, LGPL)
    Licenses/                   // license texts for bundled tools
```

---

## 6) Key Implementation Details

### 6.1 Bundled Binaries Resolution

```swift
enum Toolchain {
    static func binURL(_ name: String) -> URL {
        Bundle.main.url(forResource: "bin/\(name)", withExtension: nil)!
    }
}
```

* Mark both files as **Executable** in build phase.
* Ensure they’re **codesigned** with the app (copy phase: “Code Sign on Copy”).

### 6.2 Secure Process Launch Helper

```swift
func runProcess(executable: URL,
                arguments: [String],
                environment: [String: String] = [:],
                onStdout: @escaping (String)->Void,
                onStderr: @escaping (String)->Void) async throws -> Int32 {
    let proc = Process()
    proc.executableURL = executable
    proc.arguments = arguments
    proc.environment = environment

    let outPipe = Pipe(); let errPipe = Pipe()
    proc.standardOutput = outPipe
    proc.standardError  = errPipe

    try proc.run()

    // async readers
    Task { for try await line in outPipe.fileHandleForReading.bytes.lines { onStdout(String(line)) } }
    Task { for try await line in errPipe.fileHandleForReading.bytes.lines { onStderr(String(line)) } }

    await withCheckedContinuation { cont in
        proc.terminationHandler = { _ in cont.resume() }
    }
    return proc.terminationStatus
}
```

### 6.3 Time Parsing

```swift
func parseTime(_ s: String) -> Double? {
    // Accept "SS", "MM:SS", "HH:MM:SS"
    let parts = s.split(separator: ":").map(String.init).reversed()
    var sec = 0.0
    for (i, p) in parts.enumerated() {
        guard let v = Double(p) else { return nil }
        sec += v * pow(60, Double(i))
    }
    return sec
}
```

### 6.4 Building yt-dlp Args (examples)

```swift
func ytdlpArgs(for req: ClipRequest, tempOut: URL) -> [String] {
    var fmt: String = "bv*[height<=1080]+ba/b[height<=1080]/b"
    switch req.quality {
    case .p1080: fmt = "bv*[height=1080]+ba/b[height=1080]/b"
    case .p720:  fmt = "bv*[height=720]+ba/b[height=720]/b"
    case .p480:  fmt = "bv*[height=480]+ba/b[height=480]/b"
    case .auto1080: break
    }

    var args = ["--no-playlist", "--newline", "-f", fmt, "-o", tempOut.path]
    switch req.container {
    case .mp4:  args += ["--merge-output-format", "mp4"]
    case .webm: args += ["--merge-output-format", "webm"]
    case .m4a:  args += ["-x", "--audio-format", "m4a"]
    case .opus: args += ["-x", "--audio-format", "opus"]
    }
    if req.useDownloadSections {
        args += ["--download-sections", "*\(req.startTime)-\(req.endTime)"]
    }
    args.append(req.url)
    return args
}
```

### 6.5 Building ffmpeg Args

```swift
func ffmpegArgs(input: URL, output: URL, req: ClipRequest, start: String, end: String) -> [String] {
    var args = ["-y", "-ss", start, "-to", end, "-i", input.path,
                "-movflags", "+faststart",
                "-progress", "pipe:1", "-nostats"]

    switch req.accuracy {
    case .keyframeCopy:
        args += ["-c", "copy", output.path]
    case .frameAccurate:
        // defaults
        let vBit = (req.videoBitrateMbps ?? 5.0)
        let aBit = (req.audioBitrateKbps ?? 160)
        args += ["-c:v", "h264_videotoolbox",
                 "-b:v", "\(vBit)M",
                 "-maxrate", "\(ceil(vBit * 1.5))M",
                 "-bufsize", "\(Int(vBit * 2))M"]
        args += ["-c:a", "aac_at", "-b:a", "\(aBit)k", output.path]
    }
    return args
}
```

### 6.6 Progress Parsing (ffmpeg)

* Use `out_time_ms` from `-progress` output.
* Compute `progress = min(1.0, out_time_ms / (durationSeconds * 1000000))`.

---

## 7) Error Handling & Edge Cases

* **Network / 404 / removal:** surface yt-dlp error lines; suggest checking URL.
* **Age-restricted / region-blocked:** surface error; do not implement cookies/login.
* **Invalid times:** show inline error; disable Run button.
* **Zero-length clip:** disallow; highlight end time.
* **No disk space or write perms:** present actionable alert; allow choosing another folder.
* **ffmpeg copy mode failures** when times aren’t on keyframes → instruct to retry with frame-accurate mode.
* **Unexpected tool missing/permission:** verify binaries executable on first run; if not, re-chmod +x and retry.

---

## 8) Preferences & Filename Tokens

* Render tokens with safe replacements:

  * `{title}` → fetched from yt-dlp metadata (`-J` JSON or `%()` in filename then read back).
  * `{start}`, `{end}` → canonical `HH-MM-SS`.
  * `{res}` → selected or detected height.
  * `{container}` → `mp4/webm/m4a/opus`.
* Sanitize file name for `/ : ? * " < > |` etc.

---

## 9) Testing Plan

* **Unit tests:** time parsing, argument builders, filename templating, sanitizer.
* **Integration tests (local fixtures):** run ffmpeg on a sample file to verify cutting modes.
* **Manuals:** several real YT URLs (public domain or your own uploads), all quality/container combos, path with spaces, long titles.
* **Performance:** ensure UI remains responsive; parsing pipes on a background actor.

---

## 10) Security, Signing, Licensing

* **Code sign** app + bundled binaries with same Team ID; enable **Hardened Runtime**.
* **Notarize** before distributing.
* **Licenses:** include `Licenses/` with texts for yt-dlp and ffmpeg; if distributing an ffmpeg build, comply with its license (prefer LGPL components only; avoid GPL encoders like `libx264`).
* **Disclaimer UI:** “Only download/clip content you’re authorized to use.”

---

## 11) Future Enhancements (Out of Scope Now)

* Clip **preview** player.
* Batch queue of multiple clips.
* Metadata sidecar (CSV/JSON) export.
* Automatic chapter/title token extraction.
* Localization.

---

## 12) Acceptance Criteria

* App launches on macOS 15+.
* User can paste a YouTube URL, enter `Start`/`End`, choose quality/container, and **successfully save** a clip to a chosen folder with a chosen name.
* Progress shows **two stages** (download → cut).
* **Frame-accurate** and **keyframe-only** modes both work.
* Output filename follows the template, sanitized, with token replacement.
* Binaries bundled, signed, and run without additional installs.

---

## 13) Developer Checklist (Build Steps)

1. Add `yt-dlp` and `ffmpeg` universal2 binaries to `Resources/bin/`, mark executable, ensure “Code Sign on Copy”.
2. Implement `Toolchain`, `YtDlpService`, `FFmpegService`, and `JobOrchestrator`.
3. Build SwiftUI views and validation.
4. Wire progress parsing to UI.
5. Add preferences + filename templating.
6. Sign, enable Hardened Runtime, notarize.
7. Smoke test on Apple Silicon and Intel (or Rosetta).

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/canning1295)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/canning1295)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
