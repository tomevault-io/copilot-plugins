## sn-cis-df-practice

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This folder contains study materials for the **ServiceNow CIS-DF (Certified Implementation Specialist - Data Foundations)** certification exam covering CMDB and CSDM.

## Folder Structure

```
CIS - Data Foundation - Learning and Practice/
├── docs/                              # Documentation & study materials
│   ├── servicenow/
│   │   ├── pdf/                       # Original ServiceNow PDFs
│   │   └── markdown/                  # AI-readable markdown versions
│   ├── courses/                       # LinkedIn Learning courses
│   ├── ServiceNow CIS-DF...pdf        # Official exam blueprint
│   ├── INTERACTIVE_QUIZ_GUIDE.md      # Quiz system documentation
│   └── requirements.md                # Project requirements
│
├── practice/                          # Practice exam system
│   ├── app/                           # Web app (exam HTML files)
│   ├── results/                       # Completed quiz results
│   └── config/                        # Configuration files
│
├── tools/                             # Utility scripts
│   ├── sync-history.js                # Question history management
│   └── pdf_to_markdown.py             # PDF conversion utility
│
├── CLAUDE.md                          # AI instructions (this file)
└── README.md                          # Project overview
```

## Exam Blueprint

**Exam Code:** CIS-DF | **Questions:** 75 | **Duration:** 90 minutes

| Domain | Weight | Key Topics |
|--------|--------|------------|
| 1. Configuration | 15% | CI Class Manager, IRE (Identification & Reconciliation Engine), CMDB 360/multisource |
| 2. Ingest | 19% | Auto vs manual relationships, Service Graph Connectors, data ingestion methods, technical debt |
| 3. Govern | 35% | CMDB Health Score (6 metrics), Data Manager, deduplication, CMDB Workspace, Principal Classes, CI lifecycle |
| 4. Insight | 20% | NLQ, CMDB 360 reports, Unified Map, dependency views, Playbooks |
| 5. CSDM Fundamentals | 11% | CI mapping to CSDM domains, stakeholder collaboration, CSDM benefits |

## Question Types

Generate questions in these formats (matching the real exam):
- **Multiple Choice** - Single correct answer (A, B, C, D)
- **Multiple Select** - "Choose two/three" with multiple correct answers
- **Matching** - Pair terms with definitions or put items in sequence
- **Scenario-Based** - Real-world situations requiring applied knowledge

## Resources

- `docs/ServiceNow CIS-DF Exam Specification_Blueprint.pdf` - Official exam blueprint with sample questions
- `docs/servicenow/markdown/` - 70+ ServiceNow Zurich release documentation markdown files covering CMDB topics. Use this for AI reading.
- `docs/servicenow/pdf/` - 70+ ServiceNow Zurich release documentation PDFs. Fall back if markdown files have issues.

## How to Generate Practice Questions

1. **Read relevant files** from `docs/servicenow/markdown/` folder based on the topic
2. **Create scenario-based questions** that test practical application, not just memorization
3. **Include answer explanations** with references to concepts from documentation
4. **Weight question distribution** according to domain percentages (Govern 35% = most questions)

## Key Concepts by Domain

### Configuration (15%)
- CI Class Manager: table structure, class attributes, hierarchy
- IRE: identification rules, reconciliation rules, duplicate detection
- CMDB 360: multisource data aggregation, source precedence

### Ingest (19%)
- Service Graph Connectors (AWS, Azure, GCP, SCCM, Intune, etc.)
- IntegrationHub ETL
- Discovery vs manual CI creation
- Asset-CI synchronization (Install Status, Hardware Status sync automatically)

### Govern (35%)
- **Health Score 6 Metrics:** Correctness, Completeness, Compliance, Relationship, Staleness, Duplicate
- Data Manager policies and requirements
- Deduplication Wizard
- CMDB Workspace features
- Principal Classes definition
- CI lifecycle attributes (operational status, install status)

### Insight (20%)
- Natural Language Query (NLQ)
- CMDB Query Builder
- Unified Map configuration and usage
- Dependency Views
- Data Foundations Dashboard Playbooks structure:
  1. Summary of indicator
  2. Overview of problem
  3. Importance of addressing issue
  4. Fix or Improve

### CSDM Fundamentals (11%)
- CSDM domains and layers
- Mapping CIs to correct CSDM classes
- Business Services, Technical Services, Application Services
- Stakeholder roles in CSDM adoption

## Sample Question Format

```
**Question X (Domain: [Domain Name])**
[Scenario or question text]

A. [Option A]
B. [Option B]
C. [Option C]
D. [Option D]

<details>
<summary>Answer</summary>
**Answer: [Letter]**
[Explanation of why this is correct and why others are wrong]
</details>
```

## Session Commands

When user asks to practice, offer these options:

