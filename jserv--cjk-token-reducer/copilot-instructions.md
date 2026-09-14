## cjk-token-reducer

> Add the following to your Claude Code settings file (usually `~/.claude/settings.json`).

# Claude Code

## Configure Claude Code hook
Add the following to your Claude Code settings file (usually `~/.claude/settings.json`).
This hook intercepts your prompt before submission.

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "cjk-token-reducer"
          }
        ]
      }
    ]
  }
}
```

The tool accepts JSON input `{"prompt": "..."}` on stdin and outputs modified JSON.

## How It Works
The hook intercepts at `UserPromptSubmit`, translating CJK prompts before Claude processes them:

```
┌──────────────────────────────────────────────────────────────┐
│                      Claude Code Session                     │
├──────────────────────────────────────────────────────────────┤
│  SessionStart ─────► User types prompt (CJK)                 │
│                           │                                  │
│                           ▼                                  │
│              ┌────────────────────────────┐                  │
│              │    UserPromptSubmit        │                  │
│              │  ┌──────────────────────┐  │                  │
│              │  │  cjk-token-reducer   │  │ ◄─ Intercept     │
│              │  │  - Detect CJK        │  │                  │
│              │  │  - Check cache       │  │                  │
│              │  │  - Translate → EN    │  │                  │
│              │  │  - Preserve code     │  │                  │
│              │  └──────────────────────┘  │                  │
│              └────────────────────────────┘                  │
│                           │                                  │
│                           ▼                                  │
│                   Claude processes (English prompt)          │
│                           │                                  │
│                   ┌───────┴───────┐                          │
│                   ▼               ▼                          │
│              PreToolUse      (No tools)                      │
│                   │               │                          │
│                   ▼               │                          │
│              Tool executes        │                          │
│                   │               │                          │
│                   ▼               │                          │
│              PostToolUse          │                          │
│                   │               │                          │
│                   └───────┬───────┘                          │
│                           ▼                                  │
│                        Stop                                  │
└──────────────────────────────────────────────────────────────┘
```

---
> Source: [jserv/cjk-token-reducer](https://github.com/jserv/cjk-token-reducer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
