## planeage

> PlaneAge is a lightweight aviation tool. It calculates aircraft age by joining real-time flight data with a local FAA registration database.

# AGENTS.md - PlaneAge Project Memory

## 1. Project Identity & Goal
PlaneAge is a lightweight aviation tool. It calculates aircraft age by joining real-time flight data with a local FAA registration database.
- **Goal:** Resolve `Flight #` -> `Tail #` -> `Mfr Year` -> `User Output`.

## 2. Tech Stack (Strict)
- **Runtime:** Node.js (CommonJS / `require`).
- **Web:** Express.js (Single `server.js`).
- **Frontend:** Vanilla HTML5/JS/CSS in `/public`. No build steps.
- **Data:** FAA `master.csv` (Streamed, not read, as the file is incredibly large).

## 3. Integration Logic: AeroDataBox API
- **Endpoint:** `GET https://aerodatabox.p.rapidapi.com/flights/number/{flightNumber}/{date}`
- **Parameter Formatting:** - `flightNumber`: Strip spaces (e.g., "DL 47" -> "DL47").
    - `date`: Ensure `YYYY-MM-DD` format.
- **Mapping Path:** - The API returns an **array** of flight objects. 
    - Target: `response[0].aircraft.registration`.
- **N-Number Normalization (CRITICAL):**
    - API returns: `N12345`.
    - FAA CSV contains: `12345` (no 'N' prefix).
    - Search the FAA file and return year.

## 4. Data Schema: Local FAA `master.csv`
- **Path:** `./data/master.csv`
- **Aircraft Reference:** `./data/acftref.csv`

## 5. Implementation Rules
1. **Zero-RAM Lookup:** Use `readline` to stop reading the CSV the moment a match is found.
2. **Error States:** - If flight isn't found: "Flight details currently unavailable."
    - If tail number isn't in FAA local cache: "Aircraft specs not in local registry."
3. **UI:** Results must be "Glanceable." Big numbers, high contrast.

## 6. Accessibility (Lightweight Target)
PlaneAge is a simple app; meet basic accessibility essentials without over-engineering:
- **Keyboard:** All controls usable with keyboard only; visible focus state.
- **Forms:** Every input has a programmatic label (`<label for>` or `aria-label`).
- **Updates:** Results/errors are announced for screen readers (e.g., `role="status"` / `aria-live`).
- **Contrast:** Keep text high-contrast (aim for WCAG AA contrast where practical).

## 7. Data Automation: FAA Weekly Refresh
- **Source URL:** `https://registry.faa.gov/database/ReleasableAircraft.zip`
- **Process (scripts/refresh-faa.js):**
    1. Download ZIP to `/data/temp.zip`.
    2. Extract `MASTER.txt` and `ACFTREF.txt` from the ZIP.
    3. Perform an "Atomic Swap":
        - Rename current `master.csv` to `master.old`.
        - Rename extracted `MASTER.txt` to `master.csv`.
        - Delete `master.old` and `temp.zip` upon success.
        - Repeat for `ACFTREF.txt`.
- **Dependencies:** Use `adm-zip` for extraction and native `https` for downloads to stay minimalist.
- **Execution:** Triggered via `npm run refresh` (mapped to `node scripts/refresh-faa.js`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chrisszeluga)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chrisszeluga)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
