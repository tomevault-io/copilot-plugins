## mobile-observability

> Claude Code plugin for mobile app observability. Provides commands, agents, skills, and hooks for iOS, Android, and React Native instrumentation.

# Mobile Observability Plugin

## Project Context

Claude Code plugin for mobile app observability. Provides commands, agents, skills, and hooks for iOS, Android, and React Native instrumentation.

## Plugin Components

| Component | Location | Description |
|-----------|----------|-------------|
| Commands | `commands/` | `/instrument`, `/audit` |
| Agents | `agents/` | `codebase-analyzer`, `instrumentation-reviewer` |
| Skills | `skills/` | 8 focused instrumentation skills |
| Hooks | `hooks/hooks.json` | Anti-pattern warnings for Swift/Kotlin/TypeScript |
| References | `references/` | 20+ guides, templates, vendor docs |

## Key Commands

```bash
/instrument ios              # Generate instrumentation plan
/audit                       # Scan existing instrumentation
```

## Skills

| Skill | Trigger |
|-------|---------|
| `instrumentation-planning` | "What should I measure?" |
| `crash-instrumentation` | "How to capture crashes with context" |
| `session-replay` | "Set up session replay" |
| `interaction-latency` | "Track button response time" |
| `navigation-latency` | "Track screen load time" |
| `network-tracing` | "Trace API requests" |
| `user-journey-tracking` | "Track user funnels" |

## Reference Navigation

| Topic | File |
|-------|------|
| Methodology | `references/user-focused-observability.md` |
| What to measure | `references/jtbd.md` |
| Instrumentation checklist | `references/instrumentation-patterns.md` |
| Business logic | `references/business-logic.md` |
| Feature flags | `references/feature-flags.md` |
| Crashes | `references/crash-reporting.md` |
| Performance | `references/performance.md`, `references/ui-performance.md` |
| Database | `references/data-persistence.md` |
| iOS | `references/ios-native.md` |
| Android | `references/android-native.md` |
| React Native | `references/react-native-expo.md` |
| OpenTelemetry | `references/otel-mobile.md` |
| Vendors | `references/platforms/*.md` |

## Development

```bash
# Test plugin locally
claude plugins install ./

# Verify structure
ls -la .claude-plugin/ commands/ agents/ skills/ hooks/ references/
```

---
> Source: [nexus-labs-automation/mobile-observability](https://github.com/nexus-labs-automation/mobile-observability) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
