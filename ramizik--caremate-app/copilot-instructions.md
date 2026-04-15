## caremate-app

> **Project:** CareMate - AI Caregiver Burnout Prevention

# GEMINI.md - CareMate Hackathon Guide

**Project:** CareMate - AI Caregiver Burnout Prevention  
**Context:** 24-hour hackathon build  
**Your Role:** Research, analysis, and AI integration design assistant

> **Note:** For complete project details (tech stack, API contracts, file structure, testing), see **AGENTS.md**. This file covers gemini-cli-specific use cases only.

---

## 🎯 Quick Project Context

**Problem:** Unpaid caregivers (esp. women) burn out silently. 73% report severe stress, but no quantitative early warning system exists.

**Solution:** Mobile app that detects burnout through wearable data (HRV, sleep), AI scoring, and generates micro-interventions with delegation automation.

**Tech Stack:** Expo (React Native), FastAPI, Gemini 1.5 Pro, Twilio, Supabase, fitparse  
**Demo Personas:** Sarah (score 84/100 - Critical), Maria (52 - Moderate), Emma (24 - Managing)  
**Critical Demo Flow:** Profile → Upload sarah.fit → HRV 12ms detected → Score spikes 38→84 → AI interventions → SMS delegation

**See AGENTS.md for:** Full tech stack, API endpoints, TypeScript interfaces, testing requirements, tier roadmap.

---

## 🔧 When to Use Gemini CLI

### Use Gemini CLI For:

**AI Integration Design**
- Prompt engineering for intervention generation
- Testing Gemini 1.5 Pro response quality
- Validating delegation message tone/clarity
- Designing multimodal prompts (FIT data + transcript + context)

**Data Analysis & Validation**
- Burnout scoring formula validation (see below)
- HRV threshold research (normal: 20-100ms, critical: <15ms)
- Statistical analysis of biomarker correlations
- Mock data generation for testing

**Research Tasks**
- Caregiver burnout studies (citations for pitch)
- HRV/sleep deprivation research
- Twilio SMS best practices
- Expo/React Native library recommendations

**Multimodal Analysis**
- FIT file format deep dive
- Reviewing architecture diagrams
- Analyzing wearable data patterns
- Cross-referencing multiple project docs

**Long Context Reasoning**
- Analyzing entire codebase structure (1M token window)
- Cross-referencing AGENTS.md + PROJECT_GOALS.md + ROADMAP.md + MVP-Tier-List.md
- Identifying inconsistencies across docs
- Refactoring suggestions across multiple files

### Use Claude Code (Cursor) For:
- Day-to-day coding, debugging, refactoring
- Component implementation
- API endpoint creation
- Bug fixes during development

---

## 🤖 Gemini-Specific Use Cases for CareMate

### 1. AI Intervention Prompt Design

**Task:** Generate 3-5 micro-interventions from burnout score components.

**Example command:**
```bash
gemini "Design a system prompt for generating caregiver burnout interventions. Input: burnout score 84, HRV 12ms, sleep 3.5h, 2 kids + 1 elder care. Output: 3-5 interventions with problem/solution/impact/time_saved. Focus on delegation to spouse, sister, neighbor." @PROJECT_GOALS.md
```

**Guidelines for intervention prompts:**
- **Problem:** Quantitative + health impact (e.g., "HRV 12ms indicates extreme stress, 5x cardiac risk")
- **Solution:** Specific, actionable (e.g., "Ask spouse to handle meal prep Mon/Wed/Fri")
- **Impact:** Time saved + quality of life (e.g., "Saves 2.5h/week, reduces decision fatigue")
- **Difficulty:** Easy (ask family) < Moderate (schedule change) < Hard (hire help)
- **Action:** draft_delegation | link_resource | block_calendar | reschedule_calendar

**Test with Sarah's profile:** HRV 12ms, sleep 3.5h, caring for 2 kids + 1 elder parent

