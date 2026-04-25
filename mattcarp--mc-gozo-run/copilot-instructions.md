## mc-gozo-run

> **Race:** Il-Girja t'Ghawdex (Gozo Half Marathon)

# GozoRun - Gozo Half Marathon Companion App

**Race:** Il-Girja t'Ghawdex (Gozo Half Marathon)
**Date:** Sunday, April 26 2026, 07:30 - Xaghra Square
**Distance:** 21.1km loop course
**Website:** https://runfionarun.com
**Repo:** https://github.com/mattcarp/mc-gozo-run

## Tech Stack
- Swift + SwiftUI (iOS 17+)
- MapKit + CoreLocation + AVFoundation + Swift Charts
- Zero external dependencies (current)
- Backend TBD: Supabase Realtime or Firebase for live spectator tracking

## Architecture
- `GozoRunApp.swift` - Entry point, TabView (Track, Splits, Spectate, Settings)
- `RunTrackerViewModel.swift` - GPS tracking, stats, CLLocationManager
- `MapView.swift` - Satellite map with route overlay, KM markers, POIs
- `SpectatorView.swift` - Spectator mode (currently MVP: same-device only)
- `VoiceAlertManager.swift` - KM proximity voice alerts via AVSpeechSynthesizer
- `ThemeManager.swift` - 4 switchable themes
- `ElevationChartView.swift` - Swift Charts elevation profile
- `CountdownView.swift` - Live race countdown
- `SettingsView.swift` - Theme picker, units, voice toggle
- `GPXParser.swift` - XMLParser for 664-point GPX route
- `Models.swift` - Data models (RunSession, SpectatorLocation, KmSplit)

## Development
- Build: Xcode 16+ on macOS
- Run on physical iPhone (GPS required)
- `route.gpx` must be in Copy Bundle Resources

## Critical Rules
- NEVER remove the GPX route data
- Keep zero external dependencies if possible
- Test on physical device (simulator GPS is unreliable)
- Race day is immovable: April 26, 2026

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mattcarp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
