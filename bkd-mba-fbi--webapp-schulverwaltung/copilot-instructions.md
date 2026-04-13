## webapp-schulverwaltung

> - Don't use the `any` type or @ts-ignore, try to type things appropriately.


# Coding Style

- Don't use the `any` type or @ts-ignore, try to type things appropriately.
- In function signatures use ReadonlyArray<T> instead of T[] or Array<T>.
- Prefer immutable operations, e.g. use array spreading instead of array.push().
- Instead of a type `T | null` use the custom type Option<T>.

# Testing Style

- In a Jasmine test, don't use the word "should" as in `it("should return true", ...)`, instead use the style `it("returns true", ...)`.
- If I have focused a test with `fdescribe` or `fit`, don't remove it when doing any code changes, if the changes you are doing are on the same component/service.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bkd-mba-fbi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
