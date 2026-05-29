## full-auto-forza-edition

> A Windows desktop automation tool for Forza Horizon 6. Built with Python + CustomTkinter. Uses a hybrid OpenCV + optional OCR detection pipeline to identify game screens and simulate keyboard/mouse input.

# Full Auto Forza Edition (FAFE) — Project Summary

## What is this?
A Windows desktop automation tool for Forza Horizon 6. Built with Python + CustomTkinter. Uses a hybrid OpenCV + optional OCR detection pipeline to identify game screens and simulate keyboard/mouse input.

## File Structure
```
New APP/
├── forza_app.py          # Entry point
├── main_window.py        # Main UI window (CTk), all tabs, settings panel
├── setup_panel.py        # Collapsible Setup & Templates panel with threshold sliders
├── capture.py            # Screenshot capture, region selection, CaptureSession, NodeSession
├── detector.py           # ScreenDetector — ROI + multi-scale + edge + optional OCR matching
├── race.py               # Race Auto-Grind automation logic
├── mastery.py            # Auto Unlock 22B Mastery automation logic
├── delete_cars.py        # Delete Used Cars automation logic
├── log_widget.py         # Thread-safe log widget with colored warning support
├── app_lang.py           # All UI strings in zh-tw / zh-cn / en
├── config.py             # Config load/save, path helpers
├── updater.py            # GitHub release auto-updater
├── version.py            # VERSION = "1.2.1"
├── build_app.bat         # PyInstaller build script
├── templates/            # Template images (race/, mastery_full/, each with 1080p/1440p/2160p/custom)
└── index.html            # GitHub Pages intro/tutorial page (trilingual)
```

## Architecture

### UI (main_window.py)
- 4 tabs: Race Auto / Auto Unlock 22B / Auto Buy 22B-STi / Delete Used Cars
- Settings is inline (not popup) — ⚙ hides topbar+tabs and shows settings panel, ← Back restores
- Topbar: app title, monitor picker, ⚙ button, ☕ Support Me button (bottom-right overlay)
- Each tab has: description label, optional count input (0=unlimited), Start/Stop buttons, log widget
- Auto Unlock 22B tab also has a start-row selector (CTkSegmentedButton, values 1/2/3) between count and run controls; saved as `mastery_start_loop` in config
- Language: zh-tw (default), zh-cn, en — switching triggers app restart
- Theme: system/light/dark

### Detection (detector.py)
- `ScreenDetector.detect(frame, key, template, threshold)` returns a `MatchResult` dataclass
- ROI-first: each template key has a configured search region (see `DEFAULT_ROIS`); full-screen fallback applies a 0.94 score penalty
- **Text template fast path**: `TEXT_TEMPLATES = frozenset(DEFAULT_ROIS.keys())` — all 11 keys. Text templates skip the Canny edge channel and use only 3 scales (`detector_text_scales`, default `[0.95, 1.00, 1.05]`), reducing matchTemplate calls from 14 to 3 (~4–5× faster). Non-text templates keep the full 7-scale + edge pipeline.
- Full pipeline scales: `[0.86, 0.92, 0.97, 1.00, 1.03, 1.08, 1.14]`
- Hybrid score (non-text): `0.62 × grayscale(equalizeHist) + 0.28 × Canny edges + up to 0.10 OCR bonus`
- Text score: `grayscale(equalizeHist)` only (edge channel skipped)
- **OCR-primary mode** (beta, default off — the real cross-hardware fix): For text templates with OCR hints, OCR becomes a first-class confirmation signal rather than a +0.10 bonus. Pixel template matching of game UI text is fragile across hardware (GPU AA, font hinting, HDR, game settings all change pixels) — text content is what's actually invariant. Logic: if `image_score >= detector_ocr_skip_above` (default 0.75), skip OCR for the fast path; else run OCR; if hint matched, promote score to `max(image_score, detector_ocr_confirm_score)` (default 0.85). To prevent the per-key OCR cooldown from breaking the stability filter, an OCR confirmation is **cached** per key for the cooldown duration and applied to subsequent frames without re-running OCR (gated by `detector_ocr_cache_pixel_min`, default 0.15, to guard against the screen changing during cooldown). Enable via `detector_ocr_primary: true`. Legacy default behaviour: OCR only in narrow borderline window as +0.10 bonus.
- **OCR pre-warming**: `OptionalOCR._ensure_loaded()` is kicked off in a background thread when `ScreenDetector` is constructed, so the first detection call doesn't eat the 1–2s onnxruntime model-load cost.
- **OCR cooldown**: `detector_ocr_cooldown` (default 1.0s) — minimum gap between actual OCR calls per template key. Prevents CPU spikes during long failed-detection streaks. Under OCR-primary, the confirmation cache uses this same duration.
- **Warning log surfaces OCR text**: when detection stalls for 3s, the warning includes `OCR: 'xxx'` showing what OCR read in the borderline zone — critical for diagnosing why a template isn't matching on a given user's machine.
- Soft threshold: actual cutoff is `max(0.45, threshold * 0.92)` — user's slider is a target, not a hard wall
- Stability filter: requires N consecutive frames (default 2) above the soft threshold; `stable=False` flag bypasses this for single-shot click ops
- Optional OCR: lazy-loads `rapidocr_onnxruntime` → `rapidocr` → `pytesseract`. Hint words in `OCR_HINTS` are multilingual (en/zh-tw/zh-cn). If no OCR backend is installed, detection still works
- `ScreenDetector.wait_for(frame_cb, key, template, threshold, stop_cb, interval, timeout, on_warn)` is the loop used by race.py / mastery.py; default `timeout=float('inf')`

