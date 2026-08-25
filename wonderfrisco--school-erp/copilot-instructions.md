## school-erp

> **Never use PowerShell `Get-Content` / `Set-Content` to read or write PHP/Blade files.**

# CLAUDE.md — School ERP Project Notes

## ⚠️ PowerShell File Encoding — CRITICAL

**Never use PowerShell `Get-Content` / `Set-Content` to read or write PHP/Blade files.**

PowerShell 5.1 (Windows) treats UTF-8 files as Windows-1252 by default.  
Piping through `Set-Content` silently corrupts non-ASCII characters:
- `→` becomes `â†'`
- `·` becomes `Â·`
- Any accented letter or symbol is mangled

**Always use explicit UTF-8 when doing bulk text operations:**

```powershell
# READ
$text = [System.IO.File]::ReadAllText($filePath, [System.Text.Encoding]::UTF8)

# WRITE
[System.IO.File]::WriteAllText($filePath, $newText, [System.Text.Encoding]::UTF8)
```

Or use the Bash tool (via Claude's toolset) which handles UTF-8 correctly.

---

## Stack

| Layer | Technology |
|-------|-----------|
| Framework | Laravel 12, PHP 8.3 |
| Frontend | Blade + Alpine.js + Tailwind CSS |
| Database | MySQL |
| PDF | barryvdh/laravel-dompdf |
| Queue | Redis (default) |
| Auth | Laravel Sanctum + custom role/permission system |

---

## Key Architecture Decisions

### School Settings (school_settings table)
All per-school configuration is stored in the `school_settings` key/value table — **not** as extra columns on the `schools` table.

- Access via `current_school()->getSetting('key', $default)` / `setSetting('key', $value)`
- Typed accessors on `School` model: `passMark()`, `caWeight()`, `examWeight()`, `currency()`, `gradeScale()`, `performanceTiers()`, etc.
- Settings are loaded in one query per request and cached in-memory on the School model instance (`$_settingsCache`).
- App-level defaults live in `config/school-erp.php` — the single source of truth for fallback constants.

### Multi-tenancy
`HasSchoolTenant` global scope (on most models) appends `WHERE school_id = ?` automatically.

- In web requests: `current_school()` helper resolves the school from the authenticated user.
- In CLI/queue (null school): the scope is **not applied**, so Artisan commands/jobs process all schools. This is intentional for cleanup commands.

### DomPDF Page Margins
**DomPDF ignores `@page { margin: ... }` rules.** The reliable approach is:

```html
<style>
  @page { size: A4 portrait; margin: 0; }
</style>
<table class="wrap-tbl" style="width:100%;border-collapse:collapse">
  <tr>
    <td style="padding:12mm; padding-bottom:50mm; vertical-align:top">
      <!-- all content goes here -->
    </td>
  </tr>
</table>
```

Cell padding is always honoured. Use `position:fixed` for headers/footers.

### Grade Scale
`GradingService::getGradeScale()` resolution order:
1. School settings (`school_settings` table, key `grade_scale`)
2. `config/school-erp.php` → `grading.grade_scale`
3. Static `GradingService::$gradeScale` (GES WASSCE defaults)

### Report Card Generation
- **New (current)**: `GenerateSingleReportCardJob` — one job per student, dispatched in a loop. Low memory.
- **Old (deprecated)**: `BulkReportCardJob` — loaded all students at once. Do not use for new code.

---

## Common Gotchas

### `total_score` on `grades` table
The `total_score` column is a **stored generated column** (`class_score + exam_score`). Do **not** try to insert/update it directly — MySQL computes it automatically. Attempting to set it will cause a SQL error.

### `is_core` on subjects table
The `subjects` table does **not** have an `is_core` column. Do not include it in column-constrained eager loads (`subject:id,name,code,is_core` will fail).

### `generated_by` on report_cards table
The `report_cards` table does **not** have a `generated_by` column. Do not eager-load `generatedBy` relationship.

### GradeController::index()
Signature is `public function index(): View` — no `$request` parameter. Adding it causes P1003 (unused variable) warnings.

---

## Config Reference (`config/school-erp.php`)

```php
grading.pass_mark      // default 50
grading.ca_weight      // default 30
grading.exam_weight    // default 70
grading.grade_scale    // GES WASSCE default scale
grading.performance_tiers
documents.currency     // default 'GHS'
documents.pdf_system_name
documents.report_card_title
documents.transcript_title
queues.reports         // 'reports'
queues.documents       // 'documents'
queues.notifications   // 'notifications'
cache.report_stats     // 300 (seconds)
cache.cert_stats       // 300
pagination.default     // 20
search.student_limit   // 15
cleanup.report_cards_years // 2
```

---
> Source: [wonderfrisco/school-erp](https://github.com/wonderfrisco/school-erp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
