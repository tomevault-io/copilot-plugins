## lrs-parsing

> The LRS Parsing principles are

The LRS Parsing principles are
- Proper Long Right Scope implementation. 
  * Precedence of verbs is purely based on position and grouping. Groupings have precedence, in order of nesting, then rightmost sub-expression has precedence over expressions to the left of it. 
  * The parsing code must have exactly zero special cases for specific verbs 
  * Parsing is agnostic with regards to individual verbs. The parser relies only on attributes obtained from the Verb Registry (e.g., monadic verbs, or dyadic verbs, which evaluator to call for a given token, which string representation to use for a given token, etc.). 
- Parser operation is split into two stages: Construction of the parse tree (S-attributed expression) and Evaluation of a parse tree
- Parse trees support arguments that are parse trees (nested)
- Parse trees support verbs that include adverbs

---
> Source: [ERufian/ksharp](https://github.com/ERufian/ksharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
