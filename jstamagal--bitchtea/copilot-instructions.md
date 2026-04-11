## bitchtea

> Read AGENTS.md for full project context, module map, and build commands.

# Gemini CLI Instructions

Read AGENTS.md for full project context, module map, and build commands.

This project uses **bd (beads)** for issue tracking.

```bash
bd ready              # Find available work
bd show <id>          # View issue details  
bd update <id> --claim  # Claim work
bd close <id>         # Complete work
```

After completing work: `go build ./... && go test ./... && go vet ./...` then commit and push.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jstamagal)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jstamagal)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
