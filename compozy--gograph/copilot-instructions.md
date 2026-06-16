## no-linebreaks

> Formatting rule to eliminate unnecessary blank lines within function bodies and code blocks for cleaner, more compact code

<formatting_rule type="linebreaks">
- Don't need to add extra line breaks between variables definitions and code blocks
- When writing a function, mainly small functions, don't need line breaks
</formatting_rule>

<example type="bad_practice">
## Bad

```go
func Foo() {
    bar := "bar"

    bar = "baz"

    return
}
```
</example>

<example type="good_practice">
## Good

```go
func Foo() {
    bar := "bar"
    bar = "baz"
    return
}
```
</example>

---
> Source: [compozy/gograph](https://github.com/compozy/gograph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
