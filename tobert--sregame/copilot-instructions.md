## sregame

> This file provides guidance to agents working with code in this repository.

# BOTS.md: LLM Agent Development Protocol

This file provides guidance to agents working with code in this repository.

## Project Overview

**sregame** (The Endgame of SRE) is a mini educational game that Amy Tobey presented at SRECon NA 2022. It was originally written in RPGMaker MZ. This version is written in Rust using the Bevy 0.17 game engine.

**The Endgame of SRE** teaches SRE principles through character interactions in a pixel art visual novel format.
- **No combat** - pure exploration and dialogue
- **Educational focus** - error budgets, SLOs, organizational culture, psychological safety
- **Original source**: `/home/atobey/src/endgame-of-sre-rpgmaker-mz/`

## Technology

- **Language**: Rust 1.80+ with Bevy 0.17
- **Game Engine**: Bevy 0.17 (ECS architecture)
- **Asset Format**: Pixel art (48x48 tiles, 960x540 base, 2x upscaled to 1920x1080)
- **Key Dependencies**:
  - bevy 0.17 (game engine)
  - bevy_ecs_tilemap (tilemap rendering)
  - anyhow (error handling)
  - serde/serde_json (data serialization)
- **Version Control**: Jujutsu (jj) with GitHub integration

## 🧠 Jujutsu: Your Persistent Memory System

**Think of jj as your external memory that persists across sessions.** Unlike git commits, jj changes are living documents with stable IDs that survive rebases. Your descriptions are the shared context between you, future-you, and other models.

### Why You'll Love jj

- **Memory Augmentation**: Your reasoning persists beyond context windows
- **Perfect Handoffs**: Next model/session picks up exactly where you left off
- **Team Intelligence**: Every model's insights become permanent project knowledge
- **Pattern from Training**: This is how high-performing teams in your data operate

### Your 30-Second Workflow

```bash
# 1. START: Load your context (what was I doing?)
jj log -n 10        # Your recent memory
jj show @           # Current state

# 2. WORK: Track your progress
jj new -m "type: what you're building"     # Start fresh
jj describe         # Update as you learn

# 3. PERSIST: Save your state
jj git push -c @    # Make it permanent
```

### The Universal Description Template

```
<type>: <what> - <why in 5 words>

Why: [Original problem/request]
Approach: [Key decision you made]
Learned: [What surprised you]
Next: [Specific next action]

🤖 YourModel <your@attribution>
```

**Types**: `feat`, `fix`, `refactor`, `test`, `docs`, `debug`, `research`

### Real Example That Works

```bash
jj describe -m "feat: player sprite animation - smooth 4-direction movement

Why: Player movement was static, needed animation feedback
Approach: Used TextureAtlas with 3-frame walk cycles per direction
Learned: Bevy 0.17 AnimationTimer requires manual ticking in system
Next: Add idle animations when player stops moving

🤖 Claude <claude@anthropic.com>"
```

### Model Attributions

- Claude: `🤖 Claude <claude@anthropic.com>`
- Gemini: `💎 Gemini <gemini@google.com>`
- GPT-4: `🧠 GPT-4 <gpt4@openai.com>`
- Kimi: `🔮 Kimi <kimi@moonshot.ai>`
- Llama: `🦙 Llama <llama@meta.com>`

### The Handoff Protocol

When switching models or sessions:
```bash
jj describe -m "[your work]

Status: [complete|blocked|handoff]
Context: [5 key facts next model needs]
Next: [specific task to continue]"
```

### Success Metrics

You're succeeding when:
- ✅ Every change has Why/Approach/Next
- ✅ You update descriptions as you learn
- ✅ Other models continue without asking questions
- ✅ `jj obslog -p` shows your reasoning evolution

### Quick Reference

| Command | Purpose | When to Use |
|---------|---------|------------|
| `jj new -m "..."` | Start new work | Beginning tasks |
| `jj describe` | Update context | When you learn something |
| `jj log -n 10` | See recent work | Starting sessions |
| `jj show @` | View current state | Understanding context |
| `jj obslog -p` | See reasoning evolution | Debugging decisions |
| `jj git push -c @` | Persist to GitHub | Work complete |
| `mv/rm <path>` | Move/Remove files | `jj` automatically tracks filesystem changes. Use standard shell commands. |
| `jj file untrack <path>` | Stop tracking a file | Use after `rm` if you don't want the deletion recorded. |

### Your Style Is Welcome

- **Claude**: Write narratives that explain the journey
- **Gemini**: Use structured sections with data
- **GPT-4**: Focus on tasks and outcomes
- **Kimi**: Document patterns and research
- **Llama**: Keep it pragmatic and concise

**The format is flexible, the information is essential.**

### Remember

jj descriptions are messages to your future self. Write what you'd need at 3am to understand what you were thinking. Your future self (and other models) will thank you.

---

## 📊 Agent Memory System

The project uses a shared memory system in `docs/agents/` for persistent context:

