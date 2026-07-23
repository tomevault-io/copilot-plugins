## openattributegraph

> OpenAttributeGraph is the Swift package implementation of AttributeGraph-compatible primitives used by OpenSwiftUI. For Darwin compatibility, its Swift-facing implementation is mirrored into `DarwinPrivateFrameworks/AG`, where the private-framework package owns Swift interface generation and xcframework updates.

# OpenAttributeGraph Guide

OpenAttributeGraph is the Swift package implementation of AttributeGraph-compatible primitives used by OpenSwiftUI. For Darwin compatibility, its Swift-facing implementation is mirrored into `DarwinPrivateFrameworks/AG`, where the private-framework package owns Swift interface generation and xcframework updates.

## Everyday Workflow

1. Make the code change.
2. Format changed Swift files:

   ```sh
   Scripts/format-swift.sh
   ```

3. Build and test locally:

   ```sh
   swift build
   swift test
   ```

4. Commit with a descriptive message.

## DarwinPrivateFrameworks Sync

Run this when changes affect the AttributeGraph Swift API, shim behavior, or Swift files that must be reflected in `DarwinPrivateFrameworks`.

`DarwinPrivateFrameworks` should exist next to this checkout at `../DarwinPrivateFrameworks`. See `../DarwinPrivateFrameworks/CLAUDE.md` and `../DarwinPrivateFrameworks/AG/README.md` for the framework-side build and update details.

Sync only the Swift implementation files:

- Source: `Sources/OpenAttributeGraph/**/*.swift`
- Destination: `../DarwinPrivateFrameworks/AG/DeviceSwiftShims/**/*.swift`

Preserve the relative file layout when syncing. Do not use `.ag_template` as the handoff artifact; `Scripts/gen_ag_template.sh` is legacy for this workflow. Swift interface generation is handled inside DarwinPrivateFrameworks by its AG update scripts.

After syncing the Swift files, update DarwinPrivateFrameworks from that repo:

```sh
cd ../DarwinPrivateFrameworks
swift package update-xcframeworks --allow-writing-to-package-directory
```

Then verify this package against the local DarwinPrivateFrameworks checkout:

```sh
cd ../OpenAttributeGraph
OPENATTRIBUTEGRAPH_USE_LOCAL_DEPS=1 OPENATTRIBUTEGRAPH_COMPATIBILITY_TEST=1 swift build
```

Create and merge the DarwinPrivateFrameworks PR before updating this package's resolved dependency:

```sh
swift package update DarwinPrivateFrameworks
git add Package.resolved
git commit -m "Update DarwinPrivateFrameworks dependency"
```

## Key Commands

- `Scripts/format-swift.sh` - Format changed Swift files.
- `swift build` - Build the normal OpenAttributeGraph implementation.
- `swift test` - Run the normal local test suite.
- `OPENATTRIBUTEGRAPH_USE_LOCAL_DEPS=1 OPENATTRIBUTEGRAPH_COMPATIBILITY_TEST=1 swift build` - Build against sibling `../DarwinPrivateFrameworks`.
- `swift package update DarwinPrivateFrameworks` - Refresh the resolved DarwinPrivateFrameworks dependency after its PR lands.

## Notes

- Keep OpenAttributeGraph implementation changes and DarwinPrivateFrameworks sync changes in separate PRs when both repos need updates.
- Do not manually edit generated Swift interface files in this repo; DarwinPrivateFrameworks owns that generation.
- Prefer environment variables over temporary `Package.swift` edits for local dependency testing.

---
> Source: [OpenSwiftUIProject/OpenAttributeGraph](https://github.com/OpenSwiftUIProject/OpenAttributeGraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
