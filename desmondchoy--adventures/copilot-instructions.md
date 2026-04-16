## adventures

> This file provides guidance to Gemini (Google AI) when working with code in this repository.

# GEMINI.md

This file provides guidance to Gemini (Google AI) when working with code in this repository.

## Project Overview

Learning Odyssey is an AI-powered interactive educational storytelling platform. Children select a story world and educational topic, then progress through personalized adventures where choices shape the narrative. The application uses LLM-generated content with structured chapter types (STORY, LESSON, REFLECT, CONCLUSION, SUMMARY).

## Development Commands

### Running the Application
```bash
# Activate virtual environment (required for all Python commands)
source .venv/bin/activate

# Run development server
uvicorn app.main:app --reload
```

### Dependencies
```bash
pip install -r requirements.txt
```

### Deployment: Cache Busting
When deploying code changes (especially JS/CSS), update version strings to force cache invalidation:

**Files with version strings:**
- `app/templates/base.html` — CSS files and `font-size-manager.js`
- `app/templates/components/scripts.html` — ES6 module (`main.js`)

**Version format:** `?v=YYYYMMDDC` (date + letter increment)
- Example: `?v=20260101a` → `?v=20260101b` → `?v=20260102a`

**Update all version strings when:**
- Modifying any JavaScript file
- Modifying any CSS file
- Fixing bugs that users might have cached

## Architecture Overview

### Core Flow
1. **WebSocket Router** (`app/routers/websocket_router.py`) — Entry point for adventure sessions, handles JWT auth
2. **AdventureStateManager** (`app/services/adventure_state_manager.py`) — Central state management
3. **ChapterManager** (`app/services/chapter_manager.py`) — Chapter type sequencing and story element selection
4. **LLMServiceFactory** (`app/services/llm/factory.py`) — Dual-model architecture for cost optimization

### Dual-Model LLM Architecture
The factory pattern routes tasks to appropriate models (~50% cost reduction):
- **Gemini Flash** (29% of ops): Complex reasoning — `story_generation`, `image_scene_generation`
- **Gemini Flash Lite** (71% of ops): Simple processing — `summary_generation`, `paragraph_formatting`, `character_visual_processing`, `image_prompt_synthesis`

```python
from app.services.llm.factory import LLMServiceFactory
llm = LLMServiceFactory.create_for_use_case("story_generation")  # Returns Flash
llm = LLMServiceFactory.create_for_use_case("summary_generation")  # Returns Flash Lite
```

### State Model
`AdventureState` (in `app/models/story.py`) is the central data structure:
- `planned_chapter_types`: Pre-determined sequence of ChapterType enums
- `chapters`: List of completed ChapterData objects
- `selected_narrative_elements`, `selected_sensory_details`: Story elements chosen at initialization
- `chapter_summaries`, `lesson_questions`: Accumulated data for final summary
- `character_visuals`: Dict tracking visual descriptions for all characters (for image consistency)
- `protagonist_description`: Base visual description of protagonist
- `metadata`: Agency details, challenge history, timing data

### Chapter Type Sequencing
Configurable chapter length (currently 10) with rules:
- First chapter through second-to-last chapter: eligible for STORY, LESSON, or REFLECT
- Last chapter: always CONCLUSION
- First chapter is always STORY (with agency choice)
- One LESSON-REFLECT-STORY sequence placed randomly in the middle chapters
- Two additional non-consecutive LESSON chapters
- Remaining positions: STORY

### WebSocket Services (`app/services/websocket/`)
- `core.py`: Connection management and coordination
- `choice_processor.py`: User choice handling, triggers character visual updates
- `content_generator.py`: Chapter content generation
- `stream_handler.py`: Live streaming responses (chunk-by-chunk for performance)
- `image_generator.py`: AI image generation with two-step prompt synthesis
- `summary_generator.py`: Final adventure summary

### Data Sources
- `app/data/lessons/*.csv`: Educational questions by topic
- `app/data/stories/*.yaml`: Story categories with narrative elements, sensory details, themes

### Persistence
- **Supabase**: User auth (Google OAuth + Guest), adventure state, telemetry
- `StateStorageService`: State persistence with RLS policies
- `TelemetryService`: Analytics logging with duration tracking

## Critical Development Patterns

### 1. Dynamic Narrative Handling (CRITICAL)
Narrative content is generated dynamically by LLMs and is inherently variable. **Never hardcode narrative text in application logic or tests.**
- Rely on structure (AdventureState fields, ChapterType) not specific content
- Use metadata for reliable decision-making
- Tests should verify state transitions and structure, not narrative sentences

