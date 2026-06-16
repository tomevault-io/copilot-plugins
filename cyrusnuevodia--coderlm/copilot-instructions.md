## coderlm

> Use when a task involves many files (>10), the total content exceeds comfortable context size, or the task benefits from divide-and-conquer. Runs an agent that discovers files via shell tools, peeks strategically, and recursively decomposes into sub-agents. Bundles bashrlm context guards — output from high-output commands (cat, grep, rg, jq, find, ls, curl, etc.) is automatically truncated so the agent never floods its own context window.


## Usage

```
coderlm <agent> --prompt <file> [--max-depth N] [--allowedTools TOOLS]
```

## Examples

```bash
echo "Find all TODO comments in src/" > task.txt
coderlm codex --prompt task.txt
coderlm "bunx --bun @google/gemini-cli" --prompt task.txt

echo "Fix type errors in src/" > task.txt
coderlm claude --prompt task.txt --allowedTools "Bash,Edit"
```

---
> Source: [CyrusNuevoDia/coderlm](https://github.com/CyrusNuevoDia/coderlm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