### 2. Burnout Scoring Formula Validation

**Current formula (from AGENTS.md):**
```
score = (wearable_score * 0.40) + (calendar_task_score * 0.30) + 
        (semantic_score * 0.20) + (self_report_score * 0.10)

wearable_score = f(HRV, sleep, steps, stress)
```

**Analysis tasks:**
```bash
# Validate HRV thresholds
gemini "Research HRV (RMSSD) normal ranges for women 40-50. What HRV value indicates chronic stress? Provide citations."

# Test formula edge cases
gemini "Given burnout formula in AGENTS.md, test edge cases: (1) HRV=12ms, sleep=3.5h, no other data (2) All biomarkers normal but calendar shows 80h caregiving/week. Should weights change?" @AGENTS.md

# Statistical validation
gemini "Analyze correlation between HRV drop and burnout risk. If Sarah's HRV dropped from 45ms to 12ms in 2 weeks, how urgent is intervention?"
```

### 3. FIT File Data Extraction Design

**Task:** Validate fitparse library extracts correct biomarkers.

```bash
# Research FIT file structure
gemini "Explain Garmin FIT file format. Which data messages contain HRV (RMSSD), sleep hours, steps, stress score? Provide field names."

# Validate extraction logic
gemini "Review this FIT parsing code. Does it correctly extract HRV RMSSD vs HRV SDNN? Are sleep stages being summed correctly?" @backend/services/fit_parser.py
```

**Key biomarkers to extract:**
- `hrv_rmssd` (NOT hrv_sdnn) - Root mean square of successive differences
- `sleep_hours` - Total sleep (deep + light + REM)
- `steps` - Daily step count
- `stress_score` - Garmin proprietary stress (0-100)
- `resting_heart_rate` - Morning RHR

### 4. Multimodal Context Reasoning

**Scenario:** Sarah uploads FIT file + records voice memo saying "I'm fine, just tired."

```bash
gemini "Analyze dissonance: FIT shows HRV 12ms (critical), voice transcript says 'I'm fine'. What intervention tone should AI use? Draft message." @AGENTS.md @MOCK_DATA.md
```

### 5. Research Tasks for Pitch/Demo

```bash
# Statistics for pitch
gemini "Find 3 credible statistics on unpaid caregiver burnout, focus on women 40-55. Include sources."

# Competitive analysis
gemini "Compare CareMate approach to existing caregiver apps (Care.com, Caring Bridge). What's unique about wearable + AI scoring?"

# HRV research
gemini "Summarize relationship between HRV and chronic stress. What HRV drop % warrants urgent intervention?"
```

---

## 📊 Data Analysis Commands

### Validate Mock Persona Scores

**Sarah (Critical - 84/100):**
- HRV: 12ms (critical, normal 20-100)
- Sleep: 3.5h (critical, need 7-9)
- Care load: 2 kids + 1 elder
- Expected score: 80-90 range

```bash
gemini "Given Sarah's biomarkers in MOCK_DATA, validate burnout score 84 is accurate using formula in AGENTS.md. Show calculation." @AGENTS.md @constants/mockData.ts
```

### Generate Additional Test Personas

```bash
gemini "Create mock persona: Maria, 45, single mom, 3 kids, works full-time. HRV 18ms, sleep 5.5h. Calculate expected burnout score and generate 3 interventions." @AGENTS.md
```

---

## 🎨 Prompt Engineering for Interventions

**System prompt template (for Gemini 1.5 Pro in backend):**

