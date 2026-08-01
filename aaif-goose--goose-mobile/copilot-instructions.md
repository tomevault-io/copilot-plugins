## goose-mobile

> This is a swift app - which is a client to the goosed service (scripts here are just for helpers to launch it in dev time)

This is a swift app - which is a client to the goosed service (scripts here are just for helpers to launch it in dev time)

validating changes when needed, can do things like this (or be smarter) with iphone 17:
```sh
xcodebuild -scheme Goose -destination 'platform=iOS Simulator,name=iPhone 17' -configuration Debug build 2>&1 | tail -50
```

critical: Never git commit or change git state. 

You should try to complete the task if you can, don't stop if you think it can be done.

---
> Source: [aaif-goose/goose-mobile](https://github.com/aaif-goose/goose-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
