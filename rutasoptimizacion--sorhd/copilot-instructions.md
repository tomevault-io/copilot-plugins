## sorhd

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**FlamenGO!** (Sistema de Optimización de Rutas para Hospitalización Domiciliaria) is a route optimization platform for home hospitalization services. The system helps clinical teams optimize daily visit routes considering personnel skills, vehicle capacity, time windows, and geographic constraints.

**Current Status**: This repository contains **planning and documentation only**. No implementation code exists yet. All development work should follow the specifications in this repository.

## Documentation Structure

### Core Documents

1. **specs/requirements.md** - Functional and non-functional requirements in EARS format
   - Read this FIRST to understand what the system must do
   - Contains requirements traceability matrix
   - Defines all user roles, entities, and constraints

2. **specs/design.md** - Comprehensive system design document
   - System architecture (3-tier: Backend, Web Admin, Mobile App)
   - Data model and database schema (PostgreSQL + PostGIS)
   - API specifications (RESTful endpoints)
   - Technology stack for each component
   - Security design (JWT authentication, RBAC)
   - Deployment architecture
   - Optimization algorithm design (OR-Tools VRP solver)

3. **CHECKLIST.md** - Detailed implementation checklist with 15 phases
   - Phase 0-1: Project setup and backend foundation
   - Phase 2-6: Backend modules (resources, optimization, tracking, notifications)
   - Phase 7-10: Admin panel (React.js)
   - Phase 11-13: Mobile app (React Native)
   - Phase 14-15: Integration, testing, and deployment
   - Each phase includes tasks, acceptance criteria, and dependencies

4. **documents/requerimientos_funcionales.md** - Original functional requirements (Spanish)

## System Architecture (High-Level)

### Three Main Components

1. **Backend** (Python + FastAPI)
   - PostgreSQL + PostGIS for geospatial data
   - OR-Tools for route optimization (VRP with time windows)
   - WebSocket for real-time GPS tracking
   - Push notifications (FCM/APNS) and SMS fallback

2. **Admin Panel** (React.js + TypeScript)
   - Resource management (personnel, vehicles, patients, cases)
   - Route planning interface with map visualization
   - Live monitoring dashboard (real-time vehicle tracking)

3. **Mobile App** (React Native)
   - Clinical Team profile: view routes, update visit status, GPS tracking
   - Patient profile: view visit status, track approaching team (Uber-style)

## Key Technical Decisions

### Backend
- **Framework**: FastAPI (async, type hints, auto-generated OpenAPI docs)
- **ORM**: SQLAlchemy with GeoAlchemy2 for PostGIS integration
- **Optimization**: Google OR-Tools (primary), custom heuristic (fallback)
- **Authentication**: JWT tokens with role-based access control (Admin, Clinical Team, Patient)
- **Real-time**: WebSocket for GPS location updates (30-second intervals)

### Database
- **PostgreSQL 15+** with **PostGIS** extension for geospatial queries
- Location stored as `GEOGRAPHY(POINT, 4326)` (WGS 84)
- Audit logging for all mutations
- Distance matrix caching (optional, 24-hour TTL)

### Admin Panel
- React 18 + TypeScript + Vite
- State: Redux Toolkit (global) + React Query (server state)
- UI: Material-UI
- Maps: Leaflet or Google Maps
- Real-time updates via WebSocket

### Mobile App
- React Native (iOS + Android)
- Redux Toolkit + AsyncStorage
- Background GPS tracking with minimal battery impact
- Offline support with sync queue
- Push notifications via Firebase (Android) and APNS (iOS)

## Route Optimization Algorithm

The core optimization problem is a **Vehicle Routing Problem with Time Windows and Skills (VRP-TWS)**.

**Constraints**:
- Personnel skill requirements must match case needs
- Vehicle capacity limits
- Time window constraints (e.g., "AM only", "10:00-12:00")
- Working hours (default 8:00-17:00)

**Objectives**:
1. Minimize total travel distance
2. Minimize total travel time
3. Balance workload across vehicles
4. Maximize on-time arrivals

**Implementation**:
- Primary: OR-Tools VRP solver (powerful, handles complex constraints)
- Fallback: Nearest neighbor + 2-opt local search (faster, approximate)

## Database Schema Highlights

**Key Tables**:
- `users` - Authentication (username, password_hash, role)
- `personnel` - Clinical staff (name, skills, start_location, work_hours)
- `vehicles` - Vehicles (identifier, capacity, base_location, resources)
- `patients` - Patient records (name, location)
- `cases` - Visit requests (patient_id, care_type, location, time_window, priority)
- `routes` - Optimized routes (vehicle_id, date, status, total_distance)
- `visits` - Individual visits in routes (route_id, case_id, sequence, times, status)
- `location_logs` - GPS tracking (vehicle_id, location, timestamp)
- `notifications` - Push/SMS notifications (user_id, type, message, delivery_status)

**Relationships**:
- Many-to-many: Personnel ↔ Skills, Routes ↔ Personnel
- One-to-many: Route → Visits, Vehicle → Routes, Patient → Cases

## API Structure

All endpoints under `/api/v1/`:

