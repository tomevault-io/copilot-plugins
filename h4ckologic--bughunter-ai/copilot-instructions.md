## bughunter-ai

> This project is a Claude Code skill for autonomous bug bounty hunting.

# BugHunter AI — Claude Code Configuration

This project is a Claude Code skill for autonomous bug bounty hunting.

## Key Files
- `skills/BugBountyFramework/SKILL.md` — Main skill definition (loaded by Claude Code)
- `skills/BugBountyFramework/Agents/*.md` — 20 specialized vulnerability agents
- `skills/BugBountyFramework/Tools/*.ts` — TypeScript tools (Bun runtime)

## How It Works
When a user types `hunt <target>`, Claude Code loads SKILL.md which orchestrates a 10-phase hunt:
1. State machine initialization (hunt-orchestrator.ts)
2. Credential loading from vault (credential-vault.ts)
3. Authentication flow (auth-manager.ts)
4. Application profiling via Playwright
5. Reconnaissance
6. Parallel agent deployment (each agent is a .md file)
7. Dynamic testing via Burp + Playwright
8. Vulnerability assessment
9. Learning & pattern update
10. Report generation

## Development
- Tools are TypeScript/Bun. Run with `bun <tool>.ts --help`
- Agents are Markdown files with structured instructions
- Test against your own targets or sanctioned bug bounty programs only

---
> Source: [h4ckologic/bughunter-ai](https://github.com/h4ckologic/bughunter-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
