## nightscout-cgm-skill

> Analyze CGM blood glucose data from Nightscout. Use this skill when asked about current glucose levels, blood sugar trends, A1C estimates, time-in-range statistics, glucose variability, or diabetes management insights.


# Nightscout CGM Analysis Skill

This skill provides tools for fetching and analyzing Continuous Glucose Monitor (CGM) data from Nightscout.

## ⚠️ Before Making Changes

**Always run tests before and after modifying `cgm.py`:**

```bash
cd <skill-path>
python -m pytest tests/ -q           # Quick check (281+ tests)
python -m pytest tests/ --cov=scripts  # With coverage
```

## Available Commands

Run the `cgm.py` script from this skill's `scripts/` directory:

```bash
python <skill-path>/scripts/cgm.py <command> [options]
```

Where `<skill-path>` is the location where this skill is installed (e.g., `~/.copilot/skills/nightscout-cgm`, `.github/skills/nightscout-cgm`, or `.claude/skills/nightscout-cgm`).

### Commands

| Command | Description |
|---------|-------------|
| `current` | Get the latest glucose reading |
| `analyze [--days N]` | Analyze CGM data (default: 90 days) |
| `report [--days N] [--output PATH] [--open]` | Generate interactive, PDF-printable HTML report with charts |
| `goals [view|set|clear]` | View, configure, or clear local report goals |
| `compare --period1 P1 --period2 P2` | Compare two time periods side-by-side |
| `alerts [--days N]` | Get trend alerts for recurring patterns |
| `refresh [--days N]` | Fetch latest data from Nightscout |
| `auto-refresh [view|set|on|off]` | Configure refresh-on-query sync, no daemon required |
| `patterns [--days N]` | Find interesting patterns (best/worst times, problem areas) |
| `query [options]` | Query with filters (day of week, time range) |
| `day <date> [options]` | View all readings for a specific date |
| `worst [options]` | Find your worst days for glucose control |
| `chart [options]` | Terminal visualizations (heatmap, sparkline, day chart) |
| `pump` | Get current pump status (IOB, COB, predicted glucose) * |
| `treatments [--hours N]` | Get recent treatments (boluses, temp basals, carbs) * |
| `events [--tag TEXT] [--days N]` | Correlate glucose response around existing Nightscout treatments/events * |
| `ages [--count N]` | Get CAGE/SAGE/IAGE from Site/Sensor/Insulin Change events * |
| `profile` | Get pump profile settings (basal rates, ISF, carb ratios) * |

\* **Optional pump/treatment commands require matching Nightscout uploads.** `pump` and `profile` require Loop, OpenAPS, AndroidAPS, or similar devicestatus/profile data. `treatments`, `events`, and `ages` require treatment/event entries. CGM-only users won't see errors—commands return a structured message explaining that the data is not available.

### Report Command

Generate a comprehensive, self-contained HTML report with interactive charts:
- `--days N` - Number of days to include (default: 90)
- `--output PATH` - Custom output path (default: nightscout_report.html)
- `--open` - Open report in browser after generating

**Auto-Sync:** Reports automatically sync from Nightscout if local data is more than 30 minutes old.

**Report Features:**
- Interactive date controls (7d/14d/30d/90d/6mo/1yr/All + custom date pickers)
- All charts recalculate dynamically in browser
- Sticky section navigation for long reports
- Deterministic executive summary with concise status bullets
- Print / Save PDF action with print-friendly styles
- Time-in-Range pie chart
- Modal Day (24-hour profile with percentile bands and target range lines)
- Daily trends, Day of week comparison
- Glucose histogram, Heatmap with hover tooltips
- Weekly summary with week-over-week TIR delta, best day, and context notes
- Key stats: TIR%, GMI (estimated A1C), CV (variability)
- Goal tracking: configurable TIR, CV, GMI, and average glucose targets
- **Insulin Delivery** (if using Loop/OpenAPS): TDD breakdown (bolus/basal), stacked bar chart, carb tracking

### Goals Command

Configure local report goals stored in `config.json`:
- `goals` or `goals view` - Show active goals
- `goals set --tir PCT --cv PCT --gmi PCT --average MGDL` - Set one or more goals
- `goals clear` - Remove custom goals and return to defaults
- Goal directions: TIR is minimum target; CV, GMI, and average glucose are maximum targets

