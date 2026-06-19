## named-booleans-skill

> Heuristics for refactoring inline boolean conditions into named const predicates without adding ceremony. Use when extracting inline conditions, addressing review feedback about condition readability, or deciding whether named booleans make code clearer.


# Named Booleans

## Quick Rule

Extract inline conditions into named `const` values when the name explains a domain decision. Do not extract conditions just to eliminate every operator from an `if`, `return`, or ternary.

## Use Named Consts When

- The condition combines multiple facts into one business concept.
  - Example: `hasPositiveAmount && hasBillIdentifier`
- The same condition or sub-condition is reused.
- A sequence of guard clauses reads like a decision table.
- The inline expression hides product language, workflow state, permission scope, or data validity.
- The named value prevents subtle misreading of a boundary, fallback, or persistence rule.

## Avoid Named Consts When

- The name only restates syntax.
  - Avoid: `const valueIsNotNull = value != null`
- The extracted const makes a tiny helper longer and harder to scan.
- The condition is already inside a well-named predicate function.
- The expression is a direct, local guard and the inline version is clearer.
- Extracting the const changes behavior, including truthiness vs null checks, lazy evaluation, side effects, or expensive work.

## Shape

- Prefer one named const per meaningful concept, not one per operator.
- Keep named predicates close to their use.
- Use positive names when possible: `hasUsableTaxAmount`, `shouldUpdatePayableFields`.
- Preserve original semantics exactly. If the old condition used truthiness, do not silently switch to `!== null`.
- If the helper becomes a wall of boolean setup, collapse back to a flatter form.

## Examples

Good:

```ts
const hasPositiveAmount = payable.amountMinor != null && payable.amountMinor > 0
const hasBillIdentifier = hasBillNumber || hasTitle

return hasPositiveAmount && hasBillIdentifier
```

Too much:

```ts
const isCurrencyField = fieldName === "currency"
if (isCurrencyField) {
  const hasSupportedCurrency = normalizedCurrency !== null

  return hasSupportedCurrency
}
```

Better:

```ts
const isCurrencyField = fieldName === "currency"
const hasSupportedCurrency = normalizedCurrency !== null

if (isCurrencyField) return hasSupportedCurrency
```

---
> Source: [CasperEngl/named-booleans-skill](https://github.com/CasperEngl/named-booleans-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
