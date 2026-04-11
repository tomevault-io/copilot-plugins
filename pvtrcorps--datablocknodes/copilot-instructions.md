## datablocknodes

> **Architecture Version: 18.0 ("The Audit")**

# GEMINI.md: Datablock Nodes Technical Reference

**Architecture Version: 18.0 ("The Audit")**

This document provides a deep dive into the internal architecture of the Datablock Nodes addon. It is intended for developers.

---

### 1. Core Architecture & Execution Flow

The addon's execution is triggered when the Blender dependency graph is updated, typically after a node or property change.

#### 1.1. The Execution Cycle

The execution flow is orchestrated by a few key components:

1.  **`entry_point.depsgraph_update_handler`**: The entry point registered with Blender's `depsgraph_update_post` handler. It invokes the Orchestrator.
2.  **`engine.orchestrator.Orchestrator`**: The master controller.
    - It identifies the active node branch to be executed.
    - It calls `_evaluate_node` recursively to execute the node tree, which returns the final root `DatablockProxy`.
    - It calls the `Planner` to create an execution plan from the proxy graph.
    - It calls `_synchronize_blender_state`, which prunes old datablocks and then delegates to the `Materializer`.
3.  **`engine.planner.Planner`**:
    - Takes the root proxy and flattens the entire proxy hierarchy into a single list.
    - Builds a dependency graph based on parent-child relationships and all relationships defined in `_fn_relationships`.
    - Performs a topological sort (Kahn's algorithm) to produce a flat list of proxies in the correct dependency order (e.g., materials before objects, parents before children).
4.  **`engine.materializer.materialize_plan`**:
    - This function translates the plan from the `Planner` into actual Blender data. It operates in several distinct phases (see section 3).

#### 1.2. The `DatablockProxy`

The central data structure. It is a Python class (`proxy_types.py`) that acts as a blueprint for a Blender datablock.

-   **`path` (str):** A canonical, hierarchical path (e.g., `/Scene/objects/Character`).
-   **`fn_uuid` (UUID):** A persistent, deterministic unique identifier. It's the key for the override system.
-   **`properties` (dict):** A dictionary containing the proxy's state.
    -   **`_fn_source` (dict):** Defines the proxy's origin.
        -   `{'type': 'procedural'}`: Created by a generator node (e.g., `Create Object`).
        -   `{'type': 'file', ...}`: Imported from a `.blend` file via the `Import` node.
    -   **`_fn_globals` (dict):** On root proxies only. Holds scene-wide properties (e.g., render settings).
    -   **`_fn_relationships` (dict):** Stores all connections to other proxies. Keys include `data`, `collection_links`, `active_material`, `camera`, `world`. The values are typically paths to the target proxies.
-   **`parent`, `children`:** Defines the direct transformation hierarchy.

---

### 2. The Override System

This system allows user changes to be preserved non-destructively.

1.  **State Snapshotting:** During materialization, immediately after all procedural properties and relationships for a datablock have been set, the `Materializer` calls `utils.capture_datablock_state`. This function captures a JSON-serializable snapshot of the datablock's state, which is then stored in `tree.fn_initial_state_map` associated with the datablock's `fn_uuid`.
2.  **Override Detection (`override_handler.py`):**
    - The `depsgraph_update_post_handler` listens for any changes in Blender.
    - If a managed datablock is modified, the handler captures its new, current state.
    - It calculates the difference (`_calculate_overrides`) between the `current_state` and the `initial_state` snapshot.
    - The resulting diff (a dictionary of changed properties) is saved into `tree.fn_override_map`.
3.  **Override Application:** On the next execution, after the `Materializer` applies the base state and relationships, it reads the override data from `tree.fn_override_map` and applies it last, ensuring user changes have the final say.
4.  **Override Management (`operators.py`):**
    - `FN_OT_clear_all_overrides` and `FN_OT_clear_active_overrides` are operators exposed in a UI panel in the Node Editor's sidebar. They allow users to view and delete stored overrides, reverting datablocks to their node-defined state.

---

### 3. The Materializer In-Depth

The `materialize_plan` function in `engine/materializer.py` is the heart of the scene construction.

-   **Phase 1: Analysis & Append:**
    - It first iterates through the plan to find all `Reference` proxies (from `.blend` files).
    - It groups them by filepath and datablock type to perform efficient, batched `bpy.data.libraries.load()` operations. This avoids loading the same library multiple times.
-   **Phase 2: Procedural Creation:**
    - It iterates through the `Procedural` proxies and creates them using the functions stored in `properties._datablock_creation_map`.
    - It handles special cases like creating objects with linked data (meshes, lights).
    - It registers the newly created Blender datablock with the `UUIDManager`, linking it to the proxy's `fn_uuid`.
-   **Phase 3: State Application:**
    - This is a loop through the entire plan, now that all datablocks exist in Blender. For each datablock, the following strict order is applied:
        1.  **Apply Base Properties:** Set all simple properties from `proxy.properties`.
        2.  **Establish Relationships:** Set all relationships from `_fn_relationships` (parenting, collection links, material links, etc.). This happens *before* snapshotting.
        3.  **Snapshot Initial State:** Capture the final procedural state.
        4.  **Apply User Overrides:** Apply any stored overrides on top.

---

### 4. The Import Process & Blend Parser

Importing from a `.blend` file is a two-stage process to avoid modifying the current Blender session directly.

1.  **`nodes.import_node.py`:**
    - When the node's "Reload" button is pressed, it calls `launch_blend_parser`.
    - This function executes `engine/blend_parser.py` as a separate, headless Blender subprocess.
2.  **`engine.blend_parser.py` (The Subprocess):**
    - It opens the target `.blend` file in the background.
    - It uses `scene_assembler.SceneAssembler` to iterate through the datablocks (`objects`, `collections`, etc.) in the file.
    - For each datablock, it creates a corresponding `DatablockProxy`.
    - **Crucially**, it uses `utils.capture_datablock_state` to read the *actual* properties of the datablocks from the file.
    - It assigns deterministic UUIDs to each proxy based on a hash of the file path and the datablock's name and type.
    - It serializes the entire generated proxy tree into a `.fn_cache.json` file in the user's temp directory.
3.  **Back in the Main Process:**
    - The `import_node` loads the proxy tree from the `.fn_cache.json` file and presents it as its output.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pvtrcorps)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pvtrcorps)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
