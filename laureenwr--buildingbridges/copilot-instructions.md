## buildingbridges

> `TreeVisualization` is a client component located at `components/onboarding/TreeVisualization.tsx`. It is designed to render a hierarchical, tree-like structure for the user onboarding process.

# TreeVisualization Component Guide

`TreeVisualization` is a client component located at `components/onboarding/TreeVisualization.tsx`. It is designed to render a hierarchical, tree-like structure for the user onboarding process.

## Functionality

*   **Recursive Rendering**: The component takes a `data` object and recursively renders nodes and their children.
*   **Interactivity**: Each node in the tree is a button. Clicking a node triggers the `onNodeClick` callback function, passing the `id` of the clicked node.
*   **State Management**: The component receives the selection state (`selected`) for each node via its props. It does not manage this state internally.
*   **Animation**: It uses [Framer Motion](https://www.framer.com/motion/) to animate the appearance of nodes, providing a staggered, visually appealing effect.
*   **Accessibility**: The component includes ARIA roles (`tree`, `treeitem`, `group`) to improve accessibility for screen readers.

## Props

*   **`data`**: A `TreeNode` object that represents the root of the tree. The `TreeNode` interface requires an `id`, `label`, `selected` property, and an optional `children` array of `TreeNode` objects.
*   **`onNodeClick`**: A callback function of type `(id: string) => void` that is executed when a node is clicked.

## Usage

This component is used in the onboarding flow to allow users to make selections in a hierarchical manner. The parent component is responsible for providing the tree data and handling the state changes when a node is clicked.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/laureenwr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
