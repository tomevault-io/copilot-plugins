## smartsheet-deprecation-scanner

> Use when explicitly invoked via /smartsheet-deprecation-scanner to scan a codebase for usage of deprecated Smartsheet API endpoints and parameters, including both SDK-based and direct HTTP usage.


# Smartsheet Deprecation Scanner

## Overview

Scan a codebase for usage of deprecated Smartsheet API endpoints, query parameters, request/response fields, and seat type values. Output a structured markdown report of all findings with file paths, line numbers, code snippets, and migration guidance.

## Trigger

Only activate when explicitly invoked as `/smartsheet-deprecation-scanner`. Do not activate automatically.

---

## Step 0 — Check for Changelog Updates

**Before scanning, always do this first.**

Read the `last_changelog_check` date from this file's YAML frontmatter. If today's date is more than 7 days after that date:

1. Fetch `https://developers.smartsheet.com/api/smartsheet/changelog` using WebFetch with the prompt: "List every deprecation entry. For each one extract: (1) the deprecated item (endpoint, parameter, field, or value), (2) the deprecation date, (3) the sunset date, and (4) the replacement or recommended action."
2. Compare the results against the deprecation categories already documented in this skill (Categories 1–8 and any existing Pending Review entries).
3. For any item in the changelog that is **not already covered** in this skill file:
   - Append it to the `## Pending Review` section at the bottom of this file using the format described there.
   - Update the `last_changelog_check` date in the YAML frontmatter to today's date.
   - Use the Edit tool to write both changes to this skill file.
   - **Alert the user before proceeding with the scan:**

     > ⚠️ **Smartsheet changelog update detected.** New deprecation entries have been added to the `## Pending Review` section of the skill file (`~/.claude/skills/smartsheet-deprecation-scanner/SKILL.md`). Please review and promote or discard them before relying on this scan as complete.
     >
     > New entries found:
     > - [list each new item with its deprecation date and sunset date]
     >
     > Continuing scan with currently confirmed deprecations only…

4. If no new items are found, still update `last_changelog_check` to today's date and continue.
5. If the changelog fetch fails, note it at the top of the report and continue with the existing deprecation data:

   > ⚠️ **Changelog check skipped** — could not reach developers.smartsheet.com. Scan uses deprecation data last verified on {last_changelog_check}.

If `last_changelog_check` is within 7 days of today, skip the fetch and proceed directly to Step 1.

---

## Pending Review

<!-- New deprecations found by the auto-update check are appended here.
     Each entry should be reviewed and either promoted into the appropriate
     Category section above (with full grep patterns and migration guidance)
     or removed if not applicable.
     Format per entry:
     ### [PENDING] {short description}
     - **Deprecated:** {date}
     - **Sunset:** {date or TBD}
     - **Item:** {endpoint / parameter / field / value}
     - **Replacement:** {replacement or "none"}
     - **Source:** changelog entry dated {date}
     - **Status:** Awaiting review
-->

---

## Scan Targets

### Languages & SDKs to cover

- **Java** — `smartsheet-java-sdk`
- **C#** — `smartsheet-csharp-sdk`
- **Python** — `smartsheet-python-sdk`
- **Node.js** — `smartsheet-javascript-sdk`
- **Ruby** — `smartsheet-ruby-sdk`
- **Go, PHP, Scala, Swift, Kotlin, and any other language** — no official Smartsheet SDK exists for these; scan for direct HTTP usage only. All URL path patterns apply equally regardless of language.
- **Direct HTTP** — `axios`, `fetch`, `HttpClient`, `requests`, `RestTemplate`, `HTTParty`, `Net::HTTP`, `URLConnection`, `WebClient`, `HttpURLConnection`, `curl`, `http.Get`, `http.Post`, `GuzzleHttp`, `file_get_contents`, etc.

---

## Deprecation Reference

### Category 1 — Deprecated Endpoints (Sharing)

Asset-specific sharing endpoints are deprecated. **Deprecated:** 2025-08-04. **Sunset:** 2026-06-03.

**Replacement:** Use unified `/shares` endpoints with `assetType` and `assetId` query parameters, using `PATCH` (not `PUT`) for updates.

> **HTTP method change alert:** `PUT /.../shares/{shareId}` → `PATCH /shares/{shareId}`. This is both a URL change AND a method change. Flag it explicitly in the report — some HTTP clients will silently send the wrong method. Mark these findings with **⚠ HTTP METHOD CHANGE** in the occurrence table.

