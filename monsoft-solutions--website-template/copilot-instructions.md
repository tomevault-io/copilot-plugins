## website-template

> Contains the guidelines for the Operations with Github Projects

# GitHub Project Operations Guidelines

- THe project to manage has id: `PVT_kwDOCvCFwM4A21rI`

## Project Creation
- Use `create_project` to create new GitHub project boards
- Required parameters: `owner` (organization/username), `name` (project title)
- Optional parameter: `body` for project description
- Returns project ID, title, URL, and creation timestamp

## Project Fields and Columns
- Use `get_project_fields` to retrieve all fields with their options (parameters: `projectId`)
- Use `get_project_columns` to get status columns (parameters: `projectId`)
- Status field ID and column IDs are required for item placement

## Item Management
- Use `add_project_item` to add issues/PRs to projects (parameters: `projectId`, `contentId`)
- Use `add_project_item_with_column` to add items directly to specific columns (parameters: `projectId`, `contentId`, `fieldId`, `columnId`)
- Use `update_project_item` to move items between columns (parameters: `projectId`, `itemId`, `fieldId`, `columnId`)
- Use `list_project_items` to view all items in a project (parameters: `projectId`, optional: `first` for limit)

## Parameter Types
- `projectId`: GraphQL node ID for the project (string)
- `contentId`: GraphQL node ID for the issue or PR (string)
- `itemId`: GraphQL node ID for the project item (string)
- `fieldId`: GraphQL ID of the field to update (string)
- `columnId`: GraphQL ID of the column/status option (string)

## Error Handling
- All operations handle standard GitHub errors (authentication, not found, validation)
- Proper error responses include type and descriptive message
- Always check `success` field in response to verify operation completed

## Response Format
- All successful responses include a `success: true` property
- Project data includes ID, title, URL, and other relevant metadata
- Item data includes ID and content data where applicable

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Monsoft-Solutions) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
