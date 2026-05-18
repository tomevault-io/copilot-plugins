## torii

> When writing Rust code for Torii


# Your rule content

- Adhere to existing style
- Prefer writing out builders instead of deriving them
- Keep in mind there can be many auth plugins registered
- Focus on writing secure code
- Use Tokio as the async runtime
- Ensure .await works correctly by ensuring Send + Sync on types that cross the await boundary

---
> Source: [cmackenzie1/torii-rs](https://github.com/cmackenzie1/torii-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