| Deprecated | Replacement |
|---|---|
| `GET /sheets/{id}/shares` | `GET /shares?assetType=sheet&assetId={id}` |
| `GET /reports/{id}/shares` | `GET /shares?assetType=report&assetId={id}` |
| `GET /sights/{id}/shares` | `GET /shares?assetType=sight&assetId={id}` |
| `GET /workspaces/{id}/shares` | `GET /shares?assetType=workspace&assetId={id}` |
| `GET /sheets/{id}/shares/{shareId}` | `GET /shares/{shareId}?assetType=sheet&assetId={id}` |
| `GET /reports/{id}/shares/{shareId}` | `GET /shares/{shareId}?assetType=report&assetId={id}` |
| `GET /sights/{id}/shares/{shareId}` | `GET /shares/{shareId}?assetType=sight&assetId={id}` |
| `GET /workspaces/{id}/shares/{shareId}` | `GET /shares/{shareId}?assetType=workspace&assetId={id}` |
| `POST /sheets/{id}/shares` | `POST /shares?assetType=sheet&assetId={id}` |
| `POST /reports/{id}/shares` | `POST /shares?assetType=report&assetId={id}` |
| `POST /sights/{id}/shares` | `POST /shares?assetType=sight&assetId={id}` |
| `POST /workspaces/{id}/shares` | `POST /shares?assetType=workspace&assetId={id}` |
| `PUT /sheets/{id}/shares/{shareId}` | `PATCH /shares/{shareId}?assetType=sheet&assetId={id}` |
| `PUT /reports/{id}/shares/{shareId}` | `PATCH /shares/{shareId}?assetType=report&assetId={id}` |
| `PUT /sights/{id}/shares/{shareId}` | `PATCH /shares/{shareId}?assetType=sight&assetId={id}` |
| `PUT /workspaces/{id}/shares/{shareId}` | `PATCH /shares/{shareId}?assetType=workspace&assetId={id}` |
| `DELETE /sheets/{id}/shares/{shareId}` | `DELETE /shares/{shareId}?assetType=sheet&assetId={id}` |
| `DELETE /reports/{id}/shares/{shareId}` | `DELETE /shares/{shareId}?assetType=report&assetId={id}` |
| `DELETE /sights/{id}/shares/{shareId}` | `DELETE /shares/{shareId}?assetType=sight&assetId={id}` |
| `DELETE /workspaces/{id}/shares/{shareId}` | `DELETE /shares/{shareId}?assetType=workspace&assetId={id}` |

**SDK method patterns to flag:**

```
# Python
smartsheet.Sheets.list_shares(...)
smartsheet.Reports.list_shares(...)
smartsheet.Sights.list_shares(...)
smartsheet.Workspaces.list_shares(...)
smartsheet.Sheets.get_share(...)
smartsheet.Sheets.share_sheet(...)
smartsheet.Sheets.update_share(...)
smartsheet.Sheets.delete_share(...)
# same patterns for Reports, Sights, Workspaces

# Node.js
client.sheets.listShares(...)
client.reports.listShares(...)
client.sights.listShares(...)
client.workspaces.listShares(...)
client.sheets.getShare(...)
client.sheets.share(...)
client.sheets.updateShare(...)
client.sheets.deleteShare(...)
# same for reports, sights, workspaces

# Java
smartsheet.sheetResources().shareResources().listShares(...)
smartsheet.reportResources().shareResources().listShares(...)
smartsheet.sightResources().shareResources().listShares(...)
smartsheet.workspaceResources().shareResources().listShares(...)

# C#
smartsheet.SheetResources.ShareResources.ListShares(...)
smartsheet.ReportResources.ShareResources.ListShares(...)
smartsheet.SightResources.ShareResources.ListShares(...)
smartsheet.WorkspaceResources.ShareResources.ListShares(...)

# Ruby (SDK)
client.sheets.shares.list(...)
client.reports.shares.list(...)
client.sights.shares.list(...)
client.workspaces.shares.list(...)

# Ruby (direct HTTP — HTTParty, Net::HTTP, Faraday, etc.)
# Flag any string containing /sheets/.../shares, /reports/.../shares,
# /sights/.../shares, or /workspaces/.../shares
```

---

### Category 2 — Deprecated Endpoints (Home / "Sheets" Folder)

Deprecated 2025-03-25. The "Sheets" folder concept is being replaced by workspaces. **Sunset: TBD** (under re-evaluation).

