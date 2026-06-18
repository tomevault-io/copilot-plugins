## apextree-skill

> >


# ApexTree AI Skill

> **Framework wrapper detection — check `package.json` before generating code.**
> - `react` → use **`react-apextree`** instead of the core API.
> - `vue` → use **`vue-apextree`**.
> - `@angular/core` → use **`ngx-apextree`**.
>
> Wrappers handle `destroy()` automatically on unmount, accept reactive props, and forward events as idiomatic framework events. Use the core API directly only when no framework is detected, or when the user explicitly asks for vanilla. See `references/framework-wrappers.md`.

## 1. Critical Rules

1. **Data is a single `NestedNode`** (the root) passed to `render(data)`, **not** to the constructor. Constructor takes `(element, options)`; `render(rootNode)` paints.
2. **Every node needs `id`, `name`, and `children`.** Leaves have `children: []` — not `undefined` and not omitted.
3. **`id` must be unique across the entire tree.** Selection, edge highlighting, search, and breadcrumb all key off `id`.
4. **`render(data)` returns a `Graph` instance.** Save it — it exposes `collapse`, `expand`, `fitScreen`, `centerOnNode`, search APIs, and selection APIs.
5. **`enableExpandCollapseZoom: true` (default)** re-fits the viewBox on collapse / expand. Set to `false` to keep the camera locked.
6. **`enableSelection` is `'single' | 'multi' | false`**, not a boolean. `false` means selection is off; `'single'` and `'multi'` are the active modes.
7. **`onSelectionChange(listener)` is on the `Graph`** returned by `render`, not on `tree` and not in `options`.
8. **Use `contentKey: 'data'`** to switch the built-in template into org-card mode (avatar, name, title, subtitle, badge, accent stripe). Don't write a custom `nodeTemplate` for that — the built-in one is already there.
9. **For custom rendering use `nodeTemplate(content)`** where `content` is the value at `contentKey`. Set `contentKey: 'data'` for structured payloads.
10. **`direction` is `'top' | 'bottom' | 'left' | 'right'`** — controls where the root sits and which way the tree grows.
11. **Per-node options** live on `node.options` and can override font, border, tooltip, and node visuals for a single node.
12. **Call `destroy()`** before unmounting in React / Vue / Angular — it frees `ResizeObserver`s and tooltip DOM.
13. **Set the license key once** at app startup with `ApexTree.setLicense('KEY')` to remove the watermark.

---

## 2. Data Format — `NestedNode`

```ts
interface NestedNode<T = undefined> {
  id: string;                                          // REQUIRED, unique across the whole tree
  name: string;                                        // REQUIRED, display label (when contentKey is 'name')
  children: NestedNode<T>[];                           // REQUIRED — empty array for leaves
  data?: T;                                            // arbitrary payload, used when contentKey: 'data'
  options?: Partial<FontOptions & NodeOptions & TooltipOptions>;   // per-node overrides
}
```

Minimal example:

```js
import { ApexTree } from 'apextree';

const data = {
  id: '1', name: 'CEO',
  children: [
    { id: '2', name: 'CTO', children: [
      { id: '4', name: 'Eng Lead', children: [] },
      { id: '5', name: 'QA Lead',  children: [] },
    ]},
    { id: '3', name: 'CFO', children: [] },
  ],
};

const tree = new ApexTree(document.getElementById('chart'), {
  width: 800, height: 600,
  nodeWidth: 120, nodeHeight: 40,
  childrenSpacing: 70, siblingSpacing: 30,
  direction: 'top',
});

const graph = tree.render(data);
```

### Org-card mode (built-in `'data'` template)

Set `contentKey: 'data'` and put structured fields into `node.data`. The built-in template renders avatar / name / title / subtitle / badge / accent stripe automatically:

```js
const data = {
  id: 'ceo', name: 'Alice',
  data: {
    name: 'Alice Johnson',
    title: 'Chief Executive Officer',
    subtitle: 'Executive',
    imageURL: 'https://example.com/alice.jpg',
    accentColor: '#6366f1',
    badge: { text: 'Active', color: '#EEF2FF' },
  },
  children: [],
};

new ApexTree(el, {
  contentKey: 'data',
  nodeWidth: 220, nodeHeight: 80,
}).render(data);
```

