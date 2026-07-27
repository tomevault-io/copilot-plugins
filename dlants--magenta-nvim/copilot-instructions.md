## magenta-nvim

> Check that the changes leverage the type system to make incorrect usage hard or


# Type Representation Review

## Purpose

Check that the changes leverage the type system to make incorrect usage hard or
impossible. Prefer encoding constraints in types over enforcing them with
runtime checks, comments, or convention.

## Make invalid states non-representable

Flag types where combinations of fields can express states that should never
occur, and recommend a disjoint (discriminated) union instead.

Avoid this — `result` and `error` can both be present or both absent:

```typescript
state: {
  type: "success" | "failure";
  result?: Result;
  error?: string;
}
```

Prefer this — each variant carries exactly the fields it needs:

```typescript
state:
  | { type: "success"; result: Result }
  | { type: "error"; error: string };
```

## Additional guidance

- Prefer precise types over loose ones (avoid `string`/`number` where a literal
  union or branded type captures the real domain).
- **Never** introduce new `any` types.
- Use `undefined` as the single bottom value; avoid `null`. If an external
  library hands back `null`, convert it to `undefined` at the boundary so
  internal types never carry `null`.
- Flag optional fields (`?:`) that are really "present in some variants, absent
  in others" — these are usually a sign a discriminated union is needed.
- Watch for non-null assertions (`!`) and unchecked casts (`as`) that paper over
  a weak type; suggest a representation that removes the need for them.

---
> Source: [dlants/magenta.nvim](https://github.com/dlants/magenta.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
