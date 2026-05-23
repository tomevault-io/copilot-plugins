## tests

> Enforces flat test structure without unnecessary describe blocks

# Rules

## Use flat test structure
Tests should be written in a flat structure using only `it` blocks. `describe` blocks should only be used when explicitly requested in the test requirements.

# Examples

## Valid
```typescript
it('should render user name', () => {
  // test code
})

it('should show edit button', () => {
  // test code
})
```

## Invalid
```typescript
describe('UserComponent', () => {
  it('should render user name', () => {
    // test code
  })

  it('should show edit button', () => {
    // test code
  })
})
```

---
> Source: [ericvera/whatsapp-cloudapi](https://github.com/ericvera/whatsapp-cloudapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