### Custom `nodeTemplate`

```js
new ApexTree(el, {
  contentKey: 'data',
  nodeWidth: 200, nodeHeight: 80,
  nodeTemplate: (c) => `
    <div style="display:flex;align-items:center;gap:8px;padding:8px;">
      <img src="${c.img}" style="width:32px;height:32px;border-radius:50%;" />
      <div>
        <div style="font-weight:600;">${c.name}</div>
        <div style="font-size:11px;color:#666;">${c.role}</div>
      </div>
    </div>`,
}).render(data);
```

### Per-node style overrides

```js
const data = {
  id: 'ceo', name: 'CEO',
  options: { nodeBGColor: '#EEF2FF', borderColor: '#A5B4FC' },
  children: [
    { id: 'cto', name: 'CTO',
      options: { nodeBGColor: '#ECFDF5', borderColor: '#6EE7B7' },
      children: [] },
  ],
};
```

---

## 3. Top-Level Options

| Option | Type | Default | Notes |
|---|---|---|---|
| `width` / `height` | `number \| string` | `'100%'` / `'auto'` | Canvas size. `'auto'` sizes to content. |
| `viewPortWidth` / `viewPortHeight` | `number` | `800` / `600` | Internal SVG viewport. |
| `direction` | `'top'\|'bottom'\|'left'\|'right'` | `'top'` | Where the root sits. |
| `contentKey` | `string` | `'name'` | Key on the data object used as the label. Set to `'data'` for structured payloads. |
| `siblingSpacing` / `childrenSpacing` | `number` | `50` / `50` | Horizontal / vertical spacing (px). |
| `nodeWidth` / `nodeHeight` | `number` | `50` / `30` | Node dimensions (px). |
| `theme` | `'light'\|'dark'\|'custom'` | `'light'` | Built-in palette. `'custom'` disables CSS-var injection so host vars win. |
| `enableAnimation` | `boolean` | `true` | Animate expand/collapse transitions. |
| `enableExpandCollapse` | `boolean` | `true` | Show expand/collapse buttons on parent nodes. |
| `enableExpandCollapseZoom` | `boolean` | `true` | Re-fit viewBox on collapse / expand. |
| `enableToolbar` | `boolean` | `false` | Zoom/pan/export toolbar. |
| `enableSearch` | `boolean` | `false` | Search input in the toolbar. |
| `enableSelection` | `'single'\|'multi'\|false` | `false` | Selection mode. |
| `enableBreadcrumb` | `boolean` | `false` | Path-from-root breadcrumb. |
| `groupLeafNodes` | `boolean` | `false` | Stack leaves vertically. |
| `highlightOnHover` | `boolean` | `true` | Highlight node + connecting edges on hover. |
| `edgeStyle` | `'orthogonal'\|'curved'\|'straight'` | `'orthogonal'` | Connector shape. |
| `edgeColorMode` | `'default'\|'node'` | `'default'` | `'node'` = each edge inherits the child node's `borderColor`. |
| `nodeTemplate` | `(content) => string` | built-in | Custom node HTML. |
| `enableTooltip` | `boolean` | `false` | Hover tooltip. |
| `onNodeClick` | `(node) => void` | — | Click callback (raw node data). |
| `a11y` | `{ enabled?, label? }` | `{ true, 'Organizational chart' }` | WCAG 2.1 AA + keyboard nav. |

---

## 4. Lifecycle

```js
import { ApexTree } from 'apextree';

ApexTree.setLicense('KEY');                  // optional, once at app startup

const tree = new ApexTree(el, options);      // 1. construct
const graph = tree.render(rootNode);         // 2. paint — REQUIRED. KEEP the returned Graph.

graph.collapse('node-2');                    // imperative API
graph.expand('node-2');
graph.changeLayout('left');                  // switch direction at runtime
graph.fitScreen();
graph.centerOnNode('node-7');
graph.exportToSvg();

graph.construct(newRoot);                    // replace data + re-render

tree.destroy();                              // before unmount
```

**Re-renders**: call `graph.construct(newData)` (cheaper) or re-call `tree.render(newData)`.

---

## 5. Public API

### Tree (`ApexTree`)

