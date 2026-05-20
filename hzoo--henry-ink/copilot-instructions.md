## signals

> global state management

## signals usage
```ts
import { batch, type Signal, useComputed, useSignal } from "@preact/signals";
const count = signal(0);

// access with .value, can be used in components
// think about splitting components to save re-renders
<button onClick={() => count.value++}>{count.value}</button>;
```

use computed within components to save re-renders

```ts
function Word({ word }: { word: { text: string, index: number } }) {
  const isCurrent = useComputed(() => currentWordIndex.value === word.index);

  return (
    <span className={cn(
      "text-blue-500",
      isCurrent && "text-blue-700"
    )}>
      {word.text}
    </span>
  )
}
```

---
> Source: [hzoo/henry.ink](https://github.com/hzoo/henry.ink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
