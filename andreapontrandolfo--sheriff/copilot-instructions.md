## sheriff

> When declaring new variables, prefer having the type inferred by the initial value. If this is impossible or unconvenient, it's ok to express the type annotation explicitly, but if the type is complex, prefer declaring an interface separately and then annotate the variable with the interface.

# Your rule content

When declaring new variables, prefer having the type inferred by the initial value. If this is impossible or unconvenient, it's ok to express the type annotation explicitly, but if the type is complex, prefer declaring an interface separately and then annotate the variable with the interface.

## Examples

### Wrong

```
const x: {
    y?: string
    z?: number
} = {}
```

### Correct

```
interface X = {
    y?: string
    z?: number
}

const x: X = {}
```

---
> Source: [AndreaPontrandolfo/sheriff](https://github.com/AndreaPontrandolfo/sheriff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
