## retrodiffusion

> When creating a GitHub Issue, adhere to these guidelines

# Creating GitHub Issues Guide

**IMPORTANT PREVIEW STEP:** Before using any tools to create the Issue on GitHub, you MUST first show a complete Markdown preview of the Issue (Title and Body) to the user. Ask for their confirmation before proceeding.

This guide outlines the process for creating GitHub Issues.

**Key Issue Details:**

*   **Repository:** All Issues should be created in the `https://github.com/superbuilders/retrodiffusion` repository.
*   **Title:** (Required) Create a clear, concise, and specific title that summarizes the issue or the desired outcome.
    *   *Bad Example: "Map bug"*
    *   *Good Example: "Game crashes when opening the inventory on level 3"*
*   **Body/Description:** (Required) The issue body MUST be structured using the following Markdown headings: `### Problem`, `### Goals`, and `### Action Items`. Adapt the content under each heading as appropriate for the issue type (bug, feature, etc.).
    *   **Problem/Context:** Clearly explain the current situation, the problem to be solved, or the reason for the feature request. Provide background context.
        *   *For bugs:* Include specific steps to reproduce, expected behavior, and actual behavior.
    *   **Goals/Proposed Solution:** Clearly define what success looks like or what the proposed solution or feature should achieve.
    *   **Action Items/Tasks:** Break down the work into smaller, actionable steps. Keep these points at the top-level (no sub-lists).
    *   **Technical Details:** If relevant, mention specific files (e.g., `path/to/file` — DO NOT prefix with `@`), functions, or technologies involved.

**Issue Formatting Notes:**

*   Use Markdown only for section headings (e.g. `### Problem`) or for task lists under the `### Action Items` heading (e.g. `- [ ] Action item here`)
*   If referencing other issues or PRs, use the `#issue_number` or `#pr_number` syntax.

**Example of a Well-Structured Issue Body:**

```markdown
## Problem

The current implementation in `apps/cademy/src/lib/tiledLoader.ts` renders Tiled maps by creating individual `THREE.Mesh` objects for each tileset... This leads to performance issues due to many draw calls.

1.  **Performance**: It generates potentially large, merged BufferGeometry objects and requires a separate draw call for each mesh (tileset x layer  combination). For maps with many layers or using multiple tilesets per layer, this can lead to a significant number of draw calls, impacting rendering performance.
2.  **Scalability**: As maps grow in size or complexity, the performance cost of this approach increases.
3.  **Dynamic Updates**: Implementing features like tile animations becomes complex, as it requires identifying and manually updating specific vertex data (especially UVs) within potentially very large, shared buffer attributes.

## Goals

1.  **Improve Rendering Performance**: Significantly reduce the number of draw calls required to render the Tiled map by leveraging `THREE.InstancedMesh`
2.  **Increase Scalability**: Create a rendering architecture that handles larger and more complex maps more efficiently
3.  **Simplify Dynamic Tile Updates**: Establish a foundation where properties of individual tiles (like their texture UVs for animation) can be modified more easily via per-instance attributes, rather than direct buffer manipulation of large geometries.

## Action Items

- [ ]  Define Base Geometry: Create a single, reusable `THREE.PlaneGeometry` representing the dimensions of one tile (`mapData.tilewidth`, `mapData.tileheight`).
- [ ]  Create Custom Shaders: Write a vertex shader (`tile.vert`) that applies the `instanceMatrix` and calculates the final `vUv` coordinates for the fragment shader based on the base plane's UVs and new per-instance UV attributes and then write a fragment shader (`tile.frag`) that samples the correct tileset texture (`sampler2D` uniform) using the calculated vUv and performs necessary alpha testing. Ensure correct color space handling (e.g., `sRGBEncoding` considerations).
- [ ]  Implement ShaderMaterial: Create a `THREE.ShaderMaterial` using the custom shaders. Define uniforms for the tileset texture (`map`) and attributes for per-instance data (`instanceMatrix`, `instanceUvOffset`, `instanceUvSize`).
- [ ]  Update `loadTiledMap`: Integrate the `InstancedMesh` objects returned by the refactored creation function into the main `mapGroup`. Ensure correct positioning and layering (adjusting Z-offset if needed).
- [ ]  <other specific steps...>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/superbuilders) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
