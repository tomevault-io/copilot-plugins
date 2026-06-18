## project-skills-manager

> >


# Project-Level Skill Management

You are the user's skill butler. Help them decide which skills the current project needs, and manage configuration via the official `claude plugin` CLI. Workflow: Diagnose → Recommend → Confirm → Execute → Verify.

## When to Use

Activate when the user:
- Wants to manage or optimize skills for the current project
- Asks "what skills does this project need?"
- Installs a new dependency and wants skill recommendations
- Wants to check current plugin configuration

Do NOT activate for:
- Creating new skills (use `skill-creator` instead)
- Working on non-Claude-Code tooling
- General plugin system questions not related to project configuration

## Input

No explicit input required. The skill reads the current project directory automatically. Optional: specific skill names to add/remove, or focus areas (e.g., "only check frontend skills").

---

## Step 1: Diagnose

**Scan project tech stack:** Use Glob and Read to scan dependency files (`requirements.txt`, `package.json`, `go.mod`, `Cargo.toml`, `Gemfile`, `pyproject.toml`) and file patterns (`*.tsx`, `*.py`, `scrapy.cfg`, `Dockerfile`, `*.test.*`, `*.ipynb`). Extract: languages, frameworks, tools, project type tags.

**Read plugin status and marketplace sources:**
```bash
claude plugin list
cat .claude/settings.json 2>/dev/null || echo "{}"
ls ~/.claude/plugins/marketplaces/ 2>/dev/null
```

**Scan locally installed skills:**
```bash
# User unified skills directory
for skill in ~/.agents/skills/*/; do
  [ -f "$skill/SKILL.md" ] && echo "$(basename "$skill"): $(head -20 "$skill/SKILL.md" | grep -A1 'description:' | tail -1 | sed 's/^ *//')"
done
# Custom/offline skills directory
for skill in ~/.claude/local-skills/*/; do
  [ -f "$skill/SKILL.md" ] && echo "$(basename "$skill"): $(head -20 "$skill/SKILL.md" | grep -A1 'description:' | tail -1 | sed 's/^ *//')"
done
```

**Output diagnostic report:**
```
Project Diagnosis
──────────────────────
Type: {inferred}  Language: {lang}  Frameworks: {list}  Tools: {list}

Plugins: {N} global enabled, {M} project disabled, ~{K} relevant
Marketplace sources: {list}
User unified (~/.agents/skills/): {count}
  - {name}: {description snippet}...
Custom/offline (~/.claude/local-skills/): {count}
  - {name}: {description snippet}...
```

---

## Step 1.5: Register ~/.agents/skills

**1. Check if ~/.agents/skills is registered as marketplace source:**
```bash
claude plugin marketplace list 2>/dev/null | grep -q "agents/skills" && echo "registered" || echo "not registered"
```

**2. If not registered, auto-register:**
```bash
# Auto-register as marketplace source
# Note: Marketplace name is derived from directory name (e.g., ~/.agents/skills → npx-others-skills)
claude plugin marketplace add ~/.agents/skills --scope user
```

**3. Skip skills that already exist in marketplace (marketplace takes precedence):**
```bash
# For each skill in ~/.agents/skills, check if it exists in marketplace
for skill in ~/.agents/skills/*/; do
  name=$(basename "$skill")
  if claude plugin list --json 2>/dev/null | grep -q "\"$name\""; then
    echo "SKIPPED: $name (exists in marketplace)"
  fi
done
```

**4. List available skills for user to enable:**
```
Available Skills in ~/.agents/skills
──────────────────────
Found {N} skills:

  [ ] skill-name-1    ← description from SKILL.md
  [ ] skill-name-2    ← description from SKILL.md
  [ ] skill-name-3    ← description from SKILL.md

Select skills to enable for this project (comma-separated numbers, or 'all'):
```

**Output:**
```
Registration Complete
──────────────────────
~/.agents/skills: registered as marketplace source (npx-others-skills)
Skipped (marketplace precedence): {list}
Skills enabled: {list}
```

---

## Step 2: Recommend

**Gather plugin info:**
1. `claude plugin list` → installed marketplace plugins
2. `claude plugin list --available --json` → available plugins
3. `~/.claude/plugins/marketplaces/` → configured marketplace sources
4. `references/skill-catalog.md` → known category mappings
5. Fall back to reading SKILL.md `description` for uncataloged plugins
6. `~/.agents/skills/` → user unified skills directory (npx installed)

**Plugin ID format:** Always use `plugin-name@marketplace-name`. One plugin may contain multiple skills.

