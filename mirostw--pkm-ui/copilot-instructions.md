## always-test

> - after adding or changing code always make sure to run `bun test:unit` and `bun test:e2e` to make sure everything still works


# Your rule content

- after adding or changing code always make sure to run `bun test:unit` and `bun test:e2e` to make sure everything still works
- When adding new functionality, always add tests for it (unit tests and/or e2e tests)
- We use bun for unit tests and playwright for e2e tests

## module mocks
avoid using mock.modules with bun unit tests, instead use the following spyOn workaround 

Example:

Desired:

mock.module("../path/to/module", () => ({
  default: mock().mockImplementation(() => ({ // or mockImplementationOnce, depending on use-case
    foo: mock(),
    bar: mock(),
  });
});

Workaround:

import MyModule from "../path/to/module";
spyOn(MyModule, "default").mockImplementation(() => ({ // or mockImplementationOnce, depending on use-case
  foo: mock(),
  bar: mock(),
}));

---
> Source: [MiroStW/pkm-ui](https://github.com/MiroStW/pkm-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