### 2. Validated Streaming Pattern
Use `stream_chapter_with_live_generation()` to generate, validate, then stream:
```python
# CORRECT — Collect and validate first, then stream approved content
chapter_content = await generate_chapter_content_with_retries(...)
await stream_text_content(chapter_content.content, websocket)
await websocket.send_json({"type": "choices", "choices": ...})
```
This intentionally prioritizes correctness:
- Story chapters must have exactly 3 validated choices before content is shown
- Unvalidated live token streaming should be avoided for chapter output

### 3. Background Task Pattern
Non-critical tasks (summaries, visual extraction) run in background after streaming:
```python
# Defer task factories until after streaming completes
state.deferred_task_factories.append(create_visual_extraction_task)
# Execute after streaming
await execute_deferred_task_factories(state)
```

### 4. Two-Step Image Prompt Synthesis
Images use a sophisticated synthesis process for visual consistency:
1. Generate scene description from chapter content (`IMAGE_SCENE_PROMPT`)
2. LLM combines scene + protagonist look + agency + character_visuals into final prompt
3. Send synthesized prompt to Imagen

### 5. Character Visual Evolution
- `state.character_visuals` tracks all character appearances
- `CHARACTER_VISUAL_UPDATE_PROMPT` extracts descriptions from each chapter
- `update_character_visuals()` merges new descriptions intelligently
- **System-wide character description rules in `SYSTEM_PROMPT_TEMPLATE` are NOT duplication** — they ensure extractable descriptions in every chapter

### 6. Security: User Isolation
- WebSocket: Authenticated users access adventures via `user_id` only (no `client_uuid` fallback)
- REST APIs: `validate_user_adventure_access()` checks ownership before data access
- Guest adventures (`user_id IS NULL`) remain accessible to anyone

### 7. On-Demand Summary Generation
Missing summaries are generated lazily when the summary screen is requested (not during chapter flow):
```python
# In summary_generator.py
await ensure_all_summaries_exist(state)  # Generates only missing summaries
```
This avoids complex async task coordination and handles any scenario where summaries might be missing.

### 8. Story Chapter Validation with Retries
Story chapters MUST have exactly 3 choices. The system uses `generate_chapter_content_with_retries()` which:
- Validates LLM output has 3 choices
- Retries up to 3 times if validation fails
- Only streams content that passes validation

### 9. Chapter Update Timing
**Critical**: Send `chapter_update` message BEFORE streaming content, not after. This ensures the UI shows the correct chapter number immediately when content starts streaming.

## Code Style & Patterns

- **Virtual Environment**: ALWAYS activate `.venv` before running Python commands
- **Imports**: Use absolute imports (`from app.services.summary import SummaryService`)
- **Type hints**: Required for all functions, use Pydantic models extensively
- **Error handling**: Use custom exceptions (`StateNotFoundError`, `SummaryError`)
- **Async**: Use `async`/`await` for all I/O operations, services are async
- **Naming**: `snake_case` for functions/variables, `PascalCase` for classes
- **State management**: `AdventureState` is single source of truth, never hardcode chapter data
- **Logging**: Use structured logging with context (`logger.info("message", extra={"state_id": id})`)
- **Code reuse**: Review existing codebase before creating new functions, prioritize modular design

## Critical Implementation Rules

### State Management
- `AdventureState` MUST be single source of truth
- NEVER hardcode chapter numbers (use `state.story_length` and `planned_chapter_types`)
- Agency choice MUST be stored in `state.metadata["agency"]`
- Always convert chapter types to lowercase when storing/retrieving
- Complete state serialization required with proper type hints
- State changes must be logged with context

### Chapter Requirements
- First chapter MUST be STORY type with agency choice
- Last chapter MUST be CONCLUSION type
- 3 LESSON chapters required (1 in a LESSON-REFLECT-STORY sequence, 2 additional non-consecutive)
- REFLECT chapters MUST only follow LESSON chapters
- No consecutive LESSON chapters allowed
- No question repetition in session

### Agency Implementation
- Agency MUST be referenced in all subsequent chapters
- Agency MUST evolve in REFLECT chapters
- Agency MUST have meaningful resolution in conclusion
- Use `update_agency_references()` for tracking

### Code Implementation Guidelines (CRITICAL)