### Automation modules (race.py, mastery.py, delete_cars.py)
- Each has a `run(cfg, stop_event, log_cb, status_cb, ...)` function
- Run in a daemon thread; `stop_event` signals shutdown
- race.py / mastery.py instantiate `ScreenDetector(cfg)` per run and call its `wait_for` / `detect`
- Per-template thresholds stored in config (keys like `thresh_start_menu`, `thresh_racing`, etc.)
- After 3s without detection: fires `warn_cb` with a red warning message that includes `best.source` and `best.score` for diagnostics
- wait_for() waits indefinitely (no timeout) until detected or stopped — **intentional design**, F9 is the recovery path

#### mastery.py — snake navigation
- `_nav_keys_for_loop(loop)` returns relative WASD keys for the infinite 3-column snake (col1 down, col2 up, col3 down, …)
- `start_loop = max(1, min(3, cfg.get("mastery_start_loop", 1)))` — which row the first car is on
- `loop_count` is initialised to `start_loop - 1`; a separate `car_num` counter starts at 0
- On the first car (`car_num == 1`) navigation is always skipped — user is already positioned there
- `max_cars` limit and log messages use `car_num` (1, 2, 3…), not `loop_count`

### Capture (capture.py)
- `CaptureSession`: listens for CAPS LOCK, opens fullscreen region selector, shows preview
- `NodeSession`: waits for CAPS LOCK, opens full screenshot for clicking 6 node positions
- ESC cancels at any stage — `_on_esc` also fires `on_cancel()` when not mid-capture (not just mid-capture)
- **Multi-capture fix**: `self._stop.set()` is called immediately before `self.callback()` in the success path, ensuring the old session's CAPS LOCK listener is torn down before the next session registers its own. This prevents duplicate listeners that previously broke the 2nd/3rd capture.
- Hooks are targeted (not unhook_all) to avoid killing the F9 toggle hotkey
- **Template rescaling**: `load_template` scales by height only (`scale = current_h / meta["screen_height"]`). Game UIs are laid out relative to vertical resolution; using an average of width+height was wrong for non-standard aspect ratios (e.g. 5K2K).

### Config (config.py)
- JSON file next to exe
- Keys include: lang, theme, toggle_key, capture_key, monitor_index, race_resolution, mastery_resolution, all thresh_* values, all *_wait values
- Mastery-specific timing keys: `mastery_check_interval` (min 0.3), `mastery_post_click_wait` (min 0.5), `mastery_post_key_wait` (min 0.8), `mastery_node_click_wait` (min 0.7)
- `mastery_start_loop` (int 1–3): which row the first car is on for Auto Unlock 22B
- Optional detector tuning keys: `detector_scales` (list of floats), `detector_text_scales` (list of floats), `detector_stable_frames` (int), `detector_enable_ocr` (bool), `detector_ocr_cooldown` (float seconds, default 1.0), `detector_ocr_primary` (bool, default false — beta opt-in for OCR as first-class match signal), `detector_ocr_skip_above` (float, default 0.75), `detector_ocr_confirm_score` (float, default 0.85), `detector_ocr_cache_pixel_min` (float, default 0.15 — minimum pixel score for cached OCR confirmation to apply)

### Log widget (log_widget.py)
- Thread-safe via queue
- `log(msg)` — normal text
- `log_warning(msg)` — red text (#ff4444) via tk.Text tag

### Auto-updater (updater.py)
- Checks GitHub API on startup (2s delay)
- Downloads full zip, extracts to temp, bat replaces FAFE.exe + _internal/ only (leaves config.json and templates/ untouched)
- Bat uses a retry loop on `copy /y` and gates `robocopy /PURGE` behind copy success — a locked FAFE.exe falls through to a `:fail` branch that relaunches the existing install instead of bricking it

## Key Design Decisions
- All UI strings go through `_at(key, lang, **kwargs)` in app_lang.py — never hardcode UI text
- Sliders save on mouse release only (not on drag)
- Slider minimums are set to the lowest safe values — going lower breaks automation because the game can't keep up
- Per-template threshold sliders in Setup panel (custom mode only shows retake buttons)
- Settings sliders clamp loaded values to slider range to handle stale config
- `keyboard.unhook_all()` is NEVER called — only targeted hook cleanup to preserve F9
- `wait_for` is intentionally infinite with no auto-retry — retrying detection without changing template/threshold yields the same result; user adjusts via sliders or recaptures
- All templates are text-based → `TEXT_TEMPLATES` covers all keys; never add a non-text template without also removing it from `TEXT_TEMPLATES` and verifying the edge channel improves accuracy

## GitHub
- Repo: https://github.com/Leoncrispybacon/Full-Auto-Forza-Edition
- Pages: https://leoncrispybacon.github.io/FAFE
- Latest release: v1.2.1

## Build
Run `build_app.bat` — produces `FAFE_dist/` with `FAFE.exe`, `_internal/`, `config.json`.
Release zip should contain the full `FAFE_dist/` contents.
The build bundles `rapidocr_onnxruntime` so OCR hints work out of the box; the ONNX models add ~50 MB to the dist.

---
> Source: [Leoncrispybacon/Full-Auto-Forza-Edition](https://github.com/Leoncrispybacon/Full-Auto-Forza-Edition) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
