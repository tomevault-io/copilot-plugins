## statsig-kit

> This repository contains the Statsig client SDK for Apple platforms (iOS, macOS, watchOS, tvOS, and visionOS), with support for both Swift and Objective-C applications.

# StatsigKit: Statsig SDK for Swift and Objective-C

This repository contains the Statsig client SDK for Apple platforms (iOS, macOS, watchOS, tvOS, and visionOS), with support for both Swift and Objective-C applications.

## Formatting

Run this before opening a PR:

```bash
swift format -i -r Sources/Statsig/ Tests/ Sample/App Package.swift
```

## Testing

To run tests:

```bash
./run_tests.sh
```

This runs `xcodebuild test`, forwarding any extra arguments. For example, to run specific tests, run:

```bash
./run_tests.sh -only-testing StatsigTests/StatsigUserSpec
```

Running all tests can be flaky at the moment. When developing a feature, prefer running tests specific to the feature.

---
> Source: [statsig-io/statsig-kit](https://github.com/statsig-io/statsig-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
