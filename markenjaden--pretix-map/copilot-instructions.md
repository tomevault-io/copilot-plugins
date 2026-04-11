## pretix-map

> `pretix-map` is a high-performance plugin for the [pretix](https://github.com/pretix/pretix) ticketing system. It provides advanced geographic visualizations and sales analytics to help organizers understand their audience distribution and marketing effectiveness.

# GEMINI.md - pretix-map

## Project Overview
`pretix-map` is a high-performance plugin for the [pretix](https://github.com/pretix/pretix) ticketing system. It provides advanced geographic visualizations and sales analytics to help organizers understand their audience distribution and marketing effectiveness.

### Core Features
- **Automatic Geocoding:** Automatically converts order invoice addresses to coordinates. Supports multiple fallback strategies (Zip+City, Zip+Country) to ensure high mapping rates.
- **Nightly Reprocessing:** Background service (at 3 AM) that retries failed or missing geocodes automatically.
- **Interactive Visualization:**
  - **Pin View:** Clustered markers with detailed tooltips (Order, Date, Items, Revenue).
  - **Heatmap View:** Visualizes density, with optional **Revenue Weighting**.
  - **Density Grid:** Grid-based heatmap (Choropleth alternative) for hot zone analysis.
- **Timeline & Marketing Milestones:**
  - Time-lapse slider to visualize sales growth.
  - Integration of marketing milestones (e.g., Newsletters) from `pretix.cfg` onto the timeline.
- **Advanced Analytics:**
  - Statistics panel: Top Cities, Top Ticket Types, Total Revenue, and **Average Travel Distance**.
- **Management Tools:**
  - **List View:** Identify and manage orders with missing/failed geodata.
  - **Event Context:** Distinct Red Marker for the event location.
  - **Event Comparison:** Overlay distribution data from other events for YOY analysis.
- **Historical Processing:** CLI command `geocode_existing_orders` for processing past data.

### Technical Stack
- **Backend:** Python, Django, pretix, Celery (background tasks), Geopy (Nominatim).
- **Frontend:** Leaflet.js (Heat, MarkerCluster), Vanilla JS.

---

## Building and Running

### Development Setup
1.  **Prerequisites:** Working [pretix development setup](https://docs.pretix.eu/en/latest/development/setup.html) and Celery worker.
2.  **Installation:**
    ```bash
    python setup.py develop
    ```
3.  **Configuration (`pretix.cfg`):**
    ```ini
    [pretix_mapplugin]
    nominatim_user_agent = YourAppName/1.0 (contact@domain.com)
    milestones = 2025-01-01:Newsletter, 2025-02-15:Early Bird End
    ```
4.  **Run:** Restart pretix server and Celery worker.

### Key Commands
- **Reprocess Data:** `python manage.py geocode_existing_orders`
- **Check Syntax:** `python -m py_compile pretix_mapplugin/*.py`

---

## Development Conventions
- **Formatting:** `black`, `isort`.
- **Logic:** Data isolation via `django-scopes`.
- **Security:** Dynamic CSP headers to allow OSM tiles and inline styles.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MarkenJaden)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MarkenJaden)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
