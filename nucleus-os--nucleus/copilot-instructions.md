## nucleus

> Sometimes I will be making other changes as you work. Do not expect `git status` to be limited to only your own changes. Never revert any changes without explicit request/permission from the user, even if they are breaking builds.

Sometimes I will be making other changes as you work. Do not expect `git status` to be limited to only your own changes. Never revert any changes without explicit request/permission from the user, even if they are breaking builds.

React 19 and React Compiler are enabled - do not use useMemo, useCallback, etc.

react-native-gpui (RNGPUI) is a new out-of-tree platform for React Native that implements RN's Fabric architecture using RN's shared C++ core and a Rust implementation that bridges to Zed's GPUI (~/rngpui/zed/crates/gpui/ ). It targets desktop apps for macOS, Linux, and Windows. It does not support Android or iOS.

Patches/edits within react-native (i.e. node_modules copy) is forbidden. RNGPUI is intended to work with the publicly published RN version.

Do not attempt to build or run without explicit request from the user. Just focus on coding.

---
> Source: [nucleus-os/nucleus](https://github.com/nucleus-os/nucleus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
