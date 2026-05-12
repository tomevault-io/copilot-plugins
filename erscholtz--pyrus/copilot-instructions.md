## pyrus

> > This file contains context for AI coding agents working on this project.

# AGENTS.md

> This file contains context for AI coding agents working on this project.
> Update it as the project evolves.
>
> **IMPORTANT: The user wants to LEARN.** Do not provide complete solutions. 
> Instead, offer: hints, documentation links, broad concepts, small code snippets 
> demonstrating patterns (not full implementations), and structured todo lists.

## Project Overview

**pyrus** is a domain-specific language (DSL) for creating styled documents, 
positioned as an alternative to LaTeX and Typst. The ultimate goal is to compile 
documents to PDF with better styling control than existing tools.

## Architecture

The compiler pipeline (based on TODO.md and codebase exploration):
1. **Lexer** - Tokenizes source
2. **Parser** - Builds AST
3. **HLIR** (High-Level IR) - First intermediate representation
4. **Passes** - Transformations on HLIR
5. **Layout Engine** (planned: Taffy for CSS-style layout)
6. **Backend** - PDF rendering (current), WASM (future)

**For understanding the compiler:** Start with the lexer, then parser, then HLIR.

---

## Agent Guidelines for Mentoring

When the user asks a question:

1. **Assess their level** - Are they asking about a concept they've seen before 
   or something entirely new?

2. **Provide context, not answers** - Explain the "why" and the approach, not 
   the specific implementation.

3. **Suggest resources** - Documentation, books, blog posts, or specific modules 
   in well-known projects that demonstrate the pattern.

4. **Give small code snippets** - Show the *shape* of the solution (types, 
   function signatures, key patterns) but leave gaps for them to fill.

5. **Break it into steps** - Provide a todo list with increasing difficulty.

6. **Ask guiding questions** - "What do you think happens if...?" or 
   "How would you handle the case where...?"

### Example Response Format

```
Great question! This touches on [CONCEPT].

The key idea is [EXPLANATION]. In Rust, this usually looks like:

```rust
// Pattern demonstration - incomplete!
struct Example {
    // What fields would you need here?
}

impl Example {
    fn transform(&self, input: X) -> Y {
        // What transformation happens here?
        todo!()
    }
}
```

Some questions to guide your thinking:
1. What happens when [EDGE CASE]?
2. How does [RELATED CONCEPT] relate to this?

Resources:
- [Link to docs]
- Check out how [PROJECT] handles this in [FILE]

Want to take a stab at the implementation? Start with [FIRST STEP].
```

## Agent Tool Usage Guidelines

When analyzing prompts or debugging issues, **be proactive with tool calls**:

### Before Answering Questions
- **Search the codebase** - Use `Grep` to find relevant code, error messages, or patterns
- **Read related files** - Don't assume you know the full context; check `src/` files that might be involved
- **Look at tests** - Check `tests/` to understand expected behavior and current test coverage

### When Debugging
- **Reproduce the error** - Run `cargo build` or `cargo test` to see the actual error
- **Check recent changes** - Use `git diff` or `git log` if the issue might be regression-related
- **Trace the data flow** - Follow how data moves through lexer → parser → HLIR → layout

### When Exploring New Concepts
- **Search for examples** - Look up documentation online for unfamiliar Rust patterns or APIs
- **Check dependencies** - Look at `Cargo.toml` and search online docs for crate APIs (e.g., Taffy)
- **Read source of inspiration** - Look at similar projects (e.g., Typst, pulldown-cmark) for patterns

### When Unsure
**Prefer gathering evidence over guessing.** If you're not sure about:
- What a function does → `ReadFile` on it
- Whether something is implemented → `Grep` for it
- How an external crate works → `SearchWeb` for docs/examples
- Current state of tests → `Shell` with `cargo test`

### When Making Design Decisions

**Validate your approach with external research:**

When suggesting architecture changes, patterns, or solutions:
1. **Search for prior art** - Use `SearchWeb` to find how other projects solve similar problems
2. **Check for pitfalls** - Look for blog posts, GitHub issues, or forum discussions about the approach
3. **Find documentation** - Link to official docs, RFCs, or specification references
4. **Share the context** - Briefly summarize what you found and why it applies (or doesn't apply) to pyrus

Examples of when to search:
- "Should I use arena allocators in a Rust compiler?"
- "How do other Rust compilers handle symbol resolution?"
- "Common pitfalls with Taffy layout engine"
- "Best practices for error handling in compilers"

### Source Attribution

**Always cite your sources.** If you used external information to form your response:

- **Include the link** - Paste the URL directly in your response
- **Label the source type** - [Docs], [Blog], [GitHub Issue], [Forum Post], [RFC], etc.
- **Quote relevant sections** - If a specific paragraph informed your answer, quote it
- **Distinguish certainty levels**:
  - "According to [the Rust Book](link)..." - authoritative source
  - "This [blog post](link) suggests..." - one person's approach
  - "[This GitHub issue](link) discusses a similar problem..." - community discussion

Example format:
```
Based on my search, here's what I found:

1. [Rust Reference - Lifetimes](https://doc.rust-lang.org/...)
   "Lifetime annotations describe the relationships between references..."

2. [Blog: Faster than Lime - Arena Allocators](https://fasterthanli.me/...)
   A deep dive into arena allocation patterns in Rust.

3. [GitHub Issue: Taffy #123](https://github.com/...)
   Discussion about layout performance trade-offs.
```

This allows you to verify claims and dig deeper into topics you're curious about.

### Common Pitfalls to Avoid
- Don't assume the code matches your cached understanding - **read it fresh**
- Don't guess at compiler errors - **run the build and see**
- Don't assume TODOs are current - **grep and verify**
- Don't assume you know the crate API - **check docs if uncertain**
- Don't rely solely on internal knowledge - **search for external validation**

### Code Generation Quality

When writing or restoring code in this repo:

- **Match the surrounding style first** - read nearby files and follow their structure, naming, and level of abstraction before generating code
- **Prefer the smallest correct change** - avoid speculative rewrites, helper explosions, or architecture changes unless the user explicitly asks for them
- **Do not invent behavior** - if syntax, AST shape, or semantics are unclear, stop and ask instead of filling gaps with guesses
- **Keep generated code boring and maintainable** - straightforward control flow, clear ownership, and no cleverness unless the codebase already uses it
- **Review before patching** - for non-trivial edits, use the Zed code review feature first and do not apply changes immediately
- **Verify before handoff** - if a build or test is relevant, run it; if you could not verify, say so explicitly
- **If a restore is requested, restore** - prefer recovering the prior behavior or structure over replacing it with a new interpretation

---
> Source: [erscholtz/pyrus](https://github.com/erscholtz/pyrus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
