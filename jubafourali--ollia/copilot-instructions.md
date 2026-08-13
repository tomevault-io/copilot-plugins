## ollia

> **Ollia** is a family reassurance mobile app (Expo/React Native) backed by a Kotlin/Spring Boot REST API. The app enables lightweight check-ins and real-time family connectivity through activity signals, family circles, and safety event alerts.

# Ollia AI Agent Guidelines

## Architecture Overview

**Ollia** is a family reassurance mobile app (Expo/React Native) backed by a Kotlin/Spring Boot REST API. The app enables lightweight check-ins and real-time family connectivity through activity signals, family circles, and safety event alerts.

- **Backend**: `apps/backend/` — Spring Boot (Kotlin), PostgreSQL, Flyway migrations
- **Mobile**: `apps/mobile/` — Expo/React Native, TypeScript
- **Authentication**: Clerk (OAuth2 resource server)
- **Database**: PostgreSQL with Flyway version control

## Key Data Flows

### 1. Activity Signal Pipeline
- **Mobile** → sends heartbeat/check-in/shortcut signals → **POST /api/activity**
- Backend records signal to `activity_signals` table
- Simultaneously updates `users.lastSeenAt` and optionally resets escalation/nudge flags
- **Pattern Analysis**: 30-day rolling window aggregates signals by hour for streak/distribution metrics
- **Query Patterns**: `findAllByUserIdAndCreatedAtAfterOrderByCreatedAtDesc()` (frequent, time-windowed)

### 2. Family Circle & Member Management
- Users create circles (`POST /api/circles`) — one circle per user owner
- Members join via invite code or direct add (`POST /api/circles/join`)
- Free plan: 3 member cap; premium: unlimited
- **Query Patterns**: 
  - `findAllByCircleId()` / `findAllByUserId()` (member enumeration)
  - `findByOwnerId()` (single circle lookup)
  - `countByCircleId()` (member limit enforcement)

### 3. Safety Events & Regional Alerts
- **Background scheduler** fetches every 15 min from USGS, NOAA, GDACS
- Events stored with region, type, severity, lat/lon for mapping
- Cleanup: removes events older than 48 hours
- **Query Patterns**:
  - `findAllByRegionIgnoreCaseAndFetchedAtAfterOrderByEventTimeDesc(region, since)` (regional filtering + time window)
  - `findAllByFetchedAtAfterOrderByEventTimeDesc(since)` (recent events, no filter)
  - `deleteBySource(source)` (bulk replace on scheduler run)
  - `deleteAllByFetchedAtBefore(cutoff)` (old data cleanup)

