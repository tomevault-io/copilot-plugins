## writing-code-protocol

> Writing Code Protocol


**Writing Code Protocol**

- Place mock, test and implementation exactly near intent file.
- Export files from its folders using barrel file (ui/ui.dart, commands/commands.dart etc.).
- To find implementation, try to load intents first.
- Do not rely on strings, use enums.
- Use context.read/context.select to subscribe to observable state in build methods.
- If you create new file, make sure to check its intent. If it doesn't have an intent - create it.
- Use EquatableMixin for classes.

---
> Source: [Arenukvern/mcp_flutter](https://github.com/Arenukvern/mcp_flutter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
