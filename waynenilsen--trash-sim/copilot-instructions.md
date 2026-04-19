## trash-sim

> - Always run `bun run format` and `bun run lint` before completing code changes and handing over for review

# Cursor Rules for Trash Sim

## Code Quality
- Always run `bun run format` and `bun run lint` before completing code changes and handing over for review
- Ensure all code passes TypeScript type checking with `bun run check`
- Fix any formatting or linting issues before finalizing changes

## Tech Stack
- Runtime: Bun
- Language: TypeScript
- Build Tool: Vite
- Framework: React
- UI Components: shadcn/ui
- Styling: Tailwind CSS
- Statistics: @stdlib/stats
- Code Quality: Biome (formatting & linting)

## Project Structure
- Source code in `src/`
- UI components in `components/ui/`
- Use path aliases: `@/` for project root

## Development Workflow
- Use `bun` for package management and scripts
- Run `bun run dev` to start development server
- Run `bun run check` for TypeScript type checking
- Run `bun run format` to format code
- Run `bun run lint` to lint and auto-fix code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/waynenilsen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
