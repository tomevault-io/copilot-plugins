## code-curly-braces

> Enforces curly braces around all if statement blocks

# Rules

## Use curly braces for all if statements
All if statements must use curly braces, even for single-line statements.

# Examples

## Valid
```typescript
if (value > 0) {
  return value
}

if (isValid) {
  doSomething()
}

if (hasError) {
  handleError()
} else {
  sendSuccess()
}
```

## Invalid
```typescript
if (value > 0) return value

if (isValid)
  doSomething()

if (hasError) handleError()
else sendSuccess()
```

---
> Source: [ericvera/whatsapp-cloudapi](https://github.com/ericvera/whatsapp-cloudapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
