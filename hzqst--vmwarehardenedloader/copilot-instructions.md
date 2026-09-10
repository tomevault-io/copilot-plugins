## vmwarehardenedloader

> - Kernel C++ uses SAL annotations, `#pragma once` headers, anonymous namespaces for file-local state, `VmLoader...` exported module APIs, `k...` constexpr constants, and `g_...` global hook state.


# Code and safety conventions

- Kernel C++ uses SAL annotations, `#pragma once` headers, anonymous namespaces for file-local state, `VmLoader...` exported module APIs, `k...` constexpr constants, and `g_...` global hook state.
- Prefer explicit NTSTATUS propagation and early returns; zero/clear output structures on entry and clean pool allocations on every failure path. Existing helpers use tagged paged/nonpaged allocations and `PAGED_CODE()` where required.
- Hook lifecycle is stateful and reversible: acquire the firmware ERESOURCE around provider traversal/patching, cap list walks at 64 entries, preserve original handlers, and restore them on unload. PnP callback registration is guarded by a boolean and callback cookie.
- PnP filtering is intentionally user-mode-only (`ExGetPreviousMode() == UserMode`) and limited to enum branches ending in \\Enum\\PCI, \\Enum\\USB, or \\Enum\\HDAUDIO; VMware markers are VEN_15AD/VID_0E0F.
- Firmware filters mutate provider buffers in place; ACPI mutations must preserve table bounds and recompute checksum. Keep replacement strings equal length.
- Treat kernel symbol inputs as hostile: validate bounded PE headers, exact machine/timestamp/image-size identity, dynamic-data version/fields, file/signature size limits, local non-network path normalization, and writable + non-executable RVAs before dereference.
- Avoid broad refactors around undocumented kernel globals or SYSTEM_FIRMWARE_TABLE_HANDLER layout; Windows updates can invalidate assumptions. Keep changes local and preserve rollback behavior.

---
> Source: [hzqst/VmwareHardenedLoader](https://github.com/hzqst/VmwareHardenedLoader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
