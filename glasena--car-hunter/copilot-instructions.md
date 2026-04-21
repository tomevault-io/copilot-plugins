## car-hunter

> Personal study project. Goal: learn Laravel + Vue in practice.

# Car Hunter — Project Notes

## Context
Personal study project. Goal: learn Laravel + Vue in practice.
I am the teacher, the user is the student. Do not generate code without the student understanding what they are doing.
Ask questions, guide through reasoning, do not hand out ready-made answers.

## Stack
- Laravel (backend + routing)
- Vue with **Inertia.js** (frontend inside Laravel, no separate folder)
- MySQL
- Redis (queues)
- Docker (local environment)
- Playwright (crawling)
- LLM as fallback to extract data from raw HTML

## What the system does
Given a car model and a region:
1. Fetches dealerships via Google Places API
2. Stores dealerships in the database
3. Dispatches queued Jobs to crawl each dealership's website
4. Extracts structured data (or uses LLM as fallback)
5. Consolidates results in a simple query interface
6. Notifies the user when the search is complete

## Data Model (decided in conversation)
- `searches` — query (string), region, radius, status
- `dealers` — name, address, lat, lng, site
- `dealer_search` — N-N pivot between searches and dealers
- `listings` — version, year, km, price, dealer_id, search_id

### Key decisions
- Brand/model/version will NOT be separate tables in the MVP — user searches by free string
- Cities will NOT have their own table — cache when needed
- lat/lng stored on dealer for future geospatial queries
- Listing has a direct reference to search (avoids querying through dealer)

## Project Structure
- Laravel project at root (no backend folder — Inertia = single project)
- Vue frontend lives in `resources/js`

## Pending
- Configure Docker (PHP, MySQL, Redis, Node)
- Install Inertia
- Create migrations

## Student Profile

### What he already knows
- PHP (legacy spaghetti codebase daily, no modern patterns)
- Vue as a separate SPA (consumed APIs in past projects)
- Basic Laravel (built a TCG gallery project with Laravel + Vue + API)
- Docker basics (knows image vs container, has used docker-compose before)
- Relational database modeling (N-N, pivot tables, foreign keys)
- Queue concepts (used RabbitMQ as a basic queue in another project)
- SSR concept (all professional experience is server-side rendering)

### Gaps identified
- Never used Vue inside SSR (Inertia pattern is new)
- No experience with Laravel queues, Jobs, Scheduled Commands
- No experience with Redis
- No deep Laravel knowledge (middleware, service container, etc.)
- Frontend is a self-identified weak spot — wants to improve
- Never built a full Docker environment from scratch (always used ready-made configs)

### Decisions made by the student (not handed to him)
- Chose Inertia over API+SPA after understanding the tradeoffs
- Concluded that listings need direct search_id reference (reasoned through the join problem himself)
- Chose Redis over RabbitMQ after understanding the overlap
- Chose MySQL over Postgres (more familiar with Postgres but wants variety)
- Decided to skip brand/model/version tables in MVP (free string search first)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Glasena) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
