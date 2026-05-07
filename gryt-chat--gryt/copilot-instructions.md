## icons

> Icon library conventions — use react-icons (Material Design) only


# Icons: react-icons/md only

- **Use `react-icons/md`** (Material Design icons) for all icons.
- **Never use `lucide-react`**, `@heroicons`, `@phosphor-icons`, or other icon libraries.
- Always set an explicit `size` prop on every icon (e.g. `<MdRefresh size={16} />`).

```tsx
// BAD
import { RotateCw } from "lucide-react";
<RotateCw size={14} />

// GOOD
import { MdRefresh } from "react-icons/md";
<MdRefresh size={14} />
```

---
> Source: [Gryt-chat/gryt](https://github.com/Gryt-chat/gryt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
