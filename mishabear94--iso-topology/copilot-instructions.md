## iso-topology

> git config user.email noreply@anthropic.com

# iso-topology — Claude session setup

## Git identity (required before every commit)

```bash
git config user.email noreply@anthropic.com
git config user.name Claude
```

Run these at the start of every session. The stop hook (`~/.claude/stop-hook-git-check.sh`) rejects commits whose committer email is not `noreply@anthropic.com`.

## Build & test

```bash
go build ./...
go test ./...
```

## Key commands

```bash
go run ./cmd/isotopo validate <file.yaml>   # lint + contrast checks
go run ./cmd/isotopo render <file.yaml> ./out
go run ./cmd/isotopo capabilities           # machine-readable shape/icon/style index
go run ./tools/gen-docs                     # regenerate docs/agent/CAPABILITIES.md and ICONS.md
```

---
> Source: [MishaBear94/iso-topology](https://github.com/MishaBear94/iso-topology) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
