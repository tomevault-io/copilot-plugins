## figma-ui-thread

> This code runs in an iframe with full browser APIs but NO access to `figma.*`.


# Figma UI Thread (React) Rules

This code runs in an iframe with full browser APIs but NO access to `figma.*`.

## Available APIs

- Full DOM and React
- `fetch()` for network requests
- `window`, `localStorage`, `sessionStorage`
- `parent.postMessage()` to send messages to plugin

## Communication with Plugin

### Sending Messages to Plugin

```typescript
import type { PluginMessage } from "../shared/messages";

const postToPlugin = (message: PluginMessage) => {
  parent.postMessage({ pluginMessage: message }, "*");
};

// Usage
postToPlugin({ type: "get-selection" });
postToPlugin({ type: "create-rectangle", width: 100, height: 100 });
```

### Receiving Messages from Plugin

```typescript
import type { UIMessage } from "../shared/messages";

useEffect(() => {
  const handleMessage = (event: MessageEvent<{ pluginMessage: UIMessage }>) => {
    const msg = event.data.pluginMessage;
    if (!msg) return;

    switch (msg.type) {
      case "selection-changed":
        setSelection(msg.nodes);
        break;
    }
  };

  window.addEventListener("message", handleMessage);
  return () => window.removeEventListener("message", handleMessage);
}, []);
```

## React Patterns

### Functional Components with Hooks

```typescript
// ✓ Use functional components
function SelectionPanel() {
  const [selection, setSelection] = useState<SelectionNode[]>([]);
  // ...
}
```

### Custom Hooks for Plugin Communication

Consider extracting message handling into a custom hook:

```typescript
// src/ui/hooks/usePluginMessage.ts
function usePluginMessage<T>(type: string, handler: (data: T) => void) {
  useEffect(() => {
    const onMessage = (e: MessageEvent) => {
      if (e.data.pluginMessage?.type === type) {
        handler(e.data.pluginMessage);
      }
    };
    window.addEventListener("message", onMessage);
    return () => window.removeEventListener("message", onMessage);
  }, [type, handler]);
}
```

## Styling with Tailwind CSS

Use Tailwind utility classes with CSS variables defined in `index.css`. Never hardcode colors — always use variables with **Tailwind v4 syntax**:

```typescript
// ✓ GOOD - Use CSS variables with Tailwind v4 syntax
className="bg-(--color-bg)"
className="text-(--color-text)"
className="border-(--color-border)"

// ❌ BAD - Hardcoded colors
className="bg-[#0d0d14]"
className="text-[#e8e8e8]"

// ❌ AVOID - Legacy arbitrary value syntax
className="bg-[var(--color-bg)]"
```

## Network Requests

All API calls must happen in UI thread:

```typescript
// ✓ Fetch in UI, send result to plugin
const data = await fetch("https://api.example.com/data").then((r) => r.json());
postToPlugin({ type: "api-response", data });
```

---
> Source: [hoshikitsunoda/figma-plugins-vibe-coding-template](https://github.com/hoshikitsunoda/figma-plugins-vibe-coding-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
