## watching

> 1. Use `if` statements for conditional branching. The condition must evaluate to a boolean.


1. Use `if` statements for conditional branching. The condition must evaluate to a boolean.
2. `if` statements support optional `else` and `else if` clauses for multiple branches.
3. Use `if-case` statements to match and destructure a value against a single pattern. Example: `if (pair case [int x, int y]) { ... }`
4. If the pattern in an `if-case` matches, variables defined in the pattern are in scope for that branch.
5. If the pattern does not match in an `if-case`, control flows to the `else` branch if present.
6. Use `switch` statements to match a value against multiple patterns (cases). Each `case` can use any kind of pattern.
7. When a value matches a `case` pattern in a `switch` statement, the case body executes and control jumps to the end of the switch. `break` is not required.
8. You can end a non-empty `case` clause with `continue`, `throw`, or `return`.
9. Use `default` or `_` in a `switch` statement to handle unmatched values.
10. Empty `case` clauses fall through to the next case. Use `break` to prevent fallthrough.
11. Use `continue` with a label for non-sequential fallthrough between cases.
12. Use logical-or patterns (e.g., `case a || b`) to share a body or guard between cases.
13. Use `switch` expressions to produce a value based on matching cases. Syntax differs from statements: omit `case`, use `=>` for bodies, and separate cases with commas.
14. In `switch` expressions, the default case must use `_` (not `default`).
15. Dart checks for exhaustiveness in `switch` statements and expressions, reporting a compile-time error if not all possible values are handled.
16. To ensure exhaustiveness, use a default (`default` or `_`) case, or switch over enums or sealed types.
17. Use the `sealed` modifier on a class to enable exhaustiveness checking when switching over its subtypes.
18. Add a guard clause to a `case` using `when` to further constrain when a case matches. Example: `case pattern when condition:`
19. Guard clauses can be used in `if-case`, `switch` statements, and `switch` expressions. The guard is evaluated after pattern matching.
20. If a guard clause evaluates to false, execution proceeds to the next case (does not exit the switch).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonhygg99) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
