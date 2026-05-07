## outbound-agents

> 7 AI agents that turn a raw company list into personalized multi-channel outbound sequences.

# Brisbane Agents — B2B Outbound Pipeline

7 AI agents that turn a raw company list into personalized multi-channel outbound sequences.

## Pipeline Overview

```
Input CSV → Signal Scraper (auto-enriches) → [Review] → Lead Prioritizer → Prospect Profiler → Hook Writer → Sequence Builder → Final Sequences
            (0-enriched + 1-signals)                     (2-prioritized)    (3-profiles)        (4-hooks)     (5-sequences)
```

**Standalone agents** (run independently):
- **Reply Classifier** — classify incoming replies and recommend next actions
- **Meeting Prep** — generate pre-call briefs for booked meetings

## How to Run

### Full Pipeline
```
/outbound-pipeline input/my-companies.csv
```

Input can be as simple as a CSV with just company names — the Signal Scraper auto-enriches missing data via web search. After enrichment, you review the data before the pipeline continues.

### Individual Agents
Just ask naturally:
- "Run the signal scraper on input/my-companies.csv"
- "Classify the replies in input/sample-replies.csv"
- "Prep me for the meetings in input/sample-meetings.csv"

## ICP Configuration

Edit `input/icp-config.csv` to customize scoring and copy for your product. The lead prioritizer, hook writer, and sequence builder all read this file. Without it, agents run in generic mode.

## Agent Roles

| # | Agent | What It Does |
|---|-------|-------------|
| 1 | Signal Scraper | Auto-enriches sparse CSVs via web search, then detects buying signals |
| 2 | Lead Prioritizer | Scores leads 1-100, assigns TIER_1 through TIER_4 (uses ICP config if present) |
| 3 | Prospect Profiler | Builds 60-second actionable profiles for reps |
| 4 | Hook Writer | Crafts personalized opening lines (max 120 chars, uses ICP config if present) |
| 5 | Sequence Builder | Creates 7-touch, 21-day multi-channel sequences (uses ICP config if present) |
| 6 | Reply Classifier | Classifies replies into 7 categories with next actions |
| 7 | Meeting Prep | Generates pre-call briefs with time-blocked agendas |

## File Conventions

- Pipeline output files use numbered prefixes: `output/0-enriched.csv`, `output/1-signals.csv`, etc.
- Standalone agents write to `output/classified-replies.csv` and `output/meeting-briefs.csv`
- Input files go in `input/`
- Example output from a test run is in `examples/`

## Required Input Columns (Pipeline)

Only `company_name` is required. All other columns are optional — the Signal Scraper will research missing data via web search.

| Column | Required? | Description |
|--------|-----------|-------------|
| `company_name` | Yes | Company name |
| `industry` | No | Industry or vertical |
| `employee_count` | No | Number of employees |
| `job_postings` | No | Current open roles |
| `recent_news` | No | Recent press, announcements, events |
| `tech_stack` | No | Technologies used |
| `funding_info` | No | Funding history and amounts |
| `linkedin_activity` | No | Recent LinkedIn activity from company/leaders |

## Reply Classifier Input Columns

Sample file: `input/sample-replies.csv`

| Column | Description |
|--------|-------------|
| `company_name` | Company name |
| `contact_name` | Person who replied |
| `contact_email` | Their email |
| `original_sequence_step` | Which step they replied to |
| `reply_subject` | Email subject |
| `reply_body` | Reply text |
| `reply_timestamp` | When they replied |

## Meeting Prep Input Columns

Sample file: `input/sample-meetings.csv`

| Column | Description |
|--------|-------------|
| `company_name` | Company name |
| `contact_name` | Who you're meeting |
| `contact_title` | Their title |
| `meeting_type` | discovery or demo |
| `meeting_date` | Date of meeting |
| `meeting_time` | Time of meeting |
| `industry`, `employee_count`, `signal_type`, `signal_detail`, `priority_tier`, `profile_summary`, `key_talking_points`, `potential_pain_points`, `sequence_history` | Context fields |

---
> Source: [janskuba/outbound-agents](https://github.com/janskuba/outbound-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
