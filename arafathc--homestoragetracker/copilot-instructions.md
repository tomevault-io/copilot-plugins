## homestoragetracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a mobile-responsive Flask web application for QR code management and storage with hierarchical organization. Users can generate QR codes, scan them with camera/screenshots, upload existing codes, and organize them by categories. Storage boxes can contain tools, creating a parent-child relationship system. The app includes real-time camera scanning, public shareable URLs, and database-backed storage with SQLAlchemy.

## Core Architecture

**Web Application (`app.py`)**:
- Flask web server with Bootstrap 5 responsive UI
- `QRCodeStorageWeb` class handles all QR operations with categorization
- Mobile-first design with camera integration
- RESTful routes for CRUD operations

**Key Components**:
- **QR Generation**: Creates QR codes with metadata and category association
- **Camera Scanning**: Real-time QR scanning using HTML5-QRCode library
- **File Upload**: Process uploaded QR code images
- **Category Management**: Organize items by Storage Boxes, Tools, Home Items
- **Search/Filter**: Text search across all QR metadata fields

## Key Dependencies

- `flask`, `werkzeug`: Web framework and utilities
- `qrcode[pil]`: QR code generation
- `opencv-python-headless`: QR code detection (production-optimized)
- `gunicorn`: Production WSGI server
- `bootstrap@5.1.3`: Responsive UI framework
- `html5-qrcode@2.3.8`: JavaScript camera scanning

## Common Commands

### Development Setup
```bash
pip install -r requirements.txt
```

### Run Web Application (Development)
```bash
python app.py
```
Access at http://localhost:5000

### Run Web Application (Production)
```bash
gunicorn --bind 0.0.0.0:8080 --workers 1 --threads 8 app:app
```

### Fix Database (if needed)
```bash
python fix_db.py
```

### Deploy to Google Cloud
```bash
./deploy.sh
```
Interactive deployment with database options (SQLite or Cloud SQL PostgreSQL)

### Legacy CLI Testing
```bash
python test_qr.py  # Test core functionality
python main.py     # Interactive CLI (requires terminal input)
```

## Data Model

Enhanced JSON structure in `qr_data.json`:
```json
{
  "qr_codes": [...],
  "storage_boxes": [...],
  "tools": [...], 
  "home_items": [...]
}
```

Each QR record contains:
```json
{
  "id": "uuid",
  "data": "QR code content",
  "filename": "image_file.png",
  "item_type": "storage_box|tool|home_item",
  "item_name": "User-friendly name",
  "location": "Physical location",
  "description": "Additional details",
  "created_at": "ISO timestamp"
}
```

## Mobile Features

- **Responsive Design**: Bootstrap grid adapts to all screen sizes
- **Camera Scanning**: Native camera access for real-time QR detection
- **Screenshot Upload**: Process QR codes from uploaded images
- **Touch UI**: Large buttons and touch-friendly interactions
- **Floating Scan Button**: Quick-access mobile scanning (hidden on desktop)

## Deployment Architecture

**Containerization**: 
- Dockerfile with OpenCV system dependencies
- Production-ready with gunicorn WSGI server
- Cloud Run compatible (serverless auto-scaling)

**Google Cloud Integration**:
- Cloud Build for CI/CD pipeline
- Container Registry for image storage
- Cloud Run for serverless hosting
- Automatic HTTPS and custom domain support

## File Structure

```
├── app.py              # Main Flask application
├── main.py             # Legacy CLI interface
├── templates/          # Jinja2 HTML templates
│   ├── base.html       # Base template with mobile scanner
│   ├── index.html      # Homepage with category cards
│   ├── generate.html   # QR code generation form
│   ├── upload.html     # File upload interface
│   ├── browse.html     # QR code listing/filtering
│   ├── view_qr.html    # Individual QR code details
│   └── search.html     # Search results display
├── uploads/            # QR code image storage
├── Dockerfile          # Container configuration
├── cloudbuild.yaml     # Google Cloud Build pipeline
└── deploy.sh           # Automated deployment script
```

## Security & Production Notes

- App runs with unauthenticated access by default
- HTTPS required for camera access (automatically provided by Cloud Run)
- File uploads limited to 16MB with extension validation
- OpenCV headless version used for server environments
- Consider adding authentication for production deployments

---
> Source: [ArafathC/HomeStorageTracker](https://github.com/ArafathC/HomeStorageTracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-04 -->
