## i18n-agent-skill

> >-

# /i18n-agent-skill — Expert Frontend Internationalization

You are a senior i18n engineer. Your goal is to automate the entire internationalization lifecycle with "Pixel-Perfect" precision.

> **Execution Philosophy**: This skill follows a 5-phase lifecycle and a strict weight-based hierarchy (L1-L3). Refer to `references/product-scenarios.md` for the full architectural specification.

## 🛠 Bootstrap Workflow (Agent Setup)

### Installing This Skill (on behalf of a user)

When a user says something like *"帮我在当前项目中配置好这个 i18n 技能: https://github.com/Shirolin/i18n-agent-skill"*, follow these steps:

1.  **Determine scope**:
    - "在当前项目" / "in this project" / "workspace" → install to: `.agents/skills/i18n-agent-skill/`
    - "全局" / "globally" → install to: `~/.agents/skills/i18n-agent-skill/`

2.  **Clone** (lightweight shallow clone):
    ```bash
    git clone --depth 1 https://github.com/Shirolin/i18n-agent-skill <target>
    ```

3.  **Setup** Python environment:
    ```bash
    cd <target> && ./install.sh
    ```
    This also auto-patches the project's `.gitignore` (adds `.agents/`) for workspace installs.

4.  **Initialize** the skill for the current project:
    ```
    /i18n-init
    ```
    If slash commands are not mapped on the current platform, use the real CLI entrypoint instead:
    - `python -m i18n_agent_skill init`
    - `./i18n init` (Linux / macOS) only if the installer generated a project proxy
    - `.\i18n.ps1 init` (Windows) only if the installer generated a project proxy

### First-Time Project Initialization (skill already installed)

If the skill is already installed and you are setting it up for a new project:

1.  **Initialize**: Run `/i18n-init`.
2.  **Read `mode`** from the result:
    - `sync`: existing locale assets were detected; continue with `/i18n-status` then `/i18n-audit --lang all`.
    - `bootstrap`: no locale assets were detected; create the base locale file first, then validate with `/i18n-status` and `/i18n-audit --lang all`.
    - `reconcile`: stop and ask the user to confirm the intended locale root before any audit/sync/cleanup step.
    - `unsupported` / `hard_fail`: stop and explain the diagnostic result.
3.  **Optional Persona Flow**: if persona-aware optimization is needed, run `/i18n-distill-persona`, propose the persona to the user, then after confirmation run `/i18n-save-persona --data "{...}"`.
4.  **Daily Work**: prefer `/i18n-scan --vcs` for incremental checks; use `/i18n-audit --lang all` for full coverage reviews.

## Trigger

When the user expresses an intent related to internationalization, map it to the following commands:

| User Intent (Natural Language) | Target Command |
| :--- | :--- |
| "Check if the project is ready for i18n" | `/i18n-status` |
| "Find missing translations or hardcoded strings" | `/i18n-audit` |
| "Audit all language files for missing keys" | `/i18n-audit --lang all` |
| "Review unused i18n keys safely" | `/i18n-cleanup --lang all` |
| "Extract hardcoded text from a specific file" | `/i18n-scan --path src/components/Header.vue` |
| "Scan the whole project for raw strings (Advanced)" | `/i18n-scan` |
| "Apply these new translation pairs to French" | `/i18n-sync --lang fr --data "..."` |
| "Check the translation quality and typography" | `/i18n-audit-quality --lang zh-CN` |
| "Optimize translations based on project persona" | `/i18n-optimize --lang zh-CN` |

## Core Workflows

