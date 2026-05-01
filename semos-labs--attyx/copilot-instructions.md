## clean-code

> Clean code and architecture rules for Attyx


# Attyx Cursor Rules

## Project Philosophy

- Attyx is a deterministic VT-compatible state machine.
- Clarity > cleverness.
- Correctness > feature count.
- Minimalism > abstraction.
- The core must be testable without rendering or PTY.

---

## Architecture

### Strict Layer Separation

```
src/
term/ # Pure terminal engine (no side effects)
headless/ # Deterministic runner + tests
app/ # PTY + OS integration
render/ # GPU + font rendering
```

### Rules

- `term/` must not depend on PTY, windowing, rendering, clipboard, or platform APIs.
- `term/` must be fully deterministic and pure.
- Parser emits `Action` values.
- State applies `Action`.
- Renderer consumes state.
- Parser must never modify state directly.
- Renderer must never influence parsing or state.

---

## Implementation Order (Do Not Skip)

1. Grid + cursor + printable bytes + `\n \r \b \t`
2. Headless snapshot system + golden tests
3. ESC detection + CSI parsing skeleton
4. Minimal CSI support:
   - Cursor movement
   - Erase line/screen
   - SGR reset + 16 colors
5. Scroll + scrollback
6. Alternate screen (`?1049h / ?1049l`)
7. Damage tracking (dirty rows first)

Do not implement advanced features before the above is stable and tested.

---

## Code Style (Zig)

- Use explicit types in public APIs.
- Avoid metaprogramming unless it reduces complexity.
- No global mutable state.
- Keep modules small and focused.
- Keep functions short and readable.
- Avoid clever abstractions.
- Favor data-oriented structs.
- No hidden allocations.
- Every allocation must have a clear ownership model.
- Pass `std.mem.Allocator` explicitly.

---

## Error Handling

- Use Zig errors, not sentinel values.
- Do not swallow errors silently.
- Parsing:
  - Default mode: ignore malformed sequences gracefully.
  - Strict mode: log malformed sequences.

---

## Performance

- No per-character allocations.
- Parser operates on slices and indexes.
- Do not build temporary strings for CSI parsing.
- State updates should be O(1) per action where possible.
- No premature micro-optimizations.

---

## Testing (Mandatory)

- Every feature must include:
  - A golden snapshot test, or
  - A state hash test, or
  - A parser unit test.
- All tests must run in headless mode.
- No rendering required for core tests.

---

## Terminal Model Rules

- Maintain explicit mode flags.
- Alternate screen must be a separate buffer.
- Scrollback belongs only to the main screen.
- Do not mix rendering logic into state logic.

---

## Scope Control

Do NOT implement before MVP is complete:

- Tabs
- Splits
- Ligatures
- Kitty graphics protocol
- IME
- UI overlays
- Experimental protocol extensions

Focus on minimal VT correctness first.

---
> Source: [semos-labs/attyx](https://github.com/semos-labs/attyx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