| Deprecated | Replacement |
|---|---|
| `GET /home` | Use workspaces |
| `GET /home/folders` | Use workspaces |
| `GET /folders/personal` | Use workspaces |
| `POST /home/folders` | Create folders in workspaces |
| `POST /sheets` (top-level) | Create sheets in a workspace |
| `POST /sheets/import` (top-level) | Import into a workspace folder |

**Destination type `home`** — Any code passing `"home"` as a destination type is deprecated.

**Distinguishing top-level `POST /sheets` from workspace/folder-scoped creation:**
For SDK calls like `sheets.create(...)` or `Sheets.create_sheet(...)`, check the surrounding argument structure. If there is no `workspaceId`, `folderId`, or `workspace_id`/`folder_id` parameter in the same call or immediately preceding context, treat it as top-level and flag it. For direct HTTP calls, flag any `POST` to the bare path `/sheets` without a preceding `/workspaces/{id}/sheets` or `/folders/{id}/sheets` path segment. When ambiguous, flag with a note: "Verify no workspace/folder destination is provided."

**SDK method patterns to flag:**

```
# Python
smartsheet.Home.list_contents(...)
smartsheet.Home.list_folders(...)
smartsheet.Home.create_folder(...)
smartsheet.Folders.get_folder("personal")
smartsheet.Sheets.create_sheet(...)  # only if no workspace/folder context
smartsheet.Sheets.import_sheet(...)  # only if top-level

# Node.js
client.home.listFolders(...)
client.home.listContents(...)
client.home.createFolder(...)
client.folders.getFolder("personal")
client.sheets.importSheet(...)  # top-level

# Java
smartsheet.homeResources().listFolders(...)
smartsheet.homeResources().folderResources().createFolder(...)

# C#
smartsheet.HomeResources.ListFolders(...)
smartsheet.HomeResources.FolderResources.CreateFolder(...)

# Ruby
client.home.folders.list(...)
client.home.folders.create(...)
```

---

### Category 3 — Deprecated Endpoints (Folder & Workspace Navigation)

**Deprecated:** 2025-02-01 (folder/workspace GET endpoints) and 2025-08-04 (template endpoints). **Sunset:** 2026-06-03. Replaced by unified children endpoints.

> **Overlap with Category 4:** Calling a deprecated navigation endpoint (e.g. `list_folders`) AND passing `include_all=True` to it are two separate findings — report them under both Category 3 and Category 4 respectively. Do not merge them into a single finding.

| Deprecated | Replacement |
|---|---|
| `GET /folders/{id}` | `GET /folders/{id}/metadata` + `GET /folders/{id}/children` |
| `GET /folders/{id}/folders` | `GET /folders/{id}/children?childrenResourceTypes=folders` |
| `GET /workspaces/{id}` | `GET /workspaces/{id}/metadata` + `GET /workspaces/{id}/children` |
| `GET /workspaces/{id}/folders` | `GET /workspaces/{id}/children?childrenResourceTypes=folders` |
| `GET /templates` | `GET /workspaces/{id}/children?childrenResourceTypes=sheets,templates` |
| `GET /templates/public` | No direct replacement |

**SDK method patterns to flag:**

```
# Python
smartsheet.Folders.get_folder(folder_id)
smartsheet.Folders.list_folders(folder_id)
smartsheet.Workspaces.get_workspace(workspace_id)
smartsheet.Workspaces.list_folders(workspace_id)
smartsheet.Templates.list_public_templates()
smartsheet.Templates.list_user_created_templates()

# Node.js
client.folders.getFolder(folderId)
client.folders.listFolders(folderId)
client.workspaces.getWorkspace(workspaceId)
client.workspaces.listFolders(workspaceId)
client.templates.listPublicTemplates()
client.templates.listUserCreatedTemplates()

# Java
smartsheet.folderResources().getFolder(folderId, ...)
smartsheet.folderResources().listFolders(folderId, ...)
smartsheet.workspaceResources().getWorkspace(workspaceId, ...)
smartsheet.workspaceResources().listFolders(workspaceId, ...)
smartsheet.templateResources().listPublicTemplates(...)
smartsheet.templateResources().listUserCreatedTemplates(...)

# C#
smartsheet.FolderResources.GetFolder(folderId, ...)
smartsheet.FolderResources.ListFolders(folderId, ...)
smartsheet.WorkspaceResources.GetWorkspace(workspaceId, ...)
smartsheet.WorkspaceResources.ListFolders(workspaceId, ...)
smartsheet.TemplateResources.ListPublicTemplates(...)
smartsheet.TemplateResources.ListUserCreatedTemplates(...)

# Ruby
client.folders.get(folder_id)
client.folders.list(folder_id)
client.workspaces.get(workspace_id)
client.workspaces.list_folders(workspace_id)
client.templates.list_public(...)
client.templates.list_user_created(...)
```

