## vlarlkit

> This is a **research codebase** for VLA (Vision-Language-Action) reinforcement learning. It sits deliberately between two extremes:

# VLA-RL Project Guidelines

## Project Context

This is a **research codebase** for VLA (Vision-Language-Action) reinforcement learning. It sits deliberately between two extremes:

- **Not over-engineered**: Avoid enterprise-style abstractions, excessive inheritance hierarchies, factory patterns, or configuration systems that add indirection without scientific value. Researchers need to read and modify code quickly.
- **Not a throwaway script**: Code should be structured, reusable, and extensible. New experiments should be addable without rewriting core logic.

When in doubt, prefer **clarity and directness** over abstraction.

---

## Code Style

### General

- Write code that reads like well-structured research — clean, purposeful, and traceable.
- Avoid unnecessary wrapper classes or functions that exist solely for "good software engineering" reasons without benefiting the research workflow.
- Prefer flat structures over deep nesting. If you need more than 2–3 levels of abstraction to explain something, reconsider the design.

### Comments

Use comments **sparingly and purposefully**:

- ✅ Explain *why* a non-obvious implementation choice was made.
- ✅ Label logical steps within a longer function to help readers follow the flow.
- ❌ Do not restate what the code obviously does (`# increment counter`).
- ❌ Do not add docstrings to every function by default — only where the interface is genuinely non-obvious or the function is part of a public API.

```python
# Good: explains a non-obvious step
# Normalize advantages before computing the policy gradient to reduce variance
advantages = (advantages - advantages.mean()) / (advantages.std() + 1e-8)

# Good: section label in a long function
# --- Rollout collection ---
rollouts = self._collect_rollouts(env, policy)

# --- Reward computation ---
rewards = self._compute_rewards(rollouts, reward_model)
```

---

## Python Naming Conventions

### Private vs. Public — Strict Enforcement

This project enforces a strict naming convention for class attributes and methods:

- **Public** (`self.name`, `def method`): Intended to be accessed from outside the class. Part of the class's interface.
- **Private** (`self._name`, `def _method`): Only accessed internally within the class. Never accessed directly from outside.

This is not just style — it communicates **intent and contract** to readers.

```python
class RolloutBuffer:
    def __init__(self, capacity: int):
        self.capacity = capacity        # public: callers may read this
        self._buffer = []               # private: managed internally only
        self._ptr = 0                   # private: internal write pointer

    def add(self, transition):          # public: part of the interface
        self._buffer[self._ptr] = transition
        self._ptr = (self._ptr + 1) % self.capacity

    def _validate(self, transition):    # private: internal helper
        assert "obs" in transition
```

**Rules:**
- If an attribute is only ever set and read inside the class, it **must** use a leading underscore.
- If a method is only called by other methods within the same class, it **must** use a leading underscore.
- Do not use double underscore (`__name`) unless name mangling is genuinely needed (rare).
- When refactoring or adding to existing classes, audit and correct any violations.

---

## Project Structure Preferences

- Keep related logic together. A training script should be readable top-to-bottom without jumping across many files.
- Prefer explicit imports over wildcard imports.
- Configuration should be simple — dataclasses or plain dicts are preferred over complex config frameworks.
- Avoid premature generalization. Build for the current two or three use cases, not for every hypothetical future one.

---

## What to Avoid

| Anti-pattern | Why |
|---|---|
| Deep class hierarchies | Hard to trace for researchers unfamiliar with the codebase |
| Registry / plugin systems | Overkill unless the codebase has grown to genuinely need it |
| Comments that paraphrase code | Noise that obscures real signal |
| Public attributes that should be private | Breaks encapsulation contract and misleads readers |
| Overly defensive programming | Trust that inputs are correct in research contexts; assert at boundaries instead |

---
> Source: [VLARLKit/VLARLKit](https://github.com/VLARLKit/VLARLKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
