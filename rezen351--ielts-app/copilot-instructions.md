## ielts-app

> You are an expert **Senior Full-stack Developer** and **Certified IELTS Examiner**. You are assisting a Physics Engineering student in building a high-performance, automated IELTS training platform.

#Project Context: IELTS Prep App (Next.js + Azure AI)

You are an expert **Senior Full-stack Developer** and **Certified IELTS Examiner**. You are assisting a Physics Engineering student in building a high-performance, automated IELTS training platform.

## Technical Stack
- **Framework:** Next.js (App Router), TypeScript, Tailwind CSS.
- **Backend/Hosting:** Azure Static Web Apps (Hybrid Mode).
- **Database:** Azure Cosmos DB (MongoDB API) with Mongoose.
- **AI Integration:** - Azure OpenAI (GPT-4o-mini) for evaluation.
  - Azure AI Speech for TTS/STT.
  - Azure AI Translator for multilingual support.
- **Environment:** WSL (Ubuntu) on Windows.

## Workflow & Consistency (STRICT)
1. **The TODO Protocol:** Before starting any task or generating code, ALWAYS check the `TODO.md` file. 
   - If there are unfinished tasks, prioritize completing them before moving to new features.
   - Update the `TODO.md` immediately after a task is completed or if a new sub-task is discovered.
2. **Context Continuity:** Ensure every new piece of code is aware of previous implementations to maintain a unified architecture.

## Coding Standards & Effectiveness
- **Clean Code (SOLID):** Functions must be small, single-responsibility, and easy to test.
- **Zero Waste Policy:** Strictly avoid and remove "dead code". 
   - No unused imports, variables, or functions.
   - If a refactor makes a previous function obsolete, delete it immediately.
- **Security First:** Never hardcode API Keys. Always use `process.env`.
- **Serverless Optimization:** Database connections must use a global cached pattern (singleton).
- **Strict Types:** No `any`. Use TypeScript interfaces for everything.

## IELTS Accuracy
- Evaluation logic must strictly follow the official IELTS Band Descriptors (1.0 - 9.0).
- Focus on: Lexical Resource, Grammatical Range, Coherence, and Pronunciation.

## Automation & Post-Fix
- **Verification:** After every fix, verify the build (`npm run build`).
- **Git Protocol:** After a successful build, provide instructions to `git add`, `commit`, and `push`.

### [2026-03-26]
- **Feature Proposal (Milestone Gatekeeper):**
  - **Concept:** Implementation of a "Milestone Assessment" layer between roadmap levels.
  - **Logic:** Before advancing to a new learning level, users must pass a comprehensive diagnostic test (Milestone Quiz) covering previous topics.
  - **Adaptive Branching:** 
    - **Pass (>= 80%):** Unlock next level + Generate new content.
    - **Fail (< 80%):** AI identifies specific weaknesses and generates a "Remedial Deep Dive" module (e.g., focused Grammar/Vocab) instead of the standard next level.
  - **Technical Architecture:** Planned updates to `Learning` model (status: `milestone`, `remedial`) and new evaluation APIs for performance analysis.
- **Workflow:** Updated `TODO.md` with milestone-related tasks.
- **Status:** Planning phase completed. Ready for implementation.

### [2026-03-25]
- **UI/UX (Responsiveness & 360px Optimization):**
  - **Mobile-First Refinement:** Optimized all core dashboards (Main, Learning, Examiner) for 360px device width (iPhone SE standard).
  - **Dynamic Card History:** Replaced rigid tables with a responsive card-based layout for test results on mobile devices.
  - **Minimalist Practice Interface:** Redesigned practice headers to remove bulky titles, focusing on core session controls (Timer, Submit, Navigation).
  - **Typography & Inputs:** Calibrated font sizes and input heights across `QuestionCard` and auth pages for better mobile ergonomics.
- **Feature (Technical AI Evaluation):**
  - **Deep Linguistic Feedback:** Enhanced AI evaluation prompts to provide granular technical breakdowns for Grammar (GRA) and Lexical Resource (LR).
  - **Standardized Corrections:** Implemented a new feedback format: `Current -> Better (Rule/Reason)` for grammar and `Word -> Advanced Synonym (Example)` for vocabulary.
  - **Technical Insights:** Refocused the "English Check" dashboard tool on advanced technical points (inversion, complex tenses) with mandatory usage examples.
- **Architectural Fix (Data Consistency):**
  - **Database Source of Truth:** Migrated user profile fetching (native language, goal band, etc.) from `localStorage` to a centralized Database API (`GET /api/user/settings`).
  - **Real-time Chat Sync:** Ensured the Chat Tutor always uses the latest database-stored language preferences for its translation engine.
- **Navigation Logic:**
  - **Practice Hub Centricity:** Updated all practice module exit/completion flows to redirect users to the **Practice Hub** (`/dashboard/practice`) instead of the learning roadmap, creating a more logical training loop.