**NEVER make assumptions — always verify first.** Before writing ANY code that calls functions, imports, or references existing code:

1. **ALWAYS search and verify function names and signatures BEFORE using them**
2. **NEVER assume function names, even if they seem logical**
3. **ALWAYS check existing import patterns before adding new imports**
4. **ALWAYS verify parameter types and return values**

**Implementation order:**
1. **Search** → Find existing implementations
2. **Read** → Understand the actual code structure
3. **Verify** → Check function signatures and usage patterns
4. **Implement** → Write code based on verified information
5. **Test** → Run diagnostics to catch errors early

## Testing Guidelines

### What to Test
- State transitions and `AdventureState` updates
- Structural correctness (does chapter have content? does summary have title?)
- Correct function calls and service interactions
- Chapter type validation and sequence rules

### What NOT to Test
- Specific LLM-generated narrative text (will break tests)
- Exact sentences or character names from generated content

### Mocking
Use mocking to provide structurally correct but non-specific narrative content when testing components that consume it.

### Running Tests
```bash
# Run all tests
pytest tests/

# Run specific test file
pytest tests/test_summary_service.py
```

## Key Validation Rules

### ChapterType Enum
```python
class ChapterType(str, Enum):
    LESSON = "lesson"
    STORY = "story"
    CONCLUSION = "conclusion"
    REFLECT = "reflect"
    SUMMARY = "summary"
```

### Chapter Validation
- Story chapters: exactly 3 choices
- CONCLUSION/SUMMARY chapters: exactly 0 choices
- No consecutive LESSON chapters
- REFLECT must follow LESSON and precede STORY

## Agency Implementation

Four categories defined in `prompt_templates.py`:
- Craft a Magical Artifact
- Choose a Companion
- Take on a Profession
- Gain a Special Ability

Agency is tracked in `state.metadata["agency"]` with visual details extracted from choice text.

## Frontend Architecture

### ES6 Modules (`app/static/js/`)
- `authManager.js`: Supabase auth, session management
- `adventureStateManager.js`: localStorage operations
- `webSocketManager.js`: Connection lifecycle, reconnection with exponential backoff
- `stateManager.js`: State transitions
- `uiManager.js`: DOM manipulation, story rendering
- `main.js`: Entry point, module coordination

### Configuration Bridge
`app/templates/components/scripts.html` sets up `window.appConfig` with server-side data for client modules.

## Environment Variables
Required in `.env`:
- `GOOGLE_API_KEY`: For LLM and image generation
- `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_KEY`, `SUPABASE_JWT_SECRET`: Database/auth
- `APP_ENVIRONMENT`: development/production

## Security Audit Context

### Production Deployment Model (Railway)
- Production deploy path is **GitHub -> Railway auto-deploy only**.
- Local CLI deploys (for example `railway up`) are **not** part of the production release process.
- Local `.env` is developer-only, must remain gitignored, and must be excluded from Docker build context.

### Severity Rules for Local Secret Findings
- Presence of a local `.env` file alone is **not** automatically a production exposure.
- Escalate to `high`/`critical` only when at least one is true:
  - Secret is committed to git history.
  - Deployment source included local filesystem context.
  - Secret appears in shared logs/artifacts/images/backups.

### Required Verification for Secret-Exposure Claims
- Check deployment source in Railway deploy history (`via GitHub` vs local CLI source).
- Confirm `.env` is not tracked (`git ls-files .env` should be empty).
- Confirm `.dockerignore` excludes `.env` and `.env.*`.
- Refer to `docs/security/deployment-model.md` as the source of truth before assigning severity.

## Database Migrations
Supabase migrations are in `supabase/migrations/`. Apply with:
```bash
npx supabase db push
```

## Common Debugging Patterns
- **Method Missing at Runtime**: Check indentation — methods inside functions won't be accessible on class instances
- **AttributeError on Service Methods**: Verify method is properly indented within the class definition
- **Telemetry/Duration Issues**: Connection restarts cause chapter start times to be lost, resulting in null duration values. Check metadata keys and consider timestamp-based backfilling
- **Data Flow Tracing**: When debugging UI display issues, trace the complete data flow from storage → calculation → display rather than assuming the problem is at the UI layer

## Additional Documentation
- `memory-bank/` — Architectural decisions, implementation plans, progress logs, LLM best practices
- `wip/implemented/` — Detailed implementation history for major features (streaming optimization, Supabase integration, visual consistency, etc.)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DesmondChoy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
