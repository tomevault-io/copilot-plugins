## zh-quotes-skill

> This skill is especially important when Chinese natural language is mixed with:


# Contextual Typography Skill

## Purpose

Resolve punctuation, quotation mark, and symbol usage correctly in multilingual and mixed-syntax contexts.

This skill is especially important when Chinese natural language is mixed with:

- programming languages
- Typst
- Markdown
- LaTeX
- JSON/YAML
- shell commands
- prompts
- configuration files

The goal is NOT typography beautification.

The goal is:

1. syntax correctness
2. parser compatibility
3. language-context consistency
4. readable Chinese typography

---

# Activation Conditions

Automatically activate this skill when ANY of the following appear:

- Chinese mixed with code
- Typst content
- Markdown fenced code blocks
- JSON/YAML/TOML
- nested quotation marks
- code generation
- prompts containing quotation examples
- multilingual text
- shell commands
- regex
- HTML/XML
- LaTeX/math
- strings containing Chinese text

Especially activate when output contains BOTH:

- Chinese characters
- ASCII syntax symbols

---

# Core Principle

Typography follows CURRENT SYNTAX CONTEXT, not human language.

Priority order:

1. programming language syntax
2. markup language syntax
3. data serialization syntax
4. natural language typography

Code validity always overrides Chinese typography conventions.

---

# Context Rules

## Pure Chinese prose

Use:

- Chinese double quotes: “”
- Chinese single quotes: ‘’
- Chinese punctuation

Example:

“生成设计”是一种方法论。

Nested:

“所谓‘规则’，本质上是约束。”

---

## Code Context

Inside code:

- ALWAYS use language-native syntax quotes
- NEVER replace syntax quotes with Chinese quotes

Correct:

```python
text = "生成设计"
````

Wrong:

```python
text = “生成设计”
```

---

## Chinese Inside Code Strings

Outer quotes follow programming language syntax.

Inner quotations inside Chinese text use Chinese quotes.

Correct:

```python
text = "用户点击“生成”按钮"
```

Correct:

```rust
println!("系统提示：“生成完成”");
```

---

## Typst Context

Typst syntax overrides Chinese typography.

Use:

* ASCII syntax symbols
* English quotes for Typst syntax
* Chinese quotes only INSIDE natural Chinese text

Correct:

```typst
#text("这是正文")
```

Correct:

```typst
#quote[
他说：“生成完成”
]
```

Wrong:

```typst
#text(“这是正文”)
```

---

## Markdown Context

Outside code:

* Chinese typography allowed

Inside code spans/blocks:

* preserve exact syntax

---

# Nested Context Resolution

Always identify the CURRENT nesting level.

Example hierarchy:

Markdown
→ Python
→ Chinese string
→ Chinese quotation

Each layer follows its own syntax rules.

---

# Forbidden Behaviors

Never automatically:

* convert ASCII quotes to Chinese quotes in code
* beautify syntax characters
* normalize valid syntax
* replace parser-required symbols

---

# Output Validation

Before final output:

1. detect current syntax context
2. verify quote legality
3. verify parser compatibility
4. verify nested quotation correctness

High-risk formats:

* Typst
* JSON
* YAML
* shell
* regex
* prompts
* LaTeX
* Markdown fenced blocks

---
> Source: [amomorning/zh-quotes-skill](https://github.com/amomorning/zh-quotes-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