### Auto-Refresh Command

Configure stale-data sync before read-only queries. This does not run a persistent daemon:
- `auto-refresh` or `auto-refresh view` - Show current refresh-on-query settings
- `auto-refresh set --minutes N` - Sync before queries when the newest local reading is older than N minutes
- `auto-refresh on` / `auto-refresh off` - Enable or disable refresh-on-query
- Use `refresh --days N` for explicit manual sync

### Day Command

View detailed readings for a specific date:
- `day <date>` - Date can be 'today', 'yesterday', '2026-01-16', or 'Jan 16'
- `--hour-start H` - Start hour for time window (0-23)
- `--hour-end H` - End hour for time window (0-23)

### Worst Command

Find your worst days ranked by peak glucose:
- `--days N` - Number of days to search (default: 21)
- `--hour-start H` - Start hour for time window (0-23)
- `--hour-end H` - End hour for time window (0-23)
- `--limit N` - Number of worst days to show (default: 5)

### Query Options

The `query` command supports flexible filtering:
- `--days N` - Number of days to analyze (default: 90)
- `--day NAME` - Filter by day of week (e.g., Tuesday, or 0-6 where 0=Monday)
- `--hour-start H` - Start hour for time window (0-23)
- `--hour-end H` - End hour for time window (0-23)

### Chart Options

The `chart` command creates terminal visualizations:
- `--sparkline` - Compact trend line using Unicode blocks (▁▂▃▄▅▆▇█)
- `--hours N` - Hours of data for sparkline (default: 24)
- `--date DATE` - Specific date for sparkline (e.g., today, yesterday, 2026-01-16)
- `--hour-start H` - Start hour for sparkline time window (0-23)
- `--hour-end H` - End hour for sparkline time window (0-23)
- `--heatmap` - Weekly grid showing time-in-range by day and hour
- `--day NAME` - Hourly breakdown for a specific day of week
- `--color` - Use ANSI colors (for direct terminal, not inside Copilot)

### Pump and Treatment Commands (Optional)

These commands use optional Nightscout endpoints. The skill auto-detects capabilities and returns a helpful CGM-only message when the required data is not available.

**`pump`** - Get current pump/loop status:
- IOB (Insulin on Board) and COB (Carbs on Board)
- Predicted glucose trajectory
- Recommended bolus
- Last enacted action (temp basal or bolus)
- Pump status (manufacturer, model, suspended/bolusing)
- Phone battery level

**`treatments [--hours N]`** - Get recent treatments:
- Boluses (automatic and manual)
- Temp basals
- Carb entries
- Summary totals (total insulin, total carbs)

**`events [--tag TEXT] [--days N] [--window-minutes N]`** - Correlate existing Nightscout events:
- Matches text in event type, notes, enteredBy, foodType, or reason
- Compares the 30-minute baseline before the event to the post-event window
- Does not store local annotations or edit Nightscout data
- Useful for questions like "what happens after pizza?" when those events already exist in Nightscout

**`ages [--count N]`** - Get CAGE/SAGE/IAGE:
- CAGE: time since last `Site Change`
- SAGE: time since last `Sensor Change`
- IAGE: time since last `Insulin Change`
- Average interval in days and observed change count per type

**`profile`** - Get pump profile settings:
- Basal rates by time of day
- Total daily basal
- ISF (Insulin Sensitivity Factor) by time
- Carb ratios by time
- Target glucose ranges
- Loop settings (max bolus, pre-meal targets, override presets)

### Examples

