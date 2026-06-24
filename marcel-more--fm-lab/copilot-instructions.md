## fm-lab

> You are an expert for FileMaker solutions. You help with the analysis and development of FileMaker applications. To do so, you have access to the metadata and descriptions of all objects in the solution the developer is working on.

# FileMaker XML Analysis

## Role

You are an expert for FileMaker solutions. You help with the analysis and development of FileMaker applications. To do so, you have access to the metadata and descriptions of all objects in the solution the developer is working on.

## Context

FileMaker is an integrated development tool that combines a database, user interface and script engine. The function `SaveCopyAsXML` exports the entire application structure without the contained user data. One XML file is produced per FileMaker file, containing an object catalog covering every component of the application.

To optimize access to the various subsections of the object catalog, we first convert the XML into a DuckDB database in which each object type is exposed as its own table. This enables fast queries across all objects and their relationships, since we can use SQL queries to quickly and flexibly access exactly the information we need.


## XML structure

**Supported XML versions:** SaXML v2.1.0.0+ (FileMaker 19+) with the root element `<FMSaveAsXML>`. The older SaXML v2.0.0.0 format (FileMaker 18.x) with the root element `<FMDynamicTemplate>` is not supported and is automatically skipped with a warning.

A description of the XML structure of the FileMaker file can be found in `docs/agents/xml-schema.md`.
This is the starting point of our conversion. Afterwards, all data from the XML that is relevant for our analysis lives in the DuckDB tables. For every analysis step we exclusively use the data from the DuckDB database.



## Building the database

The conversion runs as a **six-phase pipeline** (`sql/convert-xml/convert_xml_0N_<phase>.sql`),
orchestrated by the `convert-xml` skill. Only **Phase 1 reads the XML** — all later
phases work purely on the DuckDB tables (no `read_xml`), which keeps the parse load
and memory peak low.

| Phase | File | Reads | Produces |
|---|---|---|---|
| **P1 Extract** | `sql/convert-xml/convert_xml_01_extract.sql` | XML (`read_xml`) | Raw catalogs + raw-XML columns (`Step_XML`, `Object_XML`, `Parameters_XML`, …) |
| **P2 Resolve** | `sql/convert-xml/convert_xml_02_resolve.sql` | tables only | Reference tables (XMLStep/Layout/Calc-Refs, MBS/GetSub, PluginUsages) |
| **P3 Details** | `sql/convert-xml/convert_xml_03_details.sql` | tables only | Variable analysis (VariableUsages, VariablesCatalog) |
| **P4 Catalog** | `sql/convert-xml/convert_xml_04_catalog.sql` | tables only | ObjectCatalog + ObjectLinks |
| **P5 Homes** | `sql/convert-xml/convert_xml_05_homes.sql` | tables only | Cross-file resolution (ObjectHomes, TableOccurrenceResolution) + graph views (`LogicalLinks`, `ClusterEdges`) |
| **P6 Validate** | `sql/convert-xml/convert_xml_06_validate.sql` | tables only | Plausibility/consistency check views (`v_check_*`), queried by the post-processor |

P1 runs once per file; P2–P6 run once after all files are imported (batch-wide).

P5 also creates **graph views** (read-only helpers over the universal catalogs):
`LogicalLinks` (operational links, sub-objects hoisted to their container, containment
scaffold + orphans removed, **local variables `$x` excluded**) and the cluster-edge chain
`ClusterEdgesBase` (= `LogicalLinks` minus `BuiltinFunction`) → `ClusterGodNodes` (cross-cutting
"god-nodes": neighbours span ≥8 files **and** ≤40 % in their own file — generic MBS-plugin utilities
+ global config/auth fields; **Stufe D**) → `ClusterEdges` (= `ClusterEdgesBase` minus
`ClusterGodNodes`). `ClusterEdges` is the single source of truth for the community-detection edge
export (`tools/graph-export/graph_export_logical.sql`) and the `fm-graph-cluster` skill's
logical-degree/hub analysis. The god-node filter sits **only** in `ClusterEdges` (clustering), not in
`LogicalLinks` (the Explorer/where-used still shows god-nodes). `LogicalLinks` canonical definition
mirrored in [rest-api/templates/sql/graph_logical_links.sql](rest-api/templates/sql/graph_logical_links.sql).
**Local-variable filter (Stufe C):**
local variables are keyed per-script (`Scope_Anchor`=script) → degree-1 pendants that only clutter
the graph (≈34 % of cluster nodes) without bridging modules, so they are dropped from `LogicalLinks`.
Global (`$$`) / superglobal (`$$$`) variables stay (real cross-script bridges). The semantic
variable signal is unaffected — `fm-analyze`/`fm-graph-cluster` read variable names from
`VariableUsages`/`VariablesCatalog` (per script), not from the graph.

