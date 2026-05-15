## inkframe

> E-ink photo frame with web-based photo upload. Users drag-and-drop photos from their phone or computer browser. No Google API or cloud sync needed.

# InkFrame - E-Ink Photo Frame

## Overview

E-ink photo frame with web-based photo upload. Users drag-and-drop photos from their phone or computer browser. No Google API or cloud sync needed.

## Architecture

- Flask web app serving on port 80
- Drag-drop upload via browser (mobile-first)
- Three-tier image storage: originals, display-ready 600x448, thumbnails 300x200
- Inky Impression 5.7" 7-color e-ink display with saturation control
- MockDisplay for headless dev (saves to data/mock_display.png)
- APScheduler for slideshow cycling
- SQLite for photo metadata, JSON for settings

## Project Structure

```
app.py              # Flask routes, GPIO buttons, startup
display.py          # E-ink + MockDisplay, info screens, debouncing
image_processor.py  # Upload processing, resize, thumbnails, EXIF
models.py           # SQLite DB + JSON settings
scheduler.py        # Photo cycling with APScheduler
wifi_manager.py     # WiFi AP/client mode (NetworkManager)
templates/          # Jinja2 templates
  base.html, index.html, settings.html, setup_wifi.html
static/
  css/style.css
  js/upload.js, gallery.js
data/               # Gitignored runtime data
  originals/        # Original uploads
  display/          # Pre-rendered 600x448 PNG
  thumbnails/       # 300x200 JPEG for web gallery
config/             # Gitignored settings + DB
```

## Key Commands

```bash
# Dev (on homelab, no display hardware)
source venv/bin/activate
python3 -c "from app import app; app.run(host='0.0.0.0', port=8080)"

# Service management (on target Pi)
sudo systemctl start|stop|restart inkframe
sudo journalctl -u inkframe -f
```

## API Endpoints

- `POST /api/photos/upload` - Upload photo (multipart)
- `GET /api/photos` - List photos (paginated)
- `DELETE /api/photos/<id>` - Delete photo
- `POST /api/photos/delete-bulk` - Bulk delete
- `POST /api/display/next|prev|info` - Display control
- `POST /api/display/show/<id>` - Show specific photo
- `POST /api/slideshow/start|stop` - Slideshow control
- `GET|POST /api/settings` - Settings CRUD
- `GET /api/status` - System status

## Dependencies

- Prefer system apt packages (`python3-opencv`, etc.) over pip for heavy native libraries — the target Pi is armv7l and compiling from source (e.g. opencv-python-headless) takes too long or OOM-kills
- Use `install.sh` to symlink system packages into the venv when needed
- Pure-Python packages are fine via pip/requirements.txt

## GPIO Buttons (BCM)

| Pin | Function |
|-----|----------|
| 5   | Show info screen |
| 6   | Previous photo |
| 16  | Next photo |
| 24  | Setup mode / hold: reboot |

---
> Source: [bigguy6883/inkframe](https://github.com/bigguy6883/inkframe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
