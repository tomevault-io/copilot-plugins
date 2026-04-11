## zephyr

> We are building a custom, self-hosted music scrobbling and charting service (a Maloja alternative) with a heavy focus on multi-user support, seamless single sign-on (SSO), and rich mobile charting.

# PROJECT CONTEXT: Custom Maloja Alternative & Navidrome Fork

## 1. Project Overview
We are building a custom, self-hosted music scrobbling and charting service (a Maloja alternative) with a heavy focus on multi-user support, seamless single sign-on (SSO), and rich mobile charting. 
To achieve seamless integration without requiring users to generate API keys, we are forking Navidrome to modify its native ListenBrainz scrobbling payload.

## 2. Tech Stack Architecture
* **Media Server:** Forked Navidrome (Go).
* **Custom Backend:** Go (Fiber framework) + TimescaleDB (for heavy time-series data).
* **Custom Frontend:** React Native (Expo) or Flutter (using `fl_chart`).
* **Authentication Mechanism:** Pass-through Subsonic API verification via JWT.
* **Image Serving Strategy:** No external API calls. The frontend will fetch images directly from Navidrome via the Subsonic `getCoverArt` endpoint.

## 3. Navidrome Source Modifications (COMPLETED)
The Go source code of Navidrome has been modified to inject custom internal data into the JSON payload: `navidrome_username`, `navidrome_uuid`, `cover_art_id`, and `duration_ms`.

## 4. Backend Implementation (In Progress)
* **Auth:** Subsonic pass-through authentication. Successful login returns a JWT.
* **Database:** TimescaleDB with a `scrobbles` hypertable.
* **API Endpoints (Protected via JWT):**
    * `POST /login`: Authenticates via Navidrome, returns JWT.
    * `POST /submit-listens`: Receives scrobbles from Navidrome (Public).
    * `GET /charts/top-artists`, `top-albums`, `top-tracks`.
    * `GET /charts/recent`: Most recent listens.
    * `GET /charts/timeline`: Play counts over time (using `time_bucket`).
    * `GET /charts/summary`: User stats overview.
    * `GET /charts/stats/hour`: Play distribution by hour of day (0-23).
    * `GET /charts/stats/day`: Play distribution by day of week (0-6).
    * `GET /charts/listens`: Raw scrobble history.

## Implementation Notes
- **User Isolation:** All `/charts` endpoints enforce data isolation using the `username` stored in the JWT claims.
- **TimescaleDB:** Leverages `time_bucket`, `EXTRACT(HOUR FROM ...)` and `EXTRACT(DOW FROM ...)` for efficient aggregations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nizarberyan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Nizarberyan)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
