## infusion-scheduler

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a patient self-scheduling system for infusion therapy clinics. The system consists of two main components:

1. **Rails Frontend** (`infusion_scheduler_rails/`) - Patient-facing web application
2. **FastAPI + PydanticAI Backend** (`pydanticai_infusion_scheduler/`) - Scheduling agent with smart tools

## Build & Run Commands

### Backend (FastAPI)
```bash
cd pydanticai_infusion_scheduler
uv venv -p 3.12 .venv
source .venv/bin/activate
uv pip install -r requirements.txt
PYTHONPATH=. uvicorn api:app --reload --port 8000
```

### Frontend (Rails)
```bash
cd infusion_scheduler_rails
bundle install
bin/rails db:create db:migrate
bin/rails db:seed  # Load test data
bin/dev
```

### Test Data
```bash
cd infusion_scheduler_rails
bin/rails scheduling:setup_test_data  # Creates patients, chairs, drugs, etc.
bin/rails scheduling:setup_jamie_scenario  # Creates Jamie's specific test case
```

## Architecture

### Rails Models
- `Patient` - Patient info with MRN, care plan, insurance
- `Chair` - Infusion chairs with capacity
- `Booking` - Appointments with actual duration tracking
- `Drug` - Medications with lead time requirements
- `PriorAuthorization` - PA tracking with authorized infusion count
- `CarePlan` - Treatment cadence (frequency ± tolerance)
- `Staff` - Nursing staff schedules

### FastAPI Scheduling Agent Tools
The agent has these tools for checking availability:

1. **check_drug_availability** - Verify drug is on-site (lead time check)
2. **check_chair_availability** - Verify chair + staff availability for duration
3. **check_care_plan_cadence** - Validate slot within treatment window
4. **check_insurance_eligibility** - Real-time eligibility verification
5. **get_available_slots** - Orchestrator combining all checks
6. **book_appointment** - Create confirmed booking
7. **reschedule_appointment** - Move booking within rules

### Scheduling Rules (from requirements)
1. **Drug Lead Time**: Infliximab requires 3 days on-site before appointment
2. **Service Duration**: Patient-specific (e.g., 2hr drip + 30min observation = 2.5hr slot)
3. **Care Plan Cadence**: Every N weeks ± M days tolerance
4. **Insurance Guardrails**: Real-time eligibility + PA tracking
5. **Working Hours**: 9 AM - 5 PM (end of shift)

## Code Style & Guidelines
- **Python**: Use Ruff (88 char lines), type hints, double quotes
- **Ruby**: Standard Rails conventions, RuboCop compatible
- **API**: JWT authentication between Rails and FastAPI
- **Models**: Pydantic for validation in FastAPI

## Environment Variables

### FastAPI (.env)
```
DATABASE_URL=postgresql://user:pass@localhost:5432/infusion_scheduler
JWT_SECRET_KEY=your-secret-key
OPENAI_API_KEY=your-openai-key
```

### Rails (.env)
```
DATABASE_URL=postgresql://user:pass@localhost:5432/infusion_scheduler_rails
FASTAPI_URL=http://localhost:8000
JWT_SECRET_KEY=your-secret-key
```

## Test Scenarios

Use rake tasks to set up various test scenarios:

```bash
# Basic test data
bin/rails scheduling:setup_test_data

# Jamie's scenario (from requirements)
bin/rails scheduling:setup_jamie_scenario

# Edge cases
bin/rails scheduling:setup_pa_expiry_scenario
bin/rails scheduling:setup_overlapping_bookings
```

---
> Source: [medright/infusion_scheduler](https://github.com/medright/infusion_scheduler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
