## construction-dashboard-frontend

> 專案內 RBAC—側欄、路由、按鈕如何對齊 permission 模組


# 專案模組權限（前端）

1. 完整 checklist 與 `action` 約定見後端 repo **`docs/project-module-permissions.md`**（與本專案對照實作）。
2. 新專案內頁面：`src/constants/permission-modules.ts` 必須更新三處且與後端 **`PERMISSION_MODULES` 順序一致**：陣列鍵、`NAV_PATH_PERMISSION_MODULE`、`PERMISSION_MODULE_LABELS`。後端另需 **migration 補列** 租戶範本與專案成員權限表，否則平台／租戶**模組開通**、後台**成員矩陣**、專案**成員矩陣**雖可能顯示新列，既有使用者不會自動繼承旗標。
3. 側欄／Layer2／Layer3：`navigation.ts` 的 `pathSuffix` 必須能對應到 `NAV_PATH_PERMISSION_MODULE`（`AppSidebar` 以 `canReadPath` 過濾）。
4. 頁面與操作元件：使用 **`useProjectPermission(projectId)`** 或 **`useProjectModuleActions(projectId, moduleId)`** → `can(moduleId, 'read'|'create'|'update'|'delete')`；路由守衛已用 `read` 擋頁，按鈕仍需各自檢查 **create/update/delete**。
5. **`platform_admin`** 在 `useProjectPermission` 內視為全開；**`tenant_admin`** 與 `project_user` 皆依 `projectPermissions` store（對齊後端 `my-permissions`，含租戶模組開通遮罩）。側欄僅 `platform_admin` 略過 `ensureLoaded`／全顯。

勿以 `canAccessAdmin` 取代專案模組權限（除非該 UI 僅限租戶後台／與產品一致）。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/henrya860919) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
