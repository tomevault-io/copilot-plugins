## example-php-block

> This plugin demonstrates PHP-only block registration using WordPress 7.0+ `autoRegister`.

# Example PHP Block

This plugin demonstrates PHP-only block registration using WordPress 7.0+ `autoRegister`.

## Skills

- **php-block** - Best practices for creating PHP-only blocks

## Key Patterns

1. Use `autoRegister => true` in supports
2. Register CSS/JS with `wp_register_*`, pass handles to block registration
3. JavaScript only works on frontend - enqueue in `render_callback`
4. Use `get_block_wrapper_attributes()` for block supports to apply

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bacoords) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
