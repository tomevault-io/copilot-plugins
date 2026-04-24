## maa-tribute

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project: Maa's Tribute Video

A tribute video for the book publication ceremony of "Sudarshana Singhal: In Quest of India's Cultural Paradigms" honoring Dr. Sudarshana Devi Singhal (1936-1998), created by her younger daughter-in-law Urmila Singhal.

## File Structure

```
originals/          Source video recordings (3 quality levels)
  urmila-speech-full.mp4          2.3GB - full quality, use for final render
  urmila-speech-compressed.mp4    171MB - compressed
  urmila-speech-small.mp4         27MB, 1280x720, 5:38 - use for testing
photos/             12 family photos (wedding, family, scanned)
final/              Output tribute videos
  maa-tribute-small.mp4           Built from small source
  maa-tribute-compressed.mp4      TBD
  maa-tribute-full.mp4            TBD - ceremony version
docs/
  Urmila_Speech_for_Maa.docx      Final speech text
  book.pdf                        The book (8 pages, contains extractable photos)
```

## Current Video Format (v2 - Slideshow + PIP)

- **0:00-0:15**: Full-screen Urmila (intro/namaste)
- **0:15 onward**: 12 photos cycle full-screen as continuous slideshow, Urmila as small PIP (250x140) in bottom-right corner
- Photos shown on dark blurred background, 8 seconds each, 1-second crossfade between them
- Slideshow loops (~85 seconds per cycle, ~3.8 cycles to fill video)

## Photo Slideshow Order (emotionally sequenced)

| # | Source file | Description |
|---|-----------|-------------|
| 1 | book.pdf page 4 (extracted) | Young Maa, Mongolia 1957, scholar |
| 2 | `55-MaaTaji.jpg` | Maa with Nikhil's father, Banaras |
| 3 | `99-BanarasHome.JPG` | The Banaras family home |
| 4 | `PNG image 4.png` | Wedding procession (overhead) |
| 5 | `PNG image 5.png` | Maa blessing Urmila, purple saree |
| 6 | `PNG image.png` | Wedding stage |
| 7 | `PNG image 1.png` | Full family wedding photo |
| 8 | `PNG image 3.png` | Maa close-up with two women, green saree |
| 9 | `PNG image 6.png` | Maa smiling joyfully, purple saree, fan |
| 10 | `PNG image 2.png` | Wedding group, Maa center |
| 11 | `PNG image 7.png` | Urmila and Maa eating together |
| 12 | book.pdf page 6 (extracted) | Maa in garden, red saree among flowers |

## How to Build

### Prerequisites
```bash
brew install ffmpeg poppler
pip3 install PyPDF2 Pillow
```

### Step 1: Extract photos from book PDF
```bash
pdftoppm -png -r 200 docs/book.pdf /tmp/book_images/bookpage
# Then crop Mongolia (page 4) and garden (page 6) using PIL - see build notes below
```

### Step 2: Prepare slideshow frames
Python script prepares all 12 photos as 1280x720 frames with dark blurred background. Output to `/tmp/slideshow_prep/`. Scanned photos get bottom 6% cropped to remove PhotoScan watermark.

### Step 3: Create slideshow cycle
```bash
# 12 images, 8s each, 1s crossfade = 85s cycle
ffmpeg -y \
  -loop 1 -t 8 -i /tmp/slideshow_prep/01_mongolia.png \
  -loop 1 -t 8 -i /tmp/slideshow_prep/02_maa_father.png \
  ... (all 12) \
  -filter_complex "[0][1]xfade=transition=fade:duration=1:offset=7[x1]; ..." \
  /tmp/slideshow_one_cycle.mp4
```

### Step 4: Composite final video
```bash
INPUT="originals/urmila-speech-small.mp4"  # or compressed/full
OUTPUT="final/maa-tribute-small.mp4"

ffmpeg -y \
  -i "$INPUT" \
  -stream_loop 3 -i /tmp/slideshow_one_cycle.mp4 \
  -filter_complex "
    [0]scale=250:140[pip];
    [1]setpts=PTS+15/TB,scale=1280:720[slides];
    [0][slides]overlay=0:0:enable='gte(t,15)':eof_action=pass[bg];
    [bg][pip]overlay=x=1010:y=565:enable='gte(t,15)'[out]
  " \
  -map "[out]" -map 0:a -c:v libx264 -preset medium -crf 22 -c:a copy \
  -t 339 "$OUTPUT"
```

## Technical Notes

- ffmpeg 8.0.1, poppler, PyPDF2, Pillow installed via homebrew/pip
- `-loop 1` with fade filters does NOT work for overlays; use `enable='between(t,...)'` or xfade
- xfade offsets for 12 images: 7, 14, 21, 28, 35, 42, 49, 56, 63, 70, 77
- Small video test renders in ~1 minute; full 2.3GB will take longer
- Slideshow prep images cached at `/tmp/slideshow_prep/` (need to regenerate after reboot)
- Book PDF images cached at `/tmp/book_images/` (need to regenerate after reboot)

## TODO
- [ ] Urmila reviews test video (`final/maa-tribute-small.mp4`) for timing and photo order
- [ ] Confirm 15-second intro cutover timing
- [ ] Confirm PIP size (250x140) is appropriate
- [ ] Build compressed and full-quality final versions
- [ ] Consider: Urmila may re-record without headphones (per online chat feedback)

## Urmila's Preferences
- Dislikes em dashes (feels AI-like)
- Prefers natural, conversational tone
- Sharp editorial eye, will correct details
- Prefers interview-style collaboration for creative work


- Before beginning implementation, check with your project lead for any cross-session context or coordination notes that may affect this work.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nikhilsi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
