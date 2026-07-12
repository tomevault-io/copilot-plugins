## macos-cua

> - Ultra-strict TypeScript with all strict flags enabled

# macos-cua Agent Conventions

## TypeScript Standards
- Ultra-strict TypeScript with all strict flags enabled
- ESM modules with `.js` import suffixes under Node16 resolution
- Tabs for indentation (width 3 for display), line width 120 — enforced by Biome
- No `any`, `@ts-ignore`, `@ts-expect-error`, or non-essential type assertions
- Use `import type` for type-only imports

## Project Structure
- Monorepo with pnpm workspaces
- `packages/core`: Platform-abstracted computer-use interfaces
- `packages/cli`: Command-line interface
- `packages/mcp`: Model Context Protocol server
- `packages/pi-extension`: Pi coding agent extension
- `skills/`: Agent skill definitions

## Testing, QA, and PR Evidence
- Vitest for all tests
- Test naming: `#given #when #then` pattern
- Coverage via v8 provider
- When opening a PR, write for a human reviewer who did not watch the session: state the user-facing behavior/change in plain terms and group changes by reviewer-relevant surface, not by file list.
- In PR QA/Evidence, explain each command or manual QA action with what was tested, what result was observed, where the saved artifact/log lives, and why that evidence is sufficient.
- Include residual risks as risk -> evidence -> conclusion, and never paste raw secret-bearing logs, env dumps, tokens, or credentials.

## macOS Native APIs
- Screenshot: ScreenCaptureKit/IOSurface (preferred) or `screencapture` CLI fallback
- Input: koffi-bound CoreGraphics CGEvent (CGEventPost for global, CGEventPostToPid for per-PID); no CLI fallback
- Accessibility: `osascript` for UI queries when needed

## Platform Abstraction
- Abstract `ComputerInterface` with methods: screenshot, click, doubleClick, type, key, scroll, drag, getCursorPosition
- Implementations: `MacOSHostComputer` (native), `VMComputer` (interface only), `CloudComputer` (interface only)
- All unimplemented platforms should have interface stubs with `throw new Error("Not implemented")`

---
> Source: [code-yeongyu/macos-cua](https://github.com/code-yeongyu/macos-cua) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
