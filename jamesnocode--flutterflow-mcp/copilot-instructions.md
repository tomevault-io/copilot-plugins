## flutterflow-mcp

> First-class widget workflows for reliable query/edit operations.


# Widgets Commands

Read commands:
- Inventory: `widgets.list`
- Search: `widgets.find`, `widgets.findText`
- Text-focused inventory: `textfields.list`
- Single read: `widget.get`
- Batch read: `widget.getMany`

Edit commands:
- Create: `widget.create` / `widget.insert`
- Update: `widget.set` / `widgets.updateMany`
- Structure: `widget.wrap`, `widget.unwrap`, `widget.move`, `widget.reorder`, `widget.moveMany`
- Delete: `widget.delete`, `widget.deleteSubtree`

Required args:
- Page selector: `nameOrId` or `pageId` or `fileKey`
- Widget selector: `nodeId`
- For create: `type` plus placement selector (`parentNodeId`, `beforeNodeId`, or `afterNodeId`)

Performance/safety:
- Prefer `widgets.list` filters (`type`, `include`) or `textfields.list` over N+1 `widget.get` loops.
- Resolve exact `nodeId` before write operations.
- In non-split pages, ensure key-path updates are node-scoped before set/delete.

Text updates:
- Prefer `widget.set` with `text` for common text edits.
- Use explicit key path only when required:
  - `props.text.textValue.inputValue`
  - mirror `props.text.textValue.mostRecentInputValue` when present

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jamesnocode)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jamesnocode)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
