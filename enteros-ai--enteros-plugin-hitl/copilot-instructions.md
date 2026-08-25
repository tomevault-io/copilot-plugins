## enteros-plugin-hitl

> `molecule-hitl` is a **human-in-the-loop policy plugin** that wraps

# molecule-hitl — Human-in-the-Loop Gates

`molecule-hitl` is a **human-in-the-loop policy plugin** that wraps
`builtin_tools/hitl.py`. It provides an `@requires_approval` decorator,
`pause_task`/`resume_task` tools, multi-channel notification (dashboard/Slack/email),
and RBAC bypass roles.

**Version:** 1.0.0
**Runtime:** `claude_code`

---

## Repository Layout

```
molecule-hitl/
├── plugin.yaml              — Plugin manifest
├── skills/
│   └── hitl-gates/
│       └── SKILL.md         — When/how to call request_approval, pause_task, resume_task
└── builtin_tools/           — (harness-provided, not in this repo)
    └── hitl.py              — @requires_approval decorator + tools
```

---

## What It Does

### The @requires_approval Decorator

Wrap any async callable with `@requires_approval`. Before executing, the agent
pauses and sends a notification to the configured channel. A human must approve
or deny before the callable proceeds.

### Tools

| Tool | Purpose |
|---|---|
| `request_approval` | Trigger a pause + notification for a specific action |
| `pause_task` | Pause the current task pending human input |
| `resume_task` | Resume a paused task with or without modifications |

### RBAC Bypass Roles

Certain roles bypass HITL gates entirely. Configure via workspace RBAC settings.

---

## Configuration

In workspace `config.yaml`:

```yaml
hitl:
  enabled: true
  channels:
    dashboard: true
    slack: true        # requires SLACK_WEBHOOK_URL secret
    email: false
  default_policy: warn   # warn | block
```

---

## Development

### Prerequisites

- Python 3.11+
- `gh` CLI authenticated
- Write access to `Molecule-AI/molecule-ai-plugin-molecule-hitl`

### Setup

```bash
git clone https://git.moleculesai.app/molecule-ai/molecule-ai-plugin-molecule-hitl.git
cd molecule-ai-plugin-molecule-hitl

# YAML validation
python3 -c "import yaml; yaml.safe_load(open('plugin.yaml'))"
```

### Pre-Commit Checklist

```bash
# YAML structure
python3 -c "import yaml; yaml.safe_load(open('plugin.yaml'))"

# Credential scan
python3 -c "
import re, sys
with open('plugin.yaml') as f:
    content = f.read()
patterns = [r'sk.ant', r'ghp.', r'AKIA[A-Z0-9]']
if any(re.search(p, content) for p in patterns):
    print('FAIL: possible credentials found')
    sys.exit(1)
print('No credentials: OK')
"
```

---

## Release Process

1. Review changes: `git log origin/main..HEAD --oneline`
2. Bump `version` in `plugin.yaml` (semver)
3. Commit: `chore: bump version to X.Y.Z`
4. Tag and push: `git tag vX.Y.Z && git push origin main --tags`
5. Create GitHub Release with changelog

---

## Known Issues

See `known-issues.md` at the repo root.

---
> Source: [EnterOS-AI/enteros-plugin-hitl](https://github.com/EnterOS-AI/enteros-plugin-hitl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
