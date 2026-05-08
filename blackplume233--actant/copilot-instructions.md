## endurance-testing

> Enforce endurance test maintenance when modifying agent lifecycle, state machine, or communication code


# Endurance Test Maintenance

When modifying files matching the globs above, you MUST check whether endurance tests need updating.

## Trigger Conditions

Update `*.endurance.test.ts` when changes affect:
- Agent state transitions (status, pid, processOwnership)
- LaunchMode behavior (exit handling, restart logic, recovery action)
- ProcessWatcher or RestartTracker logic
- External Spawn API (resolve/attach/detach)
- WorkspacePolicy or cleanup behavior
- RPC handlers for agent operations
- ACP/MCP communication (Phase 2+)

## Required Actions

1. **Read** `.trellis/spec/endurance-testing.md` to understand the coverage matrix
2. **Update** affected `*.endurance.test.ts` files — new scenarios, updated invariants, or adapted interfaces
3. **Verify** with `ENDURANCE_DURATION_MS=5000 pnpm test:endurance`
4. **Add** coverage phase annotation if introducing new scenario category

## Quick Reference

- Endurance test files: `packages/*/src/**/*.endurance.test.ts`
- Config: `vitest.endurance.config.ts`
- Run: `pnpm test:endurance`
- Spec: `.trellis/spec/endurance-testing.md`

---
> Source: [blackplume233/Actant](https://github.com/blackplume233/Actant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