---

### Category 4 — Deprecated Query Parameters (Offset-Based Pagination)

Offset-based pagination is deprecated on several endpoints in favor of token-based pagination using `paginationType=token`, `lastKey`, and `maxItems`.

| Endpoint | Deprecated Params | Deprecated | Sunset |
|---|---|---|---|
| `GET /sights` | `includeAll`, `modifiedSince`, `page`, `pageSize` | 2025-12-03 | 2026-06-03 |
| `GET /workspaces` | `includeAll`, `page`, `pageSize` | 2025-08-04 | 2026-06-03 |
| `GET /folders/{id}/folders` | `includeAll`, `page` | 2025-02-01 | 2026-06-03 |
| `GET /workspaces/{id}/folders` | `includeAll`, `page` | 2025-02-01 | 2026-06-03 |
| `GET /webhooks` | `includeAll` | 2025-08-04 | 2026-06-03 |

**Flag any code passing these parameters to these endpoints.** SDK-level patterns:

```
# Python — flag includeAll=True or page= on these resources
smartsheet.Sights.list_sights(include_all=True)
smartsheet.Workspaces.list_workspaces(include_all=True)
smartsheet.Webhooks.list_webhooks(include_all=True)

# Node.js
client.sights.listSights({ includeAll: true })
client.workspaces.listWorkspaces({ includeAll: true })
client.webhooks.listWebhooks({ includeAll: true })

# Java — PaginationParameters with includeAll=true on these resources
# C# — PaginationParameters with IncludeAll=true on these resources
```

---

### Category 5 — Deprecated Query Parameters (include= values on Folder/Workspace)

**Deprecated:** 2025-02-03. **Sunset:** 2026-06-03.

| Endpoint | Deprecated `include` value | Replacement |
|---|---|---|
| `GET /folders/{id}` | `distributionLink` | Remove |
| `GET /folders/{id}` | `sheetVersion` | Call `GET /sheets/{sheetId}/version` |
| `GET /folders/{id}` | `permalink` | Remove |
| `GET /workspaces/{id}` | `distributionLink` | Remove |
| `GET /workspaces/{id}` | `sheetVersion` | Call `GET /sheets/{sheetId}/version` |
| `GET /folders/personal` | `permalink` | Remove |

> **Note:** `loadAll` is a standalone query parameter (`?loadAll=true`), not an `include=` value. It is deprecated on `GET /workspaces/{id}` and is covered by Category 3 (the deprecated workspace GET endpoint). Do not report it as a Category 5 finding.

---

### Category 6 — Deprecated Query Parameters (Copy Endpoints)

**Deprecated:** 2025-12-09. **Sunset:** 2026-03-09.

| Endpoint | Deprecated Params | Replacement |
|---|---|---|
| `POST /folders/{id}/folders` (copy via body) | `include`, `exclude`, `skipRemap` | Use `POST /folders/{id}/copy` |
| `POST /workspaces` (copy via body) | `include`, `exclude` | Use `POST /workspaces/{id}/copy` |

---

### Category 7 — Deprecated Seat Type: `VIEWER`

**Deprecated:** 2026-04-30. **Sunset:** 2026-10-30.

Replace all references to `VIEWER` seat type with `CONTRIBUTOR`.

Affects:
- `GET /2.0/users` — response `seatType` field
- `GET /2.0/users/{id}/plans` — response
- `POST /2.0/users/{id}/plans/{planId}/downgrade` — request body
- `GET /users?seatType=VIEWER` — query filter
- Webhook event payloads containing seat type

**Flag any string literal `"VIEWER"` or `VIEWER` enum in seat-type context.**

---

### Category 8 — Deprecated Response Fields

These fields are being removed from responses. Flag any code that reads them.

