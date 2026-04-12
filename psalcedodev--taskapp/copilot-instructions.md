## taskapp

> This rule explains the filtering approach used in the generic `VirtualizedResizableTable` component and the recommended pattern for handling complex data structures.

# Table Data Filtering Strategy

This rule explains the filtering approach used in the generic `VirtualizedResizableTable` component and the recommended pattern for handling complex data structures.

## Filtering Logic

-   The filtering logic resides within the `useMemo` hook that calculates `filteredData` in `[resources/js/components/domain_driven/table/table_data.tsx](mdc:resources/js/components/domain_driven/table/table_data.tsx)`.
-   It performs a **generic, client-side filter** based on the `debouncedFilter` state.
-   The core logic iterates through the defined `columns` for each `item` in the `tableData`.
-   For each column specified by `col.accessorKey`, it retrieves the corresponding value from the `item`.
-   It converts this value to a string using `value?.toString()`, converts it to lowercase, and checks if it `includes()` the lowercase `filterText`.
-   An item is kept if **any** of its column values (as defined in the `columns` prop) match the filter text according to this logic.

## Handling Complex/Nested Data (e.g., Arrays of Objects)

-   The generic `toString()` approach **does not work well** for searching within nested structures like arrays of objects (e.g., searching for a child's name within an `assigned_children` array).
-   **Avoid** adding specific checks for nested data structures directly within the generic `VirtualizedResizableTable` component, as this breaks its reusability.
-   **Recommended Approach:** Pre-process the data before it reaches the table, typically on the **backend**.
    -   Modify the relevant API Resource (e.g., `[app/Http/Resources/TaskDetailResource.php](mdc:app/Http/Resources/TaskDetailResource.php)` or a dedicated list resource) to include a new, flattened attribute containing the searchable string representation.
    -   For example, create an `assigned_children_names` attribute by joining the names from the `children` relationship: `$this->children->pluck('name')->join(', ')`.
    -   Update the `ColumnDef` passed to `VirtualizedResizableTable` in the consuming component (e.g., `tasks_manager.tsx`) to use the new string attribute (`assigned_children_names`) as the `accessorKey` for that column.

This strategy keeps the frontend table component simple and generic, relying on the backend to provide data in a format suitable for straightforward display and filtering.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/psalcedodev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/psalcedodev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
