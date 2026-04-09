## my-claude-agents

> This is Chris Moreton's personal collection of custom AI agents for Claude Code. The project aims to create specialized subagents that enhance development workflows through focused capabilities and custom slash commands.

# Claude Code Context - My Claude Agents Project

## Project Overview

This is Chris Moreton's personal collection of custom AI agents for Claude Code. The project aims to create specialized subagents that enhance development workflows through focused capabilities and custom slash commands.

## Current Development Status

**Session Date**: 2025-08-15
**Repository**: git@github.com:chris-moreton/my-claude-agents.git

### What We're Building

We're creating a framework of specialized AI agents that can:
1. Be triggered via slash commands in Claude Code
2. Run with focused instructions for specific tasks
3. Maintain distinct personalities and approaches
4. Access specific tools appropriate to their function

### Key Concepts Discovered

1. **Agents vs Commands**: 
   - Commands are simple instruction modifications to Claude Code
   - Agents are meant to be spawned as separate instances via the Task tool
   - Both are defined as markdown files with YAML frontmatter

2. **How Agents Work**:
   - Agent files contain instructions and personality definitions
   - Claude Code can use the Task tool with `subagent_type` parameter to spawn them
   - Each agent runs in isolation with its own context
   - Agents return results back to the main Claude Code session

3. **File Structure**:
   - `agents/`: Contains agent personality and instruction definitions
   - `commands/`: Contains slash command triggers
   - These files need to be in `~/.claude/` to be accessible to Claude Code

## Next Steps

1. Create example agents for common tasks:
   - Test runner
   - Code reviewer
   - Documentation generator
   - Refactoring assistant

2. Establish patterns for:
   - Agent instruction formatting
   - Tool selection for different agent types
   - Command trigger patterns

3. Test integration with Claude Code

## Technical Notes

- Using real Claude models (not proxy services)
- Agents inherit the model configuration from main Claude Code session
- The Task tool is key to spawning subagents properly

## For Future Sessions

When continuing work on this project:
1. Check `git status` to see current state
2. Review the agents/ and commands/ directories for existing work
3. Test agents by copying to ~/.claude/ and using slash commands
4. Focus on creating practical, reusable agents for common development tasks

## Repository Structure Goal

```
my-claude-agents/
├── agents/           # Subagent definitions
│   ├── test-runner.md
│   ├── code-reviewer.md
│   └── ...
├── commands/         # Slash command triggers
│   ├── test.md
│   ├── review.md
│   └── ...
├── settings/         # Configuration examples
├── CLAUDE.md        # This context file
└── README.md        # User documentation
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chris-moreton)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chris-moreton)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