| Field | Endpoint(s) | Deprecated | Sunset | Notes |
|---|---|---|---|---|
| `sheetCount` | `GET /users`, `GET /users/me`, `GET /users/{id}` | 2025-04-22 | 2025-05-12 | **🔴 Past sunset** — returns `-1` |
| `totalCount` / `totalPages` | `GET /webhooks` responses | 2025-08-04 | 2026-06-03 | Returns `-1` until sunset |
| `pageNumber` / `pageSize` / `totalCount` / `totalPages` | `GET /sights`, `GET /workspaces`, folder/workspace listing responses | 2025-12-03 / 2025-08-04 | 2026-06-03 | Offset pagination response fields |
| `favorite` | `GET /sights/{id}` response | — | Removed 2025-03-31 | **🔴 Already removed** |
| `workspace.accessLevel` | `GET /sights/{id}` response | — | Removed 2025-03-31 | **🔴 Already removed** |
| `workspace.permalink` | `GET /sights/{id}` response | — | Removed 2025-03-31 | **🔴 Already removed** |
| `createdAt` / `updatedAt` | Old asset-specific sharing responses | 2025-08-04 | 2026-06-03 | Removed at sunset |

**Flag any code accessing `.sheetCount`, `.totalCount` from webhooks, `.totalPages` from webhooks, `.favorite` from sight objects, `.accessLevel` or `.permalink` from sight workspace objects.**

> **Permalink note:** `.permalink` appears in multiple contexts — it is only deprecated when accessed on a `workspace` object inside a sight response, or as an `include=permalink` value on folder/workspace endpoints. Do not flag `.permalink` on sheet or row objects, which are not deprecated.

---

## How to Scan

### Step 1 — Discover all source file extensions in the repo

Do not assume which languages are present. Run this command to enumerate every source file extension actually in use, excluding dependency directories and build output:

```bash
find . -type f -name "*.*" \
  ! -path "*/node_modules/*" ! -path "*/vendor/*" ! -path "*/.git/*" \
  ! -path "*/bin/*" ! -path "*/obj/*" ! -path "*/build/*" ! -path "*/dist/*" \
  ! -path "*/.gradle/*" ! -path "*/target/*" ! -path "*/__pycache__/*" \
  | sed 's/.*\.//' | sort | uniq -c | sort -rn | head -40
```

Review the output. Exclude extensions that are clearly not source code (e.g. `json`, `md`, `xml`, `yaml`, `yml`, `lock`, `sum`, `mod`, `png`, `svg`, `css`, `html` unless HTML contains inline scripts, `txt`). Keep every extension that could contain code making HTTP calls.

Build two values from this:

**`$ALL_EXT`** — `--include` flags for every source code extension found. This is used for the universal HTTP/URL patterns that apply to any language. Example: `--include="*.cs" --include="*.js" --include="*.ts"`

