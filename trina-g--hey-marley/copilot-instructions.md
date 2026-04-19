## hey-marley

> Create a simple web application prototype that connects to two Langflow pipelines for Iteration 1 of the onboarding workflow:

# Cursor Rules: Simple App Prototype with Langflow Integration

## Project Overview

Create a simple web application prototype that connects to two Langflow pipelines for Iteration 1 of the onboarding workflow:
- **Flow 1:** Scenario Generation (creates personalized writing prompts)
- **Flow 2:** Assessment + Plan (evaluates writing and generates learning focus areas)

## Architecture Requirements

### Tech Stack Recommendations

**Frontend:**
- Use a simple, modern framework: **React** (with Vite) or **Next.js** for quick setup
- UI Library: **Tailwind CSS** + **shadcn/ui** or **Material-UI** for rapid prototyping
- State Management: **React Context** or **Zustand** for simple state (no Redux needed)

**Backend:**
- **FastAPI** (Python) or **Express.js** (Node.js) - choose based on team preference
- REST API endpoints to handle onboarding workflow
- Load Langflow configuration from JSON file (with embedded API keys)
- Simple in-memory or file-based session storage for prototype (upgrade to PostgreSQL later)

**Langflow Configuration:**
- Configuration provided as JSON file with embedded API keys and flow endpoints
- Backend loads configuration at startup
- No direct Langflow API calls - configuration contains all necessary connection details

## Project Structure

```
app-prototype/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── OnboardingForm.jsx      # Fixed Q&A form
│   │   │   ├── WritingExercise.jsx     # Exercise submission UI
│   │   │   └── ResultsDisplay.jsx      # Assessment results
│   │   ├── services/
│   │   │   └── api.js                  # API client for backend
│   │   ├── context/
│   │   │   └── SessionContext.jsx      # Session state management
│   │   └── App.jsx
│   ├── package.json
│   └── vite.config.js
├── backend/
│   ├── app/
│   │   ├── main.py                     # FastAPI app entry
│   │   ├── routes/
│   │   │   └── onboarding.py           # Onboarding endpoints
│   │   ├── services/
│   │   │   └── config_loader.py        # Load Langflow config from JSON
│   │   └── models/
│   │       └── session.py              # Session data models
│   ├── config/
│   │   └── langflow_config.json        # JSON config with API keys
│   ├── requirements.txt
│   └── .env                            # Environment variables (optional)
└── README.md
```

## Implementation Guidelines

### 1. Frontend UI Components

**OnboardingForm Component:**
- Simple form with fixed questions (lifestyle + mindset)
- No LLM calls - just collect user input
- Submit button triggers Flow 1 call
- Show loading state while waiting for scenario generation

**WritingExercise Component:**
- Display generated scenario and 3 exercise prompts from Flow 1
- Text areas for each exercise (5-10 min writing time)
- Submit button triggers Flow 2 call
- Show loading state during assessment

**ResultsDisplay Component:**
- Display skill assessment results
- Show 3 recommended learning focus areas
- Simple, clean presentation

### 2. Backend API Endpoints

**POST /api/onboarding/scenario**
- Input: `{ lifestyle: string, mindset: string }`
- Uses Langflow Flow 1 configuration (from JSON config)
- Returns: `{ scenario: string, exercises: [string, string, string] }`
- Store session state (in-memory for prototype)

**POST /api/onboarding/assess**
- Input: `{ sessionId: string, exercises: [string, string, string] }`
- Uses Langflow Flow 2 configuration (from JSON config)
- Returns: `{ assessment: object, focusAreas: [string, string, string] }`
- Update session state

**GET /api/session/:sessionId**
- Retrieve current session state
- Used for page refresh recovery

### 3. Langflow Configuration Loading

**Configuration File Structure:**
The backend will receive a JSON configuration file (`config/langflow_config.json`) with embedded API keys and flow details:

