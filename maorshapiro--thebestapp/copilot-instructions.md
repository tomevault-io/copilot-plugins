## thebestapp

> - Source kit: Fintech Mobile App Update 1.0, node `404:1856`.

# Startup Matchaker Design System Rules

## Figma Source

- Source kit: Fintech Mobile App Update 1.0, node `404:1856`.
- Style intent: clean, minimalist, data-heavy mobile UI with low cognitive load.
- Use these rules for Figma-driven React Native/Fabric implementation work in this repo.

## Tokens

- Use `src/design-system/fintechTokens.ts` for colors, typography, spacing, radii, and shadows.
- IMPORTANT: Do not hardcode Figma hex values in components when an exported token exists.
- Primary palette: `#456EFE`, `#4B78FE`, `#13C999`, `#2DC688`.
- Neutral palette: `#FFFFFF`, `#F9F9FB`, `#23303B`, `#202A34`, `#8E949A`, `#A4A9AE`.
- Muted fills use alpha forms from the kit: `#456EFE26`, `#456EFE4D`, `#13C99926`, `#A4A9AE26`, `#A4A9AE40`, `#FF636326`.
- Typography uses Sofia Pro with weights Regular 400, Medium 500, SemiBold 600, and Bold 700.
- Spacing must stay on the 12-24px working range for tile composition: 12, 16, 20, and 24 are preferred gaps.
- Corner radii from the kit: 5px icon wells, 10px cards/tiles, 35px screen frames, 50px pills/avatars.
- Card shadow: color `#6E7588`, opacity `0.07`, offset `1x5`, radius `40`, Android elevation `4`.
- Icon glow: black opacity `0.09`, offset `8x7`, radius `48`, Android elevation `5`.

## React Native Components

- Reusable tile components live in `src/components/startup-tiles/`.
- Export templates as `HeroTile`, `ServiceTile`, and `MicrodataTile`.
- Keep components pure and memoized with `React.memo`.
- Use `StyleSheet.create` for stable style objects and keep expensive list callbacks stable with `useCallback`.
- Use `FlashList` from `@shopify/flash-list` for long startup data surfaces.

## Figma MCP Flow

- Fetch exact node context with Figma MCP before implementing visual work.
- Treat generated React/Tailwind snippets as visual reference only; convert to React Native styles.
- Validate against the Figma screenshot for color, type scale, spacing, shadows, and corner radius.

---
> Source: [MaorShapiro/THEBESTAPP](https://github.com/MaorShapiro/THEBESTAPP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
