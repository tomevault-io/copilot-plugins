## xserver-auto-renew-rs

> This program is a command-line tool designed to automate the process of renewing contracts for services on Xserver, a Japanese hosting provider. It handles logging in, solving CAPTCHAs, and performing the renewal actions automatically.

### Repository Description: xserver-auto-renew-rs

#### **Objective**

This program is a command-line tool designed to automate the process of renewing contracts for services on Xserver, a Japanese hosting provider. It handles logging in, solving CAPTCHAs, and performing the renewal actions automatically.

#### **Project Structure**

```
src/           # Rust source code
├── main.rs    # Application entry point
├── cli.rs     # Command line interface definitions
├── client/    # Logic for logging in and interacting with Xserver
├── data/      # Local data storage and utilities
├── external/  # CAPTCHA solver and webhook integrations
├── logger.rs  # Simple file based logger
├── ops.rs     # Status output and cleanup helpers
├── task.rs    # systemd timer/service management
└── update.rs  # Self‑update functionality
systemd/       # Template systemd unit files
install.sh     # Convenience installer for prebuilt binaries
release.py     # Script for tagging and releasing
```

#### **Development Guidelines**

1.  **Dependencies**: Always use `cargo add <crate-name>` to add a new library. This ensures the latest version is added to `Cargo.toml`.
2.  **Code Formatting**: Run `cargo fmt` to format the code before finalizing any changes.
3.  **Rust Edition**: This project is locked to the `2024` edition. Do not downgrade it.

---
> Source: [h-sumiya/xserver-auto-renew-rs](https://github.com/h-sumiya/xserver-auto-renew-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
