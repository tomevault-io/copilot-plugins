## utils-rs-build

> Build instructions for utils_rs Rust bindings


# Building utils_rs

When modifying Rust code in `src/utils_rs/`, rebuild the package using:

```bash
uv pip install --force-reinstall -e src/utils_rs
```

Do NOT use `maturin develop` or similar commands.

## After Editing

After making changes to the Rust code, update the Python type stubs:

- Update `src/utils_rs/utils_rs.pyi` to reflect any API changes (new functions, changed signatures, etc.)

---
> Source: [NVlabs/alpasim](https://github.com/NVlabs/alpasim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
