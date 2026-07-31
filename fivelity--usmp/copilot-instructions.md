## 04-data-management-communication

> WebSocket protocol details, API interaction patterns, and data formats.

# 04-data-management-communication

## Data Management & Communication

STRICT REQUIREMENT:
- WebSocket Protocol: Backend must send real-time sensor data updates over WebSockets, and frontend must maintain connections.
- API Interactions: Standardize endpoints for configuration retrieval and sensor discovery.
- Data Persistence: If necessary, use Firestore (local storage). Firestore paths must follow:
  - Public data: `/artifacts/{appId}/public/data/{collection}`
  - Private data: `/artifacts/{appId}/users/{userId}/{collection}`
- Firebase Authentication: Required before Firestore operations.
- Data Formats: Use JSON for API responses and WebSocket messages.
- API Endpoints: Use RESTful endpoints for configuration retrieval and sensor discovery.

---
> Source: [fivelity/usmp](https://github.com/fivelity/usmp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
