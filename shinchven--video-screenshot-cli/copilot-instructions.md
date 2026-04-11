## video-screenshot-cli

> Create a CLi in python that reads a video file, and takes screenshots at specified intervals, saving them to a designated directory.

## Overview

Create a CLi in python that reads a video file, and takes screenshots at specified intervals, saving them to a designated directory.

## Installation

Before running the CLI, the required Python packages must be installed.

```bash
pip install opencv-python pillow
```

## Project Structure and Distribution

This project will be managed using a `pyproject.toml` file to define project metadata, dependencies, and entry points.

*   **Source Control:** The project will be hosted on GitHub.
*   **Dependencies:** Project dependencies, such as `opencv-python` and `pillow`, will be listed in the `pyproject.toml` file.
*   **Installation:** The CLI tool will be installable directly from the GitHub repository using pip:

    ```bash
    pip install git+https://github.com/ShinChven/video-screenshot-cli.git
    ```
*   **Entry Point:** The `pyproject.toml` file will define a script entry point, allowing the tool to be run as `vid-scr` from the command line after installation.

## The Command Line Interface (CLI)

Utility: vid-scr
Purpose: Capture thumbnails from a video at regular time intervals and optionally combine them into a single composite image.

Usage:
```bash
vid-scr <video-file> [--interval <seconds>] [--output <directory>] [--merge] [--size <pixels>] [--count <captures>] [--limit <max>]
```

| Option | Description | Default |
|---|---|---|
| `--interval <seconds>` | Time between captured frames. | `10` |
| `--output <directory>` | Directory where thumbnails will be written. | current working directory |
| `--merge` | If provided, produce a merged contact-sheet/combined image from the captured thumbnails. This option does not remove the individual screenshot images. | disabled |
| `--size <pixels>` | Constrain the longest side of the output image to a specific length in pixels, while maintaining the original aspect ratio. | |
| `--count <captures>` | Number of thumbnails to capture, evenly spaced across the video. | |
| `--limit <max>` | Maximum number of thumbnails to keep; extra captures beyond the cap are skipped. | |

### Behavior
- The output directory will be created if it does not already exist.
- When `--count` is provided, the requested number of thumbnails are evenly spaced from the first frame through the final frame (a single capture targets the midpoint).
- With `--interval`, thumbnails are captured at the requested cadence until the end of the video.
- When `--limit` is provided, only the earliest captures up to that maximum are saved.
- Thumbnails are emitted as image files (PNG or JPEG).
- The output file name will be in the format of `<video_filename>_screenshot_<index>.<format>`.
- When --merge is used, thumbnails are laid out in a grid.
- Progress, warnings, and errors should be reported to standard output/error.

### Constraints
- Interval must be a positive numeric value (integer or float).
- Limit must be a positive integer when used.
- Input validation: confirm the video file exists and is a supported format before processing.
- Video metadata (FPS and frame count) must be readable; otherwise the CLI exits with an error message.

## Video Processing and Screenshotting

The core of the utility will be implemented using the OpenCV library (`cv2`) for video processing and Pillow (`PIL`) for image manipulation. The process is as follows:

1.  **Video Loading:** The video file will be opened and read using `cv2.VideoCapture`.
2.  **Duration and Frame Rate:** The video's duration and frame rate will be determined using properties of the `VideoCapture` object. This is essential for calculating screenshot intervals, especially when the `--count` option is used.
3.  **Frame Seeking:**
    *   If `--interval` is used, the frame number to capture at each interval will be calculated (`frame_number = interval_in_seconds * frame_rate`).
    *   If `--count` is used, capture timestamps are evenly distributed across the full duration (inclusive of both ends, or the midpoint when a single capture is requested) and the corresponding frame numbers are determined from those timestamps.
4.  **Screenshot Capture:** The `VideoCapture` object will seek to the calculated frame number and the frame data will be read.
5.  **Image Saving:** The captured frame will be saved as a PNG image file in the specified output directory using `cv2.imwrite`.
6.  **Image Resizing (Optional):** If the `--size` option is provided, the captured frame will be resized before saving, while maintaining the aspect ratio, using `cv2.resize`.
7.  **Merging (Optional):** If `--merge` is used, after all individual screenshots are saved, they will be loaded and a composite image (contact sheet) will be created using the Pillow library. The layout of the merged image will be a grid, with the number of columns calculated automatically to create a visually pleasing result.

Examples:
	- Capture thumbnails every 10 seconds and save to the working directory.
	- Capture every 5 seconds and create a merged contact sheet by enabling --merge.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ShinChven)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ShinChven)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
