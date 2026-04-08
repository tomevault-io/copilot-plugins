## surakshanet

> Multi-agent geospatial intelligence platform for real-time road safety monitoring in Kerala, India. Uses 13 specialized AI agents organized into 3 clusters (SWARM-GUARD, DRISHTI, SENTINEL), orchestrated via LangGraph with Byzantine fault-tolerant voting and Bayesian belief fusion.

    # SWARM-GUARD UNIFIED

    Multi-agent geospatial intelligence platform for real-time road safety monitoring in Kerala, India. Uses 13 specialized AI agents organized into 3 clusters (SWARM-GUARD, DRISHTI, SENTINEL), orchestrated via LangGraph with Byzantine fault-tolerant voting and Bayesian belief fusion.

    ## ABSOLUTE CONSTRAINTS — NEVER VIOLATE

    1. **Never use random number generators for risk scores.** Every risk_score must be computed from real external data (weather readings, traffic speeds, NCRB accident stats, CV detections). If data unavailable → risk_score=0.0, confidence=0.0, vote=HOLD. Never fabricate a number.

    2. **Never use `time.sleep()`.** Entire app is async. Use `asyncio.sleep()` if waiting needed. Use `asyncio.gather()` for parallel agents. Blocking the event loop is a production-killing bug.

    3. **Never hardcode credentials.** All secrets from `.env` via `python-dotenv` → `backend/config.py` Pydantic BaseSettings. App must validate ALL required env vars at startup and `sys.exit(1)` listing missing ones.

    4. **Never return placeholder causal attribution.** Attribution comes from Claude API (`claude-sonnet-4-20250514`). If API fails → log error, set `causal_attribution=None`, post Slack alert with raw scores marked "⚠️ Causal attribution unavailable — analysis engine returned error at {timestamp}. Manual review recommended." Never invent attribution. Never use template. Never say "high risk detected".

    5. **Never skip database persistence.** Every monitoring cycle output → PostgreSQL. If DB unreachable → buffer in Redis `persistence_queue` → background coroutine drains with exponential backoff. Never drop events silently.

    6. **Never exceed API rate limits.** Per-API tracking in Redis (token bucket/sliding window). If exhausted → skip agent, return confidence=0.0, vote=HOLD. Never get keys revoked.

    7. **Every file must be complete and runnable.** No `pass`, `...`, `# TODO`, `raise NotImplementedError`, stubs, or truncation. No imports referencing nonexistent modules.

    8. **Build order is sacred.** Each phase produces testable increment. Don't skip ahead.

    ## Tech Stack

    - **Backend:** Python 3.12, FastAPI, SQLAlchemy (async), PostgreSQL+PostGIS, Redis, LangGraph
    - **AI/ML:** YOLOv8, RT-DETR, OSNet, Anthropic Claude API (Sonnet for inner calls)
    - **Frontend:** Next.js 14, Leaflet.js, Recharts, Tailwind CSS, Socket.IO
    - **Infra:** Docker Compose, Alembic migrations, structlog JSON logging
    - **Alerting:** Slack Bolt SDK (Socket Mode)

    ## Project Structure

    ```
    swarm-guard-unified/
    ├── CLAUDE.md                          ← you are here
    ├── .env.example
    ├── .env                               ← never commit, never read in Claude Code
    ├── docker-compose.yml
    ├── Makefile
    ├── backend/
    │   ├── config.py                      ← Pydantic BaseSettings, @lru_cache singleton
    │   ├── requirements.txt               ← pinned versions
    │   ├── Dockerfile
    │   ├── agents/
    │   │   ├── base.py                    ← BaseAgent ABC + AgentOutput schema
    │   │   ├── weather_friction.py        ← Open-Meteo + OpenWeatherMap fallback
    │   │   ├── historical_pattern.py      ← NCRB accident data
    │   │   ├── traffic_anomaly.py         ← HERE Traffic Flow API v7
    │   │   ├── visual_risk.py             ← YOLOv8 object detection
    │   │   ├── situation_awareness.py     ← IMD warnings (DRISHTI)
    │   │   ├── resource_inventory.py      ← KSDMA resources (DRISHTI)
    │   │   ├── population_vulnerability.py ← Census 2011 (DRISHTI)
    │   │   ├── logistics_optimizer.py     ← OR-Tools VRP (DRISHTI)
    │   │   ├── counterfactual.py          ← Claude projection (DRISHTI)
    │   │   ├── edge_perception.py         ← RT-DETR (SENTINEL)
    │   │   ├── cross_camera.py            ← OSNet Re-ID (SENTINEL)
    │   │   ├── threat_escalation.py       ← Bayesian Beta-Binomial (SENTINEL)
    │   │   └── evidence_packaging.py      ← Section 65B compliance (SENTINEL)
    │   ├── api/
    │   │   ├── main.py                    ← FastAPI app, lifespan, middleware
    │   │   └── routes/
    │   │       ├── health.py              ← GET /health, GET /health/ratelimits
    │   │       ├── segments.py            ← GET /segments, /segments/{id}, /segments/{id}/events
    │   │       └── websocket.py           ← WebSocket /ws/riskmap
    │   ├── models/
    │   │   ├── base.py                    ← async SQLAlchemy engine + session
    │   │   ├── segment.py                 ← RoadSegment ORM (PostGIS geometry)
    │   │   ├── event.py                   ← RiskEvent ORM (JSONB agent_scores)
    │   │   ├── historical.py              ← HistoricalAccident (NCRB data)
    │   │   ├── disaster_resource.py       ← DisasterResource (KSDMA)
    │   │   └── census.py                  ← CensusData (2011)
    │   ├── orchestrator/
    │   │   ├── state.py                   ← GeospatialBeliefState TypedDict
    │   │   ├── voting.py                  ← Byzantine fault-tolerant weighted voting
    │   │   ├── belief_fusion.py           ← Bayesian fusion with correlation correction
    │   │   ├── coordinator.py             ← 3-stage: vote → fuse → Claude attribution
    │   │   ├── graph.py                   ← LangGraph StateGraph wiring
    │   │   └── monitor.py                 ← Continuous monitoring loop + rate limits
    │   └── services/
    │       ├── database.py                ← persist_event + Redis queue fallback
    │       └── slack_service.py           ← Bolt SDK, Block Kit alerts, slash commands
    ├── frontend/
    │   ├── package.json
    │   ├── Dockerfile
    │   └── src/app/
    │       ├── page.tsx                   ← Leaflet map + side panel
    │       ├── events/page.tsx            ← Events table
    │       └── health/page.tsx            ← System health
    ├── scripts/
    │   ├── seed_segments.py               ← 6 Kerala road segments
    │   ├── parse_ncrb.py                  ← NCRB accident data
    │   ├── seed_disaster_resources.py     ← KSDMA inventory
    │   └── seed_census.py                 ← Census 2011 data
    ├── tests/
    │   ├── test_agents/                   ← per-agent tests
    │   ├── test_orchestrator/             ← voting, fusion, coordinator, graph
    │   ├── test_api/                      ← endpoint tests
    │   ├── test_services/                 ← Slack, DB tests
    │   └── test_integration/             ← full pipeline E2E
    └── alembic/                           ← async migrations
    ```

    ## Agent Architecture

    ### AgentOutput Schema (every agent returns this):
    ```python
    agent_name: str
    risk_score: float   # 0.0-1.0, from REAL data, NEVER random
    confidence: float   # 0.0-1.0, trust level
    vote: "ESCALATE" | "HOLD" | "DISMISS"  # >0.70 ESCALATE, <0.30 DISMISS, else HOLD
    metadata: dict      # raw API data, intermediate calculations
    timestamp: datetime
    data_sources_used: list[str]
    ```

    ### Graceful failure (any agent, any error):
    ```python
    risk_score=0.0, confidence=0.0, vote="HOLD",
    metadata={"error": str(error), "fallback": True}
    ```

    ### 3 Clusters:

    **SWARM-GUARD (road safety):** weather_friction, historical_pattern, traffic_anomaly, visual_risk → run in parallel

    **DRISHTI (disaster response):** situation_awareness, resource_inventory, population_vulnerability, logistics_optimizer, counterfactual → run in parallel

    **SENTINEL (surveillance):** edge_perception + cross_camera (parallel) → threat_escalation (sequential) → evidence_packaging (sequential)

    ## Orchestration Pipeline (per segment, per cycle)

    1. **Agents run** → collect AgentOutput list
    2. **Byzantine Voting** → weighted sum: ESCALATE +1×conf, HOLD 0, DISMISS -1×conf. Sum >+0.50 ESCALATE, <-0.50 DISMISS, else HOLD. Exclude confidence=0 agents. >50% failed → degraded mode.
    3. **Bayesian Belief Fusion** → fuse scores accounting for correlation (Visual+Traffic ρ=0.6). Independent: weighted average. Correlated: down-weighted. Combined via precision weighting.
    4. **Coordinator** → if DISMISS stop. If HOLD+low belief stop. Else → Claude API causal attribution → persist → Slack alert if ESCALATE or belief >0.72 → Redis publish for WebSocket.

    ## External APIs (all verified free-tier)

    | API | Agent | Key | Limit | Endpoint |
    |-----|-------|-----|-------|----------|
    | Open-Meteo | weather_friction | No | 10K/day | api.open-meteo.com/v1/forecast |
    | OpenWeatherMap | weather_friction (fallback) | Yes | 60/min, 1K/day | api.openweathermap.org/data/2.5/weather |
    | HERE Traffic v7 | traffic_anomaly | Yes | 1K/day | data.traffic.hereapi.com/v7/flow |
    | Anthropic Claude | coordinator + counterfactual | Yes | 60 RPM | claude-sonnet-4-20250514 |
    | IMD Warnings | situation_awareness | No | ~100/day | mausam.imd.gov.in |

    ## 6 Monitored Kerala Road Segments

    1. NH-66 Edapally Junction, Ernakulam — 10.0261,76.3125 → 10.0350,76.3089 — asphalt, 60km/h, 4 lanes
    2. NH-544 Thrissur-Palakkad — 10.5276,76.2144 → 10.5400,76.1980 — asphalt, 80km/h, 4 lanes
    3. NH-66 Kasaragod Bypass — 12.5010,74.9893 → 12.5150,74.9820 — asphalt, 70km/h, 2 lanes
    4. MC Road Kottayam-Changanassery — 9.5916,76.5222 → 9.5750,76.5350 — concrete, 50km/h, 2 lanes
    5. SH-1 Wayanad Ghat Road — 11.6854,76.1320 → 11.6700,76.1200 — wbm, 30km/h, 2 lanes (9 hairpins)
    6. NH-66 Kozhikode Beach Road — 11.2588,75.7804 → 11.2500,75.7750 — asphalt, 40km/h, 4 lanes

    ## Key Formulas

    **Friction (IRC SP:73-2015):** base × (1-rain_reduction) × (1-temp_reduction) × humidity × wind
    **Historical risk:** min(1.0, district_rate / (2 × 14.2)) × time_multiplier × day_multiplier
    **Traffic risk:** 0.6 × (1 - speed/freeflow) + 0.4 × (jamFactor/10)
    **Belief fusion:** (μ_indep × prec_indep + μ_corr × prec_corr) / (prec_indep + prec_corr)

    ## Rate Limit Schedule

    MONITORING_INTERVAL_SECONDS=300 (5min). 6 segments per cycle.
    - Open-Meteo: every cycle = 1,728/day ✓ (under 10K)
    - HERE: once per segment per hour = 144/day ✓ (under 1K)
    - OpenWeatherMap: fallback only
    - Anthropic: only on ESCALATE/HOLD-above-threshold

    ## Code Standards

    - **Async everywhere:** asyncio, httpx.AsyncClient, create_async_engine, async_sessionmaker
    - **Retry:** tenacity @retry max_attempts=3, wait_exponential with jitter on all HTTP calls
    - **Logging:** structlog JSON with request_id context
    - **Types:** Full type hints on every function parameter and return
    - **Testing:** pytest + pytest-asyncio, real API calls in integration tests, mocked failures in unit tests
    - **Linting:** ruff check + ruff format

    ## Build Order (sacred)

    Phase 0-2: Scaffold + DB + FastAPI → Phase 3: BaseAgent → Phase 4: Weather → Phase 5: Historical → Phase 6: Traffic → Phase 7: Visual → Phase 8-9: Voting + Fusion → Phase 10: Coordinator → Phase 11: Graph + Monitor → Phase 12: Slack → Phase 13: DRISHTI (5 agents) → Phase 14: SENTINEL (4 agents) → Phase 15: Rate Limits → Phase 16: Frontend → Phase 17: Integration Tests

    ## Environment Variables (.env)

    ```
    ANTHROPIC_API_KEY=
    OPENWEATHERMAP_API_KEY=
    HERE_API_KEY=
    SLACK_BOT_TOKEN=xoxb-...
    SLACK_APP_TOKEN=xapp-...
    SLACK_CHANNEL_ID=
    DATABASE_URL=postgresql+asyncpg://swarmguard:swarmguard@localhost:5432/swarmguard
    REDIS_URL=redis://localhost:6379/0
    MONITORING_INTERVAL_SECONDS=300
    LOG_LEVEL=INFO
    ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sherin-SEF-AI)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Sherin-SEF-AI)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