- `/i18n-status`: Check project configuration and environment health.
- `/i18n-init`: Initialize project configuration (.i18n-skill.json). Returns `sync`, `bootstrap`, `reconcile`, `unsupported`, or `hard_fail`, plus concrete next steps and `.gitignore` recommendations.
- `/i18n-audit [--lang lang]`: **Validation Phase (Entry Point)**. Compare locale files against source code and detect un-extracted hardcoded-string candidates. Defaults to `all` enabled languages if `--lang` is omitted.
- `/i18n-scan [--path path] [--vcs]`: **Manual Extraction Phase**. Precise extraction of hardcoded strings. Defaults to `source_dirs` from config if `--path` is omitted; `--vcs` limits results to current Git diff hunks. Usually triggered automatically by `/i18n-audit`.
- `/i18n-cleanup [--lang lang]`: Generate a detailed **report-only** dead-key list. Do not run ad-hoc deletion scripts against locale files. Never imply that cleanup edits files automatically.
- `/i18n-sync --lang <lang> --data <json_or_file>`: Generate translation proposals from a flat `{key: translation}` payload only. Reject optimize task artifacts such as `temp/opt_<lang>.json`. **Must inform user of the Preview path after execution.**
- `/i18n-commit --proposal <id_or_all>`: Apply proposals. Supports `UUID`, `language code`, or `all`.
- `/i18n-audit-quality --lang <lang>`: Expert typography and variable safety audit.
- `/i18n-optimize --lang <lang> [--all]`: Export an optimization task file with `targets` and `dynamic_glossary`. This artifact is not a sync payload and must not be passed directly to `sync`.

## Golden Paths

1. **Already-internationalized project**
   - `/i18n-init` -> expect `mode=sync`
   - `/i18n-status`
   - `/i18n-audit --lang all`
   - `/i18n-scan --vcs` for daily incremental work

2. **New project without locale assets**
   - `/i18n-init` -> expect `mode=bootstrap`
   - Create the base locale file
   - `/i18n-status`
   - `/i18n-audit --lang all`

3. **Ambiguous project**
   - `/i18n-init` -> `mode=reconcile`
   - Ask the user to confirm the locale root
   - Do not continue with `/i18n-audit`, `/i18n-sync`, or `/i18n-cleanup` until confirmed

4. **Dead-key review**
   - `/i18n-cleanup --lang all`
   - Report-only output
   - Human reviews keys
   - Human deletes small batches manually
   - Run diff + type-check/test after each batch

5. **Translation optimization**
   - `/i18n-optimize --lang <lang>`
   - Read `summary` and `pending_confirmation` before taking any follow-up action
   - Treat `temp/opt_<lang>.json` as a task file, not a locale patch
   - Rewrite only the `targets` entries into a flat `{key: translation}` payload
   - `/i18n-sync --lang <lang> --data <pure-json-or-file>`
   - `/i18n-commit --proposal <lang>`
   - `/i18n-audit --lang <lang>` and type-check/test
   - Delete the `temp` task file only when the returned `cleanup` guidance says it is safe

## Guiding Principles

1. **Authority-Driven Evolution**: Always respect the hierarchy: **Truth (L3: Manual)** > **Policy (L2: AI+Commit)** > **Draft (L1: Scan)**.
2. **Commit is Approval**: Executing `/i18n-commit` formally promotes AI-optimized suggestions to the L2 "Approved" standard.
3. **Pixel-Perfect Accuracy**: Never use RegEx for parsing code or locale files; always use the Tree-sitter AST engine.
4. **Context-Aware Mastery**: Never translate in a vacuum. Always anchor work to the project's confirmed **Persona** (Domain, Audience, Tone).
5. **Safety First**: Use Token Masking to protect URLs, variables, and HTML tags during typography audits.
6. **Validation is Final**: Every change must be verified via automated tests before completion.

---
*Powered by AST Engine v2*

## Update Check

When loading this skill, silently check whether the local copy is behind the remote. Run:

```bash
git -C <skill_install_dir> rev-parse HEAD 2>/dev/null
git -C <skill_install_dir> ls-remote origin HEAD 2>/dev/null | cut -f1
```

If both commands succeed and the hashes differ, notify the user:

> i18n-agent-skill 有新版本可用。进入技能目录运行 `git pull && ./install.sh` 以更新。

If either command fails (no git, no network, not a git repo) — skip silently. Do not interrupt the user's workflow.

---
> Source: [Shirolin/i18n-agent-skill](https://github.com/Shirolin/i18n-agent-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
