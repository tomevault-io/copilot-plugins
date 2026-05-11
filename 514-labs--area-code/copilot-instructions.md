## moose-services

> Use this when working on or testing moose related services like those in /services/sync-base or /services/analytical-base

# Moose Services Rules

These rules apply to Moose-based services in the `services/` directory (analytical-base and sync-base).

## Workflow Testing Guidelines

When testing workflows in these services:

- **Always use the command:** `moose workflow run <workflow-name>` to test if a workflow is running correctly
- **Assume the Moose instance is already running** - no need to start it separately. Inspect the existing terminal
- **Code changes will hot reload automatically** - any modifications to the Moose code will be picked up by the running server without manual restart

## Development Workflow

1. Make changes to your workflow code
2. Save the file (hot reload will handle the rest)
3. Test using: `moose workflow run <workflow-name>`
4. Check logs and output to verify behavior

## Example Usage

```bash
# Testing a specific workflow
moose workflow run eventsPipeline

# The Moose instance will already be running, so this command
# will execute against the current running instance
```

## Important Notes

- Do not restart the Moose server unnecessarily - it supports hot reload
- Focus on the workflow logic rather than server management
- Use the workflow run command for quick iteration and testing 

---
> Source: [514-labs/area-code](https://github.com/514-labs/area-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