```json
{
  "flow_1": {
    "name": "scenario_generation",
    "endpoint": "https://langflow-instance.com/api/v1/run/{flow_id}",
    "flow_id": "scenario-generation-flow-id",
    "api_key": "embedded_api_key_here",
    "input_type": "json",
    "output_type": "json"
  },
  "flow_2": {
    "name": "assessment_plan",
    "endpoint": "https://langflow-instance.com/api/v1/run/{flow_id}",
    "flow_id": "assessment-plan-flow-id",
    "api_key": "embedded_api_key_here",
    "input_type": "json",
    "output_type": "json"
  },
  "langflow_base_url": "https://langflow-instance.com"
}
```

**Config Loader Service:**
```python
# Example structure
import json
from pathlib import Path
from typing import Dict

class LangflowConfigLoader:
    def __init__(self, config_path: str = "config/langflow_config.json"):
        self.config_path = Path(config_path)
        self.config: Dict = {}
        self.load_config()
    
    def load_config(self):
        """Load configuration from JSON file at startup"""
        with open(self.config_path, 'r') as f:
            self.config = json.load(f)
    
    def get_flow_config(self, flow_name: str) -> Dict:
        """Get configuration for a specific flow"""
        return self.config.get(f"flow_{flow_name}", {})
    
    def get_api_key(self, flow_name: str) -> str:
        """Get API key for a specific flow"""
        flow_config = self.get_flow_config(flow_name)
        return flow_config.get("api_key", "")
    
    def get_flow_endpoint(self, flow_name: str) -> str:
        """Get endpoint URL for a specific flow"""
        flow_config = self.get_flow_config(flow_name)
        return flow_config.get("endpoint", "")
```

**Initialization:**
- Load configuration file at application startup
- Store configuration in memory (singleton pattern recommended)
- Validate configuration structure on load
- Handle missing or invalid configuration files gracefully

**Security Notes:**
- Keep `langflow_config.json` in `.gitignore` (never commit API keys)
- Use environment variable for config file path if needed
- Consider encrypting sensitive values in production

### 4. Session Management

**Prototype Approach:**
- Use in-memory dictionary (Python) or Map (Node.js)
- Key: session_id (UUID)
- Value: session data (lifestyle, mindset, exercises, results)
- Add session expiration (e.g., 1 hour)

**Session Data Model:**
```python
{
    "session_id": "uuid",
    "lifestyle": "string",
    "mindset": "string",
    "scenario": "string",
    "exercises": ["string", "string", "string"],
    "exercise_responses": ["string", "string", "string"],
    "assessment": {},
    "focus_areas": ["string", "string", "string"],
    "created_at": "timestamp"
}
```

### 5. UI/UX Guidelines

**Design Principles:**
- Keep it simple and clean
- Focus on functionality over aesthetics for prototype
- Use clear labels and instructions
- Show loading states for all async operations
- Display errors clearly
- Mobile-responsive (basic)

**User Flow:**
1. User lands on onboarding form
2. Fills out lifestyle + mindset questions
3. Clicks "Generate Scenario" → Loading → Shows scenario + exercises
4. User writes responses (5-10 min)
5. Clicks "Submit for Assessment" → Loading → Shows results

**Accessibility:**
- Use semantic HTML
- Add ARIA labels where needed
- Ensure keyboard navigation works
- Test with screen reader (basic)

## Environment Setup

### Frontend Environment Variables

**Create `frontend/.env` file:**
```env
# Backend API URL
VITE_API_URL=http://localhost:8000

# Note: OpenAI API keys should NOT be stored here
# API keys must be stored in backend configuration only
```

**Important:**
- Variables prefixed with `VITE_` are exposed to frontend code
- Access using: `import.meta.env.VITE_API_URL`
- Never store API keys in frontend `.env` files
- Add `.env` to `.gitignore`

### Backend Environment Variables

**Optional Backend `.env` file (`backend/.env`):**
```env
# Backend Server Configuration
PORT=8000

# Langflow Configuration File Path
CONFIG_PATH=config/langflow_config.json
```

### Backend Configuration File

