## belowdeck

> A modular Stream Deck Plus application combining media controls, calendar, home automation, weather, and more.

# CLAUDE.md

A modular Stream Deck Plus application combining media controls, calendar, home automation, weather, and more.

See [ROADMAP.md](ROADMAP.md) for detailed architecture plans and feature backlog.

## Project Structure

```
cmd/
  demo/        - Basic Stream Deck Plus demo
  belowdeck/   - Main application
```

## Running

```bash
# Build and run
go build -o ./bin/belowdeck ./cmd/belowdeck && ./bin/belowdeck
```

## Hardware

Stream Deck Plus: 8 LCD keys (72x72px), 4 rotary dials, touch strip (800x100px), USB-C.

## Dependencies

- `media-control` - macOS now-playing info
  ```bash
  brew tap ungive/media-control && brew install media-control
  ```

## Development Notes

- Only one app can control the device at a time (quit Elgato software when testing)
- Using `rafaelmartins.com/p/streamdeck` for Go bindings (has dial/strip support)
- SVG icons from Lucide, rendered with `oksvg`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phinze)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/phinze)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
