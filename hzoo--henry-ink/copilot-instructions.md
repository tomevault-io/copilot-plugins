## imports

> importing files

use absolute imports (using "@/folder") over relative imports

```diff
- import { activeDocuments } from "../store/whiteboard";
+ import { activeDocuments } from "@/src/store/whiteboard";
```

never use `import *` syntax, always be explicit with imports.

---
> Source: [hzoo/henry.ink](https://github.com/hzoo/henry.ink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
