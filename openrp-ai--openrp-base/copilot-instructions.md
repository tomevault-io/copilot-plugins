## testing

> how to write tests for this project

- test files are placed in the same folder as the source code
- test files have the suffix `.test`, so `Component.tsx` will have a test `Component.test.tsx`
- write vitest tests
- try to mock as little as possible - never mock unless absolutely necessary. If a test tests a component that references other components that we wrote, NEVER mock these and test real behavior.
- the only exception for above are for API route tests, where we should mock lib/api responses.
- this includes file systems and databases. Just use node's fs with a temp folder and we have an in-memory pglite database in [server_api.test.helper.ts](mdc:lib/drizzle/server_api.test.helper.ts).
- remember separation of concerns. If component A includes component B, then you should test A is rendering B (by using data-testid) but not what B is rendering - that's for B's tests.
- for mocks that we change values across tests, reset them in `before()` and introduce properly scoped variables for them.
- put all test objects or helper/utility methods in `file.test.helper.ts`. Reuse existing test objects from logical places, like test users in `lib/api/users.test.helper.ts`.
- for database test objects, see @server_api.test.helper.ts for seeded objects for testing.

---
> Source: [openrp-ai/openrp-base](https://github.com/openrp-ai/openrp-base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
