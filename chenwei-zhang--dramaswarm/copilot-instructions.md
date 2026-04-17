## dramaswarm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run all tests (102 tests)
pytest tests/ -v

# Run specific test suites
pytest tests/test_crisis_engine.py tests/test_temporal_graph.py -v
pytest tests/test_persona_agent.py tests/test_audience.py tests/test_memory_store.py -v
pytest tests/test_crisis_engine.py tests/test_experiment.py -v

# Run single test
pytest tests/test_crisis_engine.py -v -k "test_name"

# Run crisis simulation demo (rule mode, no LLM needed)
python -m demos.crisis_simulation

# Web visualization (includes crisis simulation tab)
python run_viz.py
# http://localhost:8765 → 知识图谱 tab + 危机模拟 tab

# Scraper test (mock mode)
python test_scraper.py mock 杨幂

# Scraper test (real mode, needs network)
python test_scraper.py real 杨幂

# Update celebrity data (Baidu Baike + Weibo, needs WEIBO_COOKIE in .env)
python update_weibo_data.py --names 杨幂 赵丽颖
python update_weibo_data.py --all
python update_weibo_data.py --validate  # 仅验证 Cookie
```

## Environment Setup

Copy `.env.example` to `.env` and set `GEMINI_API_KEY`. Other keys (`OPENAI_API_KEY`, `ANTHROPIC_API_KEY`) optional. Config via env vars: `LLM_PROVIDER` (default `gemini`), `LLM_MODEL` (default `gemini-3-flash-preview`), `MAX_AGENTS`, `SIMULATION_TICK_INTERVAL`, `DEFAULT_TURNS`.

## Architecture

### Crisis Simulation Flow

```
CrisisSimulation.step() → per day:
  1. Timeline      → CrisisTimeline.advance_day() → determine CrisisPhase
  2. Intervention  → InterventionSystem.check() → apply user's what-if conditions
  3. Decision      → randomized agent order, sequential decisions with peer_actions + audience_reactions
  4. MessageBus    → audience reactions + celebrity actions broadcast to bus
  5. Effects       → CrisisActionSpace or FreeActionSpace (mode-dependent) → approval/heat/brand deltas
  6. Debunk        → STATEMENT/LAWSUIT can debunk rumors (40%/60% chance)
  7. Reactions     → InformationVacuumDetector → silence → event-driven rumor cascade
  8. Update        → trending topics, media headlines, brand statuses
  9. Decay         → heat -10%, approval regresses toward role-specific target
 10. Termination   → heat <15 for 3 days OR regulatory level 5 → early end
```

### Key Module Relationships

```
swarmsim/graph/
├── knowledge_graph.py    # networkx MultiDiGraph engine, loads from JSON/mock
└── temporal.py           # TemporalKnowledgeGraph — date-indexed, person timelines

swarmsim/crisis/          # Crisis simulation engine
├── models.py             # CrisisPhase(6), PRAction(10), CrisisRole(4), InteractionMode, FreeAction(11), CrisisState, etc.
├── timeline.py           # 1 turn = 1 day, 6-phase lifecycle
├── action_space.py       # 10 PR actions × 6 phases effect matrix + Big Five mods
│                         # FreeActionSpace — 11 free actions × 6 phases + personality mods + low-approval penalty
├── persona_agent.py      # CelebrityPersonaAgent (rule/LLM), builds persona from GraphRAG
│                         # crisis_role (PERPETRATOR/VICTIM/ACCOMPLICE/BYSTANDER) injected by engine
│                         # gossip_type (CHEATING/SCANDAL/DIVORCE/DRUGS/TAX_EVASION/OTHER) drives decision
│                         # Role-aware + gossip-type-aware decision: perp banned from comeback, drugs bans counterattack
│                         # Extended peer_influence: strength-weighted (relation strength 0-1 scales boost)
│                         # Memory-driven: consecutive silence → urgency, repeated apology → strategy switch
│                         # Semantic repeat penalty: similar action groups share penalty (SILENCE+HIDE, etc.)
│                         # Audience semantic analysis: keyword extraction → action-specific boosts
│                         # Structured memory via MemoryStore (add/search/get_recent/get_important)
│                         # Personality natural language description for LLM prompts
│                         # LLM prompt: graph context + timeline events + memory summary + personality description
│                         # Free mode: generate_free_response() / _rule_decide_free() / _llm_decide_free()
│                         # Enhanced free LLM prompt: includes state/role/gossip_type/audience
├── vacuum_detector.py    # Silence → rumor cascade, probability escalates with days
│                         # Event-driven rumor generation: extract topic from scenario description
│                         # (e.g. "代言带货/虚假宣传", "出轨/婚变") → combine with rumor angles
│                         # (escalation/coverup/chain/insider/digging) → context-relevant rumors
│                         # Priority: LLM > event-context rumor > title-referenced fallback
│                         # Debunk mechanism: STATEMENT/LAWSUIT can debunk rumors (severity → 20%)
├── intervention.py       # User what-if conditions with trigger types (TIME_ABSOLUTE/TIME_RELATIVE/STATE_THRESHOLD)
│                         # External events (MEDIA_REPORT/VIDEO_LEAK/COMPETITOR_ANNOUNCE/etc.)
├── experiment.py         # ExperimentManager, Experiment, ExperimentGroup, ComparisonResult — A/B testing
├── scenario_engine.py    # CrisisScenarioEngine (load + role inference) + CrisisSimulation (async run loop)
│                         # Scenario description uses gossip content (not just title + persons)
│                         # list_crisis_scenarios() returns content field for rumor context
│                         # _infer_person_roles(): auto-infer for CHEATING/DRUGS/TAX_EVASION/DIVORCE/SCANDAL
│                         # Randomized decision order per day (avoids first-mover bias)
│                         # MessageBus integration: audience reactions + celebrity actions broadcast
│                         # Dynamic termination: heat <15 for 3 days or regulatory level 5
│                         # Free mode effects computed via FreeActionSpace (with phase modifiers)
├── message_bus.py        # Agent-to-agent message bus (broadcast/direct/per-type filtering)
├── audience.py           # AudiencePool (30 agents: 粉丝/路人/理中客/黑粉) + reaction templates
│                         # AudienceAgent has comment memory: repeated action → lower comment probability
│                         # Semantic similarity groups: SILENCE+HIDE, APOLOGIZE+CHARITY share repeat penalty
└── outcome_analyzer.py   # Compare sim vs historical baseline, generate PR recommendations

