## proborge

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`proborge` is a CLI pipeline that generates competitive programming problems and packages them in **CMS Italian import format** for deployment to a Contest Management System (CMS) instance at `cms.bhos.tech`.

**CMS has no HTTP API for task import.** Tasks are imported via `cmsAddTask` CLI on the server. The pipeline produces a zip ready for `scp` + `cmsAddTask`.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run full pipeline on a problem
python main.py problems/input/1.md

# Run single stage
python main.py --stage statement problems/input/1.md
python main.py --stage solution problems/input/1.md
python main.py --stage generator problems/input/1.md
python main.py --stage stress problems/input/1.md
python main.py --stage testcases problems/input/1.md
python main.py --stage package problems/input/1.md

# Deploy to CMS server (requires env vars below)
python main.py --deploy problems/input/1.md
```

## Environment Variables

```
ANTHROPIC_API_KEY=...          # required for AI stages
CMS_SSH_HOST=cms.bhos.tech     # optional: enables auto-deploy
CMS_SSH_USER=ubuntu
CMS_SSH_KEY=/path/to/key.pem
CMS_CONTEST_ID=1
```

## Pipeline Stages

6 sequential stages, each writes to `problems/output/<N>/`:

| Stage | Module | Input → Output |
|-------|--------|----------------|
| 1 Statement | `pipeline/statement.py` | `.md` → `working/1_az.md` + `cms_package/statement/statement.pdf` |
| 2 Solution | `pipeline/solution.py` | `1_az.md` → `working/1_solution.cpp` + `working/1_brute.cpp` |
| 3 Generator | `pipeline/generator.py` | `1_az.md` → `working/1_gen.cpp` (compiled as `./gen`) |
| 4 Stress test | `pipeline/stress.py` | solution + brute + gen → `working/stress_log.txt` (200 rounds) |
| 5 Test cases | `pipeline/testcases.py` | gen + solution → `cms_package/input/`, `output/`, `gen/GEN`, `task.yaml` |
| 6 Package | `pipeline/cms_package.py` | `cms_package/` → `rhyme_cms.zip` |

## CMS Italian Format — Critical Details

### File naming
- `input/inputN.txt` and `output/outputN.txt`
- N is **0-indexed, no leading zeros** (0, 1, 2, ...)
- `statement/statement.pdf` — exactly this path

### task.yaml required fields
```yaml
name: rhyme                  # short URL-safe name, matches task directory
title: "Qafiyə"             # Azerbaijani title
n_input: 20                  # must match actual file count exactly
time_limit: 1.0              # float, seconds
memory_limit: 512            # int, MiB
score_mode: max_subtask      # use max_subtask if subtasks exist, else max
token_mode: infinite
primary_language: az
public_testcases: "0,1,2"   # 0-indexed samples, visible without tokens
# total_value: 100           # only when no subtasks
```

### gen/GEN file (subtask grouping)
CMS reads this to detect GroupMin scoring. **CMS does not re-run the generator** — the actual `inputN.txt` files must already exist. Lines are informational only.

```
# ST: 30
input 1 10 42
input 1 10 7
# ST: 30
input 1 1000 42
# ST: 40
input 1 10000 1
```

### Generator convention
Generator must accept `./gen <seed> <subtask_number>`. Subtask controls constraint size.

### Test distribution
Given `sample_count=3`, `test_count=20`, subtasks `[30, 30, 40]`:
- Tests 0–2: samples → `public_testcases: "0,1,2"`
- Tests 3–8: subtask 1 (small constraints)
- Tests 9–14: subtask 2 (medium constraints)
- Tests 15–19: subtask 3 (full constraints)

### task_name derivation
- Input file named `rhyme.md` → `task_name = "rhyme"`
- Input file named `1.md` → slugify the Azerbaijani title

## Metadata Parsing

After Stage 1, parse `1_az.md` to extract `time_limit`, `memory_limit`, `title`, `subtasks`, and sample I/O. Patterns (Azerbaijani):
- Title: `^# Məsələ (\d+): (.+)$`
- Time: `Vaxt limiti:\*\* ([\d.]+) saniyə`
- Memory: `Yaddaş limiti:\*\* (\d+) MiB`
- Subtasks: table rows matching `\|\s*(\d+)\s*\|\s*(\d+)\s*\|\s*(.+?)\s*\|`
- Samples: between `**Giriş:**\n``` ... ``` ` blocks

## Packaging & Deploy

`finalize_cms_package()` in `cms_package.py`:
1. Validates all file counts match `n_input`
2. Prints rich validation table
3. Zips `cms_package/` → `problems/output/<task_name>_cms.zip`
4. Prints `scp` + `cmsAddTask` instructions
5. If `CMS_SSH_HOST` set: auto-deploys via SSH

Manual deploy:
```bash
scp problems/output/rhyme_cms.zip user@cms.bhos.tech:/tmp/
ssh user@cms.bhos.tech
unzip /tmp/rhyme_cms.zip -d /tmp/task_rhyme/
cmsAddTask -c 1 /tmp/task_rhyme/cms_package/
```

## Validation Checklist

Before packaging, verify:
- `input/input0.txt` exists (0-indexed, not 1-indexed)
- Input and output file counts both equal `n_input` in `task.yaml`
- All input/output files non-empty
- `statement/statement.pdf` exists
- `gen/GEN` exists if subtasks defined; GEN scores sum to `max_score`
- `task_name` is URL-safe (lowercase, underscores, no spaces)

---
> Source: [huseynovvusal/proborge](https://github.com/huseynovvusal/proborge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