### Topic Quiz (Learning Mode) - Interactive
- `"Quiz me on Configuration"` - 7-10 questions, untimed, with explanations
- `"Quiz me on Ingest"` - 7-10 questions on data ingestion
- `"Quiz me on Govern"` - 10-15 questions (largest domain)
- `"Quiz me on Insight"` - 7-10 questions on reporting/visualization
- `"Quiz me on CSDM"` - 5-7 questions on CSDM fundamentals

**Topic Quiz Flow:**
1. Present ONE question at a time using AskUserQuestion tool
2. User selects answer (A/B/C/D)
3. Show result (Correct/Incorrect) with explanation
4. Provide **Topic Deep Dive**: detailed concept explanation, how it works, exam tips
5. Proceed to next question
6. At end: Save results to `practice/results/{topic}-{YYYYMMDD-HHMMSS}.md`

### Full Practice Test (Exam Simulation Mode) - WEB APP
- `"Full practice test"` or `"Start exam simulation"`

**How It Works:**
1. Read `practice/config/exam-config.json` to check question deduplication settings
2. If `avoidRepeatQuestions` is enabled, read `practice/config/question-history.json` and avoid generating questions that match existing hashes
3. Claude reads documentation and generates 75 UNIQUE randomized questions (avoiding previously asked questions when enabled)
4. Creates timestamped HTML file: `practice/app/practice-test-YYYYMMDD-HHMMSS.html`
5. Update `practice/config/question-history.json` with the newly generated questions
6. User opens that specific file in browser
7. Take exam with full timer and navigation

**Exam Simulation Features (Web App):**
- 75 questions (weighted: Config 11, Ingest 14, Govern 26, Insight 15, CSDM 9)
- 90 minutes STRICT timer - auto-submits when time expires
- NO explanations during exam (real exam environment)
- Question navigator sidebar - jump to any question
- Flag for review functionality
- Results page with domain-wise breakdown and improvement areas
- Download report as markdown
- Each exam file preserved with unique timestamp for future reference

### Other Commands
- `"Explain [topic]"` - Read relevant markdown files and explain the concept
- `"Show my quiz history"` - List completed quizzes from practice/results folder
- `"Review [quiz-file]"` - Review a specific past quiz

### Cleanup Commands
Use these commands to clear generated files and reset progress:

**Clear Quiz Results** (practice/results/*.md files)
- `"Clear the results"` or `"Delete my quiz history"` or `"Clear quiz results"`

**Clear Generated Exam Simulations** (practice/app/*.html files)
- `"Clear the practice exams"` or `"Delete generated exams"` or `"Clear simulation exams"`

**Clear Both Results and Exams**
- `"Clear all practice data"` or `"Reset practice files"`

**Clear Question History** (reset deduplication - allows questions to repeat)
- `"Clear question history"` or run: `node tools/sync-history.js --clear`

See `docs/INTERACTIVE_QUIZ_GUIDE.md` for complete documentation of the quiz system.

## Question Deduplication System

The practice exam system tracks previously asked questions to avoid repetition.

### Configuration File
**Location:** `practice/config/exam-config.json`

```json
{
  "avoidRepeatQuestions": true,
  "warnOnRepeats": true
}
```

- `avoidRepeatQuestions`: When `true`, generate new questions that haven't been asked before
- `warnOnRepeats`: When `true` and all questions exhausted, warn user but proceed with repeats

### Question History
**Location:** `practice/config/question-history.json`

Contains SHA-256 hashes of all previously asked questions. Updated automatically after each exam generation.

### Sync Utility
**Location:** `tools/sync-history.js`

Commands:
```bash
node tools/sync-history.js          # Sync history from all sources
node tools/sync-history.js --clear  # Clear all history (start fresh)
node tools/sync-history.js --status # Show current history stats
node tools/sync-history.js --help   # Show help
```

Sources scanned:
- **HTML files** in `practice/app/` - Extracts embedded questions from exam simulations
- **Markdown files** in `practice/results/` - Extracts questions from completed quiz results

### How Deduplication Works

When generating a new practice exam:
1. Read `practice/config/exam-config.json`
2. If `avoidRepeatQuestions` is `true`:
   - Read `practice/config/question-history.json`
   - Generate only NEW questions that don't match existing hashes
   - Use SHA-256 hash of normalized question text for comparison
3. If not enough new questions can be generated:
   - Warn user about which questions will repeat
   - Prioritize least-recently-asked questions for repeats
   - Proceed with exam including necessary repeats
4. After generating exam, update `question-history.json` with new questions

### To Disable Deduplication
Set `avoidRepeatQuestions` to `false` in `practice/config/exam-config.json` to allow question repetition.

---
> Source: [rtnsht/sn-cis-df-practice](https://github.com/rtnsht/sn-cis-df-practice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
