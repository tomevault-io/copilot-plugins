## bsky-schwartz

> AI-powered Schwartz Value analysis for Bluesky posts.

# SKILL.md - bsky-schwartz

AI-powered Schwartz Value analysis for Bluesky posts.

## What It Does

Analyzes Bluesky posts using 19 Schwartz values via AI models. Scores each value 0-6 based on post content.

## Quick Start

```bash
# Build
cd feed-generator && make build

# Collect posts
./bin/feedgen -n 50

# Analyze with specific model
./bin/feedgen -model=gpt-4o-mini analyze

# Analyze with specific model + limit
./bin/feedgen -model=ministral-8b -l 100 analyze

# Random sample
./bin/feedgen -sample 50 -model=qwen3-vl-8b analyze
```

## Commands

| Command            | Description                                  |
| ------------------ | -------------------------------------------- |
| `collect-profiles` | Collect posts from profiles in profiles.json |
| `analyze`          | Analyze unanalyzed posts with AI             |
| `from-file <file>` | Load posts from JSON file                    |

## Flags

| Flag      | Default | Description                          |
| --------- | ------- | ------------------------------------ |
| `-n`      | 40      | Number of posts to collect           |
| `-l`      | 0       | Limit posts to analyze (0=all)      |
| `-model`  | ""      | Filter by model name                 |
| `-sample` | 0       | Random sample size                  |
| `-images` | true    | Include images in analysis          |
| `-limit`  | 0       | Posts per model                     |
| `-total`  | 0       | Total posts for weighted collection |

**Important**: Flags must come BEFORE subcommand:
```bash
./bin/feedgen -model=qwen3-14b analyze  # OK
./bin/feedgen analyze -model=qwen3-14b  # WRONG
```

## Models

| Model          | Provider    | Images | Notes                    |
| -------------- | ----------- | ------ | ------------------------ |
| gpt-4o-mini    | OpenRouter  | ✅     | Default, reliable        |
| ministral-8b   | OpenRouter | ✅     | Cheap, EU                |
| qwen3-14b     | SiliconFlow | ❌     | Text-only, 14B params    |

## Database

SQLite at `data.db`.

Tables:

- `posts` - Raw post data
- `analyses` - AI analysis results (19 value columns)
- `users` - User preferences (value weights)
- `weights` - User-value weights

Query examples:

```bash
# Check posts count
sqlite3 data.db "SELECT COUNT(*) FROM posts;"

# Check analysis by model
sqlite3 data.db "SELECT model, COUNT(*) FROM analyses GROUP BY model;"

# Get value averages
sqlite3 data.db "SELECT AVG(power), AVG(caring) FROM analyses;"

# Delete all posts (keeps analyses with broken references)
sqlite3 data.db "DELETE FROM posts;"

# Delete all analyses
sqlite3 data.db "DELETE FROM analyses;"

# Full reset (delete both)
sqlite3 data.db "DELETE FROM posts; DELETE FROM analyses;"
```

## Feed Service

```bash
cd feed-service && make run
```

Serves personalized feeds based on user value preferences.

## Analysis Scripts

Python scripts in `data-analysis/`:

```bash
cd data-analysis && source .venv/bin/activate

# Generate plots + CSV
python main.py

# Generate outlier analysis
python outliers_visualization.py
```

Output: `data-analysis/plot/*.png`, `*.csv`

## Prompt Versions

- v3: Original prompt
- v4: Updated with better key names + zero-score guidelines

```bash
./bin/feedgen -prompt=v4 analyze  # uses PROMPT_V4.md
```

## Common Tasks

### Re-analyze all posts with new model

```bash
sqlite3 data.db "DELETE FROM analyses WHERE model = 'qwen3-14b';"
./bin/feedgen -model=qwen3-14b analyze
```

### Weighted collection (proportional distribution)

```bash
./bin/feedgen -total 100 -n 50 collect-profiles
# Distributes 50 posts proportionally across profiles based on their limits
```

### Check value distribution

```bash
cd data-analysis && source .venv/bin/activate && python main.py
# See plot/values_avg.csv
```

### Add new profile to collection

Edit `profiles.json` with handle and value targets.

## Files

```
bsky-schwartz/
├── data.db                    # SQLite database
├── feed-generator/            # CLI tool
│   ├── main.go               # Entry + commands
│   ├── analyze.go             # Analysis logic
│   ├── profiles.json          # Profile list for collection
│   ├── prompts/PROMPT_V4.md   # AI prompt
│   └── models.go              # Model configs
├── feed-service/             # HTTP server
│   ├── algos.go              # Feed algorithms
│   └── templates/            # Templ HTML
└── data-analysis/            # Python analysis
    ├── main.py               # Generate plots
    └── plot/                 # Output PNG/CSV
```

---
> Source: [davide-ferrara/bsky-schwartz](https://github.com/davide-ferrara/bsky-schwartz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