```bash
# Get current glucose
python scripts/cgm.py current

# Generate interactive HTML report (opens in browser)
python scripts/cgm.py report --days 90 --open

# Configure report goals
python scripts/cgm.py goals view
python scripts/cgm.py goals set --tir 75 --cv 34 --gmi 6.8 --average 145

# Configure refresh-on-query sync
python scripts/cgm.py auto-refresh view
python scripts/cgm.py auto-refresh set --minutes 15

# Analyze last 30 days
python scripts/cgm.py analyze --days 30

# Find patterns automatically (best/worst times, problem areas)
python scripts/cgm.py patterns

# What happened yesterday during lunch (11am-2pm)?
python scripts/cgm.py day yesterday --hour-start 11 --hour-end 14

# Find my worst lunch days in the last 3 weeks
python scripts/cgm.py worst --days 21 --hour-start 11 --hour-end 14

# What happens on Tuesdays after lunch?
python scripts/cgm.py query --day Tuesday --hour-start 12 --hour-end 15

# How are my overnight numbers on weekends?
python scripts/cgm.py query --day Saturday --hour-start 22 --hour-end 6
python scripts/cgm.py query --day Sunday --hour-start 22 --hour-end 6

# Morning analysis across all days
python scripts/cgm.py query --hour-start 6 --hour-end 10

# Show sparkline of last 24 hours (compact visual trend)
python scripts/cgm.py chart --sparkline --hours 24

# Show sparkline for a specific date and time range (with colors)
python scripts/cgm.py chart --date yesterday --hour-start 11 --hour-end 15 --color

# Show heatmap of time-in-range by day/hour
python scripts/cgm.py chart --heatmap

# Show hourly breakdown for a specific day
python scripts/cgm.py chart --day Saturday

# Generate interactive HTML report (opens in browser)
python scripts/cgm.py report --days 90 --open

# Compare two time periods
python scripts/cgm.py compare --period1 "last 7 days" --period2 "previous 7 days"
python scripts/cgm.py compare --period1 "this week" --period2 "last week"

# Get trend alerts (recurring patterns)
python scripts/cgm.py alerts --days 30

# Refresh data from Nightscout
python scripts/cgm.py refresh

# Correlate existing Nightscout event/treatment notes
python scripts/cgm.py events --tag pizza --days 90

# Get site/sensor/insulin change ages
python scripts/cgm.py ages --count 100
```

## Example Questions You Can Ask

With the pattern analysis capabilities, you can ask natural questions like:

- "What's my current glucose?"
- "Generate a report of my last 90 days"
- "Compare this week to last week"
- "How am I doing vs last month?"
- "What trend alerts do you see?"
- "Analyze my blood sugar for the last 30 days"
- "What patterns do you see in my data?"
- "What's happening on Tuesdays after lunch?"
- "When are my worst times for blood sugar control?"
- "How are my overnight numbers?"
- "When do I tend to go low?"
- "What day of the week is my best for time-in-range?"
- "Show me my morning patterns"
- "Show me a sparkline of my last 24 hours"
- "Show me a heatmap of my glucose"
- "What does Saturday look like?"
- "What was my worst lunch this week?" (you'll be asked what hours you eat lunch)
- "Show me what happened yesterday during dinner"
- "What were my worst days for breakfast the last two weeks?"

### Pump-Related Questions (if using Loop/OpenAPS)

- "What's my current IOB?"
- "How much insulin on board do I have?"
- "Show me my pump status"
- "What's my predicted glucose?"
- "What treatments have I had in the last 6 hours?"
- "How much insulin did I take today?"
- "What are my basal rates?"
- "What's my carb ratio?"
- "What are my ISF settings?"
- "Show me my Loop profile"

## Output Interpretation

### Time in Range (TIR)
- **Very Low** (<54 mg/dL): Dangerous hypoglycemia
- **Low** (54-69 mg/dL): Hypoglycemia
- **In Range** (70-180 mg/dL): Target range
- **High** (181-250 mg/dL): Hyperglycemia
- **Very High** (>250 mg/dL): Significant hyperglycemia

### Key Metrics
- **GMI**: Glucose Management Indicator (estimated A1C from CGM data)
- **CV**: Coefficient of Variation (<36% indicates stable glucose)
- **Hourly Averages**: Shows patterns throughout the day

## Configuration (Required)

Set the `NIGHTSCOUT_URL` environment variable to your Nightscout API endpoint:

```bash
# Linux/macOS
export NIGHTSCOUT_URL="https://your-nightscout-site.com/api/v1/entries.json"

# Windows PowerShell
$env:NIGHTSCOUT_URL = "https://your-nightscout-site.com/api/v1/entries.json"

# Windows (persistent)
[Environment]::SetEnvironmentVariable("NIGHTSCOUT_URL", "https://your-nightscout-site.com/api/v1/entries.json", "User")
```

The script will not run without this environment variable configured.

---
> Source: [shanselman/nightscout-cgm-skill](https://github.com/shanselman/nightscout-cgm-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
