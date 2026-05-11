## 11plus-tutor

> This file provides guidance to Claude Code when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

**ExamTutor** - An AI-powered learning platform for UK students preparing for 11+ entrance exams and GCSEs. Built on the architecture of DeepTutor/AIEducator.

## Architecture

- **Backend**: Python 3.12+, FastAPI (port 8001)
- **Frontend**: Next.js, React, TailwindCSS (port 3782)
- **RAG Engine**: LightRAG for knowledge graph-based retrieval
- **LLM Support**: OpenAI, Ollama, LM Studio
- **Content API**: Oak National Academy (Open Government License)

### Key Directories

```
src/
├── api/            # FastAPI backend endpoints
├── agents/         # AI agent implementations (explain, practice, mock_exam)
├── knowledge/      # RAG and curriculum knowledge base
├── question_bank/  # Question storage and retrieval
├── progress/       # Student progress tracking
├── tools/          # Tools for agents
└── core/           # Core utilities and configuration

web/
├── app/            # Next.js app router pages
├── components/     # React components
├── context/        # React context providers
└── lib/            # Utility functions

data/
├── curriculum/     # National curriculum documents
├── questions/      # Question bank data
└── oak_content/    # Cached Oak Academy content

config/
├── main.yaml       # Server configuration
├── agents.yaml     # Agent configurations
└── exams.yaml      # Exam board specifications
```

## Development Commands

### Setup
```bash
# Create Python virtual environment
python3.12 -m venv venv
source venv/bin/activate

# Install Python dependencies
pip install -r requirements.txt

# Install frontend dependencies
cd web && npm install
```

### Running
```bash
source venv/bin/activate
python scripts/start_web.py

# Frontend: http://localhost:3782
# Backend: http://localhost:8001/docs
```

### Testing
```bash
pytest tests/
cd web && npm run lint
```

## Key APIs

### Oak National Academy
- Base URL: https://open-api.thenational.academy/
- License: Open Government License
- Used for: Curriculum-aligned lessons, videos, quizzes

### Exam Board Past Papers
- Link to official sources, don't host
- AQA: https://www.aqa.org.uk/find-past-papers-and-mark-schemes
- Edexcel: https://qualifications.pearson.com/en/support/support-topics/exams/past-papers.html

## Content Strategy

1. **Primary**: Oak API content (free, legal)
2. **Secondary**: AI-generated questions based on patterns
3. **Tertiary**: Links to official exam board resources

## Exam Types Supported

### 11+ (Year 5-6, ages 10-11)
- GL Assessment (most common)
- CEM (Durham University)
- Subjects: Verbal Reasoning, Non-Verbal Reasoning, Maths, English

### GCSEs (Year 10-11, ages 14-16)
- AQA, Edexcel, OCR, WJEC
- Core: English, Maths, Science
- Options: History, Geography, Languages, etc.

## Key Files

- `PLAN.md` - Comprehensive project plan and research
- `settings.py` - Pydantic settings for configuration
- `config/exams.yaml` - Exam specifications and question types

## 🐛 PRIORITY FIXES (from user testing, March 2026)

These bugs were reported by Jon after real-world testing. Fix all 4:

### 1. Repetitive questions in practice tests
**Problem**: The same question appears multiple times within a single practice test.
**Fix**: Track question IDs/hashes within a test session. Before presenting a question, check it hasn't already been shown. Use a Set of question hashes (question text + options) to deduplicate.
**Files**: `src/agents/practice_agent.py` — `generate_question()` needs a session-level dedup mechanism.

### 2. Code word tests are too easy
**Problem**: Code word questions have answers that are too obvious because each coded word starts with different letters. Need more distractors that share the same first 1-2 coded letters.
**Fix**: In `_build_vr_prompt()` for `code_words` type, update the generation prompt to explicitly require:
- Multiple answer options must share the same first 1-2 coded letters
- Distractors should be plausible (only differ by 1-2 letters from correct answer)
- Add instruction: "Make sure at least 2-3 options start with the same coded letters to increase difficulty"
**Files**: `src/agents/practice_agent.py` — `_build_vr_prompt()`, the `code_words` example/instruction section.

### 3. Letter line shown for wrong question types
**Problem**: The alphabet/letter line helper is displayed for question types that don't need it (should only show for code words, letter sequences, etc.)
**Fix**: Only render the letter line component when `question_type` is in a whitelist: `['code_words', 'letter_codes', 'letter_sequences', 'code_breakers']`. Check both the backend response and frontend rendering.
**Files**: Frontend — check `web/app/practice/page.tsx` and related components for letter line rendering logic.

### 4. Questions repeating across sessions
**Problem**: Same questions keep coming back across different practice sessions.
**Fix**: Store a history of recently-seen question hashes (last 200-500) in the student progress/session state. Exclude these when generating or selecting new questions. If the question bank is small for a type, allow repeats only after all questions have been seen.
**Files**: `src/progress/` for tracking, `src/agents/practice_agent.py` for filtering.

---
> Source: [jhammant/11plus-tutor](https://github.com/jhammant/11plus-tutor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
