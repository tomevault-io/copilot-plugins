## relative-import

> не надо делать локальные импорты через относительные пути:

не надо делать локальные импорты через относительные пути:

```go
// плохо
import "../../config"

// хорошо
import "github.com/comerc/budva43/some/path/config"
```

---
> Source: [comerc/budva43](https://github.com/comerc/budva43) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
