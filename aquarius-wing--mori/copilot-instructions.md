## vstack

> When use LazyVStack, use index as id instead of using id of data

When use LazyVStack, use index as id instead of using id of data

Example:
```swift
ScrollView {
    LazyVStack(spacing: 16) {
        ForEach(events.indices, id: \.self) { index in
            CalendarEventDetailRow(event: events[index])
                .padding(.horizontal, 20)
        }
    }
    .padding(.top, 20)
    .padding(.bottom, max(20, geometry.safeAreaInsets.bottom))
}
```

---
> Source: [aquarius-wing/mori](https://github.com/aquarius-wing/mori) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
