## parquet-viewer

> - It compiles Parquet, Arrow, Datafusion, OpenDAL to WebAssembly.

# Architecture
- It compiles Parquet, Arrow, Datafusion, OpenDAL to WebAssembly.

# UI guideline
- Use Tailwind CSS for styling. Use light theme. Be compact and minimalistic.
- Use DaisyUI (https://daisyui.com) for components.
- Consider both dark and light themes.
- Every visible text must be copiable, meaning that if you hover any text, make sure it's selectable.

# Reactivity guideline
- Prefer explicit reactivity: pass signals/ReadSignal props and read them inside hooks instead of wrapping with `use_reactive!`.

---
> Source: [XiangpengHao/parquet-viewer](https://github.com/XiangpengHao/parquet-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
