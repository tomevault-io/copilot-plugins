## svg-to-swiftui

> Converts SVG code into SwiftUI `Shape` structures. Monorepo managed with bun + Turborepo.

# SVG to SwiftUI

Converts SVG code into SwiftUI `Shape` structures. Monorepo managed with bun + Turborepo.

## Project Structure

- `packages/svg-to-swiftui-core/` - Core conversion library (published to npm)
- `apps/nextjs/` - Next.js web app (hosted at svg-to-swiftui.quassum.com)
- `packages/figma-to-swiftui/` - Figma plugin
- `tooling/` - Shared configs (eslint, prettier, tsconfig, tailwind)

## Commands

```sh
bun install          # Install dependencies
bun dev              # Run all packages in dev mode
bun run build        # Build all packages
bun run test         # Run tests (jest, in svg-to-swiftui-core)
bun run lint         # Lint all packages
bun run format       # Check formatting
bun run typecheck    # Type-check all packages
bun run visual-test  # Visual regression tests (macOS only, compares SVG vs Swift rendering)
```

## Requirements

- Bun >= 1.x

---
> Source: [bring-shrubbery/SVG-to-SwiftUI](https://github.com/bring-shrubbery/SVG-to-SwiftUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