swarmsim/memory/base.py   # MemoryStore ABC → InMemoryStore, SQLiteStore
│                         # MemoryEntry: id, agent_id, timestamp, content, source, importance, tags, metadata
│                         # get_recent() — by timestamp, get_important() — by importance
│                         # decay_importance() — forgetting curve (importance -= rate × days)
swarmsim/llm/client.py    # LLM client: Gemini, OpenAI, Anthropic via get_client()
│                         # generate_with_retry() — exponential backoff retry (3 attempts)
celebrity_scraper/        # Independent module: 5 web spiders + mock data
```

### Crisis Simulation Architecture

`CrisisSimulation.step()` runs one simulated day, branching on `interaction_mode` (CRISIS vs FREE):

**CRISIS mode** (original flow):
1. `timeline.advance_day()` → determine CrisisPhase (breakout→escalation→peak→mitigation→resolution→aftermath)
2. `intervention_system.check()` → apply user's what-if conditions
3. **Sequential decision making**: agents act in order; each sees `peer_actions` (earlier agents' actions) + `audience_reactions` from `AudiencePool`
4. **Relationship influence**: agent queries `kg.get_relationship_context()` for each peer's action, adjusts weights (spouse apologizes → I soften; rival counterattacks → I harden)
5. **Audience reactions**: `AudiencePool` (30 agents: 粉丝/路人/理中客/黑粉) generates comments based on action templates + persona bias
6. **Propagation**: `_check_propagation()` marks agents whose close relations took significant actions
7. `InformationVacuumDetector` checks silence → generates rumors
8. `CrisisActionSpace.compute_effect()` → apply approval/heat/brand deltas
9. Generate trending topics, media headlines, update brand statuses
10. Daily decay: heat -10%, approval regresses toward 50

**FREE mode** (open interaction):
- Agents choose from `FreeAction` enum: SPEAK, SUPPORT, CRITICIZE, COLLABORATE, SOCIALIZE, ANNOUNCE, IGNORE, PRIVATE_MSG, MEDIATE, RUMOR, RETREAT
- `FreeActionSpace` computes simplified effects without phase modifiers
- `persona_agent.py` methods: `generate_free_response()`, `_rule_decide_free()`, `_llm_decide_free()`

**Intervention system** (frontend simplified): two-tab panel — "强制动作" (force PR action on a person) or "注入事件" (inject external event), triggered by day number. Backend still supports `TIME_ABSOLUTE`, `TIME_RELATIVE`, `STATE_THRESHOLD` triggers and relationship changes via API.

**External events** (`ExternalEventType` enum): MEDIA_REPORT, VIDEO_LEAK, COMPETITOR_ANNOUNCE, REGULATORY_ACTION, BRAND_DECISION, CUSTOM — injectable via intervention conditions.

**A/B Experiments** (`crisis/experiment.py`): backend API only (frontend panel removed). API endpoints: `POST /experiment/create`, `POST /experiment/{id}/run`, `GET /experiment/{id}/compare`, `GET /experiments`

`CrisisAction` has `triggered_by` and `trigger_relation` fields for tracking inter-agent causality, plus `free_action` field for free-mode actions. `CrisisState` includes `audience_reactions` and `interaction_log`. Agent personality is built from GraphRAG data (biography keywords → Big Five traits). The persona_agent supports rule-based mode (decision matrix) and LLM mode (structured prompt → parse PRAction). Rule mode needs no API key.

## Code Conventions

- **Language**: All UI strings, comments, docstrings, and user-facing text are in Chinese. Variable/method names are English snake_case.
- **No build system**: `requirements.txt` only, no `pyproject.toml` or `setup.py`.
- **Testing**: pytest with class-based tests. Async tests use `pytest-asyncio`. No CI configured.
- **Deterministic personality**: `CelebrityPersonaAgent` uses `hashlib.md5(persona_name)` to seed random decisions, ensuring the same persona always produces the same personality traits across runs.
- **Always respond in Chinese** when communicating with the user.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chenwei-zhang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
