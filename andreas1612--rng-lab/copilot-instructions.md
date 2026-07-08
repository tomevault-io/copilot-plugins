## rng-lab

> generates a plain-English gap analysis, and produces a downloadable PDF report.

# Finalogic Pre-Audit Tool — Claude Guide

## What This Project Is
A pre-audit RNG readiness assessment tool for iGaming. Clients upload a binary RNG output file;
the tool runs NIST SP 800-22 statistical tests, scores results against 4 jurisdiction configs,
generates a plain-English gap analysis, and produces a downloadable PDF report.

**This is NOT an accredited audit tool.** Every page of every report carries the verbatim
IMPORTANT NOTICE from docs/legal/LEGAL.md. Do not remove or abbreviate this notice.

---

## Before Writing Any Code — Always Read These First
1. STATUS.md — current sprint, what is done, what is next
2. NEXT_SESSION.md — full handoff notes and next sprint prompt
3. docs/legal/LEGAL.md — disclaimer language (verbatim in all reports)
4. docs/legal/AUP.md — acceptable use policy
5. docs/algorithms/THRESHOLDS.md — jurisdiction thresholds (source of truth for JSON configs)
6. src/engine/main.py — current API endpoints
7. src/engine/nist/runner.py — NIST test runner
8. src/engine/scoring.py — jurisdiction scoring engine

---

## Directory Structure
```
finalogic-preaudit-tool/
├── CLAUDE.md                    ← This file
├── README.md                    ← Project overview and setup
├── STATUS.md                    ← Sprint log and decisions
├── NEXT_SESSION.md              ← Handoff notes for next Claude session
├── CLAUDE_CODE_PROMPT.md        ← Original Sprint 1 prompt (reference)
├── package.json                 ← Root workspace scripts
├── .gitignore
│
├── docs/
│   ├── research/
│   │   ├── RESEARCH_DONE.md     ← All research complete — do not re-research
│   │   └── SOURCES.md
│   ├── algorithms/
│   │   ├── ALGORITHMS.md        ← NIST SP 800-22 + Dieharder + PractRand explained
│   │   └── THRESHOLDS.md        ← Per-jurisdiction RTP floors and p-value thresholds
│   └── legal/
│       ├── LEGAL.md             ← Liability framework + mandatory disclaimer text
│       └── AUP.md               ← Acceptable Use Policy
│
├── src/
│   ├── engine/                  ← Python FastAPI backend (run from this dir)
│   │   ├── main.py              ← FastAPI app: /health, /analyse, /report
│   │   ├── scoring.py           ← Jurisdiction scoring engine
│   │   ├── requirements.txt     ← pip dependencies
│   │   ├── nist/
│   │   │   ├── runner.py        ← Real NIST SP 800-22 runner (all 15 tests)
│   │   │   └── sp800_22_tests/  ← Cloned NIST Python port (git clone)
│   │   └── report/
│   │       ├── gap_analysis.py  ← Plain-English gap analysis generator
│   │       └── generator.py     ← ReportLab PDF builder
│   │
│   ├── jurisdictions/           ← One JSON per jurisdiction
│   │   ├── mga.json             ← Malta Gaming Authority
│   │   ├── ukgc.json            ← UK Gambling Commission
│   │   ├── denmark.json         ← Spillemyndigheden
│   │   └── canada_cga.json      ← Canadian Gaming Association
│   │
│   ├── report/
│   │   └── template.html        ← Jinja2 HTML template (for preview + future WeasyPrint)
│   │
│   └── ui/                      ← React + TypeScript + Vite frontend
│       ├── package.json
│       ├── vite.config.ts       ← Proxies /api → localhost:8000
│       ├── index.html
│       └── src/
│           ├── main.tsx
│           ├── App.tsx          ← Main UI component
│           └── api/
│               └── client.ts    ← axios instance + checkHealth(), analyseFile(), generateReport()
│
└── src/engine/test_data/        ← gitignored — put test .bin files here
```

---

## How to Run

### Backend
```bash
cd src/engine
pip install -r requirements.txt
python -m uvicorn main:app --port 8081
# API docs: http://localhost:8081/docs
```

### Frontend (requires Node 20 LTS — not yet installed on this machine)
```bash
cd src/ui
npm install
npm run dev
# UI: http://localhost:5173
```

### Quick test (no UI needed)
```bash
# Generate a test binary
python -c "import os; open('test_rng.bin','wb').write(os.urandom(200000))"

# JSON analysis
curl -X POST http://localhost:8081/analyse -F "file=@test_rng.bin"

# PDF report (saves to disk)
curl -X POST http://localhost:8081/report -F "file=@test_rng.bin" -o report.pdf
```

---

## Key Technical Decisions (Do Not Revisit Without Good Reason)
| Decision | What | Why |
|---|---|---|
| PDF library | ReportLab (not WeasyPrint) | WeasyPrint needs GTK3 DLLs; corporate machine blocks admin install |
| UKGC accreditation std | ISO/IEC 17025 | 17025 = testing labs; 17065 = product cert bodies (wrong) |
| Canada RTP floor | 85% Ontario baseline, not authoritative | Province floors vary — see canada_cga.json notes |
| GLI/BMM | Both listed on approved-lab lists | GLI acquired by BMM 2022; same parent, separate entities |
| NIST stdout | Suppressed via redirect_stdout | NIST library prints to stdout — would pollute API logs |
| Dieharder/PractRand | Deferred | Dieharder needs WSL; PractRand binary not placed yet |

---

## Jurisdiction Config Schema (src/jurisdictions/*.json)
```json
{
  "id": "mga",
  "name": "Malta Gaming Authority",
  "short_name": "MGA",
  "min_rtp_percent": 85,
  "p_value_floor": 0.01,
  "p_value_warning_floor": 0.05,
  "confidence_interval_percent": 95,
  "statistical_standard": "NIST SP 800-22",
  "lab_accreditation_standard": "ISO/IEC 17025",
  "approved_labs": ["GLI", "BMM Testlabs", "iTech Labs", "eCOGRA", "Quinel"],
  "approved_labs_note": "...",
  "notes": "..."
}
```

## API Response Shape (/analyse)
```json
{
  "filename": "rng_output.bin",
  "generated_at": "2026-05-06T14:00:00+00:00",
  "nist_result": {
    "suite": "NIST SP 800-22",
    "sample_info": { "size_bits": 1600000, "size_mb": 0.19, "sufficient": true, "warnings": [] },
    "tests": [{ "name": "Frequency (Monobit)", "p_value": 0.512, "status": "pass", "threshold_used": 0.01 }]
  },
  "jurisdiction_scores": [
    { "id": "mga", "name": "Malta Gaming Authority", "short_name": "MGA",
      "overall": "pass", "tests_passed": 15, "tests_warning": 0, "tests_failed": 0,
      "nist_check": { "result": "pass", "detail": "All 15 tests passed" },
      "rtp_floor_check": { "result": "not_checked", "detail": "..." } }
  ]
}
```

## DO NOT
- Remove or shorten the IMPORTANT NOTICE disclaimer — it must appear verbatim on page 1 and last page of every PDF
- Add authentication or a database (outside scope)
- Implement Dieharder or PractRand without first verifying the environment supports them
- Change the jurisdiction JSON schema without updating scoring.py and generator.py
- Modify files in docs/ without a specific reason — they are reference documents

---
> Source: [andreas1612/rng_lab](https://github.com/andreas1612/rng_lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