**`$SDK_EXT`** — the subset of `$ALL_EXT` that corresponds to a language with an official Smartsheet SDK (Java, C#, Python, Node.js/TypeScript, Ruby). SDK method name patterns are only run against these extensions. If none of the detected extensions have an official SDK, skip the SDK-specific grep commands — the HTTP patterns still cover everything.

The official Smartsheet SDKs are:
| Language | Extensions | SDK |
|---|---|---|
| Java | `.java`, `.groovy`, `.kt`, `.scala` | `smartsheet-java-sdk` |
| C# | `.cs` | `smartsheet-csharp-sdk` |
| Python | `.py` | `smartsheet-python-sdk` |
| Node.js | `.js`, `.ts`, `.mjs`, `.cjs` | `smartsheet-javascript-sdk` |
| Ruby | `.rb` | `smartsheet-ruby-sdk` |

Any other extension (`.go`, `.php`, `.swift`, `.ex`, `.exs`, `.rs`, `.cpp`, `.c`, `.elm`, `.dart`, etc.) has no official SDK — include it in `$ALL_EXT` for HTTP patterns but not in `$SDK_EXT`.

### Step 2 — Run grep patterns

**Two rules that are never broken:**
1. **Every category is always searched** — never skip a category because of the detected languages.
2. **HTTP/URL patterns run against `$ALL_EXT`** — these are language-agnostic and catch direct HTTP calls in any language.
3. **SDK method patterns run against `$SDK_EXT` only** — these are SDK-specific and would produce noise against languages without those SDKs.

Send all commands in a single Bash tool call so they execute in parallel. All commands use `-i` (case-insensitive).

Replace `--include="*.cs"` with your actual `$ALL_EXT` or `$SDK_EXT` value as indicated by the comment on each command.

```bash
# ── Cat 1: Sharing endpoints ──────────────────────────────────────────────────

# [SDK_EXT] SDK methods
grep -rni "shareresources\|list_shares\b\|listshares\b\|share_sheet\b\|sharesheet\b\|update_share\b\|updateshare\b\|delete_share\b\|deleteshare\b\|get_share\b\|getshare\b\|shares\.\(list\|get\|update\|delete\)\b" --include="*.cs" .

# [ALL_EXT] Direct HTTP — path contains "shares/"
grep -rni "shares/" --include="*.cs" .
# [ALL_EXT] Direct HTTP — terminal "/shares" with no trailing slash
grep -rni '/shares\b' --include="*.cs" .
# [ALL_EXT] Direct HTTP — standalone segment in URL builders, path arrays, route annotations
grep -rni "['\"]shares['\"]" --include="*.cs" .

# [ALL_EXT] ⚠ HTTP METHOD CHANGE — PUT used with a shares path must migrate to PATCH.
# Flag all hits with ⚠ HTTP METHOD CHANGE in the report.
grep -rni "put.*shares\|shares.*put\|CURLOPT_CUSTOMREQUEST.*\"PUT\"" --include="*.cs" .

# ── Cat 2: Home / "Sheets" folder ─────────────────────────────────────────────

# [SDK_EXT] SDK methods — home resource access
grep -rni "homeresources\|gethome\b\|get_home\b\|listcontents\|list_contents" --include="*.cs" .

# [SDK_EXT] SDK methods — create folder (flag if called on home resource; triage context)
grep -rni "createfolder\b\|create_folder\b" --include="*.cs" .

# [SDK_EXT] SDK methods — top-level sheet create (flag if no workspace/folder context; triage carefully)
grep -rni "createsheet\b\|create_sheet\b" --include="*.cs" .

# [ALL_EXT] Direct HTTP — "home/" catches any quote style and any ID variable form
grep -rni "home/" --include="*.cs" .
# [ALL_EXT] Direct HTTP — terminal "/home" with no trailing slash
grep -rni '/home\b' --include="*.cs" .
# [ALL_EXT] Direct HTTP — standalone "home" segment (high false-positive rate; triage for URL context)
grep -rni "['\"]home['\"]" --include="*.cs" .

# [ALL_EXT] Direct HTTP — folders/personal as combined segment
grep -rni "folders/personal" --include="*.cs" .
# [ALL_EXT] Direct HTTP — standalone "personal" segment in URL builders
grep -rni "['\"]personal['\"]" --include="*.cs" .

# [ALL_EXT] Destination type "home"
grep -rni "destinationtype\.home\|destination.*home\|home.*destination" --include="*.cs" .

# [ALL_EXT] Top-level sheet import
grep -rni 'sheets\.import\|importsheet\|import_sheet\|sheets/import' --include="*.cs" .

# ── Cat 3: Folder & workspace navigation ──────────────────────────────────────

# [SDK_EXT] SDK methods
grep -rni '\bgetfolder\b\|get_folder\b\|listfolders\b\|list_folders\b\|\bgetworkspace\b\|get_workspace\b' --include="*.cs" .

# [ALL_EXT] Direct HTTP — folders/
# Triage: DEPRECATED if path ends at ID or continues to /folders;
#         SAFE if path continues to /children or /metadata (replacement endpoints).
grep -rni "folders/" --include="*.cs" .
# [ALL_EXT] Direct HTTP — standalone "folders" segment
grep -rni "['\"]folders['\"]" --include="*.cs" .

# [ALL_EXT] Direct HTTP — workspaces/ (same triage rule)
grep -rni "workspaces/" --include="*.cs" .
# [ALL_EXT] Direct HTTP — standalone "workspaces" segment
grep -rni "['\"]workspaces['\"]" --include="*.cs" .

# [SDK_EXT] SDK methods — templates
grep -rni "templateresources\|listpublictemplates\|list_public_templates\|listusercreatedtemplates\|list_user_created_templates" --include="*.cs" .

# [ALL_EXT] Direct HTTP — templates/
grep -rni "templates/" --include="*.cs" .
# [ALL_EXT] Direct HTTP — terminal "/templates"
grep -rni '/templates\b' --include="*.cs" .
# [ALL_EXT] Direct HTTP — standalone "templates" segment
grep -rni "['\"]templates['\"]" --include="*.cs" .

# ── Cat 4: Deprecated pagination parameters ───────────────────────────────────
# [ALL_EXT] includeAll / include_all
grep -rni 'includeall.*true\|include_all.*true' --include="*.cs" .
# [ALL_EXT] modifiedSince / modified_since
grep -rni 'modifiedsince\|modified_since' --include="*.cs" .
# [ALL_EXT] page as a query param
grep -rni '\bpage\s*[=:]\|"page"\s*[=:]\|'"'"'page'"'"'\s*[=:]' --include="*.cs" .
# [ALL_EXT] pageSize / page_size
grep -rni '\bpagesize\s*[=:]\|\bpage_size\s*[=:]\|"pagesize"\|"page_size"' --include="*.cs" .

# ── Cat 5: Deprecated include= values on folder/workspace ────────────────────
# [ALL_EXT]
grep -rni 'distributionlink\|sheetversion\|loadall' --include="*.cs" .
# [ALL_EXT] permalink as an include= value (triage: flag only if passed to include param on folder/workspace)
grep -rni 'include.*permalink\|permalink.*include' --include="*.cs" .

# ── Cat 6: Deprecated copy parameters ─────────────────────────────────────────
# NOTE: Cat 6 sunset was 2026-03-09 — these are 🔴 BROKEN. Flag all hits as broken.
# [ALL_EXT]
grep -rni 'skipremap\|skip_remap' --include="*.cs" .
# [SDK_EXT] SDK copy methods — triage for include/exclude body params (also broken)
grep -rni 'copyfolder\b\|copy_folder\b\|copyworkspace\b\|copy_workspace\b' --include="*.cs" .

# ── Cat 7: Deprecated VIEWER seat type ───────────────────────────────────────
# [ALL_EXT] String literals in any quote style
grep -rni "['\"]viewer['\"]" --include="*.cs" .
# [ALL_EXT] Unquoted enum/constant: SeatType.VIEWER, SeatType::VIEWER, :VIEWER, = VIEWER
grep -rni '\.viewer\b\|::viewer\b\|:viewer\b\|= viewer\b' --include="*.cs" .
# [ALL_EXT] Seat-type param context
grep -rni 'seattype.*viewer\|seat_type.*viewer\|viewer.*seattype\|viewer.*seat_type' --include="*.cs" .

# ── Cat 8: Deprecated response fields ────────────────────────────────────────
# [ALL_EXT] — (\.|-?>) covers dot accessor (most languages) and arrow accessor (PHP)
grep -rni 'sheetcount\b\|sheet_count\b' --include="*.cs" .
grep -rni '(\.|-?>)(totalcount\|total_count\|totalpages\|total_pages\|pagenumber\|page_number\|pagesize\|page_size)\b' --include="*.cs" .
grep -rni '(\.|-?>)favorite\b\|getfavorite\b\|get_favorite\b' --include="*.cs" .
grep -rni '(\.|-?>)(accesslevel\|access_level)\b\|getaccesslevel\b\|get_access_level\b' --include="*.cs" .
grep -rni '(\.|-?>)permalink\b\|getpermalink\b\|get_permalink\b' --include="*.cs" .
# createdAt / updatedAt — very high false-positive rate; triage: flag only on shares/share response objects
grep -rni '(\.|-?>)(createdat\|created_at\|updatedat\|updated_at)\b' --include="*.cs" .
```

### Step 3 — Triage results

For each grep hit, read the surrounding 3–5 lines of context. Determine:
1. Is this actually a Smartsheet API call or SDK method (not a comment or unrelated variable)?
2. Which deprecation category does it fall under?
3. Is there already a migration in progress?

Discard false positives (comments describing old behavior, unrelated packages sharing a method name, etc.).

### Step 3b — Classify by sunset status

**Always use the sunset date, not the deprecation date, to assign severity.** These are different: the deprecation date is when the change was announced; the sunset date is when the endpoint/field actually stops working. A finding deprecated in August 2025 with a June 2026 sunset is still functional today.

Compare today's date to the **sunset date** from the deprecation reference tables above:

| Condition | Label | Meaning |
|---|---|---|
| Sunset date has passed (or field explicitly noted as "already removed") | **🔴 BROKEN** | Already removed or permanently returning `-1`/null. Fix immediately. |
| Sunset within 90 days | **🟠 URGENT** | Sunset imminent. Prioritize migration. |
| Sunset TBD or more than 90 days away | **🟡 DEPRECATED** | Deprecated but still functional. Plan migration. |

Use these labels in each finding's header, e.g.: `**🟡 DEPRECATED** — sunset 2026-06-03`

### Step 4 — Fetch official migration guidance

**Skip this step entirely if no findings were found after triage (Steps 3–3b).** Only fetch if there is at least one confirmed deprecated usage to migrate.

If findings exist, fetch the official Smartsheet migration guide index:

```
https://developers.smartsheet.com/api/smartsheet/guides/updating-code
```

Use WebFetch with the prompt: "List every linked sub-page URL on this page exactly as written. Include the link text and the full URL for each link."

**The index page contains no inline code examples — it only links to sub-guides.** From the URLs returned by that fetch, identify which sub-pages are relevant to the confirmed findings and fetch each one individually. Use WebFetch on each relevant sub-page URL with the prompt: "Extract all code examples. For each example note the language (Java, C#, Python, Node.js) and the deprecated construct it replaces."

> **⚠ NEVER construct or guess sub-page URLs.** Only fetch URLs that were explicitly returned from the index page. Guessing paths (e.g. appending `/unified-sharing-endpoints`) will produce 404s and is indistinguishable from hallucination.

Use the examples from the sub-pages — not hand-crafted approximations — as the **Migration** section content for each finding in the report. If any fetch fails, note it in the report and omit the code examples for that finding rather than substituting guessed ones:

> ⚠️ **Migration guide unavailable** — could not reach developers.smartsheet.com. See `https://developers.smartsheet.com/api/smartsheet/guides/updating-code` for official migration examples.

### Step 5 — Write the report file

Write the report to a file named `smartsheet-deprecation-report.md` in the root of the scanned directory. Use the Write tool, not terminal output. After writing, confirm the file path to the user.

If no deprecated usage is found, write a file containing only:

```
No Smartsheet API deprecation usage found.
```

---

## Output Format

Write the following structure to `smartsheet-deprecation-report.md`:

```markdown
# Smartsheet API Deprecation Report

_Generated: {date}_

## Summary

| Category | Findings | Worst Status |
|---|---|---|
| Deprecated sharing endpoints | N | 🟡/🟠/🔴 |
| Deprecated home/folder endpoints | N | 🟡/🟠/🔴 |
| Deprecated folder/workspace navigation | N | 🟡/🟠/🔴 |
| Deprecated pagination parameters | N | 🟡/🟠/🔴 |
| Deprecated include= values | N | 🟡/🟠/🔴 |
| Deprecated copy parameters | N | 🟡/🟠/🔴 |
| Deprecated VIEWER seat type | N | 🟡/🟠/🔴 |
| Deprecated response field access | N | 🟡/🟠/🔴 |
| **Total** | **N** | |

> **SDK upgrade may be required.** Replacement methods for many of these findings (e.g. unified sharing, `getWorkspaceMetadata`, `getWorkspaceChildren`) are only available in recent SDK versions. Before migrating, check that your installed SDK version supports the replacement methods — update the SDK package first if needed.

---

## Findings

### [CAT-001] Deprecated Sharing Endpoint

**Status:** 🟡 DEPRECATED — sunset date TBD
**Deprecated since:** 2025-08-04

#### Occurrences

| # | File | Line | Snippet |
|---|---|---|---|
| 1 | `src/api/sharing.js` | 42 | `client.sheets.listShares(sheetId)` |
| 2 | `src/api/sharing.js` | 67 | `await axios.get(\`/sheets/${id}/shares\`)` |

#### Migration

Replace asset-specific sharing calls with unified `/shares` endpoint.

**Before:**
\`\`\`javascript
// Deprecated
const shares = await client.sheets.listShares(sheetId);
\`\`\`

**After:**
\`\`\`javascript
// Current
const shares = await smartsheet.get('/shares', {
  params: { assetType: 'sheet', assetId: sheetId }
});
\`\`\`

---

### [CAT-002] Deprecated VIEWER Seat Type

...
```

---

## Migration Reference

Official migration guides with code examples for Java, C#, Python, and Node.js are maintained at:

**https://developers.smartsheet.com/api/smartsheet/guides/updating-code**

Always fetch this page (Step 4) rather than using the examples that were previously inlined here. The official guide reflects the current SDK methods and is updated alongside SDK releases; hand-crafted examples in this skill file will drift out of date.

---
> Source: [smartsheet/smartsheet-deprecation-scanner](https://github.com/smartsheet/smartsheet-deprecation-scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
