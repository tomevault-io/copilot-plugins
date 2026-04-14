## sw-expert-academy

> Hello Gemini-CLI! You only need to care about Gemini CLI — ignore other agent platforms (e.g., Roo Code, Claude Code).

# SW Expert Academy
Hello Gemini-CLI! You only need to care about Gemini CLI — ignore other agent platforms (e.g., Roo Code, Claude Code).
Solve coding problems from SW Expert Academy using Python 3.7 (PyPy 3.7).
Supports both https://swexpertacademy.com/ (home, ID/PW login) and https://swexpertacademy.samsung.com/ (company, SSO login). Set `SWEA_BASE_URL` in `.env` to choose. The company mirror uses alphanumeric problem IDs with a difficulty prefix (e.g., `VH1234`, `H1234`, `MH1234`) instead of plain numbers.

## Skills

Skills are located in `.gemini/skills/`. Each skill has a `SKILL.md` with full instructions.

| Skill | Purpose |
|-------|---------|
| `fetching-problem <problem_id>` | Fetch problem statement from the website and save as `problem.md`. |
| `solving-problem <problem_id> [--resume \| --restart] [--auto \| --manual]` | Full solve loop: plan → write → validate → submit → reflect. `--auto` submits via browser; `--manual` (default) pauses for user verdicts. Max 10 trials. |
| `planning-solution <problem_id> [trial_number]` | Plan an algorithm and save as `plan_N.md`. Auto-increments if trial omitted. |
| `writing-solution <problem_id> [trial_number]` | Implement a plan as `solution_N.py` and run local tests. Uses latest plan if trial omitted. |
| `validating-solution <problem_id> <trial_number>` | Static code review to catch prohibited imports, forbidden syntax, and output format issues before submission. |
| `stress-testing <problem_id> <trial_number>` | Generate max-size inputs and measure execution time to catch TLE/RE before submission. |
| `submitting-solution <problem_id> <trial_number>` | Submit a solution via Playwright and return the judge verdict. |
| `diagnosing-failure <problem_id> <trial_number> <verdict>` | Diagnose a failed submission and produce a revised plan. |

## Prerequisites

- **`.env` file** — Copy `env.example` to `.env` and fill in `SWEA_BASE_URL`, `SWEA_ID`, and `SWEA_PW`. Required by `fetching-problem` and `submitting-solution`.
- **Playwright MCP** — Required by `fetching-problem`, `submitting-solution`, and `solving-problem --auto`. Enable the Playwright MCP server in your gemini-cli MCP settings and restart.

## Project layout

```
.env                                    # Credentials (git-ignored)
env.example                             # Template for .env
problem_bank/{problem_id}/
  problem.md                          # Problem statement
  progress.md                         # Solve loop progress log
  plan_{trial_number}.md              # Algorithm plan per trial
  python/solution_{trial_number}.py   # Solution per trial
  tests/                              # Stress test input files
```

## Constraints

- **No `import sys`** — use `input()` / `print()` for I/O.
- **No external packages** (numpy, pandas, etc.).
- **No walrus operator** (`:=`) — requires Python 3.8+.
- Target the Python/PyPy time and memory limits.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Taekyo-Lee)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Taekyo-Lee)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
