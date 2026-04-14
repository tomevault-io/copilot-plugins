## agentic-cli

> 1. Always explain what you are doing before modifying code.

# GEMINI.md

## Agent Rules

1. Always explain what you are doing before modifying code.
2. Add comments explaining important logic.
3. Log each major step.
4. Add the current date to generated documentation.
5. Prefer clear variable names over short ones.

## Documentation Rules

When generating docs:
- Include a step-by-step explanation.
- Include the current date.
- Explain the purpose of the code.

Format:

Date: YYYY-MM-DD

Step 1: Explain action  
Step 2: Explain modification  
Step 3: Explain outcome

Write it in a an markdown file inside a folder updates, with the file name as the 
an incremnetion of previois numnber

keeps updating it, when your encounter a problem write about it, when you solve it and write
about how you have approched it and the journey each time

# Skill System (Claude-style Simulation)

## Skill Loading Rules

* Skills are stored in `.claude/skills/` directory
* Each skill is a markdown file
* When a task matches a skill, you MUST:

  1. Identify the most relevant skill(s)
  2. Read and follow all instructions inside the skill file
  3. Apply its rules strictly in your response

## Skill Execution Behavior

* Treat skills as authoritative instructions
* Override default behavior if skill specifies constraints
* Combine multiple skills if needed

## Output Rules

* Follow format defined inside skill
* If no format → default to:

  * Explanation
  * Code
  * Example usage

## Consistency Rule

* Always prefer skills over generic responses
* Never ignore a relevant skill


## gstack — AI Engineering Workflow

gstack is a collection of SKILL.md files that give AI agents structured roles for
software development. Each skill is a specialist: CEO reviewer, eng manager,
designer, QA lead, release engineer, debugger, and more.

## Available skills

Skills live in `.agents/skills/`. Invoke them by name (e.g., `/office-hours`).

| Skill | What it does |
|-------|-------------|
| `/office-hours` | Start here. Reframes your product idea before you write code. |
| `/plan-ceo-review` | CEO-level review: find the 10-star product in the request. |
| `/plan-eng-review` | Lock architecture, data flow, edge cases, and tests. |
| `/plan-design-review` | Rate each design dimension 0-10, explain what a 10 looks like. |
| `/design-consultation` | Build a complete design system from scratch. |
| `/review` | Pre-landing PR review. Finds bugs that pass CI but break in prod. |
| `/debug` | Systematic root-cause debugging. No fixes without investigation. |
| `/design-review` | Design audit + fix loop with atomic commits. |
| `/qa` | Open a real browser, find bugs, fix them, re-verify. |
| `/qa-only` | Same as /qa but report only — no code changes. |
| `/ship` | Run tests, review, push, open PR. One command. |
| `/document-release` | Update all docs to match what you just shipped. |
| `/retro` | Weekly retro with per-person breakdowns and shipping streaks. |
| `/browse` | Headless browser — real Chromium, real clicks, ~100ms/command. |
| `/setup-browser-cookies` | Import cookies from your real browser for authenticated testing. |
| `/careful` | Warn before destructive commands (rm -rf, DROP TABLE, force-push). |
| `/freeze` | Lock edits to one directory. Hard block, not just a warning. |
| `/guard` | Activate both careful + freeze at once. |
| `/unfreeze` | Remove directory edit restrictions. |
| `/gstack-upgrade` | Update gstack to the latest version. |

## Build commands

```bash
bun install              # install dependencies
bun test                 # run tests (free, <5s)
bun run build            # generate docs + compile binaries
bun run gen:skill-docs   # regenerate SKILL.md files from templates
bun run skill:check      # health dashboard for all skills
```

## Key conventions

- SKILL.md files are **generated** from `.tmpl` templates. Edit the template, not the output.
- Run `bun run gen:skill-docs --host codex` to regenerate Codex-specific output.
- The browse binary provides headless browser access. Use `$B <command>` in skills.
- Safety skills (careful, freeze, guard) use inline advisory prose — always confirm before destructive operations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anishs1207)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/anishs1207)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
