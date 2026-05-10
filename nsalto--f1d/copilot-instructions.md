## f1d

> - **REMOVE "F1 Team" suffix EVERYWHERE**: Hass F1 Team → Hass, Alpine F1 Team → Alpine, Cadillac F1 Team → Cadillac, etc.

# F1 2026 Dashboard - Design & Architecture Decisions

## Critical Design Rules

### Team Naming
- **REMOVE "F1 Team" suffix EVERYWHERE**: Hass F1 Team → Hass, Alpine F1 Team → Alpine, Cadillac F1 Team → Cadillac, etc.
- This applies to: standings tables, driver rows, constructor lists, anywhere team names appear
- Implement in: display filters, API response formatters, not just in component templates
- Use `getTeamIdByName()` consistently which already handles this normalization

### Circuit Visualization
- Circuits must be **prominently visible** with good contrast
- SVG black strokes require filter: `invert(1) sepia(1) saturate(3) hue-rotate(320deg) brightness(0.9)` to render as F1 red on dark backgrounds
- **Dashboard**: Circuit should be large and prominent below countdown (future: show live position overlay)
- **Live Timing**: Circuit sidebar element for quick reference
- **Circuit Maps**: Must be large and clear enough to potentially display driver positions during race

### Team Logo Resolution
- RedBull has special case: team name variations = "rb", "red bull", "red bull racing", "oracle red bull racing" → all map to teamId "red-bull"
- Logo file always: `/teams/logos/red-bull.webp` (not `rb.webp`)
- Implement in both client and server versions of `drivers-2026.ts`
- Test: any team name variation should resolve to correct logo

## Architecture

### Data Flow
- SSE connection for real-time timing → sectors, catching status, lap times
- Database sync for historical races and standings
- Always show only available data (no simulated/fake data per user requirement)

### Visual Feedback
- Position changes: Green flash on gain (+position), Red flash on loss (-position)
- Lap updates: Yellow flash on new lap time
- Catching up: Green text for gap when catching (gap < 1s)
- Sectors: MiniSectors component shows S1/S2/S3 when available

### Styling
- Dark theme: #080808, #0f0f0f base colors
- F1 Red: #e10600 for accents and highlights
- Team colors from `team-colors.ts` for position indicators
- Font stack: `font-timing` for monospace numbers, regular for text

## Future Enhancements
- Driver position overlay on circuit SVG during live races (use lap distance data)
- Team strategy visualization (pit stop timing, tire compound strategy)
- Sector comparison heatmaps

---
> Source: [nsalto/f1d](https://github.com/nsalto/f1d) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
