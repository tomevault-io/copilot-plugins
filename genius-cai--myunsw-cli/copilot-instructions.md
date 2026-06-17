## myunsw-cli

> |


# myUNSW CLI — UNSW Student Portal Automation

Automate myUNSW operations (enrollment, grades, timetable, fees, personal info) via AI agent + curl.

## Prerequisites

Cookie file at `/tmp/myunsw_active.txt` with JSESSIONID (path=/active), AWSALB, AWSALBCORS.
Session expires ~30 min. Validate: `curl -s -b /tmp/myunsw_active.txt 'https://my.unsw.edu.au/active/studentClassEnrol/years.xml' | grep -c 'bsdsSequence'` (1=valid).

## Core Pattern (BSDS Framework)

Every request follows:
1. GET page → extract `bsdsSequence` from hidden input
2. POST to SAME URL → `bsdsSequence` + `bsdsSubmit-{action}` + params
3. Follow 302 redirect → next state
4. Extract new `bsdsSequence`, repeat

Rules: always use freshest sequence. Sequential only. Stale sequence = silently ignored.

```bash
COOKIE="/tmp/myunsw_active.txt"
BASE="https://my.unsw.edu.au/active"
extract_seq() { grep -oP 'bsdsSequence" value="\K[0-9]+' "$1"; }
```

## Capabilities

| Action | How |
|--------|-----|
| Enrollment status | GET years.xml → POST → courses.xml → `?data=classes` returns JSON |
| Timetable | Navigate to ENR2.TTBL → `timetable.xml?data=classes` returns JSON |
| Grades | POST to `studentResults/results.xml` with `term=XXXX&bsdsSubmit-reload=Go` |
| Academic statement | Navigate studentAcadStatement → `statement.pdf` |
| Fees | `studentFees/reset.xml` → select statement |
| Class search | `studentClassSearch` with subject/catalogNbr/term params |
| Personal info | studentAddress, studentPhone, studentEmail, emergencyContact (read+write) |
| Waitlist | In ENR2.CRS, `bsdsSubmit-manage-waitlist` |

## Term Codes

5253=T1'25, 5256=T2'25, 5259=T3'25, 5263=T1'26, 5266=T2'26, 5269=T3'26. Pattern: +3 per term, ~+4 across years.

## Detailed Documentation

See `docs/api-reference.md` for complete endpoint documentation, JSON schema, and all 27 endpoints.
See `docs/bsds-framework.md` for BSDS state machine reverse engineering notes.

---
> Source: [Genius-Cai/myunsw-cli](https://github.com/Genius-Cai/myunsw-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
