## davidshaevel-marketplace

> <!-- If CLAUDE.local.md exists, read it for additional context (account IDs, resource details, etc.) -->

# davidshaevel-marketplace - Claude Context

<!-- If CLAUDE.local.md exists, read it for additional context (account IDs, resource details, etc.) -->

## Project Overview

Personal Claude Code plugin providing development conventions, skills, and project templates. It standardizes development workflows across all of David Shaevel's projects by injecting conventions at session start, providing reusable skills, and offering project bootstrapping templates.

**Key Technologies:**
- **Languages:** Shell (Bash/Zsh), Markdown
- **Platform:** Claude Code Plugin System
- **Tooling:** jq, gh CLI, direnv

**Project Management:**
- **Issue Tracking:** Linear (Team Tacocat)
- **Version Control:** GitHub
- **Branching Strategy:** Feature branches with PR workflow

---

## Architecture

```
Plugin Load Flow:
  Claude Code starts
    → Loads .claude-plugin/plugin.json
    → Registers skills, commands, hooks
    → session-start hook fires
      → hooks/session-start.sh executes
      → Injects conventions/development-standards.md into context
    → Skills and commands available for the session
```

---

## Repository Structure

```
davidshaevel-marketplace/
│
├── .bare/                             # Bare git repository
├── .git                               # Points to .bare
│
├── main/                              # Main branch worktree
│   ├── .claude-plugin/                # Plugin manifest
│   │   ├── plugin.json                # Plugin definition (skills, hooks, commands)
│   │   └── marketplace.json           # Marketplace metadata
│   │
│   ├── commands/                      # Slash commands
│   │   ├── bootstrap-project.md       # /bootstrap-project command
│   │   └── resolve-code-review.md     # /resolve-code-review command
│   │
│   ├── conventions/                   # Auto-injected conventions
│   │   └── development-standards.md   # Git, PR, worktree, Linear conventions
│   │
│   ├── config/                        # Plugin configuration
│   │   ├── backup-config.json         # Backup config (gitignored, repo-specific)
│   │   └── backup-config.json.example # Backup config template (committed)
│   │
│   ├── scripts/                       # Plugin scripts
│   │   └── backup-local-config.sh     # Backs up gitignored files to Google Drive
│   │
│   ├── hooks/                         # Plugin hooks
│   │   ├── hooks.json                 # Hook definitions
│   │   ├── session-start.sh           # Session start hook script
│   │   └── run-hook.cmd               # Windows hook runner
│   │
│   ├── skills/                        # Plugin skills
│   │   ├── bootstrap-project/SKILL.md # Project initialization skill
│   │   ├── resolve-code-review/SKILL.md # Code review resolution skill
│   │   ├── session-handoff/SKILL.md   # Session handoff skill
│   │   └── backup-local-config/SKILL.md # Local file backup skill
│   │
│   ├── templates/                     # Project templates
│   │   ├── CLAUDE.md.template         # CLAUDE.md template
│   │   ├── CLAUDE.local.md.template   # CLAUDE.local.md template
│   │   ├── CLAUDE.local.md.example    # Example CLAUDE.local.md
│   │   ├── SESSION_LOG.md.template    # SESSION_LOG.md template
│   │   ├── cursorrules.template       # .cursorrules template
│   │   └── gitignore-additions.txt    # Gitignore entries for agent files
│   │
│   ├── CLAUDE.md                      # Public project context (this file)
│   ├── CLAUDE.local.md                # Sensitive project context (gitignored)
│   ├── SESSION_LOG.md                 # Cross-agent memory (gitignored)
│   ├── README.md                      # Public documentation
│   └── LICENSE                        # MIT License
│
└── <feature-worktrees>/               # Feature branch worktrees (flat!)
```

---

## Important File Locations

| File | Purpose |
|------|---------|
| `.claude-plugin/plugin.json` | Plugin manifest — defines skills, commands, hooks |
| `conventions/development-standards.md` | Development conventions injected at session start |
| `config/backup-config.json` | Backup destination and file list configuration |
| `scripts/backup-local-config.sh` | Shell script that backs up gitignored files via rclone |
| `hooks/session-start.sh` | Hook that injects conventions into Claude context |
| `skills/*/SKILL.md` | Skill definitions (bootstrap, code review, session handoff) |
| `skills/backup-local-config/SKILL.md` | On-demand backup skill definition |
| `templates/*` | Project bootstrap templates |

---

## Helpful Commands

```bash
# Test the session-start hook locally
bash hooks/session-start.sh

# Check plugin structure
cat .claude-plugin/plugin.json | jq .

# Version bump (update plugin.json version field)
jq '.version = "X.Y.Z"' .claude-plugin/plugin.json > tmp && mv tmp .claude-plugin/plugin.json
```

---

## Environment Variables

No environment variables required for the plugin itself. Individual projects bootstrapped by this plugin use `.envrc` with direnv.

---

## References

- **GitHub Repository:** [davidshaevel-marketplace](https://github.com/davidshaevel-dot-com/davidshaevel-marketplace)
- **Linear Project:** [Team Tacocat](https://linear.app/davidshaevel-dot-com)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davidshaevel-dot-com)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/davidshaevel-dot-com)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
