## aya

> - Repository: aya (Rust library and tooling for working with eBPF programs).

# AGENTS NOTES

- Repository: aya (Rust library and tooling for working with eBPF programs).
- Development tooling:
  - Do not regenerate public API fixtures; the user handles that.
  - Many crates only build on Linux; on macOS lint and type check using

  ```sh
  cargo xtask clippy -- --target x86_64-unknown-linux-musl
  ```

- Coding guidelines:
  - Use github or bootlin permalinks when referencing kernel sources.
- Testing (if local machine is not Linux):
  - Prepare integration tests:

    ```sh
    .github/scripts/download_kernel_images.sh \
      test/.tmp/debian-kernels/<arch> <arch> [VERSIONS]...
    ```

  - Run in a VM:

    <!-- markdownlint-disable line-length -->

    ```sh
    find test/.tmp -name '*.deb' -print0 | sort -Vz | xargs -t -0 sh -c \
      'cargo xtask integration-test vm [-p <PACKAGE>] --cache-dir test/.tmp "$@" -- <test-filter> [ARGS]...' _
    ```

    <!-- markdownlint-restore -->

---
> Source: [aya-rs/aya](https://github.com/aya-rs/aya) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