- `/auth/*` - Login, logout, token refresh
- `/personnel/*` - Personnel CRUD
- `/vehicles/*` - Vehicle CRUD
- `/patients/*` - Patient CRUD
- `/cases/*` - Case CRUD
- `/routes/*` - Route optimization and management
  - `POST /routes/optimize` - Trigger route optimization
  - `GET /routes/active` - Get active routes
- `/tracking/*` - GPS tracking
  - `POST /tracking/location` - Upload GPS location
  - `WS /tracking/live` - WebSocket for real-time updates
- `/notifications/*` - Notification management

## Development Workflow (When Implementation Starts)

### Initial Setup
```bash
# Backend
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
alembic upgrade head  # Run migrations
uvicorn app.main:app --reload

# Admin Panel
cd admin
npm install
npm run dev

# Mobile App
cd mobile
npm install
npx react-native run-ios  # or run-android
```

### Docker Development Environment
```bash
docker-compose up
# Backend: http://localhost:8000
# Admin Panel: http://localhost:5173
# PostgreSQL: localhost:5432
```

### Testing
```bash
# Backend
pytest tests/ --cov=app --cov-report=term-missing

# Admin Panel
npm run test

# Mobile App
npm test
```

## Implementation Phases

Follow the phases in **CHECKLIST.md** sequentially:

1. **Phase 0-1** (5-7 days): Setup + Backend foundation
   - Database schema, authentication, base API
   
2. **Phase 2-6** (21-28 days): Backend modules
   - CRUD operations, optimization engine, GPS tracking, notifications

3. **Phase 7-10** (14-18 days): Admin Panel
   - Resource management, route planning, live monitoring

4. **Phase 11-13** (11-14 days): Mobile App
   - Clinical team features, patient tracking

5. **Phase 14-15** (9-12 days): Testing & Deployment
   - E2E testing, performance testing, production deployment

**Total Estimated Duration**: 60-79 days (~2.5-3.5 months)

## Security Considerations

- **Authentication**: JWT with short-lived access tokens (1 hour) and refresh tokens (7 days)
- **Password Hashing**: bcrypt with cost factor 12
- **Authorization**: Role-based access control (Admin, Clinical Team, Patient)
- **Data Protection**: TLS 1.3 for all communication, database encryption at rest
- **API Security**: Rate limiting, input validation (Pydantic), SQL injection protection (ORM)
- **Mobile Security**: Certificate pinning, secure storage (Keychain/Keystore)
- **Audit Logging**: All mutations logged with user ID, timestamp, changes (JSONB)

## Performance Requirements

- Route optimization (50 cases): < 60 seconds
- API response time (95th percentile): < 500ms
- Map load time (50 vehicles): < 2 seconds
- WebSocket latency: < 1 second
- Database queries: < 100ms (95th percentile)
- Mobile app cold start: < 3 seconds

## Important Constraints and Business Rules

1. **Personnel Skills**: Each case requires specific skills (e.g., physician, kinesiologist). Personnel assigned to a route must collectively have all required skills for assigned cases.

2. **Time Windows**: Cases have flexible time windows:
   - Specific: "10:00-12:00"
   - General: "AM" (08:00-12:00), "PM" (12:00-17:00)

3. **Vehicle Capacity**: Limited by number of personnel and equipment

4. **Working Hours**: Default 08:00-17:00, customizable per personnel

5. **Deletion Protection**: Cannot delete personnel, vehicles, or cases that are part of active routes

6. **Location Data**: All coordinates in WGS 84 (EPSG:4326)
   - Latitude: -90 to +90
   - Longitude: -180 to +180

7. **GPS Tracking**: Mobile app uploads location every 30 seconds during active routes

8. **Notifications**: Push notification first, SMS fallback on failure

9. **Distance Providers**: Google Maps (primary) → OSRM (secondary) → Haversine (fallback)

## Common Pitfalls to Avoid

1. **PostGIS Setup**: Ensure PostGIS extension is enabled before creating tables
   ```sql
   CREATE EXTENSION IF NOT EXISTS postgis;
   ```

2. **Coordinate Order**: PostGIS uses (longitude, latitude), but many APIs use (latitude, longitude). Always verify order.

3. **Time Window Validation**: Ensure arrival time ≤ time_window_end AND departure time ≥ time_window_start

4. **Skill Matching**: Must check that ALL required skills are present in assigned personnel, not just ANY skill

5. **JWT Token Expiration**: Implement automatic token refresh before expiration to avoid disrupting user sessions

6. **WebSocket Reconnection**: Mobile app must handle connection drops gracefully and reconnect automatically

7. **Offline Sync**: Mobile app must queue status updates when offline and sync when connection restored

8. **Battery Optimization**: Use geofencing or smart intervals to reduce GPS polling frequency when vehicle is stationary

## Spanish Language

All user-facing content should be in Spanish:
- UI labels and messages
- Notification templates
- Error messages
- API documentation examples

Example notification:
- ✅ "El equipo médico llegará en 15 minutos"
- ❌ "Medical team will arrive in 15 minutes"

## Future Enhancements (Post-MVP)

- Advanced geocoding from street addresses
- EHR/Ficha Clínica integration
- Predictive analytics for visit duration
- Multi-day route planning
- Vehicle maintenance tracking
- Patient satisfaction surveys
- Machine learning for optimization improvements

---
> Source: [rutasoptimizacion/sorhd](https://github.com/rutasoptimizacion/sorhd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
