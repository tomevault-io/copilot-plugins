## remote-sensors

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Remote PC sensor collection system that monitors CPU/GPU temperatures and memory usage from multiple machines and displays them on a centralized web dashboard with interactive charts.

**Architecture**: Two-part system
- **Agent** (client): Runs on each monitored PC, collects sensor data every 60 seconds, sends to server via HTTP
- **Server**: FastAPI application that receives sensor data and displays it in a real-time web dashboard

**Technology Stack**: Python 3.x, FastAPI, psutil, pynvml (NVIDIA GPU support), httpx, Chart.js

## Common Commands

### Server
```bash
# Install dependencies
cd server
pip install -r requirements.txt

# Run server (development)
uvicorn main:app --reload

# Run server (production)
uvicorn main:app --host 0.0.0.0 --port 8000

# Access dashboard
# http://localhost:8000
```

### Agent
```bash
# Install dependencies
cd agent
pip install -r requirements.txt

# Run agent
python main.py

# Configure server endpoint
# Edit agent/config.py and change SERVER_URL
```

## Architecture Details

### Agent Structure
- **[main.py](agent/main.py)**: Main event loop, collects and sends data with retry logic
- **[sensors.py](agent/sensors.py)**: Platform-agnostic sensor reading:
  - CPU temperature via psutil (coretemp, cpu_thermal, k10temp)
  - GPU temperature via NVML (NVIDIA only)
  - Memory usage via psutil (total, used, available, percent)
- **[config.py](agent/config.py)**: Configuration (server URL, collection interval, retry settings)

The agent handles multiple sensor platforms and gracefully degrades when sensors are unavailable.

### Server Structure
- **[main.py](server/main.py)**: FastAPI application with three main endpoints:
  - `POST /ingest`: Receives sensor data from agents
  - `GET /`: Renders HTML dashboard
  - `GET /api/current`: JSON API for current readings
  - `GET /api/history/{hostname}`: JSON API for historical data
- **[database.py](server/database.py)**: In-memory data store with current + historical readings (max 100 per machine)
- **[templates/dashboard.html](server/templates/dashboard.html)**: Real-time dashboard with Chart.js visualizations
- **[static/charts.js](server/static/charts.js)**: Chart.js implementation for historical temperature graphs

### Data Flow
1. Agent reads CPU/GPU temps and memory usage using platform-specific APIs
2. Data sent as JSON: `{hostname, os, timestamp, cpu_temp, gpu_temp, memory_usage: {total, used, available, percent}}`
3. Server stores in memory (current + historical readings)
4. Dashboard displays real-time values and interactive time-series charts
5. Charts auto-update every 5 seconds via `/api/history/{hostname}` endpoint

### Visualization Features
- **Real-time cards**: Display current CPU temp, GPU temp, and memory usage percentage
- **Time-series charts**: Interactive line graphs using Chart.js with dual Y-axes:
  - Left axis: CPU and GPU temperatures (°C)
  - Right axis: Memory usage (%)
- **Time range selection**: View last 10min, 30min, or 1 hour of data
- **Auto-refresh**: Charts update every 5 seconds without page reload
- **Responsive design**: Charts adapt to screen size
- **Tooltips**: Hover over data points for precise readings

## Key Design Decisions

**No Security**: This is a proof-of-concept with no TLS, authentication, or API keys for rapid prototyping.

**In-Memory Storage**: Data stored in Python dict/list structures (ephemeral). To persist data, replace `database.py` with SQLite implementation.

**Graceful Degradation**: Sensors return `None` when unavailable rather than crashing. Dashboard displays "N/A" for missing readings.

**Retry Logic**: Agent retries failed sends up to 3 times with 5-second delays.

## Testing

When testing sensor reading on a development machine:
- CPU temp may not be available in VMs or containers
- GPU temp requires NVIDIA GPU + drivers + pynvml
- Use mock data if sensors unavailable for testing

## Future Enhancements

See [project_details.md](project_details.md) for potential additions:
- SQLite persistence for data across server restarts
- Additional metrics (disk, memory, network)
- TLS and authentication
- Export historical data to CSV
- Alert thresholds for high temperatures

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/huseyintemiz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
