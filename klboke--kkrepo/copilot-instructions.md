## kkrepo

> - Check the official protocol and Nexus behavior before changing hosted, proxy, or group repository behavior.


# Protocol Compatibility Instructions

- Check the official protocol and Nexus behavior before changing hosted, proxy, or group repository behavior.
- Keep URL layout compatible with `/repository/<repo>/...` unless a protocol-specific endpoint requires another layout.
- For hosted writes, preserve write policy, checksum, metadata, and error response semantics.
- For proxy repositories, treat upstream status, headers, cache TTLs, negative cache, conditional requests, and body-read failures carefully.
- For group repositories, preserve member ordering and deterministic conflict handling.
- Add or update black-box compatibility tests under `compat-test` when protocol behavior changes.
- Do not mark protocol work complete until real client behavior or compatibility tests cover the changed path.

---
> Source: [klboke/kkRepo](https://github.com/klboke/kkRepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
