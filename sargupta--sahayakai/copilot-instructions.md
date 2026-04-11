## sahayakai

> **Status:** Blueprint Phase (B.L.A.S.T. Protocol)

# SahayakAI Project Map (gemini.md)

## Project Overview
**Status:** Blueprint Phase (B.L.A.S.T. Protocol)
**Last Updated:** 2026-01-29
**Project Identity:** "Bharat First" AI Teaching Assistant for Rural India.

## B.L.A.S.T. Protocol Progress
- [x] **B - Blueprint:** Discovery synthesized from Strategic Review & Roadmap
- [x] **L - Link:** Verified Gemini/Genkit connectivity via `tools/verify_link.ts`
- [x] **A - Architect:** Comprehensive Ecosystem SOPs roll-out (Master, Interactive, Support, Lesson Plan) - Layer 1 Complete.
- [ ] **S - Stylize:** Premium "Bharat" UX/UI Refinement (In Progress: Added Copy button to Instant Answer card)
- [ ] **T - Trigger:** Automation & Deployment triggers

## Discovery Questions (Synthesized)
*   **North Star:** Transform SahayakAI from a single-user tool to a nationwide ecosystem platform. Immediate priority: Completing the "Indian Context" transformation and establishing a "Semantic Cache" for sustainability.
*   **Integrations:** 
    *   **Core:** Google Gemini (via Genkit), Firebase (Hosting/Functions/Auth).
    *   **Priority:** Semantic Caching layer (to be built), Supabase/Realtime DB migration for curriculum mapping.
    *   **Future:** Student/Parent communication (WhatsApp/SMS).
*   **Source of Truth:** 
    *   Current: `src/lib/indian-context.ts` (Context Data), `src/ai/flows/` (AI Prompts).
    *   Technical: `STRATEGIC_REVIEW.md` and `RURAL_INDIA_ROADMAP.md` as policy guides.
*   **Delivery Payload:** Structured, culturally relevant lesson plans, localized UI (10+ languages), and offline-ready PWA assets.
*   **Behavioral Rules:** 
    *   **The "Bharat-First" Constraint:** Mandatory use of Indian rural contexts (Agriculture, local geography, zero-cost resources).
    *   **Resource Awareness:** Default assumption is "Chalk & Blackboard" environment.
    *   **Tone:** Supportive, pedagogical, and highly accessible (voice-first).
    *   **Reliability:** Deterministic business logic must wrap LLM outputs.
    *   **Planning Phase Constraint:** MUST prepare a plan (`implementation_plan.md`) and divide work into tasks (`task.md`) before executing any implementation changes. Without doing this, do not move forward.

## Data Schema (Lesson Plan Payload)
```json
{
  "title": "string",
  "gradeLevel": "string (Required)",
  "duration": "string (Required)",
  "subject": "string (Required)",
  "objectives": ["string"],
  "keyVocabulary": [{"term": "string", "meaning": "string"}],
  "materials": ["string"],
  "activities": [
    {
      "phase": "Engage | Explore | Explain | Elaborate | Evaluate",
      "name": "string",
      "description": "string",
      "duration": "string",
      "teacherTips": "string (Optional)",
      "understandingCheck": "string (Optional)"
    }
  ],
  "assessment": "string",
  "homework": "string (Optional)"
}
```

## Data Schema (Teacher Training Payload)
```json
{
  "introduction": "string (Required) - Empathetic acknowledgment of the teacher's concern",
  "advice": [
    {
      "strategy": "string (Required) - Actionable classroom technique",
      "pedagogy": "string (Required) - Core pedagogical principle name (e.g., 'Scaffolding', 'Zone of Proximal Development')",
      "explanation": "string (Required) - Simple explanation with Indian context analogy"
    }
  ],
  "conclusion": "string (Required) - Motivational closing statement"
}
```

}
```

## Data Schema (Quiz Payload)
```json
{
  "title": "string (Required)",
  "questions": [
    {
      "questionText": "string (Required)",
      "questionType": "multiple_choice | fill_in_the_blanks | short_answer | true_false",
      "options": ["string"],
      "correctAnswer": "string (Required)",
      "explanation": "string (Required) - Pedagogical explanation tied to Bharat context",
      "difficultyLevel": "easy | medium | hard"
    }
  ],
  "teacherInstructions": "string (Optional) - Classroom management tips",
  "gradeLevel": "string (Optional)",
  "subject": "string (Optional)"
}
```

## Architectural Boundaries
- **Logic Layer:** All business rules (NCERT mapping, Bharat-context injection, pedagogical grounding) happen in the AI System Prompt guided by `architecture/` SOPs.
- **Validation Layer:** Deterministic checks for "Westernisms", metadata completeness, and mandatory pedagogical citations.

## Git Standards & Branching Rules

### Branch Strategy
- **`main`** — production branch. Auto-deploys to Cloud Run (`sahayakai-hotfix-resilience`) on push. **NEVER commit directly to main.**
- **`feature/<name>`** — new features (e.g., `feature/jarvis-omniorb`, `feature/chat-phase1`)
- **`fix/<name>`** — bug fixes and hotfixes (e.g., `fix/vft-audio-input`, `fix/tts-auth`)
- **`chore/<name>`** — tooling, deps, config (e.g., `chore/update-deps`)

### Workflow (mandatory)
```
# 1. Always branch off main
git checkout main && git pull origin main
git checkout -b fix/<descriptive-name>

