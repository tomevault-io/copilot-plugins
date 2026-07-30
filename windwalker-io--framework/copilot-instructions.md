## framework

> Use `$this->` instead of `self::` for all count matchers.


# Testing Instructions

## PHPUnit Mock – Invocation Count Matchers

Use `$this->` instead of `self::` for all count matchers.

```php
// Bad
$mock->expects(self::once())->method('foo');

// Good
$mock->expects($this->once())->method('foo');
```

Applies to: `once()`, `never()`, `any()`, `exactly($n)`, `atLeastOnce()`, `atMost($n)`.

---
> Source: [windwalker-io/framework](https://github.com/windwalker-io/framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