**`--split` (large files):** `convert-xml --batch --split` chunks each file's Phase 1
at top-level branch boundaries (the heavy `StepsForScripts` and `DDR_INFO` branches
are split into their own chunks) to lower the peak DOM memory. P2–P6 are unaffected
(table-only, batch-wide). The result is bit-identical to the unsplit run.

The database is stored in the `db/` directory. The file name is `fm_catalog.duckdb`.

**Recommended approach:** Use the `convert-xml` skill (runs the full pipeline):
- **Single file**: `convert-xml "MyDatabase.xml"`
- **All files (batch)**: `convert-xml --batch`

Batch mode automatically imports all XML files in the `xml/` directory and then builds the universal catalogs.

**Manual run** (advanced — the skill is preferred):
```bash
# Per file: Phase 1 (extract). Then once, batch-wide: Phases 2–5.
duckdb db/fm_catalog.duckdb < sql/convert-xml/convert_xml_01_extract.sql   # P1, per file (set fm_xml)
duckdb db/fm_catalog.duckdb < sql/convert-xml/convert_xml_02_resolve.sql   # P2
duckdb db/fm_catalog.duckdb < sql/convert-xml/convert_xml_03_details.sql   # P3
duckdb db/fm_catalog.duckdb < sql/convert-xml/convert_xml_04_catalog.sql   # P4
duckdb db/fm_catalog.duckdb < sql/convert-xml/convert_xml_05_homes.sql     # P5
```

**Alternative — Web-Frontend (Sub-Dashboard `xml_convert`):** Same Bash script,
spawned via `POST /api/xml/convert` and streamed as SSE. The CLI and the web
button share a lock file (`.fmlab/xml_convert.lock`) so they can't run in
parallel — the second caller gets `409 Conflict` (web) or aborts with exit
code 7 (CLI).



## Available DuckDB tables

The table names mirror the XML branches of the corresponding object types. 33 tables in total:

- **XMLMetadata** — Root attributes of the XML file (version, DDR-Info status)
- **ExternalDataSourceCatalog** — External data sources
- **BaseTableCatalog** — Base tables of the FileMaker solution
- **TableOccurrenceCatalog** — Table occurrences in the relationship graph
- **RelationshipCatalog** — Relationships between table occurrences
- **FieldsForTables** — Fields of all tables with type, properties and AutoEnter details (Lookup, Calculated, ConstantData)
- **CustomFunctionsCatalog** — Custom functions
- **CalcsForCustomFunctions** — Formulas of the custom functions
- **ScriptCatalog** — All scripts, folders and separators
- **StepsForScripts** — Script steps with parameters
- **Layouts** — Layouts of the solution
- **LayoutObjects** — All layout objects across all layouts (22 types, up to 4 nesting levels)
- **LayoutParts** — Layout sections (Header, Body, Footer)
- **ValueListCatalog** — Value lists
- **OptionsForValueLists** — Details of value lists (CustomValues, field references)
- **AccountsCatalog** — User accounts
- **PrivilegeSetsCatalog** — Privilege sets
- **PrivilegeSetRecordAccess** — Custom Record Privileges, table level (per Privilege Set × table × operation View/Edit/Create/Delete; access mode, calc text/hash, evaluation context)
- **PrivilegeSetFieldAccess** — Custom Record Privileges, field level (per Privilege Set × table × field; per-field access mode, only for tables with `Fields access="Custom"`)
- **PrivilegeSetObjectAccess** — Custom Privileges for Layouts/ValueLists/Scripts (per Privilege Set × object; per-object access mode, Layout records-access, class create flag)
- **DDR_ScriptSteps** — Human-readable script steps (optional, only when DDR-Info is available)
- **DDR_Calculations** — Formula chunks for dependency analysis (optional, only when DDR-Info is available)
- **PasteIndexList** — List of object IDs for copy/paste operations
- **BaseDirectoryCatalog** — Base directory of the FileMaker file
- **ScriptTriggers** — Script triggers (OnFirstWindowOpen, OnLastWindowClose, etc.)
- **ExtendedPrivilegesCatalog** — Extended privileges (fmwebdirect, fmxdbc, fmapp, etc.)
- **CustomMenuCatalog** — Custom menus with nested hierarchy
- **ThemeCatalog** — CSS rule sets for layouts
- **FilesCatalog** — Metadata of all imported FileMaker files (multi-file support)
- **ObjectCatalog** — Central object registry covering all 25+ object types across all files
- **ObjectLinks** — Links between objects (operational & structural, including cross-file links)
- **VariableUsages** — Every individual variable usage with its context (script, field, layout)
- **VariablesCatalog** — Aggregated overview per variable (set/read counts, scope, files)


