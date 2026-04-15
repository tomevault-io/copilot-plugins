## north-thai-monitor

> Thailand Intelligence Monitor is a real-time situational awareness dashboard and news aggregation system inspired by World Monitor. It acts as an intelligence center covering all regions of Thailand (National, North, Northeast, Central, East, West, South).

# Thailand Intelligence Monitor

## Project Overview

Thailand Intelligence Monitor is a real-time situational awareness dashboard and news aggregation system inspired by World Monitor. It acts as an intelligence center covering all regions of Thailand (National, North, Northeast, Central, East, West, South).

The system continuously crawls data from various sources (RSS feeds, HTML websites, public Facebook pages, and YouTube), categorizes the intel based on keywords (e.g., Disaster, Economy, Security), assesses the severity (Critical, High, Medium, Low), and plots the information on an interactive map. It features a dark-themed, Heads-Up Display (HUD) style UI with multilayer controls for News, Social Pulse, and Live Cameras.

### Architecture
- **Backend:** Python with FastAPI. It handles REST API endpoints, database interactions via SQLAlchemy (SQLite), and runs asynchronous background crawlers (`aiohttp`, `feedparser`, `beautifulsoup4`).
- **Frontend:** Vanilla HTML, CSS, and JavaScript. It uses Leaflet.js (with CartoDB Dark Matter tiles) for the map, creating a high-tech intelligence dashboard.
- **Database:** SQLite (`north_thai_monitor.db`) storing `NewsItem`, `SocialMediaPost`, and `Source` configurations.

## Building and Running

### Prerequisites
- Python 3.9+

### Setup and Execution
1. **Install Dependencies:**
   Navigate to the `backend` directory and install the required Python packages:
   ```bash
   cd backend
   pip install -r requirements.txt
   ```

2. **Run the Server:**
   You can start the FastAPI server using the provided batch scripts from the root directory:
   ```cmd
   run.bat
   # OR
   start-server.bat
   ```
   Alternatively, run it manually from the `backend` directory:
   ```bash
   cd backend
   python run.py
   # OR
   uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
   ```

3. **Access the Dashboard:**
   Open your web browser and navigate to:
   ```text
   http://localhost:8000
   ```
   *Note: The root URL automatically redirects to `/static/index.html`.*

## Development Conventions & Key Components

- **Database-Driven Sources:** The crawler dynamically fetches its target URLs from the SQLite database (`sources` table) rather than static JSON files. This allows users to add/remove/toggle sources directly from the UI Settings panel.
- **Smart Crawler (`backend/app/services/rss_crawler.py`):** If a provided URL is not a standard RSS feed, the crawler utilizes a BeautifulSoup fallback to intelligently scrape article links and titles from regular HTML pages.
- **Social Pulse Fallback:** If the Facebook scraper fails (due to API restrictions or structural changes), the system uses an "Intelligence Fallback" that pipes recent relevant news items into the Social Pulse feed to ensure the dashboard remains active.
- **HUD UI (`frontend/`):** The UI is designed to look like a command center. It uses absolute positioning over a full-screen Leaflet map. State management is handled globally via the `appState` object in `frontend/js/app.js`, polling the backend APIs every 20-30 seconds.
- **Settings Management:** Users can manage data sources and trigger immediate system crawls via the gear icon in the UI.

### Key API Endpoints
- `GET /api/v1/news/latest`: Fetches the latest categorized news items.
- `GET /api/v1/social/feed`: Retrieves the latest social media pulse data.
- `GET /api/v1/settings/webcams`: Returns configured live camera locations and iframe URLs.
- `POST /api/v1/settings/sources/add`: Adds a new intelligence source to the database.
- `POST /api/v1/crawl/trigger`: Manually forces the background crawler to run immediately.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kongsak4807017)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kongsak4807017)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
