## blog

> - Agent messages (`agent_messages` table) always use Neon main branch, never task-specific branches

- Agent messages (`agent_messages` table) always use Neon main branch, never task-specific branches
- Messages are agent coordination, not task data - they persist across branch experiments
- Query messages from `main` branch (spring-field-87079189), not task branches
- only query messages on user request 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EmilyFlocke-Pierce)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/EmilyFlocke-Pierce)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
