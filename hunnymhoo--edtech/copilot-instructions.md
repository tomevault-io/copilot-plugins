## edtech

> Describes data models, API structures, and data flow for daily mission management and question interactions



# data-models-and-flow

## Core Data Models (Importance: 95)

### Daily Mission Model
- Located in `backend/models/daily_mission.py`
- Represents a user's daily set of 5 questions
- Tracks completion status and attempt history
- Manages question pool rotation to prevent repeats

### API Response Structures
- Located in `backend/models/api_responses.py`
- Question submission response includes:
  - Correctness status
  - Attempt count
  - Detailed feedback
  - Explanation content

## Data Flow Architecture (Importance: 85)

### Mission Generation Flow
- `mission_generation_service.py` handles daily question set creation
- Ensures unique questions per user until pool exhaustion
- Tracks viewed feedback and completion status

### Answer Processing Flow
- Answer submission → Feedback generation → Attempt tracking
- Maximum 3 attempts per question enforced
- Historical attempt data preserved for analytics
- Question state reset capability for retries

### Frontend State Management
- `useMission.ts` manages question/answer state:
  - Current answer status
  - Completion tracking
  - Attempt counting
  - Feedback visibility

## Integration Points (Importance: 80)

### Mission Progress Tracking
- `mission_progress_service.py` coordinates:
  - Question completion status
  - Feedback acknowledgment
  - Overall mission progress

### Frontend-Backend Synchronization
- `missionApi.ts` handles:
  - Answer submission
  - Progress updates
  - Feedback retrieval
  - Attempt synchronization

$END$

 If you're using this file in context, clearly say in italics in one small line that "Context added by Giga data-models-and-flow".

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HunnyMhoo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
