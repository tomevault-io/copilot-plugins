## verdoy

> Expert in: Python, FastAPI, PostgreSQL, SQLAlchemy, HTML, CSS, vanilla JavaScript, scientific data management.

# LIMS (Laboratory Information Management System) .cursorrules

Expert in: Python, FastAPI, PostgreSQL, SQLAlchemy, HTML, CSS, vanilla JavaScript, scientific data management.

## System Overview

Laboratory Information Management System for research labs, R&D facilities, and fermentation monitoring. Core features: experiment tracking, sensor data collection, equipment integration, knowledge graph relationships.

## Architecture Principles

1. **Frontend**: Server-rendered HTML, progressive enhancement, no JS frameworks
2. **Backend**: FastAPI + PostgreSQL + TimeseriesDB, async Python, REST APIs
3. **Data**: Time-series sensor readings, experiment relationships, audit trails
4. **Performance**: HTML-first, edge caching, async operations


### High-Level Architecture Concept
Clients, 
1. IoT Devices (ESP32)
2. Web Frontend (HTML/HTML5 & CSS first, as little JS as needed)
3. Mobile Apps (future)

Interact with the backend,
- LMS Core API (FastAPI Backend)

Which manages data in,
- PostgreSQL (Tabular and Graph-relational data) + TimescaleDB (Time-series Data)


### Key Components

- **IoT Devices**: ESP32 microcontrollers with sensors
- **Web Frontend**: Next.js dashboard for data visualization
- **API Backend**: FastAPI application with REST and WebSocket endpoints
- **Database**: PostgreSQL with TimescaleDB extension for time-series data
- **Real-time Communication**: WebSocket connections for live data streaming

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lutze) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
