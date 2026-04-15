## videogofaster

> * **App Name:** VideoGoFaster

# Implementation Spec: VideoGoFaster (High-Fidelity 10-bit)

## 1\. Project Overview

* **App Name:** VideoGoFaster
* **Goal:** A single-screen Android application that ingests a video file, discards audio, and
produces three distinct high-quality output videos at 2x, 4x, and 8x speeds.
* **Core Mechanism:** Speed is achieved by **blending** frames (averaging pixel data) rather than
dropping them. This creates a smooth "motion blur" aesthetic.
* **Quality Requirement:** The pipeline must operate in **10-bit color depth** to prevent "binning" or
color banding during the blending process. All outputs are generated directly from the original
source to avoid generation loss.

## 2\. Tech Stack

* **Language:** Kotlin
* **UI Framework:** Jetpack Compose (Material 3)
* **Media Engine:** `FFmpegKit` (Min-GPL or Full-GPL variant required for filter support).
* **Architecture:** MVVM (Model-View-ViewModel).
* **Concurrency:** Kotlin Coroutines (`Dispatchers.IO` for processing).

## 3\. Architecture & Data Flow

### A. The Processing Strategy (Parallel/Direct)

To maximize quality and avoid re-compression artifacts, the app must **not** use the output of the
2x video to generate the 4x video.

* **Correct Flow:**
    * Original Source → Process 2x → Output A
    * Original Source → Process 4x → Output B
    * Original Source → Process 8x → Output C

### B. The 10-bit Pipeline (Crucial)

Standard video processing often happens in 8-bit (0-255). When blending two similar colors (e.g.,
100 and 101), 8-bit math rounds the result, losing detail.

* **Requirement:** The input stream must be explicitly promoted to 10-bit (`yuv420p10le`) **before**
  any filtering occurs. This expands the color space (0-1023), allowing the mathematical blend to
  retain sub-pixel precision before final encoding.

## 4\. Implementation Steps

### Step 1: Dependencies

Add `FFmpegKit` to `build.gradle.kts` (app module). Use the `full` or `min-gpl` version to ensure
complex filter support.

```kotlin
dependencies {
    // FFmpegKit Full (GPL) for filter availability and x265 support
    implementation("com.arthenica:ffmpeg-kit-full-gpl:6.0")

    // UI & Core
    implementation(platform("androidx.compose:compose-bom:2023.08.00"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.material3:material3")
    implementation("androidx.activity:activity-compose:1.8.0")
}
```

### Step 2: Permissions & Manifest

* **Permissions:** `READ_MEDIA_VIDEO` (Android 13+), `READ_EXTERNAL_STORAGE` (Legacy).
* **Legacy Support:** If targeting \< Android 10, include `WRITE_EXTERNAL_STORAGE`.
* **Manifest:** Ensure `android:requestLegacyExternalStorage="true"` is set if supporting older
  Android versions.

### Step 3: The Video Processor Logic

Create a class `VideoProcessor`.

#### The FFmpeg Command Structure

The command must perform these actions in strict order:

1. **Input:** Read file.
2. **Format Promotion:** Convert to `yuv420p10le` (10-bit).
3. **Blend (T-Mix):** Mix $N$ frames together.
4. **Decimation (Select):** Keep only every $N$th frame.
5. **Retiming (SetPTS):** Adjust timestamps for the new speed.
6. **Encoding:** Encode using `libx265` (HEVC Software Encoder) to guarantee 10-bit support and wide
   device compatibility.

#### Helper Function

```kotlin
/**
 * @param blendFactor: The speed multiplier (2, 4, or 8).
 * This determines how many frames are blended and the decimation rate.
 */
suspend fun processVideo(
    inputPath: String,
    outputPath: String,
    blendFactor: Int
): FFmpegSession {
    // 1. Calculate PTS modifier (e.g., 0.5 for 2x speed)
    val ptsModifier = 1.0 / blendFactor

    // 2. Build the Filter String
    // - format=yuv420p10le: Promotes 8-bit inputs to 10-bit immediately.
    // - tmix=frames=N: Blends N frames.
    // - select='not(mod(n,N))': Keeps only the blended frames (effectively dropping intermediates).
    // - setpts=M*PTS: Speeds up playback to match frame reduction.
    val filterChain =
        "format=pix_fmts=yuv420p10le,tmix=frames=$blendFactor,select='not(mod(n\\,$blendFactor))',setpts=$ptsModifier*PTS"

    // 3. Construct Command
    // -c:v libx265: Use software HEVC encoder for consistent 10-bit support.
    // -crf 20: High quality variable bitrate.
    // -an: Remove audio.
    val cmd =
        "-y -i \"$inputPath\" -vf \"$filterChain\" -c:v libx265 -crf 20 -preset medium -pix_fmt yuv420p10le -an \"$outputPath\""

    return FFmpegKit.execute(cmd)
}
```

### Step 4: Software Fallback & Error Handling

While `libx265` is a software encoder (CPU based) and highly compatible, it is intensive.

1. **Error Catching:** Check `session.returnCode`. If it fails, log the `session.failStackTrace`.
2. **Fallback (Optional but Robust):** If the 10-bit command fails (rare, usually due to extreme
   resolution limits), retry with standard 8-bit `pix_fmt yuv420p`, but keep the 10-bit path as the
   primary default.

### Step 5: User Interface (Jetpack Compose)

A single screen `MainActivity.kt`.

1. **State Management:**

    * `isLoading`: Boolean
    * `currentStep`: String ("Idle", "Processing 2x...", "Processing 4x...", "Processing 8x...")
    * `outputLogs`: List\<String\> (Paths of saved files)

2. **Components:**

    * **Button:** "Select Video" (Launches `ActivityResultContracts.PickVisualMedia`).
    * **Progress Indicator:** `LinearProgressIndicator` (visible when `isLoading` is true).
    * **Text Output:** Displays the status and resulting file paths.

### Step 6: Workflow Implementation (The "Jules" Logic)

When a video is selected:

1. **Copy to Cache:** Copy the Uri to a local `File` in `context.cacheDir` (FFmpeg needs a real file
   path, not a content Uri).
2. **Output Setup:** Prepare destination paths in `Environment.DIRECTORY_MOVIES` or App Storage.
3. **Sequential Execution (Coroutine):**
   ```kotlin
   scope.launch(Dispatchers.IO) {
       // Step A: 2x
       updateStatus("Blending 2x (High Quality)...")
       processVideo(inputFile, output2x, blendFactor = 2)
       
       // Step B: 4x (From Original)
       updateStatus("Blending 4x (High Quality)...")
       processVideo(inputFile, output4x, blendFactor = 4)
       
       // Step C: 8x (From Original)
       updateStatus("Blending 8x (High Quality)...")
       processVideo(inputFile, output8x, blendFactor = 8)
       
       updateStatus("All Completed.")
   }
   ```

## 5\. File Naming Convention

To ensure clarity for the user, outputs should be named:

* `[OriginalName]_2x_10bit.mp4`
* `[OriginalName]_4x_10bit.mp4`
* `[OriginalName]_8x_10bit.mp4`

## 6\. Summary of Key Requirements for Code Generation

1. **Use `libx265`**: Do not rely on hardware encoders (`hevc_mediacodec`) as they often fail with
   custom 10-bit pixel formats on Android.
2. **Filter Order**: `format=pix_fmts=yuv420p10le` MUST be the first filter.
3. **Recursive Avoidance**: Always use the cached **original** file as input for every speed, never
   the output of the previous step.
4. **Audio**: Use `-an` to discard audio.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/salamanders)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/salamanders)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