```
You are a compassionate AI caregiver coach. Analyze burnout data and generate 3-5 micro-interventions.

Input Format:
{
  "profile": { "name": "Sarah", "age": 48, "care_type": ["kids", "elder"], "contacts": {...} },
  "score": 84,
  "biomarkers": { "hrvrmssd": 12, "sleephours": 3.5, "steps": 3200 },
  "calendar_load": "60h caregiving/week"
}

Output Requirements:
1. Problem: State biomarker + health impact (e.g., "HRV 12ms = 5x cardiac risk")
2. Solution: Specific action with contact (e.g., "Ask spouse to handle meal prep Mon/Wed/Fri")
3. Impact: Time saved + quality improvement (e.g., "Saves 2.5h/week, reduces decision fatigue")
4. Difficulty: Easy | Moderate | Hard
5. Action: draft_delegation | link_resource | block_calendar

Tone: Empathetic but urgent. Quantify risks. Focus on delegation to existing contacts first.

Generate interventions:
```

**Test this prompt:**
```bash
gemini "Use above prompt template to generate interventions for Sarah (score 84, HRV 12ms, sleep 3.5h). Output JSON matching Intervention interface from AGENTS.md." @AGENTS.md
```

---

## 🔍 Architecture Review Commands

```bash
# Validate API flow
gemini "Review CareMate architecture. Is there a race condition between FIT upload and score recalculation? Trace data flow." @AGENTS.md @backend/

# Security check
gemini "Review authentication flow. Are FIT files stored securely? Is Twilio SMS vulnerable to injection?" @AGENTS.md

# Performance analysis
gemini "Identify bottlenecks in critical path: Upload FIT → Parse → Calculate score → Generate interventions. What takes longest?" @AGENTS.md
```

---

## 📚 Cross-Document Analysis

**Find inconsistencies:**
```bash
gemini "Compare burnout scoring formula across AGENTS.md, PROJECT_GOALS.md, and backend code. Are weights consistent?" @AGENTS.md @PROJECT_GOALS.md @backend/services/scoring.py
```

**Validate tier scope:**
```bash
gemini "Is Tier 2 feature set achievable in 6 hours based on ROADMAP.md? Identify risks." @ROADMAP.md @MVP-Tier-List.md
```

---

## 🚨 Demo Risk Analysis

**1 day before hackathon:**
```bash
gemini "Review CareMate demo flow. Identify 3 highest-risk failure points. Propose fallbacks for each." @AGENTS.md @ROADMAP.md
```

**During testing:**
```bash
gemini "FIT upload failed 3 times on iOS. Parse error logs and suggest fixes." @logs/fit_upload_errors.log @backend/services/fit_parser.py
```

---

## 💡 Usage Tips

**Load multiple files with @:**
```bash
gemini "your question" @AGENTS.md @PROJECT_GOALS.md @ROADMAP.md
```

**For code review, use file paths:**
```bash
gemini "review this FIT parser" @backend/services/fit_parser.py
```

**For long analysis, use `-o output.md` to save:**
```bash
gemini "analyze entire codebase architecture" @. -o architecture_analysis.md
```

**For JSON output:**
```bash
gemini "generate 5 test interventions as JSON array" @AGENTS.md --json
```

---

## ✅ Acceptance Criteria for AI Components

**Intervention Generation (Gemini 1.5 Pro):**
- ✅ Generates 3-5 interventions in <5 seconds
- ✅ Includes quantitative health impacts
- ✅ Drafts delegation messages for existing contacts
- ✅ Matches Intervention TypeScript interface
- ✅ Tone is empathetic but urgent

**Delegation Message Quality:**
- ✅ Specific ask (not vague "help me")
- ✅ Time-bounded (Mon/Wed/Fri, this week)
- ✅ Quid pro quo offered (I'll do groceries)
- ✅ Warm tone (emoji okay, not formal)
- ✅ <160 chars for SMS compatibility

**Voice Check-In (Gemini STT + Mood):**
- ✅ Transcribes 30s audio in <3 seconds
- ✅ Extracts 3-5 mood keywords (stressed, tired, overwhelmed)
- ✅ Detects dissonance (says "fine" but voice analysis shows stress)

---

**Remember:** Gemini CLI is your research and analysis tool. For actual coding, switch to Claude Code in Cursor IDE. See AGENTS.md for full development workflow.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ramizik)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ramizik)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
