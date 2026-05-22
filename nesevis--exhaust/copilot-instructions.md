## exhaust

> - Documentation: You MUST follow @DOCUMENTATION_STYLE.md for all doc comments.

- Documentation: You MUST follow @DOCUMENTATION_STYLE.md for all doc comments.
- Variable naming: You MUST NOT use two and three letter abbreviations. `x…z`, `i…k` and `n` are fine for iterations
- Testing: You MUST NOT weaken a test assertion without asking for permission
- Control flow: You MUST NOT use a semicolon to put two statements on one line
- Control flow: You MUST NOT use if/else clauses to assign to a variable. Switch expressions are acceptable.
- Control flow: You MUST NOT use Objective-C parameter alignment style. If a declaration is too long, split each parameter into its own line like
```swift
let foo = Bar(
  x: "foo",
  y: "bar"
)
```
- Formatting: You MUST fix Swiftlint warnings where they occur

---
> Source: [nesevis/exhaust](https://github.com/nesevis/exhaust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
