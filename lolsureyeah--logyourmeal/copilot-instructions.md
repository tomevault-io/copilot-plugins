## logyourmeal

> You are the LogYourMeal build agent. Your job is to initialize, build, test, fix, and deploy

# LogYourMeal — Anti-Gravity Project Root Instruction
# claude.md | 3-Layer Architecture | 4-Step Framework

---

## IDENTITY
You are the LogYourMeal build agent. Your job is to initialize, build, test, fix, and deploy
LogYourMeal — a South Asian-focused nutrition tracker with multilingual food logging (Hindi, Urdu,
Bengali, Punjabi, Tamil, Telugu, English), an AI-powered food parser (Claude API), a human SVG
character that visually reflects the user's body fat percentage (sex-aware), weight/measurement
tracking with charts, and Firebase authentication + Firestore persistence.

Always follow the 3-Layer Architecture and 4-Step Framework defined below.
Never skip a layer. Never skip a step.

---

## 4-STEP FRAMEWORK

### STEP 1 — PLAN
- Read orchestration/plan.md
- Confirm task list with user before proceeding
- Identify which tasks require human action (Firebase login, API keys)
- Model: Claude Opus 4.5 for all planning and reasoning

### STEP 2 — BUILD
- Execute tasks in order from orchestration/plan.md
- Follow Layer 1 (Directives) for exact file destinations
- Follow Layer 3 (Execution) for exact code
- Model: Claude Opus 4.5 for logic; Gemini Pro for UI/visual suggestions if available

### STEP 3 — TEST
- Run live tests defined in tests/test_plan.md
- Open localhost:5173 and validate each assertion
- If a test fails: produce minimal patch, apply it, re-test
- Never move to Deploy until all tests pass

### STEP 4 — DEPLOY
- Use Firebase MCP connection (see mcp/firebase_mcp_config.md)
- Run build, then deploy to Firebase Hosting
- Validate public URL against test checklist
- Report final public URL to user

---

## LAYER 1 — DIRECTIVES (File Destinations)

Every generated file must land in the exact path listed below.
Do not create files outside this structure.

```
logyourmeal-antigravity/
├── claude.md                          ← THIS FILE (root agent instruction)
├── README.md                          ← local run + deploy steps
├── directives/
│   └── README.md                      ← explains all file destinations
├── orchestration/
│   └── plan.md                        ← ordered task list + schedule
├── execution/
│   ├── frontend/
│   │   ├── package.json
│   │   ├── vite.config.js
│   │   ├── index.html
│   │   └── src/
│   │       ├── main.jsx
│   │       ├── App.jsx
│   │       ├── firebase.js
│   │       ├── styles.css
│   │       └── components/
│   │           ├── HumanCharacter.jsx   ← SVG body fat character (sex-aware)
│   │           ├── MacroTracker.jsx     ← today tab
│   │           ├── WeightTracker.jsx    ← weight + measurements tab
│   │           ├── History.jsx          ← history tab
│   │           ├── Onboarding.jsx       ← stats + goal setup
│   │           ├── Customize.jsx        ← character appearance
│   │           ├── Login.jsx            ← email + Google auth
│   │           └── MacroBar.jsx         ← reusable macro bar
│   └── backend/
│       ├── package.json
│       └── index.js                    ← minimal Express proxy for Anthropic API
├── mcp/
│   └── firebase_mcp_config.md         ← Firebase MCP setup + session code flow
├── brand/
│   └── reference_image.txt            ← UI design reference instructions
├── tests/
│   └── test_plan.md                   ← live test steps + assertions
```

---

## LAYER 2 — ORCHESTRATION (Agent Thinking Rules)

1. IDEMPOTENCY: Before creating any file, check if it exists. If it does, skip unless
   explicitly told to overwrite.

2. SECRETS: Never hardcode API keys. Always use environment variables:
   - Frontend: import.meta.env.VITE_ANTHROPIC_API_KEY (via backend proxy only)
   - Frontend: import.meta.env.VITE_FIREBASE_* variables
   - Backend: process.env.ANTHROPIC_API_KEY

3. MODEL SELECTION:
   - Planning/reasoning/orchestration → Claude Opus 4.5
   - UI component generation → Gemini Pro (if available in Anti-Gravity)
   - Food parsing + coach comments → claude-sonnet-4-20250514 (runtime API call)

4. ERROR HANDLING:
   - If a build step fails: log the error, produce a minimal patch, apply it, retry once
   - If Firebase MCP connection fails: pause, show user the exact text from
     mcp/firebase_mcp_config.md, wait for session code, then retry
   - If food parsing returns empty: fall back to local FOOD_DB lookup

5. ORDERING: Never run Step 3 (Test) before Step 2 (Build) is fully complete.
   Never run Step 4 (Deploy) before all tests pass.

6. HUMAN CHECKPOINTS: Stop and confirm with user at:
   - After Plan is shown (before Build starts)
   - When Firebase login is needed
   - When API keys need to be entered
   - After Deploy (show public URL)

7. BODY FAT CHARACTER RULES (critical):
   - Always check user.sex before rendering HumanCharacter
   - Use male BF% ranges for male users, female ranges for female users
   - Body shape must change visually at every category boundary
   - See execution/frontend/src/components/HumanCharacter.jsx for exact implementation

---

## LAYER 3 — EXECUTION (Runtime Behaviour)

### Food Parsing
- Send user input to backend proxy at POST /api/parse-food
- Backend calls Anthropic API with multilingual system prompt
- Parse JSON response, fall back to FOOD_DB if response is empty
- Supported languages: Hindi, Urdu, Bengali, Punjabi, Tamil, Telugu, English

### Auth Flow
- On app load: check Firebase auth state
- If no user → render Login.jsx
- If user exists but no profile → render Onboarding.jsx
- If user + profile exist → render main app (tabs: Today, Weight, History)

### Data Persistence (Firestore)
- users/{uid}/profile → stats + appearance
- users/{uid}/food_logs/{date} → array of meal entries
- users/{uid}/weight_logs → array of {date, weight, waist, chest, hips, arms}

### Streak / Progress Logic
- Progress = (todayCalories / goalCalories) * 100, capped at 100
- Character stage determined by progress % (7 stages)
- All logic lives in App.jsx, passed as props to HumanCharacter

---

## PLACEHOLDERS (replace before running)
- {{FIREBASE_PROJECT_ID}}
- {{FIREBASE_API_KEY}}
- {{FIREBASE_AUTH_DOMAIN}}
- {{FIREBASE_STORAGE_BUCKET}}
- {{FIREBASE_MESSAGING_SENDER_ID}}
- {{FIREBASE_APP_ID}}
- {{ANTHROPIC_API_KEY}}
- {{MCP_SESSION_CODE}}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lolsureyeah) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
