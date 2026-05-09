## 4th-devs-cobol

> - **All logic must be in pure COBOL** - no Python, no jq, no sed, no shell helpers, no other languages

# COBOL Project Standards

## Language & Architecture
- **All logic must be in pure COBOL** - no Python, no jq, no sed, no shell helpers, no other languages
- **CALL "SYSTEM" ONLY for curl** HTTP requests - nothing else
- **CALL "C$SLEEP"** is OK for retry delays (GnuCOBOL built-in)
- **Direct C/POSIX calls** via GnuCOBOL FFI are OK (e.g. socket, mkdir, libcurl)

## AI/LLM Integration
- **AI/LLM is a required feature** in every task
- Use **function calling** (tool_use) where the LLM orchestrates and COBOL executes tools
- **Critical decisions must be deterministic** - do computation in COBOL, not in LLM (e.g. haversine matching, distance calculations)
- LLM role: orchestration, interpretation, text generation. COBOL role: computation, API calls, data processing
- Design tools that minimize LLM roundtrips (bulk operations > many small calls)

## JSON Handling
- **Escape all dynamic content** before embedding in JSON strings
- Character-by-character escaping in COBOL: `"` → `\"`, `\` → `\\`, newline (X"0A") → `\n`, CR (X"0D") → `\r`, tab (X"09") → `\t`
- Use a dedicated ESCAPE paragraph (see S01E04 READ-AND-ESCAPE as reference pattern)
- Use `WS-QT PIC X(1) VALUE '"'` for embedding double quotes in STRING statements
- Use `WS-NL` (X"5C" + "n") for JSON newline escape sequences in prompts

## Configuration - CRUCIAL
- **ZERO hardcoded URLs in source code** - no https://, no http://, no domain names in .cob files
- **ZERO hardcoded API keys or secrets** in source code
- **ALL URLs and keys come from environment variables** via `ACCEPT FROM ENVIRONMENT`
- Available env vars in CI:
  - `HUB_API_KEY` - Hub API key
  - `HUB_API_URL` - Hub base URL (e.g. for /verify, /data/, /api/)
  - `OPENAI_API_KEY` - OpenAI API key
  - `OPENAI_API_URL` - OpenAI chat completions endpoint
  - `OPENROUTER_API_KEY` - OpenRouter API key
  - `OPENROUTER_API_URL` - OpenRouter chat completions endpoint
  - `GEOCODING_API_URL` - Open-Meteo geocoding endpoint
- If you need a new env var, ASK - don't hardcode the URL
- Task-specific constants from task spec (task name, route codes, sender IDs) are OK to hardcode
- Build API paths dynamically: `STRING TRIM(WS-HUB-URL) "/dane/doc/" "file.md"`

## Data & Documentation
- **Fetch documentation dynamically** via curl - don't hardcode doc content in prompts
- Task input data (CSV, logs, JSON) must be fetched from Hub API at runtime
- Cache fetched files locally to avoid re-fetching on retries

## Code Style (GnuCOBOL Fixed Format)
- 7-space indent for Area A (divisions, sections, paragraphs, FD, 01 level)
- 11-space indent for Area B (statements, lower-level data items)
- Lines must stay under 72 characters (column limit)
- Column 7: `*>` for comments
- Use `FUNCTION ALL INTRINSIC` in REPOSITORY paragraph
- Compile check: `cobc -fsyntax-only app.cob`

## HTTP & Retry Logic
- All HTTP via curl through `CALL "SYSTEM"`
- Write request body to temp file, use `curl -d @file`
- Handle 503 (retry after short wait) and 429 (respect Retry-After header)
- Typical retry: 5-10 attempts with `CALL "C$SLEEP"` between

## Error Handling
- Parse Hub API responses for `"code"` and `"message"` fields
- Detect flags via `INSPECT ... TALLYING ... FOR ALL "FLG"`
- Retry loop with error feedback to LLM on subsequent attempts
- Display response previews for debugging

## CI/CD (GitHub Actions)
- Pipeline: rate-limit-gate → S01E01 → S01E02 → S01E03 → S01E04 → S01E05 → S02E01 → S02E02 → S02E03 → S02E04 → S02E05 → S03E01 → S03E02 → S03E03 → S03E04 → S03E05 → S04E01 → S04E02 → S04E03 → S04E04 → S04E05 → S05E01 → S05E02 → S05E03 → S05E04 → S05E05
- Each job needs: `actions/checkout@v5`, `gnucobol4` install, env vars for secrets
- Jobs that use AI need: `OPENAI_API_KEY`, `OPENAI_API_URL` in env
- Flag detection: `grep -q 'FLG:'` in output, mask with `sed 's/{FLG:[^}]*}/{FLG:XXX}/g'`

## Reference Implementations
- **S01E04** (SendIt) - wzorcowy: dynamic doc fetching, READ-AND-ESCAPE, LLM request building
- **S02E02** (Electricity) - wzorcowy: PNG processing, vision API, all standards met
- **S01E03** (Proxy) - wzorcowy: TCP server, libcurl FFI, 4 JSON escape paragraphs, tool-calling loop
- **S01E02** (FindHim) - wzorcowy function calling: `get_closest_suspect_to_plant` bulk tool, deterministic matching + LLM orchestration

---
> Source: [ddnowicki/4th-devs-cobol](https://github.com/ddnowicki/4th-devs-cobol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