# 2. Make changes, commit with conventional commits
git add <specific files>   # never git add -A or git add .
git commit -m "fix(scope): description"

# 3. Merge to main via --no-ff so branch is visible in history
git checkout main
git merge fix/<name> --no-ff -m "merge(fix): <description>"
git push origin main

# 4. Clean up
git branch -d fix/<name>
```

### Commit Message Convention (Conventional Commits)
```
<type>(<scope>): <short description>

Types: feat | fix | chore | docs | refactor | test | perf | merge
Scope: cost | tts | vft | quiz | auth | ci | deps | soul | ...

Examples:
  fix(tts): add Authorization header for prod middleware
  feat(chat): add Firestore real-time message listener
  fix(cost): remove unnecessary grounding from lesson-plan
  chore(deps): upgrade firebase-admin to 13.x
  merge(fix): vft audio input + cross-page context resolution
```

### Rules Claude Must Follow
1. **Never work directly on `main`.** Always create a `fix/` or `feature/` branch first.
2. **Never `git add -A` or `git add .`** — stage specific files only (avoid committing .env, secrets, large binaries).
3. **Never `git push --force`** to main.
4. **Never `--no-verify`** (skip hooks) unless explicitly asked.
5. **Merge with `--no-ff`** so branch history is preserved in main's graph.
6. **Always confirm before `git push`** unless the user says "you have all permissions, deploy".
7. **Co-author every commit** with `Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>`.

### Deployment
- Deployment = `git push origin main`
- Firebase App Hosting auto-deploys on every push to main
- Service: `sahayakai-hotfix-resilience` (Cloud Run, asia-south1)

## Maintenance Log
*   **2026-01-29:** Project initialized. Discovery Questions answered via strategic analysis. Handshake verified Gemini API link. Created `architecture/lesson_plan_generation_sop.md`.
*   **2026-01-29:** Commenced **Phase 4: S - Stylize**. Added Copy button and improved layout for the Instant Answer component on the homepage.
*   **2026-01-29:** **Schema First Pattern Established** - Defined Teacher Training payload schema in `gemini.md`, created `validate_teacher_training.ts` validator, and integrated it into the flow. Tests confirm deterministic checks for empathy, pedagogy citations, and encouragement.
*   **2026-01-29:** **Agentic Skills Deployed** - Created `senior-software-engineer` and `scrum-master` skills. These enforce "Business Serious" operating protocols for Code Architecture and Project Governance.
*   **2026-01-29:** **Skills Upgraded to Expert Level** - Enhanced skills with "Phased Workflows", bundled Reference Templates (ADR, Checklists, Standups), and Python Automation Scripts (`scaffold_test.py`, `generate_status_report.py`).
*   **2026-01-29:** **Micro-Lesson Generator DEPLOYED (Vertical Slice 1)** - Completed Logic, UI, and PDF Export.
    *   **Architecture:** Enforced 5-slide rule via Validator.
    *   **Quality:** Passed Senior Engineer Audit (Complexity Check) and Vitest functional tests.
    *   **Feature:** Client-side PDF generation producing 5-slide, high-contrast decks.
*   **2026-03-09:** **BAKTA Persona & OmniOrb Consolidation** - Injected Mr. Abhishek Gupta's blunt, truth-speaking persona into the AI voice. Consolidated OmniOrb microphones and resolved domain mapping conflicts for `sargupta.in`.
*   **2026-03-04:** **Infrastructure & UX Hardening** - Fixed TTS Authorization ("Unauthorized" error) and implemented audio priming. Refactored "Grade" to "Class" terminology for Bharat-First localization.
*   **Next Step:** Apply Schema First methodology to remaining services (Quiz, Worksheet, Field Trip) and refine the UI to match the data contracts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sargupta)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sargupta)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
