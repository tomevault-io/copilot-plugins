## morphogenesis

> - **Constants may change externally**: If you see that a constant has changed from under you, assume I did that intentionally and don't change it back without asking first.

# Claude Code Guidelines

## Code Changes & Constants

- **Constants may change externally**: If you see that a constant has changed from under you, assume I did that intentionally and don't change it back without asking first.

- **Avoid literal comments**: Don't leave comments that include literals which we have in code since it makes it annoying to edit the literals and creates repetitive maintenance overhead.
## Example

**Bad:**
```zig
const PARTICLE_SIZE = 0.008; // Physical radius of particles is 0.008
```

**Good:**  
```zig
const PARTICLE_SIZE = 0.008; // Physical radius of particles
```

This keeps comments descriptive without duplicating values that may change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MaxBittker)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MaxBittker)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
