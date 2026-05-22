## crm-assistant

> When working on the development roadmap phases:

# Auto-Continue Development Roadmap Rule

## Rule: Complete Entire Development Roadmap Automatically

When working on the development roadmap phases:

1. **Continue automatically** if everything is successful - no user input needed
2. **Create a new feature branch** for each phase (feature/phase{N}-{description})
3. **Commit and push** each feature branch after phase completion
4. **Keep going** until the entire roadmap is complete
5. **Only stop** if there are errors that require user intervention

### Phase Workflow:
1. Complete phase implementation
2. Test phase thoroughly
3. Create feature branch: `feature/phase{N}-{description}`
4. Commit with comprehensive message
5. Push to GitHub
6. Move to next phase automatically

### Success Criteria:
- All tests pass
- All requirements met
- No errors in implementation
- Clean git history with proper feature branches

Continue through all 11 phases of the development roadmap without stopping for approval unless there are blocking errors.

---
> Source: [ty-hayes-82/crm-assistant](https://github.com/ty-hayes-82/crm-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
