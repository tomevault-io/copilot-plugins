## next-saas-template

> Shared UI library usage and React component conventions


- **Source of UI components**
  - Import primitives from `@superscale/ui`; see available exports in ([packages/ui/src/index.ts](mdc:packages/ui/src/index.ts))
  - Prefer composition of primitives over custom ad-hoc components

- **Class name order**
  - Follow Biome rule `nursery.useSortedClasses` to keep Tailwind classes sorted ([biome.json](mdc:apps/web/biome.json))

- **Client components**
  - Add `"use client"` only where interactivity is required; keep most components server-side for perf

```tsx
// ✅ DO: Use shared Button
import { Button } from '@superscale/ui';

export function SubmitBar({ onSubmit }: { onSubmit: () => void }) {
  return (
    <div className="flex items-center justify-end gap-2">
      <Button onClick={onSubmit}>Save</Button>
    </div>
  );
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iantanwx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
