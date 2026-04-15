## network-graph-search

> Wh


# Reagraph Playbook

## 1) Next.js integration (never break SSR)

- Always render `GraphCanvas` on the client.
- Add `'use client'` at the top of components that use Reagraph.
- Use `next/dynamic` with `{ ssr: false }`.
  🔗 [Docs – Next.js Integration](https://reagraph.dev/docs/getting-started/NextJS)

\`\`\`ts
'use client';
import dynamic from 'next/dynamic';

export const GraphCanvas = dynamic(
	() => import('reagraph').then((m) => m.GraphCanvas),
	{ ssr: false }
);
\`\`\`

---

## 2) Layouts (be explicit)

- Default: `forceDirected2d`.
- Use hierarchical, circular, concentric, or noOverlap when data shape requires.
- Tune with `layoutOverrides`.
  🔗 [Docs – Layouts](https://reagraph.dev/docs/getting-started/Layouts)

\`\`\`tsx
<GraphCanvas
	layoutType="forceDirected2d"
	layoutOverrides={{ linkDistance: 60, nodeStrength: -120 }}
/>
\`\`\`

---

## 3) Override positions safely

- Implement `getNodePosition` and respect active drags.
  🔗 [Docs – Layouts](https://reagraph.dev/docs/getting-started/Layouts)

\`\`\`ts
getNodePosition: (id, { drags }) =>
	drags?.[id]?.position ?? { x: 0, y: 0, z: 0 };
\`\`\`

---

## 4) Clustering (force-directed only)

- Use `clusterAttribute` to group nodes visually.
- Style clusters via theme.
  🔗 [Docs – Clustering](https://reagraph.dev/docs/advanced/Clustering)

\`\`\`tsx
<GraphCanvas layoutType="forceDirected2d" clusterAttribute="category" />
\`\`\`

---

## 5) Sizing strategy (pick one and document it)

- Use `pageRank`, `centrality`, or `attribute`.
- Configure bounds via `minNodeSize`/`maxNodeSize`.
  🔗 [Docs – Sizing](https://reagraph.dev/docs/advanced/Sizing)

\`\`\`tsx
<GraphCanvas
	sizingType="attribute"
	sizingAttribute="score"
	minNodeSize={5}
	maxNodeSize={18}
/>
\`\`\`

---

## 6) Selection UX

- Prefer the `useSelection` hook (built-in hotkeys: `⌘+A`, `Esc`, `⌘+click`).
  🔗 [Docs – Selection](https://reagraph.dev/docs/advanced/Selection)

\`\`\`tsx
const { selections, onNodeClick, onCanvasClick } = useSelection({
	ref,
	nodes,
	edges,
});

<GraphCanvas
	selections={selections}
	onNodeClick={onNodeClick}
	onCanvasClick={onCanvasClick}
/>;
\`\`\`

---

## 7) Edge & label readability

- Use `labelType="auto"` for perf.
- Prefer curved edges when dense.
  🔗 [Docs – API](https://reagraph.dev/docs/API/Api)

---

## 8) Performance defaults

- Keep `animated={true}` for small/medium graphs.
- Switch to `forceAtlas2` for large networks, cap iterations.
  🔗 [Docs – Layouts](https://reagraph.dev/docs/getting-started/Layouts)

---

## 9) Camera & interactions

- Default `cameraMode="pan"`.
- Use `onCanvasClick` to deselect on background click.
  🔗 [Docs – API](https://reagraph.dev/docs/API/Api)

---

## 10) Event contracts

- Consistently wire `onNodeClick`, `onNodeDragged`, `onClusterDragged`, etc.
  🔗 [Docs – API](https://reagraph.dev/docs/API/Api)

---

## 11) Theming & accessibility

- Use a centralized theme.
- For labels, only `.ttf`, `.otf`, `.woff` fonts.
  🔗 [Docs – API](https://reagraph.dev/docs/API/Api)
  🔗 [Docs – Clustering](https://reagraph.dev/docs/advanced/Clustering)

---

## 12) Aggregation & de-overlap

- Use `aggregateEdges` to reduce visual clutter.
- Apply `noOverlap` or concentric layouts if nodes collide.
  🔗 [Docs – Layouts](https://reagraph.dev/docs/getting-started/Layouts)

---

## 13) Custom layouts (last resort)

- Implement `getNodePosition` or a pure layout function.
- Document required node fields.
  🔗 [Docs – Layouts](https://reagraph.dev/docs/getting-started/Layouts)

---

## 14) Minimal scaffold

\`\`\`tsx
'use client';
import dynamic from 'next/dynamic';
import { useRef } from 'react';
import type { GraphCanvasRef, GraphNode, GraphEdge } from 'reagraph';
import { useSelection } from 'reagraph';

export const GraphCanvas = dynamic(
	() => import('reagraph').then((m) => m.GraphCanvas),
	{ ssr: false }
);

const nodes: GraphNode[] = [
	{ id: 'a', label: 'A' },
	{ id: 'b', label: 'B' },
];
const edges: GraphEdge[] = [{ id: 'e1', source: 'a', target: 'b' }];

export default function GraphDemo() {
	const ref = useRef<GraphCanvasRef | null>(null);
	const { selections, onNodeClick, onCanvasClick } = useSelection({
		ref,
		nodes,
		edges,
	});
	return (
		<GraphCanvas
			ref={ref}
			nodes={nodes}
			edges={edges}
			layoutType="forceDirected2d"
			selections={selections}
			onNodeClick={onNodeClick}
			onCanvasClick={onCanvasClick}
			sizingType="centrality"
			clusterAttribute="group"
			animated
		/>
	);
}

Context Menu
reagraph supports context menus on nodes and edges. Out of the box, reagraph supports:

Radial Menu
Custom Menu
Radial Menu
The setup the RadialMenu component, we need to setup the theme first. The radial menu uses CSS variables to define colors. Here is an example of how to define those colors:


body {
  --radial-menu-background: #fff;
  --radial-menu-color: #000;
  --radial-menu-border: #AACBD2;
  --radial-menu-active-color: #000;
  --radial-menu-active-background: #D8E6EA;
}
Once those are defined, we can use the contextMenu callback prop to return a radial menu component. The callback provides the model (node/edge), contextual information around a node’s collapse state, and a callback to close the menu.


import { GraphCanvas, RadialMenu } from 'reagraph';

export const MyApp = () => (
  <GraphCanvas
    nodes={nodes}
    edges={edges}
    contextMenu={({ data, additional, onClose }) => (
      <RadialMenu
        onClose={onClose}
        items={[
          {
            label: 'Add Node',
            onClick: () => {
              alert('Add a node');
              onClose();
            }
          },
          {
            label: 'Remove Node',
            onClick: () => {
              alert('Remove the node');
              onClose();
            }
          }
        ]}
      />
    )}
  />
);
Custom Menu
The contextMenu callback prop can be used to return a custom menu. Below is an example of how to setup a simple menu that displays the label of the node.


import { GraphCanvas } from 'reagraph';

export const Node = () => (
  <GraphCanvas
    nodes={nodes}
    edges={edges}
    contextMenu={({ data, additional, onClose }) => (
      <div
        style={{
          background: 'white',
          width: 150,
          border: 'solid 1px blue',
          borderRadius: 2,
          padding: 5,
          textAlign: 'center'
        }}
      >
        <h1>{data.label}</h1>
        <button onClick={onClose}>Close Menu</button>
      </div>
    )}
  />
);

\`\`\`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kytewy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
