## tests

> - Use other tests as a guidance on how the code should behave


- Use other tests as a guidance on how the code should behave
- Use pnpm to run the test

# Prefer fuzzy matching over precise matching

Prefer using `expect.objectContaining` instead of perfect matching when using `expect(...).toHaveBeenCalledWith` assertions.

Examples:
```
  expect(mockStorage.rawFind).toHaveBeenCalledWith(
    expect.objectContaining({
      resource: "users",
      where: {},
    })
  );
```

The only exception to this is where it's relevant to a test that a property is not present.

---
> Source: [pedroscosta/live-state](https://github.com/pedroscosta/live-state) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
