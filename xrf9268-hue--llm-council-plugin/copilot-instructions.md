## llm-council-plugin

> Subagents operate in separate context windows, preventing context pollution and enabling longer sessions.

# LLM Council Plugin - Subagent Definitions

Subagents operate in separate context windows, preventing context pollution and enabling longer sessions.

## Council Chairman Agent

**Location**: `agents/council-chairman.md`

**Purpose**: Synthesizes multi-model responses into unified decision during Phase 3.

**Model**: `claude-opus-4-5-20251101` (maximum reasoning for synthesis/arbitration)

**Tools**: Read, Write

**Input**: Original question + `.council/` directory with stage1/stage2 responses

**Output**: `.council/final_report.md` with consensus, disagreements, and recommendations

**Constraints**:
- Remain neutral (no vendor favoritism)
- Base judgments on technical merit only
- Handle missing members gracefully

---

## Adding New Subagents

1. Create `agents/agent-name.md` with frontmatter:
   ```markdown
   ---
   name: agent-name
   description: Brief description. Use when you need [specific use case].
   model: claude-sonnet-4-5-20250929
   tools: Read, Write
   ---

   [System prompt here...]
   ```

2. Register in `.claude-plugin/plugin.json`:
   ```json
   { "agents": ["./agents/council-chairman.md", "./agents/agent-name.md"] }
   ```

3. Validate: `claude plugin validate .`

---

## Best Practices

- **Models**: Opus 4.5 (complex), Sonnet 4.5 (balanced), Haiku 4.5 (fast/simple)
- **Tools**: Limit to minimum required; `Read, Write` for analysis-only
- **Descriptions**: Include "Use when you need..." trigger phrase

## References

- [Subagents](https://code.claude.com/docs/en/sub-agents)
- [Memory Management](https://code.claude.com/docs/en/memory)

---
> Source: [xrf9268-hue/llm-council-plugin](https://github.com/xrf9268-hue/llm-council-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