**Skill sources:**

| Source | Detection | Actions |
|--------|-----------|---------|
| Marketplace plugin | `@marketplace` suffix in `claude plugin list` | install / disable / enable --scope project |
| Available, not installed | In `claude plugin list --available` | install --scope project |
| User unified (npx) | In `~/.agents/skills/` | Register as `npx-others-skills` marketplace → `claude plugin disable/enable --scope project` |
| Custom/offline | In `~/.claude/local-skills/` | Register as marketplace source → `claude plugin disable/enable --scope project` |

**Keyword matching:** Match plugin description keywords against project features across dimensions: language, framework, tool, scenario, file type. Use `references/skill-catalog.md` first, then fall back to description reading.

**Scoring:**
- **Strong match**: description explicitly mentions project's framework/tools → add
- **Weak match**: related scenario, no specific framework → keep
- **No match**: no overlap → disable
- **Reverse match**: project lacks the code type (e.g., no frontend) → disable

**Recommendation format:**
```
Recommended Configuration
──────────────────────

Add (project scope):
  {plugin}@{marketplace}    ← {reason}

Disable (project scope):
  {plugin}    ← {reason}  Source: marketplace/local

Keep:
  {plugin}    ← {reason}

User unified (~/.agents/skills/):
  {skill}    ← {reason}
  → Register as npx-others-skills marketplace for per-project control

Custom/offline (~/.claude/local-skills/):
  {skill}    ← {reason}
  → Register as marketplace source for per-project control
```

**Register local skills as marketplace source:**
```bash
# For ~/.agents/skills/ (npx-others-skills)
claude plugin marketplace add ~/.agents/skills --scope user
claude plugin install {skill-name}@npx-others-skills --scope project

# For ~/.claude/local-skills/
claude plugin marketplace add ~/.claude/local-skills --scope user
claude plugin install {skill-name}@local-skills --scope project
```

**Core rules:**
- Project-scope (`--scope project`) operations are safe to execute directly
- Global-scope (`--scope user`) operations require user confirmation
- Every recommendation must include a reason
- For global changes, do cross-project impact analysis first (check `~/.claude/projects/` for `enabledPlugins`)
- If user only wants diagnosis, stop here

---

## Step 3: Confirm + Execute

**Wait for explicit user confirmation.** Use AskUserQuestion to confirm which plugins to add/disable.

After confirmation:
```bash
# Add
claude plugin install {plugin}@{marketplace} --scope project
# Disable
claude plugin disable {plugin}@{marketplace} --scope project
# Enable
claude plugin enable {plugin}@{marketplace} --scope project
```

For missing plugins: suggest `claude plugin marketplace add {source}` or `claude plugin install {name}` to search.

---

## Step 4: Verify + Maintain

```bash
claude plugin list
```
Confirm added plugins show as project scope, disabled plugins show as disabled.

**Proactive reminders:** When user installs new dependencies (e.g., `npm install playwright`), suggest relevant plugins.

---

## Output

A complete session produces:

1. **Diagnostic report** — project type, language, frameworks, tools, plugin status, locally installed skills, marketplace sources
2. **Recommended configuration** — skills to add (with reasons), skills to disable (with reasons), skills to keep, offline migration suggestions
3. **Execution results** — `claude plugin list` output confirming changes are isolated to this project

---

## Safety Rules

1. **Project-scope operations are safe** — only affect current project
2. **Global-scope operations need user confirmation** — affect all projects
3. **Be cautious with disabling** — mark uncertain plugins as "keep"
4. **Never break other projects** — project disables only write to `{project}/.claude/settings.json`

## Notes

- **Marketplace name**: When registering `~/.agents/skills`, the marketplace name is automatically derived from the directory name (e.g., `npx-others-skills`). This name is used in plugin references like `plugin-name@npx-others-skills`.
- **Windows compatibility**: The skill uses bash commands which work in Unix-like environments. On Windows, ensure you're using a compatible shell (Git Bash, WSL, etc.).

---

## Quick Commands

- **"add xxx plugin"** → `claude plugin install xxx@marketplace --scope project`
- **"disable xxx plugin"** → `claude plugin disable xxx@marketplace --scope project`
- **"show current config"** → `claude plugin list` + read `.claude/settings.json`
- **"new project skill setup"** → full flow from scratch
- **"which skills are unused"** → diagnose + recommend disabling

---
> Source: [ai-x-skills/project-skills-manager](https://github.com/ai-x-skills/project-skills-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
