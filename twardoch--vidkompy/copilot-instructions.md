## vidkompy

> - Iterate gradually, avoiding major changes

## 1. Coding style

<guidelines>
# When you write code

- Iterate gradually, avoiding major changes
- Minimize confirmations and checks
- Preserve existing code/structure unless necessary
- Use constants over magic numbers
- Check for existing solutions in the codebase before starting
- Check often the coherence of the code you’re writing with the rest of the code.
- Focus on minimal viable increments and ship early
- Write explanatory docstrings/comments that explain what and WHY this does, explain where and how the code is used/referred to elsewhere in the code
- Analyze code line-by-line
- Handle failures gracefully with retries, fallbacks, user guidance
- Address edge cases, validate assumptions, catch errors early
- Let the computer do the work, minimize user decisions
- Reduce cognitive load, beautify code
- Modularize repeated logic into concise, single-purpose functions
- Favor flat over nested structures
- Consistently keep, document, update and consult the holistic overview mental image of the codebase.

Work in rounds:

- Create `PLAN.md` as a detailed flat plan with `[ ]` items.
- Identify the most important TODO items, and create `TODO.md` with `[ ]` items.
- Implement the changes.
- Update `PLAN.md` and `TODO.md` as you go.
- After each round of changes, update `CHANGELOG.md` with the changes.
- Update `README.md` to reflect the changes.

## 2. Keep track of paths

In each source file, maintain the up-to-date `this_file` record that shows the path of the current file relative to project root. Place the `this_file` record near the top of the file, as a comment after the shebangs, or in the YAML Markdown frontmatter.

## 3. When you write Python

- Use `uv pip`, never `pip`
- Use `python -m` when running code
- PEP 8: Use consistent formatting and naming
- Write clear, descriptive names for functions and variables
- PEP 20: Keep code simple and explicit. Prioritize readability over cleverness
- Use type hints in their simplest form (list, dict, | for unions)
- PEP 257: Write clear, imperative docstrings
- Use f-strings. Use structural pattern matching where appropriate
- ALWAYS add "verbose" mode logugu-based logging, & debug-log
- For CLI Python scripts, use fire & rich, and start the script with

```
#!/usr/bin/env -S uv run -s
# /// script
# dependencies = ["PKG1", "PKG2"]
# ///
# this_file: PATH_TO_CURRENT_FILE
```

Ask before extending/refactoring existing code in a way that may add complexity or break things.

When you’re finished, print "Wait, but" to go back, think & reflect, revise & improvement what you’ve done (but don’t invent functionality freely). Repeat this. But stick to the goal of "minimal viable next version". Lead two experts: "Ideot" for creative, unorthodox ideas, and "Critin" to critique flawed thinking and moderate for balanced discussions. The three of you shall illuminate knowledge with concise, beautiful responses, process methodically for clear answers, collaborate step-by-step, sharing thoughts and adapting. If errors are found, step back and focus on accuracy and progress.

## 4. After Python changes run:

```
./cleanup.sh;
```

Be creative, diligent, critical, relentless & funny! </guidelines>

# `vidkompy`