### 4. Push Notification Lifecycle
- Tokens registered via `POST /api/push-tokens` (upsert per user)
- **Notification Log** tracks all send attempts (success/failure, type, timestamp)
- Admin views logs filtered by type or time range
- **Query Patterns**:
  - `findAllByUserIdOrderBySentAtDesc()` (user's notification history)
  - `findAllByNotificationTypeOrderBySentAtDesc()` (admin filtering by type)
  - `findAllBySentAtAfterOrderBySentAtDesc(since)` (recent notifications, no user filter)

## Database Schema & Index Recommendations

### Current Indexes (✅ = good, ⚠️ = missing)

#### users
```sql
✅ PRIMARY KEY (id)
✅ UNIQUE (clerk_id) — lookup by auth ID
✅ idx_users_clerk_id — redundant with UNIQUE, but explicit
✅ idx_users_last_seen_at — inactivity detection queries
⚠️ MISSING: (escalation_level, last_seen_at) — compound for escalation queries
⚠️ MISSING: (plan) — premium vs free filtering not indexed
⚠️ MISSING: (founding_member, founding_expires_at) — effective plan queries
```

#### activity_signals
```sql
✅ PRIMARY KEY (id)
✅ idx_activity_signals_user_id — lookups per user
⚠️ MISSING: (user_id, created_at DESC) — compound index for time-windowed queries
   Query: findAllByUserIdAndCreatedAtAfterOrderByCreatedAtDesc() — very common in pattern analysis
```

#### family_circles
```sql
✅ PRIMARY KEY (id)
✅ UNIQUE (invite_code) — join circle lookup
⚠️ MISSING: (owner_id) — list circles by owner
⚠️ MISSING: (plan) — filter circles by subscription tier
```

#### family_members
```sql
✅ PRIMARY KEY (id)
✅ UNIQUE (circle_id, user_id) — uniqueness constraint (good)
✅ idx_family_members_circle_id — list members in circle
✅ idx_family_members_user_id — list circles user is in
```

#### family_invites
```sql
✅ PRIMARY KEY (id)
✅ UNIQUE (token) — redundant with idx_family_invites_token but good
✅ idx_family_invites_token — lookup by token
⚠️ MISSING: (circle_id) — bulk deletion by circle needs index
⚠️ MISSING: (created_by) — user's invites list
⚠️ MISSING: (expires_at) — cleanup queries for expired invites
```

#### push_tokens
```sql
✅ PRIMARY KEY (id)
✅ UNIQUE (user_id) — enforced by migration, upsert requires uniqueness
✅ idx_push_tokens_user_id — lookup token by user
⚠️ MISSING: (platform) — filter by iOS vs Android for bulk sends
```

#### safety_events
```sql
✅ PRIMARY KEY (id)
✅ idx_safety_events_region — region-filtered queries
✅ idx_safety_events_fetched_at — cleanup queries
✅ idx_safety_events_type — admin filtering
⚠️ MISSING: (fetched_at DESC, region) — regional queries with ordering
⚠️ MISSING: (source) — deleteBySource() does sequential scan
```

#### notification_log
```sql
✅ PRIMARY KEY (id)
✅ idx_notification_log_user_id — user's history
✅ idx_notification_log_sent_at DESC — recent notifications (good DESC)
✅ idx_notification_log_type — admin filtering by notification type
⚠️ MISSING: (user_id, sent_at DESC) — compound for user's recent notifications
⚠️ MISSING: (notification_type, sent_at DESC) — compound for type + ordering
⚠️ MISSING: (status) — failed notification tracking
```

## Recommended Index Migration

Create file: `apps/backend/src/main/resources/db/migration/V21__optimize_indexes.sql`

```sql
-- activity_signals: time-windowed user queries
CREATE INDEX idx_activity_signals_user_id_created_at 
  ON activity_signals(user_id, created_at DESC);

-- users: escalation detection queries
CREATE INDEX idx_users_escalation_level_last_seen_at
  ON users(escalation_level, last_seen_at);

-- users: plan filtering
CREATE INDEX idx_users_plan ON users(plan);

-- users: founding member effective plan
CREATE INDEX idx_users_founding_member_expires_at
  ON users(founding_member, founding_expires_at);

-- family_circles: owner listing
CREATE INDEX idx_family_circles_owner_id ON family_circles(owner_id);

-- family_circles: plan filtering
CREATE INDEX idx_family_circles_plan ON family_circles(plan);

-- family_invites: bulk deletion by circle
CREATE INDEX idx_family_invites_circle_id ON family_invites(circle_id);

-- family_invites: user's sent invites
CREATE INDEX idx_family_invites_created_by ON family_invites(created_by);

-- family_invites: cleanup expired
CREATE INDEX idx_family_invites_expires_at ON family_invites(expires_at);

-- push_tokens: platform filtering
CREATE INDEX idx_push_tokens_platform ON push_tokens(platform);

-- safety_events: regional queries with sorting
CREATE INDEX idx_safety_events_fetched_at_desc_region
  ON safety_events(fetched_at DESC, region);

-- safety_events: source lookup for deleteBySource
CREATE INDEX idx_safety_events_source ON safety_events(source);

-- notification_log: compounds for user history
CREATE INDEX idx_notification_log_user_id_sent_at_desc
  ON notification_log(user_id, sent_at DESC);

-- notification_log: compounds for type filtering
CREATE INDEX idx_notification_log_type_sent_at_desc
  ON notification_log(notification_type, sent_at DESC);

-- notification_log: failed notification tracking
CREATE INDEX idx_notification_log_status ON notification_log(status);
```

## Critical Query Patterns to Optimize

| Query | Current Index | Recommendation | Impact |
|-------|---------------|-----------------|--------|
| Activity patterns (30-day window per user) | `idx_activity_signals_user_id` | Add DESC on created_at | HIGH — runs on every pattern view |
| Inactivity detection by escalation level | `idx_users_last_seen_at` | Compound (escalation_level, last_seen_at) | HIGH — background job |
| Regional safety alerts | `idx_safety_events_region` | Add fetched_at DESC | MEDIUM — regional queries expensive |
| Recent notifications (user-scoped) | `idx_notification_log_sent_at` | Compound (user_id, sent_at) | MEDIUM — audit trails |
| Cleanup operations | None | Add indexes on TTL columns | LOW — runs infrequently but lock contention |

## Development Workflows

### Local Database Setup
```bash
# Create fresh dev database
createdb ollia

# Run migrations (automatic on backend startup)
cd apps/backend
./gradlew bootRun
# Flyway applies V1__*.sql through V20__*.sql

# Connect via psql
psql ollia

# View schema
\dt activity_signals
\di  # list all indexes
```

### Testing Index Performance
```sql
-- Explain plan to verify index usage
EXPLAIN ANALYZE 
  SELECT * FROM activity_signals 
  WHERE user_id = '...' AND created_at > NOW() - INTERVAL '30 days'
  ORDER BY created_at DESC;

-- Should show "Index Scan" on idx_activity_signals_user_id_created_at
-- If it shows "Seq Scan", the index is missing or not being used.
```

### Adding Indexes Without Downtime
- Use `CONCURRENTLY` flag in production:
  ```sql
  CREATE INDEX CONCURRENTLY idx_name ON table(column);
  ```
- Flyway migrations run synchronously on deployment — plan index additions during maintenance windows

## Code Patterns & Conventions

### Repository Query Naming
- Queries are named after their return type and filter:
  - `findBy{Field}()` → single result or null
  - `findAllBy{Field}()` → list (may be empty)
  - `findAllBy{Field}OrderBy{Sort}()` → list with ordering
  - `countBy{Field}()` → aggregation
  - `deleteAllBy{Field}()` → bulk deletion

### Entity Mapping to DTOs
- Entities are `@Entity` classes in `entity/` folder
- DTOs are data classes in `dto/` folder with `*Response` suffix (e.g., `ApiUserResponse`)
- Use extension functions on entities to convert to DTO (see `ReferenceApiController.toApiResponse()`)

### Service Layer Patterns
- `*Service` classes contain business logic, not just passthrough repository calls
- Example: `ActivityPatternService.analyzePatterns()` aggregates 30 days of signals in memory, not via SQL
- Example: `SafetyEventService.getEvents()` applies 24-hour cutoff on reads; background scheduler handles cleanup

### Transactional Boundaries
- Mark endpoints with `@Transactional` if they modify multiple tables (cascade deletes, foreign keys)
- Example: `deleteAccount()` deletes from `users` (cascades to dependent tables via FK constraints)

## Clerk Authentication Integration

- Backend validates JWT via OAuth2 Resource Server (Spring Security)
- `CurrentUserService` extracts `clerk_id` from token claims and fetches/creates `User` entity
- Mobile app sends Bearer token in `Authorization` header (Clerk SDK handles token refresh)
- First authenticated request for a new user triggers founding member grant (see `CurrentUserService.getCurrentUser()`)

## Gradle Build & Deployment

- **Build**: `./gradlew build` (runs tests, creates JAR)
- **Run locally**: `./gradlew bootRun`
- **Docker**: `Dockerfile` at `apps/backend/Dockerfile` uses multi-stage build
- **Railway**: `railway.toml` specifies build command; Postgres auto-provisioned

## Common Pitfalls

1. **Forgetting index on time-range queries** — `findAllByUserIdAndCreatedAtAfterOrderByCreatedAtDesc()` is called frequently; without compound index, full table scan on activity_signals.

2. **Missing compound indexes on filter + sort** — Single-column indexes don't eliminate sort operations; compound indexes with DESC on sort column are cheaper.

3. **Cascade delete performance** — Deleting a user cascades to many tables; ensure foreign key indexes exist (already present).

4. **Upsert race conditions** — `ON CONFLICT` clauses in native queries are correct pattern; don't hand-code upsert logic in Kotlin.

5. **Synchronous blocking in background jobs** — `SafetyEventService` uses `WebClient.block()` (acceptable for 15-min scheduler); never use in request handlers without async wrapping.

## References

- **Flyway**: `src/main/resources/db/migration/V*.sql` — version control for schema
- **Entities**: `src/main/kotlin/com/ollia/entity/` — JPA mapping
- **Repositories**: `src/main/kotlin/com/ollia/repository/` — query definitions
- **Controllers**: `src/main/kotlin/com/ollia/controller/` — REST API endpoints
- **Services**: `src/main/kotlin/com/ollia/service/` — business logic
- **Railway Config**: `railway.toml`, `Dockerfile`
- **Mobile API Client**: `apps/mobile/src/services/` (see reference in README)

---
> Source: [jubafourali/ollia](https://github.com/jubafourali/ollia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