### [2026-03-22]
- **Feature (Learning Module):**
  - **IELTS AI Tutor:** Implemented a personalized learning ecosystem at `/dashboard/learning`.
  - **Adaptive Diagnostic Assessment:** New users undergo a 5-question diagnostic test to determine their English baseline. AI analyzes the results to skip known topics and focus on weaknesses.
  - **Skill-based Roadmap:** Generates a personalized learning path (e.g., Cohesion, Grammar Range) based on the user's current band and target goal.
  - **AI-Curated Blog Materials:** Generates high-quality, blog-style articles for each topic, including:
    - **Mini-Explainers:** AI "did you know" sidebars for quick tips.
    - **Quick Checks:** Interactive quizzes with instant feedback and explanations.
    - **Smart Flashcards:** Flip-cards for key terminology and concepts.
  - **Context-Aware Tutor:** Integrated the Chat Tutor to automatically receive the context of the current learning material, allowing for immediate and relevant assistance.
- **Feature (Security):**
  - **Change Password:** Implemented a secure password change system in the Settings page. This includes a dedicated API route (`/api/user/settings/change-password`) with `bcrypt` verification for current passwords and strict validation for new passwords.
  - **Security UI:** Added a new "Security" card in the dashboard settings with intuitive fields for current password, new password, and password confirmation.
- **Feature (Persistence & Content Reuse):**
...
  - **Recent Practices Dashboard:** Implemented a "Recent Practices" section in Listening, Reading, Writing, and Speaking modules. This allows users to pick up where they left off or revisit previously generated AI content, saving costs and token usage.
  - **Pagination Engine:** Developed a client-side pagination system (5 items per page) for practice history to maintain a clean UI even with large amounts of saved content.
  - **Atomic Content Retrieval:** Updated the content generation API to support `GET` requests for specific module practices, sorted by the most recent.
- **Feature (AI Learning Insights):**
  - **English Vibe Check ⚡:** Created a high-engagement dashboard feature that generates instant English learning tips (vocabulary, grammar, strategies) using Gen Z slang for a modern, relatable experience.
  - **Idiomatic Localization:** Integrated a "Native Mode" that provides idiomatic translations (e.g., natural Indonesian slang) rather than literal translations, making the AI feel like a real native tutor.
  - **On-Demand Refresh:** Implemented a real-time refresh mechanism allowing users to get new insights instantly without reloading the entire dashboard.
- **Fix (Stability):**
  - **Import Resolution:** Fixed `ReferenceError` related to missing `ChevronRight` icons across multiple dashboard pages.
  - **API Efficiency:** Unified the Insight API to return both English and Translated versions in a single call, reducing latency and external translator dependency.

### [2026-03-20]
- **Robust Generation Engine (v2.0):**
  - **Self-Correcting Loop:** Implemented a sophisticated dialogue loop in `generateWithRetry`. If the AI returns malformed or incomplete data, the system now provides specific feedback and forces the model to correct itself (up to 10 attempts).
  - **Adaptive Intelligence:** Automatically lowers model "temperature" during retries to increase deterministic accuracy for complex JSON structures.
  - **Deep Validation:** Added a programmatic quality control layer that strictly enforces IELTS standards (e.g., minimum option counts, mandatory gap-fill placeholders `__________`, and answer-key alignment).
  - **Module-Specific Logic:** Calibrated validators to distinguish between "Section" modules (Listening/Reading) and "Task/Part" modules (Writing/Speaking), preventing false-positive rejections.
- **Feature (Incremental Persistence):** 
  - **Atomic Saves:** Implemented immediate database commits after every successful section generation. If a full test generation is interrupted, all previous work is safely stored, preventing token waste and allowing seamless "Resume" functionality.
- **UI/UX (Professional Examiner Suite):**
  - **Multi-Input Detection:** Developed an intelligent gap-fill system that auto-detects multiple blanks in a single question, labels them `(1)`, `(2)`, etc., and renders independent input fields for a professional computer-delivered exam experience.
  - **Speaking Part 2 (Cue Card):** Redesigned the Speaking interface to include a high-fidelity "Cue Card" display with clear Topic and Bullet Point guidance, matching official IELTS Part 2 standards.
  - **Interactive Feedback:** Enhanced the recording UI with pulsating "Listening..." states and "Captured" badges for better user confirmation.
- **Fix (Stability):** 
  - **React Render Protection:** Implemented strict string-type enforcement for question options to prevent "Objects are not valid as a React child" crashes caused by inconsistent AI outputs.
  - **Universal Field Mapping:** Updated frontend rendering to be field-agnostic, ensuring questions display correctly whether the AI uses `question`, `text`, or raw string formats.
- **Status:** All generation, UI, and interactivity systems are verified and stable. **In Progress:** Refining the scoring and evaluation logic.

### [2026-03-18]
- **Fix:** Implemented lazy-loading singleton pattern for Azure OpenAI client to resolve `Missing credentials` build errors in CI/CD (Azure Static Web Apps/Oryx).
- **Fix:** Resolved `TypeError` in `examiner/route.ts` caused by uninitialized nested objects in writing/speaking generation.
- **Feature:** Implemented **Hybrid Model Strategy**. Using GPT-4o for high-quality structure/prompts and GPT-4o-mini for cost-effective mass question generation.
- **Optimization:** Added model selection logic in `azure-openai.ts` and API routes.
- **Workflow:** Updated `GEMINI.md` with strict build-and-push rules.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rezen351)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Rezen351)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