[![PyPI](https://img.shields.io/pypi/v/vidkompy.svg)](https://pypi.org/project/vidkompy/) [![License](https://img.shields.io/github/license/twardoch/vidkompy.svg)](

**Intelligent Video Overlay and Synchronization**

`vidkompy` is a powerful command-line tool engineered to overlay a foreground video onto a background video with exceptional precision and automatic alignment. The system intelligently handles discrepancies in resolution, frame rate, duration, and audio, prioritizing content integrity and synchronization accuracy over raw processing speed.

The core philosophy of `vidkompy` is to treat the **foreground video as the definitive source of quality and timing**. All its frames are preserved without modification or re-timing. The background video is dynamically adapted—stretched, retimed, and selectively sampled—to synchronize perfectly with every frame of the foreground content, ensuring a seamless and coherent final output.

---

## 5. Features

### 5.1. Video Composition

- **Automatic Spatial Alignment**: Intelligently detects the optimal x/y offset to position the foreground video within the background, even if they are cropped differently.
- **Advanced Temporal Synchronization**: Aligns videos with different start times, durations, and frame rates, eliminating temporal drift and ensuring content matches perfectly over time.
- **Foreground-First Principle**: Guarantees that every frame of the foreground video is included in the output, preserving its original timing and quality. The background video is adapted to match the foreground.
- **Drift-Free Alignment**: Uses optimized sliding window algorithms to create globally optimal, monotonic alignment, preventing the common "drift-and-catchup" artifacts.
- **High-Performance Processing**: Leverages multi-core processing, direct pixel comparison, and optimized video I/O to deliver results quickly.
- Sequential video composition is 10-100x faster than random-access methods.
- **Smart Audio Handling**: Automatically uses the foreground audio track if available, falling back to the background audio. The audio is correctly synchronized with the final video.
- **Flexible Operation Modes**: Supports specialized modes like `mask` for letterboxed content and `smooth` blending for seamless visual integration.

### 5.2. Thumbnail Detection (`align` module)

- **Multi-Precision Analysis**: Progressive refinement system with 5 precision levels (1ms to 200ms processing time)
- **Advanced Algorithm Suite**: 6 specialized algorithms including template matching, feature matching, phase correlation, and hybrid approaches
- **Parallel Processing**: Numba JIT compilation and ThreadPoolExecutor optimization for maximum performance
- **Multiple Detection Methods**: Template matching, AKAZE/ORB/SIFT feature detection, histogram correlation, and sub-pixel refinement
- **Intelligent Fallbacks**: Automatic algorithm selection and graceful fallback between detection methods
- **Rich Analysis**: Comprehensive confidence metrics, processing statistics, and alternative result comparison

## 6. How It Works

The `vidkompy` pipeline is a multi-stage process designed for precision and accuracy:

1.  **Video Analysis**: The tool begins by probing both background (BG) and foreground (FG) videos using `ffprobe` to extract essential metadata: resolution, frames per second (FPS), duration, frame count, and audio stream information.

2.  **Spatial Alignment**: To determine _where_ to place the foreground on the background, `vidkompy` extracts a sample frame from the middle of each video (where content is most likely to be stable). It then calculates the optimal (x, y) offset.

3.  **Temporal Alignment**: This is the core of `vidkompy`. To determine _when_ to start the overlay and how to map frames over time, the tool generates "fingerprints" of frames from both videos and uses Dynamic Time Warping (DTW) to find the best alignment path. This ensures every foreground frame is matched to the most suitable background frame.

4.  **Video Composition**: Once the spatial and temporal alignments are known, `vidkompy` composes the final video. It reads both video streams sequentially (for maximum performance) and, for each foreground frame, fetches the corresponding background frame as determined by the alignment map. The foreground is then overlaid at the correct spatial position.

5.  **Audio Integration**: After the silent video is composed, `vidkompy` adds the appropriate audio track (preferring the foreground's audio) with the correct offset to ensure it's perfectly synchronized with the video content.

## 7. Modular Architecture

`vidkompy` features a clean, modular architecture that separates concerns and enables easy extension:

### 7.1. Thumbnail Detection (`align` module)

- **Core Classes**: `ThumbnailFinder` orchestrates the detection process
- **Algorithms**: Multiple detection algorithms including template matching, feature matching, phase correlation, and hybrid approaches
- **Precision System**: Progressive refinement with 5 precision levels (0=ballpark ~1ms to 4=precise ~200ms)
- **Performance**: Numba-optimized computations with parallel processing support
- **Rich Output**: Comprehensive result display with confidence metrics and processing statistics

### 7.2. Video Composition (`comp` module)

- **Alignment Engines**: Specialized engines for different content types (Full, Mask)
- **Temporal Synchronization**: Advanced DTW-based frame mapping with zero-drift constraints
- **Audio Processing**: Intelligent audio track selection and synchronization
- **Performance Optimization**: Sequential processing with 10-100x speedup over random access

### 7.3. Key Design Principles

- **Separation of Concerns**: Each module handles a specific aspect of the processing pipeline
- **Algorithm Flexibility**: Multiple algorithms available with automatic fallbacks
- **Performance Focus**: Optimized for both speed and accuracy with configurable trade-offs
- **Extensibility**: Clean interfaces allow easy addition of new algorithms and features

## 8. The Algorithms

`vidkompy` employs several sophisticated algorithms to achieve its high-precision results.

### 8.1. Thumbnail Detection Algorithms (`align` module)

The modular thumbnail detection system provides multiple specialized algorithms for robust image matching:

#### 8.1.1. Template Matching Algorithm

- **Multi-Scale Processing**: Tests multiple scale factors in parallel using ThreadPoolExecutor
- **Ballpark Estimation**: Ultra-fast histogram correlation for initial scale estimation (~1ms)
- **Parallel Optimization**: Numba JIT compilation for critical computational functions
- **unscaled Bias**: Small preference for exact scale matches when confidence is similar
- **Normalized Cross-Correlation**: Uses OpenCV's TM_CCOEFF_NORMED for reliable matching

#### 8.1.2. Feature Matching Algorithm

- **Multiple Detectors**: Supports AKAZE (default), ORB, and SIFT feature detectors
- **Robust Matching**: Ratio test filtering and RANSAC-based outlier rejection
- **Transformation Estimation**: Uses estimateAffinePartial2D and homography methods
- **Confidence Calculation**: Based on inlier ratios and geometric consistency
- **Automatic Fallbacks**: Graceful degradation when detection methods fail

#### 8.1.3. Phase Correlation Algorithm

- **FFT-Based Processing**: Uses scikit-image's phase_cross_correlation
- **Sub-Pixel Accuracy**: 10x upsampling factor for precise position detection
- **Scale Integration**: Works with scale estimates from other algorithms
- **Error Conversion**: Transforms phase correlation error into confidence metrics

#### 8.1.4. Hybrid Matching Algorithm

- **Multi-Method Combination**: Intelligently combines feature, template, and phase correlation
- **Weighted Selection**: Results weighted by confidence and method reliability
- **Adaptive Strategy**: Adjusts approach based on initial feature detection success
- **Cascaded Processing**: Feature → Template → Phase correlation pipeline

#### 8.1.5. Histogram Correlation Algorithm

- **Ultra-Fast Processing**: Provides ballpark scale estimation in ~1ms
- **Multi-Region Sampling**: Tests correlation across multiple image regions
- **Normalized Histograms**: Robust to brightness and contrast variations
- **Numba Optimization**: JIT-compiled correlation functions for maximum speed

#### 8.1.6. Sub-Pixel Refinement Algorithm

- **Precision Enhancement**: Refines position estimates with sub-pixel accuracy
- **Local Search**: Tests sub-pixel offsets around initial estimates
- **Normalized Correlation**: Direct correlation calculation for fine-tuning
- **Quality Improvement**: Enhances results from other algorithms

### 8.2. Multi-Precision Analysis System

The system offers 5 precision levels with different speed/accuracy trade-offs:

- **Level 0 (Ballpark)**: Histogram correlation only (~1ms)
- **Level 1 (Coarse)**: Parallel template matching with wide steps (~10ms)
- **Level 2 (Balanced)**: Feature + template matching combination (~25ms, default)
- **Level 3 (Fine)**: Hybrid algorithm with multiple methods (~50ms)
- **Level 4 (Precise)**: Sub-pixel refinement for maximum accuracy (~200ms)

### 8.3. Video Composition Algorithms (`comp` module)

#### 8.3.1. Frame Fingerprinting (Perceptual Hashing)

Instead of comparing the millions of pixels in a frame, `vidkompy` creates a tiny, unique "fingerprint" (a hash) for each frame. Comparing these small fingerprints is thousands of times faster and smart enough to ignore minor changes from video compression.

---

The `FrameFingerprinter` module is designed for ultra-fast and robust frame comparison. It uses perceptual hashing, which generates a compact representation of a frame's visual structure.

The process works as follows:

1.  **Standardization**: The input frame is resized to a small, standard size (e.g., 64x64 pixels) and converted to grayscale. This ensures consistency and focuses on structural information over color.
2.  **Multi-Algorithm Hashing**: To improve robustness, `vidkompy` computes several types of perceptual hashes for each frame, as different algorithms are sensitive to different visual features:

- `pHash` (Perceptual Hash): Analyzes the frequency domain (using DCT), making it robust to changes in brightness, contrast, and gamma correction.
- `AverageHash`: Computes a hash based on the average color of the frame.
- `ColorMomentHash`: Captures the color distribution statistics of the frame.
- `MarrHildrethHash`: Detects edges and shapes, making it sensitive to structural features.

3.  **Combined Fingerprint**: The results from these hashers, along with a color histogram, are combined into a single "fingerprint" dictionary for the frame.
4.  **Comparison**: To compare two frames, their fingerprints are compared. The similarity is calculated using a weighted average of the normalized Hamming distance between their hashes and the correlation between their histograms. The weights are tuned based on the reliability of each hash type for video content. This entire process is parallelized across multiple CPU cores for maximum speed.

### 8.4. Spatial Alignment (Template Matching)

To find the correct position for the foreground video, the tool takes a screenshot from the middle of it and searches for that exact image within a screenshot from the background video.

---

Spatial alignment determines the `(x, y)` coordinates at which to overlay the foreground frame onto the background. `vidkompy` uses a highly accurate and efficient method based on template matching.

1.  **Frame Selection**: A single frame is extracted from the temporal midpoint of both the foreground and background videos. This is done to get a representative frame, avoiding potential opening/closing titles or black frames.
2.  **Grayscale Conversion**: The frames are converted to grayscale. This speeds up the matching process by 3x and makes the alignment more robust to minor color variations between the videos.
3.  **Template Matching**: The core of the alignment is `cv2.matchTemplate` using the `TM_CCOEFF_NORMED` method. This function effectively "slides" the smaller foreground frame image across the larger background frame image and calculates a normalized cross-correlation score at each position.
4.  **Locating the Best Match**: The position with the highest correlation score (from `cv2.minMaxLoc`) is considered the best match. This location `(x_offset, y_offset)` represents the top-left corner where the foreground should be placed. The confidence of this match is the correlation score itself, which typically approaches `1.0` for a perfect match.
5.  **Scaling**: The system checks if the foreground video is larger than the background. If so, it is scaled down to fit, and the scale factor is recorded.

### 8.5. Temporal Alignment Engines

`vidkompy` offers two high-performance temporal alignment engines optimized for different scenarios:

- **Full** (default): Direct pixel comparison with sliding windows for maximum accuracy
- **Mask**: Content-focused comparison with intelligent masking for letterboxed content

---

Temporal alignment is the most critical and complex part of `vidkompy`. The goal is to create a mapping `FrameAlignment(fg_frame_idx, bg_frame_idx)` for every single foreground frame. `vidkompy` provides two optimized engines for this task:

#### 8.5.1. Full Engine (Default)

The **Full Engine** uses direct pixel-by-pixel frame comparison with a sliding window approach for maximum accuracy:

1. **Bidirectional Matching**:

   - **Forward Pass**: Starts from the first FG frame, searches for best match in BG within a sliding window
   - **Backward Pass**: Starts from the last FG frame, searches backward
   - Merges both passes for robust alignment

2. **Sliding Window Constraint**:

   - Enforces monotonicity by design - can only search forward from the last matched frame
   - Window size controls the maximum temporal displacement
   - Prevents temporal jumps and ensures smooth progression

3. **Direct Pixel Comparison**:
   - Compares actual pixel values between FG and BG frames
   - No information loss from hashing or fingerprinting
   - More sensitive to compression artifacts but potentially more accurate

**Characteristics:**

- Processing time: ~40 seconds for an 8-second video (d10-w10 configuration)
- Zero drift by design due to monotonic constraints
- Perfect confidence scores (1.000)
- Best overall performance for standard videos

#### 8.5.2. Mask Engine (Content-Focused)

The **Mask Engine** extends the Full engine approach with intelligent masking for letterboxed or pillarboxed content:

1. **Content Mask Generation**:

   - Automatically detects content regions (non-black areas) in FG frames
   - Creates binary mask to focus comparison on actual content
   - Helps with letterboxed or pillarboxed videos

2. **Masked Comparison**:

   - Only compares pixels within the mask region
   - Ignores black borders and letterboxing
   - More robust for videos with varying aspect ratios

3. **Same Bidirectional Approach**:
   - Uses forward and backward passes like Full engine
   - Applies mask during all comparisons
   - Maintains monotonicity constraints

**Characteristics:**

- Processing time: ~45 seconds for an 8-second video (d10-w10 configuration)
- Perfect confidence scores (1.000)
- Better handling of videos with black borders
- Ideal for videos where content doesn't fill the entire frame

#### 8.5.3. Engine Comparison

| Aspect         | Full                    | Mask                            |
| -------------- | ----------------------- | ------------------------------- |
| **Algorithm**  | Direct pixel comparison | Masked pixel comparison         |
| **Speed**      | ~5x real-time           | ~5x real-time                   |
| **Drift**      | Zero (monotonic)        | Zero (monotonic)                |
| **Memory**     | Medium                  | Medium                          |
| **Confidence** | Perfect (1.000)         | Perfect (1.000)                 |
| **Best For**   | Standard videos         | Letterboxed/pillarboxed content |

## 9. Usage

### 9.1. Prerequisites

You must have the **FFmpeg** binary installed on your system and accessible in your system's `PATH`. `vidkompy` depends on it for all video and audio processing tasks.

### 9.2. Installation

The tool is a Python package. It is recommended to install it from the repository to get the latest version.

```bash
# Clone the repository
git clone https://github.com/twardoch/vidkompy.git
cd vidkompy

# Install using uv (or pip)
uv pip install .
```

### 9.3. Command-Line Interface (CLI)

`vidkompy` offers two main commands: video composition and thumbnail detection.

**Video Composition Examples:**

```bash
# Full engine (default) - direct pixel comparison with zero drift
python -m vidkompy comp --bg background.mp4 --fg foreground.mp4

# Mask engine for letterboxed/pillarboxed content
python -m vidkompy comp --bg background.mp4 --fg foreground.mp4 --engine mask

# Custom output path
python -m vidkompy comp --bg bg.mp4 --fg fg.mp4 --output result.mp4

# Fine-tune performance with drift interval and window size
python -m vidkompy comp --bg bg.mp4 --fg fg.mp4 --drift_interval 10 --window 10
```

**Thumbnail Detection Examples:**

The refactored thumbnail detection system now uses the `align` command with enhanced precision levels:

```bash
# Find thumbnail in image (new align command)
python -m vidkompy align foreground.jpg background.jpg

# Find thumbnail in video frame
python -m vidkompy align foreground.jpg background.mp4

# Precision levels 0-4 (0=ballpark ~1ms, 2=balanced ~25ms, 4=precise ~200ms)
python -m vidkompy align foreground.jpg background.jpg --precision 4

# Enable verbose logging for detailed analysis
python -m vidkompy align foreground.jpg background.jpg --verbose

# Multi-scale search (both no and scaled results)
python -m vidkompy align foreground.jpg background.jpg --unscaled false

# Process multiple video frames for better accuracy
python -m vidkompy align foreground.mp4 background.mp4 --num_frames 10

# Backward compatibility: 'find' command still works
python -m vidkompy find foreground.jpg background.jpg
```

**CLI Help:**

```bash
# Main command help
python -m vidkompy --help

# Video composition help
python -m vidkompy comp --help

# Thumbnail detection help (new align command)
python -m vidkompy align --help

# Backward compatibility
python -m vidkompy find --help
```

The CLI now supports these main commands:

- `comp`: Video composition with intelligent alignment
- `align`: Advanced thumbnail detection with multi-precision analysis
- `find`: Backward compatibility alias for thumbnail detection

**Video Composition Parameters:**

- `--bg`: Background video path
- `--fg`: Foreground video path
- `--output`: Output video path (auto-generated if not provided)
- `--engine`: Temporal alignment engine - 'full' (default) or 'mask'
- `--margin`: Border thickness for border matching mode (default: 8)
- `--smooth`: Enable smooth blending at frame edges
- `--verbose`: Enable verbose logging

**Thumbnail Detection Parameters (align command):**

- `fg`: Foreground image/video path (first positional argument)
- `bg`: Background image/video path (second positional argument)
- `--precision`: Precision level 0-4 (0=ballpark ~1ms, 2=balanced ~25ms, 4=precise ~200ms, default: 2)
- `--num_frames`: Maximum number of frames to process for videos (default: 7)
- `--unscaled`: If True, only search at 100% scale; if False, search both no and multi-scale (default: True)
- `--verbose`: Enable detailed output and debug logging

## 10. Performance

Recent updates have significantly improved `vidkompy`'s performance and accuracy:

### 10.1. Thumbnail Detection Performance (`align` module)

The modular architecture delivers excellent performance across different precision levels:

| Precision Level | Processing Time | Use Case | Algorithms Used |
| --- | --- | --- | --- |
| **Level 0 (Ballpark)** | ~1ms | Ultra-fast scale estimation | Histogram correlation only |
| **Level 1 (Coarse)** | ~10ms | Quick template matching | Parallel template matching |
| **Level 2 (Balanced)** | ~25ms | Default balanced approach | Feature + template matching |
| **Level 3 (Fine)** | ~50ms | High-quality detection | Hybrid multi-algorithm |
| **Level 4 (Precise)** | ~200ms | Maximum accuracy | Sub-pixel refinement |

**Key Performance Features:**

- **Numba JIT Optimization**: Critical functions compiled for 5-20x speed improvements
- **Parallel Processing**: ThreadPoolExecutor for concurrent scale testing
- **Intelligent Caching**: Algorithm results cached for repeated operations
- **Memory Efficiency**: Optimized data structures and streaming processing
- **Graceful Degradation**: Automatic fallbacks maintain performance when algorithms fail

### 10.2. Video Composition Performance (`comp` module)

Based on actual benchmarks with an 8-second test video (1920x1080 background, 1920x870 foreground, ~480 frames):

| Engine | Processing Time | Speed Ratio | Confidence | Notes |
| --- | --- | --- | --- | --- |
| **Full (default)** | 40.9 seconds | ~5x real-time | 1.000 (perfect) | Fastest overall with zero drift |
| **Mask** | 45.8 seconds | ~6x real-time | 1.000 (perfect) | Best for letterboxed content |

**Key Performance Insights:**

- **Full Engine**: Delivers perfect confidence scores (1.000) with ~5x real-time processing. Uses direct frame mapping which completely eliminates drift while maintaining excellent performance.

- **Mask Engine**: Slightly slower than Full engine but achieves perfect confidence. Ideal for content with black borders or letterboxing where content-focused comparison is beneficial.

### 10.3. Technical Optimizations

- **Zero Drift Design**: Both engines use sliding window constraints that enforce monotonicity by design, completely eliminating temporal drift.
- **Optimized Compositing**: Sequential frame reading instead of random access yields a **10-100x speedup** in the final composition stage.
- **Direct Pixel Comparison**: Frame comparison uses actual pixel values without information loss from hashing or compression.
- **Bidirectional Matching**: Forward and backward passes are merged for robust alignment results.
- **Efficient Memory Usage**: Both engines use streaming processing with reasonable memory footprints.

### 10.4. Choosing the Right Engine

**Use the Full Engine (default) when:**

- Working with standard videos without letterboxing
- You need the fastest processing with perfect accuracy
- Videos have consistent content filling the frame
- General-purpose video synchronization

**Use the Mask Engine when:**

- Working with letterboxed or pillarboxed content
- Videos have significant black borders
- Content doesn't fill the entire frame
- Aspect ratio mismatches between foreground and background

## 11. Development

To contribute to `vidkompy`, set up a development environment using `hatch`.

### 11.1. Setup

1.  Clone the repository.
2.  Ensure you have `hatch` installed (`pip install hatch`).
3.  The project is managed through `hatch` environments defined in `pyproject.toml`.

### 11.2. Key Commands

Run these commands from the root of the repository.

- **Run Tests**:

```bash
hatch run test
```

- **Run Tests with Coverage Report**:

```bash
hatch run test-cov
```

- **Run Type Checking**:

```bash
hatch run type-check
```

- **Check Formatting and Linting**:

```bash
hatch run lint
```

- **Automatically Fix Formatting and Linting Issues**:

```bash
hatch run fix
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/twardoch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
