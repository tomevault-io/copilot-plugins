## gcp-project-deny-policies

> - For external search of latest GCP documentation, prefer to use the context7 mcp.

- For external search of latest GCP documentation, prefer to use the context7 mcp.

## Active Technologies
- Bash 4.0+ (POSIX-compliant shell scripting) + gcloud CLI (Google Cloud SDK) - ONLY dependency per constitution (001-fetch-enabled-apis)
- Local filesystem for output file (plain text) (001-fetch-enabled-apis)
- Bash 4.0+ + gcloud CLI, curl/wget for web fetching, jq for JSON parsing (002-deny-policy)
- Local filesystem (deny-policies/policies/ for JSON, deny-policies/commands/ for shell scripts, results/ for input) (002-deny-policy)
- Bash 4.0+ (POSIX-compliant shell scripting) + Standard Unix utilities (rm, find, test), gcloud CLI (already present) (003-cleanup-dirs)
- Local filesystem (targets: deny-policies/commands/, deny-policies/policies/, results/) (003-cleanup-dirs)

## Recent Changes
- 001-fetch-enabled-apis: Added Bash 4.0+ (POSIX-compliant shell scripting) + gcloud CLI (Google Cloud SDK) - ONLY dependency per constitution
- allow a max timeout of 5 min when performing a gcloud cli request

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JairAviles)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JairAviles)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
