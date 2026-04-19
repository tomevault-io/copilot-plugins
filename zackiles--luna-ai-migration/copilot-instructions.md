## luna-ai-migration

> After completing a user's request that involved a large change to the code you should automatically update the Repomix file for them using the following steps:

# Keep Your Context Up To Date
After completing a user's request that involved a large change to the code you should automatically update the Repomix file for them using the following steps:

- Run the script `<project-root>/.ai/build-context.sh`
- If the script is succesful, it will generate a new Repomix file at <project-root>/.ai/context/codebase-summary.xml`
- If `codebase-summary.xml` is in your context window, then reread it and load the new file that was generated from the `build-context.sh` script

## Summary
Taking these steps will ensure you always have the most recent context on the codebase from the repomix file at `<project-root>/.ai/context/codebase-summary.xml`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zackiles) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