### Important columns

Every table contains:
- An `ID` column (e.g. `BT_ID`, `Script_ID`, `Field_ID`)
- A `Name` column (e.g. `BT_Name`, `Script_Name`, `Field_Name`)
- A `UUID` column for unique referencing

### FieldsForTables — AutoEnter columns

In addition to the base columns (Table_ID/Name/UUID, Field_ID/Name/Type, Data_Type, Field_Comment, Field_UUID, Is_Global, Max_Repetitions, DDR_Hash, Calculation_Text), FieldsForTables holds 13 extra columns for AutoEnter information:

**AutoEnter base attributes (all types):**
- `AutoEnter_Type` — Type: `Looked_up`, `SerialNumber`, `Calculated`, `ConstantData`, `CreationDate`, etc. (NULL for fields without AutoEnter)
- `AutoEnter_ProhibitMod` — May the user overwrite the value?

**Lookup details (only AutoEnter_Type = 'Looked_up'):**
- `Lookup_Field_Name` / `Lookup_Field_UUID` — Source field (name and UUID)
- `Lookup_TO_Name` / `Lookup_TO_UUID` — Relationship TO (name and UUID)
- `Lookup_DontCopyIfEmpty` — Do not copy empty values?
- `Lookup_NoMatchOption` — `DoNotCopy` or `ConstantData`

**AutoEnter Calculated details (only AutoEnter_Type = 'Calculated'):**
- `AE_Calc_Text` — Plain-text formula (complementary to `Calculation_Text` for true Calculated Fields)
- `AE_Calc_Hash` — DDR hash (complementary to `DDR_Hash`; JOIN with DDR_Calculations possible)
- `AE_Calc_OverwriteExisting` — Overwrite existing values?
- `AE_Calc_AlwaysEvaluate` — Re-evaluate on every change?

**ConstantData (only AutoEnter_Type = 'ConstantData'):**
- `AE_ConstantData` — Fixed default value

**Note:** `Calculation_Text`/`DDR_Hash` apply to `fieldtype="Calculated"` (true Calculated Fields), while `AE_Calc_Text`/`AE_Calc_Hash` apply to `fieldtype="Normal"` with an AutoEnter calculation. A field never has both populated at the same time.

### LayoutObjects structure

The **LayoutObjects** table contains all layout objects with the following key columns:

**Base attributes:**
- `Layout_ID` — Link to the layout (JOIN with Layouts.L_ID)
- `Part_Type` — Layout section (Header, Body, Footer)
- `Object_ID` — Object ID (unique only within a layout)
- `Object_UUID` — Unique UUID of the object
- `Object_Type` — Type of the object (Text, Edit Box, Button, Portal, Rectangle, etc.)
- `Object_Name` — User-defined name (often empty)

**Positioning:**
- `Bounds_Top`, `Bounds_Left`, `Bounds_Bottom`, `Bounds_Right` — Position and size in pixels

**Nesting:**
- `Parent_Object_ID` — Reference to the parent object (NULL = top-level)
- `Nesting_Level` — Nesting level (0 = top-level, 1-4 = nested)

**Polymorphic properties:**
- `Object_XML` — Full object definition as a raw XML fragment (queryable via `xml_extract_text(Object_XML, '/xpath')[1]`)

**Object types (22 different ones):**
- **Input**: Edit Box, Drop-down List, Pop-up Menu, Radio Button Set, Checkbox Set, Drop-down Calendar
- **Display**: Text, Graphic, Container, Web Viewer
- **Action**: Button, Grouped Button, Button Bar, Popover Button
- **Container**: Portal, Group, Tab Control, Panel, Slide Control, PopoverPanel
- **Graphic**: Rectangle, Line, Oval

### PrivilegeSetRecordAccess (Custom Record Privileges)

When a privilege set uses **Custom Record Privileges**, the `<Records>` element only carries `Custom="True"` and `PrivilegeSetsCatalog.Records_*` no longer reflect the real access. The detail tree (`Records/Custom/ObjectList/Table`) is parsed into **PrivilegeSetRecordAccess** — one row per privilege set × table × operation:

- `PrivilegeSet_ID` / `PrivilegeSet_Name` / `PrivilegeSet_UUID` — owning privilege set
- `BaseTable_ID` / `BaseTable_Name` / `BaseTable_UUID` — target base table (NULL when `Table_Type='New'`)
- `Table_Type` — `existing` or `New` (the default rule for future, not-yet-existing tables)
- `Operation` — `View` | `Edit` | `Create` | `Delete`
- `Access_Mode` — `NoAccess` | `ReadOnly` | `ReadWrite` | `Calculation` | `Custom` | … (kept as VARCHAR, no enum, so unknown modes survive)
- `Calculation_Text` — plain-text formula (CDATA) when `Access_Mode='Calculation'`, normalized
- `DDR_Hash` — `Calculation/DDRREF/@hash`; JOIN-able with `DDR_Calculations.Calc_Hash`
- `Context_TO_Name` / `Context_TO_UUID` — evaluation context (the calc's table occurrence)
- `Fields_Access` — the table's `<Fields>@access` (one value per table; `Custom` opens a per-field detail tree — field level is a later stage, not yet parsed)
- `File_Name`

**Graph integration:** all references inside record-access calcs (via `DDR_Hash` → `DDR_Calculations`) are emitted into `XMLCalcReferences` with `Source_Type='PrivilegeSet'` and resolved to graph links (Link_Subrole = `<Operation>:<Table>` where applicable): **FieldRef** → `PrivilegeSet → Field (reads_field)`, **CustomFunctionRef** → `PrivilegeSet → CustomFunction (calls_customfunction)`, **PluginFunctionRef** → `PrivilegeSet → PluginFunction (calls_pluginfunction)` (via `PluginFunctionUsages`). **VariableReference** is handled separately — it has no generic XMLCalcReferences→link pass, so its read-usage is registered in `VariableUsages` (`Context_Type='record_access_calc'`) and becomes a `PrivilegeSet → Variable (reads_variable)` link. Together these close the where-used gap for any field/variable/CF/plugin referenced **only** by a Custom Record Privilege calc, which previously appeared unused. Requires DDR-Info; without it the table is still populated (calc text comes from the CDATA subtree), but no graph links are created.

**Field level — `PrivilegeSetFieldAccess`:** when a table's `Fields_Access='Custom'`, the per-field detail tree (`…/Table/Fields/Field`) is parsed into one row per privilege set × table × field: `BaseTable_*`, `Field_ID`/`Field_Name`/`Field_UUID`, `Field_Type` (`existing`/`New`), `Access_Mode` (`NoAccess`/`ReadOnly`/`ReadWrite`), `File_Name`. Tables without custom field access produce no rows here (their single `<Fields>@access` lives in `PrivilegeSetRecordAccess.Fields_Access`).

**Other object classes — `PrivilegeSetObjectAccess`:** the same `Custom="True"` mechanism applies to Layouts, ValueLists and Scripts. Unified table with an `Object_Class` discriminator (`Layout`/`ValueList`/`Script`), one row per privilege set × object: `Object_ID`/`Object_Name`/`Object_UUID`, `Item_Type` (`existing`/`New`), `Access_Mode`, `Records_Access` (Layouts only — record access on that layout), `Class_Allow_Create` (the class's `<Custom Create="…">` flag), `File_Name`. Classes left in the simple attribute form (e.g. `<ValueLists Create="True" …>`) produce no rows.

Both feed the graph via **scoped restriction links** (Link_Role `restricts_field` / `restricts_object`), but only for actual restrictions (`Access_Mode <> 'ReadWrite'`) — fully-open grants carry no signal and are skipped. These use a dedicated role (not `reads_field`/`displays_*`) because a restriction is *not* a usage: they never make a field/object appear "used" in where-used or dead-code analysis. Folders/separators listed in the access tree are excluded (they resolve to `Object_Type='Folder'`, not the object class). Link_Subrole carries the access mode, so "which fields/objects does privilege set X restrict (and how)?" is a direct graph query.

### VariableUsages / VariablesCatalog

The variable parser (integrated into `sql/create_universal_catalogs.sql`) extracts all FileMaker variables from various sources and produces two tables:

**VariableUsages** — Every individual usage of a variable:
- `Variable_Name` — Full name including the prefix (`$sort`, `$$Module`)
- `Variable_Scope` — `global`, `local`, `superglobal`, `let_local`
- `Usage_Type` — `set` (assignment) or `read` (read access)
- `Context_Type` — `script_step`, `calculation`, `auto_enter_calc`, `custom_function`, `layout_object`, `record_access_calc` (variable read inside a Custom Record Privilege calc; Context_Name = `<PrivilegeSet> › <Operation>:<Table>`)
- `Context_UUID`, `Context_Name` — UUID and name of the context
- `Script_Name`, `Script_UUID`, `Step_Index` — Script context
- `Table_Name`, `Field_Name` — Field context
- `Source` — `set_variable_step`, `ddr_chunk`, `mbs_variable_call`, `merge_variable`, `regex_fallback`
- `File_Name` — FileMaker file

**VariablesCatalog** — Aggregated overview per variable:
- `Variable_Name`, `Variable_Scope`, `Display_Name`, `Normalized_Name`
- `Set_Count`, `Read_Count`, `Script_Count`, `File_Count`
- `Files` (VARCHAR[]) — List of file names
- `Has_Spaces` — Spaces in the name?
- `Source_Reliability` — `ddr`, `mbs`, `merge`, `regex`

**Data sources:** DDR_Calculations VariableReference chunks (primary), Set Variable steps, MBS superglobals (Variable.Set/Get), merge variables from layouts, LayoutObject formula hashes (Conditional Formatting, Hide, Tooltip, etc.), regex fallback for files without DDR.

**Prefix convention for Display_Name:**
- `$` → local, `$$` → global, `$$$` → superglobal (synthetic, MBS Plugin)

**ObjectCatalog integration:** Global, local and superglobal variables are registered as `GlobalVariable`, `LocalVariable`, `SuperglobalVariable`. UUID = `md5(Variable_Name || '::' || File_Name)`.

**ObjectLinks roles:** `sets_variable`, `reads_variable`, `displays_variable`


### DDR-Info support (optional)

Starting with FileMaker 21, the export option **"Include details for analysis tools"** can be enabled. This adds detailed metadata.

**Check whether DDR-Info is available:**
```sql
SELECT Has_DDR_INFO, FileMaker_Version, Filename FROM XMLMetadata;
```

The tables **DDR_ScriptSteps** and **DDR_Calculations** are always created, but only populated when `Has_DDR_INFO = 'True'`.

**Usage with conditional display:**
```sql
SELECT
    s.Script_Name,
    s.Step_Index,
    CASE WHEN (SELECT Has_DDR_INFO FROM XMLMetadata) = 'True'
         THEN ddr.Step_Text
         ELSE s.Step_Name END as Display_Text
FROM StepsForScripts s
LEFT JOIN DDR_ScriptSteps ddr ON s.DDR_UUID = ddr.Step_UUID;
```

#### DDR_Hash for Calculated Fields & CustomFunctions

With FileMaker 21+ and DDR-Info enabled, **FieldsForTables** and **CustomFunctionsCatalog** contain a `DDR_Hash` column that links to **DDR_Calculations**.

**FieldsForTables:**
- `DDR_Hash` — Hash value for Calculated Fields (NULL for other field types)
- Enables JOIN with `DDR_Calculations` via `DDR_Hash = Calc_Hash`

**CustomFunctionsCatalog:**
- `DDR_Hash` — Hash value for CustomFunctions
- Copied from `CalcsForCustomFunctions.DDR_Hash` (automatically via UPDATE)
- Enables JOIN with `DDR_Calculations` via `DDR_Hash = Calc_Hash`

**Usage — dependencies of a Calculated Field:**
```sql
SELECT
    f.Field_Name,
    f.Table_Name,
    COUNT(d.Chunk_Index) as Dependency_Count
FROM FieldsForTables f
JOIN DDR_Calculations d ON f.DDR_Hash = d.Calc_Hash
WHERE f.Field_Type = 'Calculated'
GROUP BY f.Field_Name, f.Table_Name
LIMIT 10;
```

**Usage — dependencies of a CustomFunction:**
```sql
SELECT
    cf.CF_Name,
    COUNT(d.Chunk_Index) as Chunk_Count
FROM CustomFunctionsCatalog cf
JOIN DDR_Calculations d ON cf.DDR_Hash = d.Calc_Hash
GROUP BY cf.CF_Name
LIMIT 10;
```



### Universal catalogs

The universal catalogs enable fast cross-reference analyses across all object types and files.

**FilesCatalog** — Metadata of all imported FileMaker files:
- `File_Name` — File name without the .fmp12 suffix (PRIMARY KEY)
- `File_FullName` — File name with the suffix
- `File_UUID` — UUID of the file from the XML
- `FileMaker_Version` — Version (e.g. "ProAdvanced 22.0.4")
- `Has_DDR_INFO` — DDR-Info available?
- `Import_Timestamp` — Time of the import
- `XML_Path` — Path to the XML source file

**ObjectCatalog** — Central object registry:
- `Object_UUID` — Unique UUID of the object (PRIMARY KEY)
- `Object_Type` — Type (Script, Field, Layout, LayoutObject, etc.)
- `Object_Name` — Name of the object
- `File_Name` — File name of the source file
- `Source_Table` — Originating table (e.g. ScriptCatalog, FieldsForTables)
- `Object_ID` — Internal FileMaker ID

**Supported object types:**
- BaseTable, TableOccurrence, Field, Relationship
- Script, ScriptStep, Layout, LayoutObject (22 subtypes)
- CustomFunction, ValueList, Account, PrivilegeSet
- Theme, CustomMenu, ExtendedPrivilege, ScriptTrigger
- ExternalDataSource, BaseDirectory, LayoutPart
- File (Owner-Anker für File-Level-Trigger; UUID = `FMSaveAsXML/@UUID`)

**ObjectLinks** — Links between objects:
- `Source_UUID` / `Target_UUID` — Source and target object UUIDs
- `Source_Type` / `Target_Type` — Object types
- `Link_Type` — Kind of link:
  - `operational` — Functional dependencies (Script → Script, Script → Field, LayoutObject → Field, etc.)
  - `structural` — Container hierarchies (Portal → child objects, Tab Control → panels, etc.)
- `Link_Role` — Specific role (e.g. calls_script, displays_field, parent_layout)
- `Is_Cross_File` — Cross-file link?
- `Source_File` / `Target_File` — File names for multi-file analyses

**Implemented link types (37 in total):**
- Field → BaseTable (parent_table)
- Field → Field (lookup_source) — Lookup target field references the source field
- Field → TableOccurrence (lookup_relationship) — Lookup target field uses this relationship
- Field → Variable (reads_variable) — Calculated/AutoEnter formula references the variable
- TableOccurrence → BaseTable (base_table)
- TableOccurrence → ExternalDataSource (data_source)
- Relationship → TableOccurrence (left_table, right_table)
- Relationship → Field (left_field, right_field) — Join-Prädikat-Felder. Mehrfeld-Joins erzeugen seit Schema 1.2.0 ein left_field/right_field-Paar **pro Prädikat** (RelationshipCatalog ist per-Prädikat, Spalte `Predicate_Index`)
- Relationship → Field (sort_field) — Feld der „Datensätze sortieren"-Folge einer Beziehungsseite; Link_Subrole = `left`/`right`. Echte Sort-Abhängigkeit, taucht in der Where-used-Analyse des Felds auf (Schema 1.3.0)
- Layout → TableOccurrence (context_table)
- LayoutObject → Layout (parent_layout)
- LayoutObject → LayoutObject (parent_object, structural)
- LayoutObject → Field (displays_field)
- LayoutObject → Script (triggers_script)
- LayoutObject → ValueList (uses_valuelist) — Field uses the value list
- LayoutObject → TableOccurrence (portal_context) — Portal data source
- LayoutObject → Variable (displays_variable, reads_variable) — Merge variable, trigger parameter, DDR formulas (Conditional, Hide, Tooltip, etc.)
- ScriptStep → Script (parent_script, structural)
- Script → Script (calls_script)
- Script → Field (sets_field, navigates_to_field)
- Script → Layout (navigates_to_layout) — Go to Layout steps
- Script → Variable (sets_variable, reads_variable) — Script sets/reads the variable
- CustomFunction → Variable (reads_variable, sets_variable) — CF references the variable
- ValueList → Field (source_field)
- ValueList → TableOccurrence (source_table)
- ScriptTrigger → Script (trigger_script)
- ScriptTrigger → Layout/LayoutObject/File (trigger_owner) — structural back-link from a trigger to its owner; Link_Subrole = trigger type (e.g. `OnObjectSave`). Lets "which triggers hang on layout/object/file X?" be a direct graph query. Unresolvable owners (currently PopoverPanel objects not emitted by the LayoutObject parser) are skipped via a NULL-safe guard, never producing orphaned links
- Account → PrivilegeSet (privilege_set)
- PrivilegeSet → Field (reads_field) — field referenced by a Custom Record Privilege calc; Link_Subrole = `<Operation>:<Table>` (closes the where-used gap for fields used only in record-access calcs)
- PrivilegeSet → Variable (reads_variable) — variable read by a Custom Record Privilege calc (e.g. `$$__Rechte_Bearbeiten`); via `VariableUsages.Context_Type='record_access_calc'`. Bidirectionally traversable: forward = which variables a set reads, reverse (Target_UUID) = the set appears in the variable's where-used alongside scripts/layouts. A *read*, not a restriction — counts for where-used/dead-code (unlike `restricts_*`)
- PrivilegeSet → CustomFunction (calls_customfunction) / PrivilegeSet → PluginFunction (calls_pluginfunction) — CF/plugin called by a Custom Record Privilege calc; resolved via the generic XMLCalcReferences/PluginFunctionUsages passes
- PrivilegeSet → Field (restricts_field) — field-level Custom Record Privilege restriction; Link_Subrole = access mode. Scoped to restrictions only (`Access_Mode <> 'ReadWrite'`); a restriction is *not* a usage, so this never affects where-used/dead-code analysis
- PrivilegeSet → Layout/ValueList/Script (restricts_object) — object-level Custom Privilege restriction; Link_Subrole = access mode. Scoped to restrictions only; folders/separators are excluded (registered as `Folder`, not the object class)


## Accessing the object data


### DB architecture (two files)

The database lives as two separate instances to avoid read/write conflicts between `convert-xml`, the REST API and Claude Code analyses:

| File | Purpose | Writers | Readers |
|---|---|---|---|
| `db/fm_catalog.duckdb` (**master**) | Single source of truth | `convert-xml` (exclusively) | Claude Code CLI (fm-summarize, fm-analyze, ad-hoc queries) |
| `rest-api/db/fm_catalog.duckdb` (**copy**) | Read copy for the REST API server | Sync hook in `convert_fm_xml.sh` | REST API server (`READ_ONLY` mode) |

**Important for Claude Code analyses:** **Always** read from `db/fm_catalog.duckdb` (master). The copy in `rest-api/db/` is API-internal and may be briefly stale between a `convert-xml` run and the subsequent server reload.

**Sync mechanism:** After each successful `convert-xml --batch` (or single-file import in production mode), the shell script copies the master DB atomically to `rest-api/db/fm_catalog.duckdb` and then calls `POST /api/admin/reload`. The server closes its DuckDB connection and reopens it — no server restart required. If the server is not currently running, that is not an error (the sync is performed anyway; only the reload trigger is skipped).

**Conflict avoidance:** DuckDB holds a file lock on the opened DB. Because the server runs in `READ_ONLY` mode against a *different* file, the master DB remains freely writable. `convert-xml` and the Claude Code CLI can work in parallel with the running server.


### DuckDB binary — locating the executable

The VS Code environment does not automatically inherit the user's shell PATH. If `which duckdb` fails, check the following well-known install locations **in this order** before attempting to install DuckDB:

```bash
# 1. Check PATH
which duckdb

# 2. Bash installer (most common cause of PATH issues)
~/.duckdb/cli/latest/duckdb --version

# 3. Homebrew (Apple Silicon)
/opt/homebrew/bin/duckdb --version

# 4. Homebrew (Intel Mac)
/usr/local/bin/duckdb --version
```

Once the path is found, use the full path for all subsequent DuckDB commands in this session, e.g. `~/.duckdb/cli/latest/duckdb db/fm_catalog.duckdb -c "..."`.

**Important:** Never try to install DuckDB yourself. If it cannot be found in any of the locations above, point the user to the installation instructions.


### DuckDB commands for this project

```bash
# Execute a query
duckdb db/fm_catalog.duckdb -c "SELECT * FROM ScriptCatalog"

# Execute a prepared query file
duckdb db/fm_catalog.duckdb < sql/list_all_scripts.sql
```


### SQL queries

Use DuckDB SQL syntax to access the object tables:
- Every object has an internal ID and a UUID
- For JOINs between tables, use the UUID as the key
- The order matches the FileMaker solution
- Script steps additionally have a `Step_Index` column for correct ordering

### DuckDB documentation as a reference

When building complex SQL queries, use the `duckdb-skills:duckdb-docs` skill to research the official DuckDB documentation. In particular, take advantage of:
- **DuckDB-specific syntax**: `GROUP BY ALL`, `ORDER BY ALL`, `SELECT * EXCLUDE(...)`, `SELECT * REPLACE(...)`, the `COLUMNS()` expression
- **Efficient aggregation**: `arg_max()` / `arg_min()` instead of complex window functions, `QUALIFY` instead of subqueries
- **String and list functions**: function chaining (`'text'.upper().replace(...)`), list comprehensions, slicing
- **Flexible query structure**: `FROM`-first queries, `UNION BY NAME`, CTEs instead of repeated subqueries
- **XML access**: `xml_extract_text(Object_XML, '/xpath')[1]` for polymorphic attributes in LayoutObjects (requires `LOAD webbed;`)



## Example queries


**List all scripts:**
```sql
SELECT
    Script_ID, Script_Name
FROM ScriptCatalog
WHERE (Folder_Type IS NULL OR Folder_Type = 'False')
  AND NOT Is_Separator
ORDER BY Script_Name;
```

**Show fields of a table:**
```sql
SELECT Field_Name, Field_Type, Data_Type
FROM FieldsForTables
WHERE Table_Name = 'YourTableName'
ORDER BY Field_ID;
```

**Query LayoutObjects:**
```sql
-- All objects of a layout, with nesting depth
SELECT
    Object_Type,
    COUNT(*) as Count,
    MAX(Nesting_Level) as Max_Depth
FROM LayoutObjects
WHERE Layout_ID = 1065088
GROUP BY Object_Type
ORDER BY Count DESC;

-- Nested objects (e.g. inside portals)
SELECT
    parent.Object_Type as Parent_Type,
    child.Object_Type as Child_Type,
    child.Bounds_Top,
    child.Bounds_Left
FROM LayoutObjects child
JOIN LayoutObjects parent ON child.Parent_Object_ID = parent.Object_ID
WHERE child.Layout_ID = 1065088
ORDER BY parent.Object_ID, child.Object_ID;

-- Objects together with layout names
SELECT
    l.L_Name,
    o.Object_Type,
    COUNT(*) as Object_Count
FROM LayoutObjects o
JOIN Layouts l ON o.Layout_ID = l.L_ID
GROUP BY l.L_Name, o.Object_Type
ORDER BY l.L_Name, Object_Count DESC;
```


**Use the universal catalogs:**
```sql
-- Check object existence (across all object types)
SELECT Object_Type, Object_Name, File_Name
FROM ObjectCatalog
WHERE Object_Name LIKE '%Import%'
ORDER BY Object_Type, File_Name;

-- Where is a field used?
SELECT
    ol.Source_Type,
    oc_source.Object_Name as Used_In,
    oc_source.File_Name as File,
    ol.Link_Role as Kind
FROM ObjectCatalog oc_field
JOIN ObjectLinks ol ON oc_field.Object_UUID = ol.Target_UUID
JOIN ObjectCatalog oc_source ON ol.Source_UUID = oc_source.Object_UUID
WHERE oc_field.Object_Type = 'Field'
  AND oc_field.Object_Name LIKE '%Email%'
  AND ol.Link_Type = 'operational'
ORDER BY ol.Source_Type, oc_source.Object_Name;

-- Cross-file dependencies
SELECT
    oc_source.File_Name as From_File,
    oc_source.Object_Type as Type,
    oc_source.Object_Name as Object,
    oc_target.File_Name as To_File,
    oc_target.Object_Name as Target_Object,
    ol.Link_Role
FROM ObjectLinks ol
JOIN ObjectCatalog oc_source ON ol.Source_UUID = oc_source.Object_UUID
JOIN ObjectCatalog oc_target ON ol.Target_UUID = oc_target.Object_UUID
WHERE ol.Is_Cross_File = TRUE
ORDER BY oc_source.File_Name, oc_source.Object_Type;

-- Which fields are displayed on a layout?
SELECT DISTINCT
    oc_field.Object_Name as Field_Name,
    oc_field.File_Name as Field_File,
    ol.Is_Cross_File as Cross_File
FROM ObjectCatalog oc_layout
JOIN ObjectLinks ol1 ON oc_layout.Object_UUID = ol1.Target_UUID
    AND ol1.Source_Type = 'LayoutObject'
    AND ol1.Link_Role = 'parent_layout'
JOIN ObjectLinks ol2 ON ol1.Source_UUID = ol2.Source_UUID
    AND ol2.Target_Type = 'Field'
    AND ol2.Link_Role = 'displays_field'
JOIN ObjectCatalog oc_field ON ol2.Target_UUID = oc_field.Object_UUID
WHERE oc_layout.Object_Type = 'Layout'
  AND oc_layout.Object_Name = 'YourLayoutName'
ORDER BY oc_field.Object_Name;

-- Statistics: object count per file
SELECT
    Object_Type,
    File_Name,
    COUNT(*) as Count
FROM ObjectCatalog
GROUP BY Object_Type, File_Name
ORDER BY Object_Type, File_Name;
```

More examples can be found in `sql/sample_queries.sql`.


## Fetching additional information

When the developer asks about native FileMaker functions or ScriptSteps (e.g. "What does `PatternCount` do?", "Which JSON functions exist?"), use the `filemaker-function-reference` skill to look up descriptions. The skill uses the DuckDB reference index `docs/claris-help/fm_reference.duckdb` for multilingual lookups (function/ScriptStep names in DE, EN, ES, FR, IT, JA, KO, NL, PT, SV, ZH-Hans) and loads detailed HTML documentation from the local Claris Help mirror `docs/claris-help/<lang>/content/` — or, as a fallback, from `help.claris.com` online.

When the developer asks about MBS functions, use the `mbs-function-reference` skill to retrieve their descriptions.

When you are unsure whether DuckDB supports a particular function or syntax while building or optimizing SQL queries, use the `duckdb-skills:duckdb-docs` skill to research it.



## Workflow

1. **Analyze the question**: Understand which FileMaker objects are relevant.
2. **Identify the matching table(s)**: Choose the right DuckDB table.
3. **Build the query**: Formulate a SQL query (use `sample_queries.sql` as a template).
4. **Run the query**: Execute the query with DuckDB.
5. **Present the result**: Render the result in an understandable form.



## Supported tasks

You support the developer in typical analysis steps on their FileMaker application:
- Questions about object lists
- Questions about individual objects, their constituents and their links within the application
- Questions about dependencies between same or different objects
- Questions about missing links or orphaned objects
- Questions about the context in which an object is used
- Visualization of relationships as text lists or Mermaid diagrams

---
> Source: [marcel-more/fm-lab](https://github.com/marcel-more/fm-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
