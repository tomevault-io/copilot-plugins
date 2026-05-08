## figma-messages

> Messages are the **only** way to communicate between the plugin sandbox and UI iframe.


# Figma Plugin Message Protocol

Messages are the **only** way to communicate between the plugin sandbox and UI iframe.

## Message Type Definitions

Define all messages in `src/shared/messages.ts` using discriminated unions:

```typescript
/**
 * Messages sent from UI thread to Main thread (plugin sandbox)
 */
export type PluginMessage =
  | { type: "get-selection" }
  | { type: "create-rectangle"; width: number; height: number }
  | { type: "update-node"; nodeId: string; properties: NodeProperties };

/**
 * Messages sent from Main thread to UI thread
 */
export type UIMessage =
  | { type: "selection-changed"; nodes: SelectionNode[] }
  | { type: "plugin-ready" }
  | { type: "error"; message: string };
```

## Design Principles

### 1. Use Discriminated Unions

The `type` field enables type-safe message handling:

```typescript
// TypeScript narrows the type based on msg.type
switch (msg.type) {
  case "create-rectangle":
    // msg is now typed as { type: 'create-rectangle'; width: number; height: number }
    createRect(msg.width, msg.height);
    break;
}
```

### 2. Serialize Data, Not References

Never include Figma node references in messages:

```typescript
// ❌ BAD - node references can't be serialized
{ type: 'node-selected', node: figma.currentPage.selection[0] }

// ✓ GOOD - plain data only
{ type: 'node-selected', nodeId: '1:23', name: 'Frame 1', width: 100 }
```

### 3. Keep Payloads Small

Only send the data you need:

```typescript
// ❌ BAD - sending entire node tree
{ type: 'document-data', root: serializeEntireDocument() }

// ✓ GOOD - send only what UI needs
{ type: 'frame-list', frames: frames.map((f) => ({ id: f.id, name: f.name })) }
```

### 4. Define Serialized Types

Create interfaces for serialized node data:

```typescript
/**
 * Serialized node data safe for postMessage
 */
export interface SelectionNode {
  id: string;
  name: string;
  type: string;
  width: number;
  height: number;
}
```

## Adding New Messages

When adding a new message type:

1. Add the type to `PluginMessage` or `UIMessage` union
2. Add the handler in the appropriate switch statement
3. Test the full round-trip: UI → Plugin → UI

---
> Source: [hoshikitsunoda/figma-plugins-vibe-coding-template](https://github.com/hoshikitsunoda/figma-plugins-vibe-coding-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
