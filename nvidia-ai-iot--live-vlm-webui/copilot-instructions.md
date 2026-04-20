## live-vlm-webui

> When creating reference, design, or planning documents (e.g., setup guides, verification steps, architecture docs, planning docs):

# Cursor AI Rules for live-vlm-webui Project

## Reference Documentation Guidelines

When creating reference, design, or planning documents (e.g., setup guides, verification steps, architecture docs, planning docs):

1. **Location**: ALWAYS place them in `docs/cursor/` directory
2. **Naming**: Use ALL CAPS with underscores (e.g., `MAC_SETUP.md`, `API_DESIGN.md`)
3. **Purpose**: These are for local reference only and should NOT be committed to git
4. **Rationale**: Keeps the repo clean while maintaining helpful local documentation

### Examples of Reference Docs to Put in `docs/cursor/`:
- Setup guides: `MAC_SETUP.md`, `WINDOWS_SETUP.md`
- Verification checklists: `MAC_VERIFICATION_STEPS.md`
- Design docs: `CHANGES_FOR_MAC_SUPPORT.md`, `API_DESIGN.md`
- Testing scripts: `test_*.py` (if for one-time testing)
- Helper scripts: `package_for_transfer.sh` (if temporary)

### Examples of Docs to Keep in Project Root:
- `README.md` - Main project documentation
- `ROADMAP.md` - Public project roadmap
- `CONTRIBUTING.md` - Contribution guidelines
- `LICENSE` - Project license
- Other user-facing documentation

## Code Organization

- Production code: Project root or appropriate subdirectories
- Tests: `tests/` directory (if created)
- Documentation: `docs/` for public docs, `docs/cursor/` for internal/reference docs

## Documentation Cross-References

**IMPORTANT**: Never reference `docs/cursor/` files in public-facing documentation

- ❌ BAD: "See `docs/cursor/MULTI_USER_ANALYSIS.md` for details"
- ✅ GOOD: Inline the relevant information or create a proper public doc in `docs/`

**Rationale**:
- `docs/cursor/` files are discussions between individual developers and Cursor AI, particulaly made for the developement at that point of time.
- These files are gitignored and won't exist for other developers or users
- Public docs (README.md, docs/development/, docs/setup/, etc.) must be self-contained

## Git Guidelines

- The `docs/cursor/` directory is ignored in `.gitignore`
- Reference docs will stay local on developer machines
- Only production-ready code and public documentation should be committed

---
> Source: [NVIDIA-AI-IOT/live-vlm-webui](https://github.com/NVIDIA-AI-IOT/live-vlm-webui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
