## veracity

> Order use statements as std, vstd, crate types, chapter modules, macros


# Use Statement Order

All `use` statements should appear immediately after the module declaration, in this order:

## Order

1. `use std::...` - standard library imports
2. *(blank line)*
3. `use vstd::prelude::*;` - Verus prelude
4. `use crate::Types::Types::*;` - project types
5. `use crate::Chap05::...::*;` - chapter modules (glob import)
6. `use crate::XLit;` - macro imports (not glob)

## Example

```rust
pub mod MyModule {

    use std::fmt::{Debug, Display};
    use std::hash::Hash;

    use vstd::prelude::*;
    use crate::Types::Types::*;
    use crate::Chap05::SetStEph::SetStEph::*;
    use crate::SetLit;

    // ... rest of module
}
```

## Rules

- APAS modules always use a single glob import (`*`) - never `{*, SomeType}`
- Macros (like `SetLit`, `RelationLit`) are imported by name, not glob
- Keep a blank line between std imports and the rest

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/briangmilnes)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/briangmilnes)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