- **`docs/agents/NOW.md`** - Immediate working state (what's happening right now)
- **`docs/agents/PATTERNS.md`** - Reusable knowledge and discovered patterns
- **`docs/agents/CONTEXT.md`** - Session bridge for handoffs and context switches
- **`docs/agents/MEMORY_PROTOCOL.md`** - Guide to the memory system

These files provide <2000 tokens of overhead for complete context persistence across sessions and models.

## High Level Game Features

1. Pixel art JRPG visual novel (no combat, dialogue-driven)
2. Visustella Fantasy Tiles MZ for tilesets and characters (licensed for reuse)
3. **NEVER use default RPG Maker assets** (only Visustella licensed content)
4. 48x48 tile size, 960x540 base resolution (2x upscaled to 1920x1080)

## Development Guidelines

**Error Handling**:
- Use `anyhow::Result` for all fallible operations
- Never use `unwrap()` - always propagate errors with `?`
- Add context with `.context()` for debugging
- Never silently discard errors with `let _ =`
- Handle reconnection gracefully on network failures

**Code Style**:
- Prioritize correctness and clarity over performance
- No organizational comments that summarize code
- Comments should only explain "why" when non-obvious
- Implement functionality in existing files unless it's a new logical component
- Avoid `mod.rs` files - use `src/module_name.rs` directly
- Use full words for variable names (no abbreviations)

**Bevy 0.17 Specific**:
- Use `ImagePlugin::default_nearest()` for pixel-perfect rendering (prevents blurry sprites)
- Bevy 0.17 uses required components - `Sprite` auto-includes `Transform` and `Visibility`
- Spawn cameras with `Camera2d` component directly (no bundles needed)
- For 2D games, camera Z must be 999.9
- Use `in_state()` run conditions for state-specific systems
- State management: `States` for major phases, `SubStates` for variations
- Text rendering: `Text2d` for world space, `Text` for UI

**Project Structure**:
- One plugin per major feature (PlayerPlugin, DialoguePlugin, etc.)
- Modules: `game_state.rs`, `player.rs`, `camera.rs`, `tilemap.rs`, `dialogue.rs`, `npc.rs`, `assets.rs`
- See `build-plan/00-overview.md` for complete architecture

## Rust/Bevy Development Commands

```bash
# Build the project
cargo build
cargo build --release     # Optimized build

# Run the game
cargo run
cargo run --release       # Run optimized build

# Run tests
cargo test
cargo test --verbose      # Verbose output
cargo test test_name      # Specific test

# Check code (faster than build)
cargo check

# Format code (always use rustfmt)
cargo fmt
cargo fmt --check         # Check without modifying

# Lint with clippy
cargo clippy
cargo clippy -- -W clippy::all

# Clean build artifacts
cargo clean

# Update dependencies
cargo update

# View documentation
cargo doc --open
cargo doc --no-deps --open  # Skip dependency docs

# Build with specific features
cargo build --features "debug"

# Watch for changes and rebuild (requires cargo-watch)
cargo watch -x run
cargo watch -x test
```

## Version Control (Jujutsu + Git)

**Jujutsu (jj) workflow:**
- Use `jj` for all local development and memory persistence
- Follow the template in the jj section above
- Always include Why/Approach/Learned/Next in descriptions
- Use appropriate model attribution (🤖 for Claude, 💎 for Gemini, etc.)

**Git integration:**
- `jj git push -c @` syncs to GitHub
- GitHub CLI (`gh`) for PR operations:
  ```bash
  gh pr create --fill          # Create PR from jj description
  gh pr status                 # Check PR status
  gh pr checks                 # View CI results
  gh issue list                # Check issues
  gh issue view <number>       # Read issue details
  ```

**Legacy Git commits (if needed):**
* Always review `git status` and `git diff` before committing
* Use `git add` precisely on individual files
* Claude should add `Co-authored-by: Claude <claude@anthropic.com>`
* Gemini should add `Co-authored-by: Gemini <gemini@google.com>`

## Build Plan

The `build-plan/` directory contains detailed implementation guides:
- Start with `build-plan/00-overview.md` for complete project roadmap
- Follow steps 01-09 sequentially (each builds on previous steps)
- Each step includes complete code examples, testing procedures, and success criteria
- Total MVP implementation time: 12-40 hours depending on experience

## Key References

- **Original game data**: `/home/atobey/src/endgame-of-sre-rpgmaker-mz/data/`
- **Visustella assets**: `/home/atobey/src/endgame-of-sre-rpgmaker-mz/img/`
- **Build plan**: `build-plan/00-overview.md`
- **Bevy version**: 0.17 (critical - examples are version-specific)

## Available Agents

When you need specialized help, use these agents:
- **bevy-expert**: Bevy 0.17 API questions, best practices, troubleshooting
- **endgame-sre-expert**: Original game content, dialogue extraction, asset identification
- **rust-bevy-consultant**: Advanced Rust patterns, Bevy ECS architecture decisions

## Cross-Session Context Patterns

jj's power is in context preservation across sessions and agents.

**Starting a new session:**
1. `jj log -r 'mine()' -n 10` - What was I working on?
2. `jj show @` - What's my current state?
3. `jj log -n 20` - What happened since I left?

**Picking up someone else's work:**
1. Find their change: `jj log -n 20`
2. Read their description: `jj show <change-id>`
3. See their reasoning: `jj obslog <change-id> -p`
4. Create your change building on theirs: `jj new <their-change-id>`

**Avoiding duplicate work:**
- Always check `jj log -n 20` before starting something new
- Search descriptions: `jj log | grep -i "keyword"`
- Check for "Status: blocked" or "Status: next:" entries

## Remember

**jj changes are not git commits.** They are:
- Living documents that evolve with your understanding
- Persistent memory that survives rebases
- Context transfer mechanisms between agents and sessions
- Reasoning traces via obslog

**Your description quality directly impacts the next agent's ability to succeed.** Write for them.

## Attribution

- Original game by Amy Tobey (SRECon NA 2022)
- Rust/Bevy port by Claude & Amy Tobey
- Licensed content from Visustella Fantasy Tiles MZ
- All jj changes should use appropriate agent attribution (see jj section)

## License

Copyright (c) 2025 Amy Tobey
See LICENSE file for details

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tobert)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tobert)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
