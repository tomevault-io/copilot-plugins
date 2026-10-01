## osaurus

> Every new or renamed setting must update SettingsSearchIndex, its landing anchor, and the self-find probe.


# Settings catalog

`SettingsSearchIndex` is the single catalog for Management search and the
Orchestrator (`osaurus_help` action `find`). Do not add a second settings
list to the orchestrator prompt.

When you add, remove, rename, move, or relabel a user-facing setting:

1. Update the row in `SettingsSearchIndex.swift` (id, tab, exact UI title,
   keywords, `subTab`, `disambiguation`, `declarativeSection`).
2. Put the same id on the control (`settingsLandingAnchor` / `anchorId`).
3. Fix any guide path that names the old location (`guide-settings.md` and
   related topics).
4. Add the on-screen label to `SettingsSearchSelfFindProbe.swift`.

A control that is not in the catalog is not shipped.

```swift
// BAD — new toggle, no catalog row
SettingsToggle(title: L("Smooth Streaming"), isOn: $smooth)

// GOOD — same id in the index and on the control
SettingsToggle(title: L("Smooth Streaming"), isOn: $smooth)
    .settingsLandingAnchor("settings.chat.smoothStreaming")
```

---
> Source: [osaurus-ai/osaurus](https://github.com/osaurus-ai/osaurus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
