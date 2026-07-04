## mowglinext

> Repository-local working rules for code agents.

# AGENTS.md

Repository-local working rules for code agents.

## ROS2 formatting

- Any change under `ros2/` that touches C++ source or headers (`.cpp`, `.cc`, `.hpp`, `.hh`) must be `clang-format` clean before finishing.
- Use the repository style file:

```bash
clang-format -i -style=file:ros2/.clang-format <touched-files>
```

- Before concluding work on `ros2/` files, verify formatting explicitly. Preferred check:

```bash
git clang-format --diff origin/main -- <touched-files>
```

- If `origin/main` is unavailable locally, at minimum run:

```bash
clang-format -n -style=file:ros2/.clang-format <touched-files>
```

## Non-root builds and tools

- Never build as `root`.
- Never run `pio`, `platformio`, `colcon`, `cmake`, `make`, `ninja`, `npm`, formatters, or generated-file workflows as `root`.
- Always use the normal project user so the repository does not accumulate root-owned files.
- If a previous run created root-owned artifacts, fix ownership or remove those artifacts before continuing.

---
> Source: [mowglinext/mowglinext](https://github.com/mowglinext/mowglinext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