| Method | Description |
|---|---|
| `new ApexTree(el, opts?)` | Construct; applies dimensions to `el`. |
| `render(rootNode)` | Paint. Returns a `Graph`. **Required.** |
| `destroy()` | Tear down. |
| `getInstanceId()` | Unique chart instance id. |
| `ApexTree.setLicense(key)` | Static; call once at app startup. |

### Graph (returned by `render`)

| Method | Description |
|---|---|
| `construct(data)` | Replace tree data and re-render. |
| `changeLayout(direction)` | Switch direction at runtime (`'top'\|'bottom'\|'left'\|'right'`). |
| `collapse(id)` / `expand(id)` | Programmatic expand/collapse. |
| `fitScreen()` | Re-fit viewBox to all visible nodes. |
| `centerOnNode(id)` | Pan/zoom the camera to a node, keeping the current zoom. |
| `getNodeMap()` | Map of all node ids → node objects. |
| `getRootNodeId()` | Id of the root. |
| `getNodeLabel(id)` | Resolved display label for a node. |
| `findNodesByQuery(q)` | Returns node ids whose labels contain `q` (case-insensitive). |
| `setSearchHighlight(ids)` | Highlight nodes; pass `[]` to clear. |
| `getSelection()` | Currently selected node ids (insertion order). |
| `setSelection(ids)` | Replace selection. In `'single'` mode only the first id is applied. |
| `clearSelection()` | Reset. |
| `onSelectionChange(listener)` | Subscribe; pass `null` to unsubscribe. |
| `setBreadcrumbHandler(fn)` | Click-on-segment callback for the breadcrumb. |
| `exportToSvg()` | Trigger SVG file download. |

---

## 6. Pitfalls — ❌ Wrong vs ✅ Correct

### 1. Data on the constructor
❌ `new ApexTree(el, { id: '1', name: 'A', children: [...] })`
✅ `new ApexTree(el, options).render(rootNode)`

### 2. Missing `children: []` on a leaf
❌ `{ id: '5', name: 'Leaf' }` — runtime errors when iterating.
✅ `{ id: '5', name: 'Leaf', children: [] }`

### 3. Duplicate ids
❌ Two nodes with `id: 'a'` — selection and edge highlighting target the wrong one.
✅ Make ids globally unique across the whole tree.

### 4. Missing `render()`
❌ `new ApexTree(el, opts)` — nothing paints.
✅ `tree.render(data)`.

### 5. Treating `enableSelection` as a boolean
❌ `enableSelection: true` — TypeScript errors / no selection.
✅ `enableSelection: 'single'` or `enableSelection: 'multi'`.

### 6. Subscribing to selection changes via options
❌ `onSelectionChange` doesn't exist on `TreeOptions`.
✅ `graph.onSelectionChange((ids) => …)` after `render()`.

### 7. Custom org-card with a custom template
❌ Hand-rolling avatar HTML when the built-in works.
✅ Set `contentKey: 'data'` and supply `data: { name, title, subtitle, imageURL, accentColor, badge }` — the built-in template renders the org card.

### 8. Forgetting `destroy()` in React / Vue
❌ ResizeObservers leak across unmounts.
✅ Return `() => tree.destroy()` from `useEffect` cleanup / `onBeforeUnmount` / `ngOnDestroy`.

### 9. Re-rendering the whole tree just to swap data
❌ `tree.destroy(); new ApexTree(el, opts).render(newData)`.
✅ `graph.construct(newData)` — same chart instance, re-renders with new data.

### 10. Mutating the data tree in place
❌ `data.children.push(newChild)` — chart doesn't update.
✅ `graph.construct({ ...data, children: [...data.children, newChild] })`.

### 11. Hovering shows nothing
❌ `enableTooltip` is `false` by default.
✅ `enableTooltip: true` (and optionally pass a `tooltipTemplate`).

### 12. License watermark in production
❌ No license call.
✅ `ApexTree.setLicense('KEY')` once at app startup.

---

## 7. Reference Routing Table

| Topic | Reference File |
|---|---|
| `NestedNode` shape, org-card mode, per-node options, `nodeTemplate` | `references/data-format.md` |
| Graph API, search, breadcrumb, selection, layout, exporting | `references/graph-api.md` |
| React / Vue / Angular wrappers | `references/framework-wrappers.md` |

---
> Source: [apexcharts/apextree-skill](https://github.com/apexcharts/apextree-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
