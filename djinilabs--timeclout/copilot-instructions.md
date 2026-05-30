## business-rules

> - The business rules for the API should be placed inside the `libs/business-logic` folder.


- The business rules for the API should be placed inside the `libs/business-logic` folder.
- The GraphQL resolver code should simply call this business logic.
- Organize these business functions in folders, according to the main subject of that function.
- A business function should always verify the permissions, implement the business logic, and then return the expected value.
- If required, a business function should only explicitely throw `@hapi/boom` errors.

---
> Source: [djinilabs/timeclout](https://github.com/djinilabs/timeclout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