**Create `backend/config/langflow_config.json` with embedded API keys:**
```json
{
  "flow_1": {
    "name": "scenario_generation",
    "endpoint": "https://langflow-instance.com/api/v1/run/{flow_id}",
    "flow_id": "scenario-generation-flow-id",
    "api_key": "your_langflow_api_key_here",
    "openai_api_key": "sk-proj-your-openai-api-key-here",
    "input_type": "json",
    "output_type": "json"
  },
  "flow_2": {
    "name": "assessment_plan",
    "endpoint": "https://langflow-instance.com/api/v1/run/{flow_id}",
    "flow_id": "assessment-plan-flow-id",
    "api_key": "your_langflow_api_key_here",
    "openai_api_key": "sk-proj-your-openai-api-key-here",
    "input_type": "json",
    "output_type": "json"
  },
  "langflow_base_url": "https://langflow-instance.com"
}
```

**Security Notes:**
- **Important:** Add `config/langflow_config.json` to `.gitignore` to prevent committing API keys
- Replace `sk-proj-your-openai-api-key-here` with your actual OpenAI API key
- This file contains sensitive credentials and should never be committed to version control
- Use environment variable for config file path if needed

## Testing Strategy

**Prototype Testing:**
- Manual testing of complete user flow
- Test error scenarios (network failures, invalid inputs, missing config file)
- Test session persistence (page refresh)
- Verify configuration file loads correctly at startup
- Test with provided Langflow configuration JSON file

**No Unit Tests Required:**
- Focus on getting working prototype first
- Add tests in next iteration if needed

## Deployment Considerations

**Local Development:**
- Frontend: `npm run dev` (Vite) or `npm run dev` (Next.js)
- Backend: `uvicorn app.main:app --reload` (FastAPI) or `npm run dev` (Express)

**Simple Deployment Options:**
- Frontend: Vercel or Netlify
- Backend: Railway, Render, or Fly.io
- Ensure CORS is configured correctly

## Next Steps After Prototype

1. Add PostgreSQL for persistent session storage
2. Add proper authentication (if needed)
3. Add input validation and sanitization
4. Add rate limiting
5. Add monitoring/logging
6. Improve error handling
7. Add loading animations
8. Polish UI/UX

## Key Constraints

- **Cost:** Keep API calls minimal (2 LLM calls per user)
- **Latency:** Aim for <10s total response time
- **Privacy:** Don't store PII unnecessarily in prototype
- **Simplicity:** Focus on core functionality, not edge cases

## Langflow Configuration Reference

**Configuration File Location:**
- File: `backend/config/langflow_config.json`
- Loaded at application startup
- Contains all necessary connection details and API keys

**Expected JSON Structure:**
```json
{
  "flow_1": {
    "name": "scenario_generation",
    "endpoint": "https://langflow-instance.com/api/v1/run/{flow_id}",
    "flow_id": "scenario-generation-flow-id",
    "api_key": "embedded_api_key_here",
    "input_type": "json",
    "output_type": "json"
  },
  "flow_2": {
    "name": "assessment_plan",
    "endpoint": "https://langflow-instance.com/api/v1/run/{flow_id}",
    "flow_id": "assessment-plan-flow-id",
    "api_key": "embedded_api_key_here",
    "input_type": "json",
    "output_type": "json"
  },
  "langflow_base_url": "https://langflow-instance.com"
}
```

**Note:** The backend does not make direct Langflow API calls. The configuration file contains all necessary details for connecting to the Langflow pipelines. The actual integration method will depend on how the Langflow pipelines are exposed (could be via API, SDK, or other method as specified in the config).

## Notes

- This is a **prototype** - prioritize working functionality over perfection
- Use TypeScript if team prefers, but JavaScript is fine for prototype
- Keep code simple and readable - refactor later if needed
- Document any Langflow-specific quirks or workarounds discovered
- Load and validate the Langflow configuration JSON file before considering prototype complete
- Ensure the config file path is correctly referenced and the file exists

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Trina-G) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
