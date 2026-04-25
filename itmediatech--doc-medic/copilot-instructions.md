## doc-medic

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Doc Formatter & Hyperlink Fixer — Architecture & Implementation Outline | Project Name = "Doc_Medic"

## Development Commands

Since this project uses .NET 9, standard .NET CLI commands apply:
- **Build solution**: `dotnet build src/App.sln --configuration Release`
- **Run tests**: `dotnet test src/App.sln --configuration Release`
- **Run specific test**: `dotnet test --filter "FullyQualifiedName~TestClassName.TestMethodName"`
- **Restore packages**: `dotnet restore src/App.sln`
- **Clean**: `dotnet clean src/App.sln`
- **Local Velopack build**: `.\build.ps1 -Version "1.0.0" -Configuration "Release"`

## Implementation Status

✅ **COMPLETED**
1. Complete project structure with all assemblies (App.UI, App.Domain, App.Core, App.Infrastructure, App.Services, App.Tests)
2. All core interfaces implemented (App.Core)
3. Full OpenXML document processing pipeline (App.Infrastructure)
4. WPF UI with Prism MVVM and Material Design theming (App.UI)
5. Real service integrations replacing all mock implementations
6. Velopack deployment configuration with GitHub Actions CI/CD
7. Settings and secrets management with DPAPI encryption
8. Comprehensive logging with Serilog
9. HTTP client with Polly resilience policies

## Implementation Priority (Historical)
1. ✅ Start with the explicit project structure (section 2)
2. ✅ Implement interfaces first (from section 9) 
3. ✅ Follow the pseudocode in section 21
4. ✅ Use the OpenXML helpers in section 8 as implementation guides
5. ✅ The regex patterns and processing logic are production-ready

> Purpose: Modern desktop app to batch‑format .docx files and repair specific hyperlinks via a Power Automate API. This document is a coding brief for generating the full solution with Claude CLI.

---

## 0) Goals & Non‑Goals

**Goals**

* Batch select many `.docx` files and apply user‑selected transforms reliably and fast.
* Fix **only** hyperlinks that match defined patterns (CMS, TSRC, or `?docid=`) using a Power Automate API.
* Standardize styles (Normal, Heading 1, Heading 2, Hyperlink) to the specified rules.
* Optional cleanups: collapse multiple spaces, normalize “Top of Document” links, center images.
* Seamless in‑app updates from GitHub Releases.
* Modern, sleek, responsive WPF UI with theming, persistent settings, and verbose logging.

**Non‑Goals**

* Editing formats other than `.docx`.
* General link validation beyond the specified patterns.

---

## 1) Platform & Tech Stack

* **Runtime**: .NET 9 (Windows)
* **UI**: WPF + Prism (MVVM, modular), modern theme via **MaterialDesignInXaml**.
* **Document IO**: Open XML SDK (`DocumentFormat.OpenXml`). No Word COM automation.
* **Dependency Injection**: Prism + DryIoc.
* **Updater**: Velopack
* **HTTP**: `HttpClient` w/ Polly for retry & timeout.
* **Config**: JSON settings at `%AppData%/Company/App/settings.json`, secrets (there is no API key for Power Automate HTTP Request) encrypted via DPAPI.
* **Logging**: Serilog (rolling file logs + optional console sink in debug).
* **Testing**: xUnit (regex/unit), approval tests for XML transforms.

---

## 2) Solution Structure

```
repo-root/
  src/
    App.UI/                    # ✅ WPF shell (Prism), views, viewmodels, navigation
    App.Domain/                # ✅ Domain models, enums, constants, value objects
    App.Core/                  # ✅ Abstractions: services, ports, pipeline stages
    App.Infrastructure/        # ✅ OpenXML impl, HTTP client, storage, updater, logging
    App.Services/              # ✅ Orchestration services (document pipeline, styles, links)
    App.CLI/                   # (Optional) headless batch runner using same services
  tests/
    App.Tests/                 # ✅ Unit + integration tests
  .github/
    workflows/
      release.yml              # ✅ GH Actions to build, sign, and publish releases
  tools/
    schemas/                   # JSON Schemas for API payloads/responses (if needed)
  velopack.json                # ✅ Velopack package configuration
  deployment.json              # ✅ Production deployment settings  
  build.ps1                    # ✅ Local build and packaging script
  Directory.Packages.props     # ✅ Centralized package management
  src/App.sln                  # ✅ Visual Studio solution file
  CLAUDE.md                    # this file
```

**Key assembly boundaries**

* `App.Core`: ✅ Interfaces + contracts (stable). No external deps.
* `App.Domain`: ✅ Pure C# types (POCOs), regexes, constants.
* `App.Infrastructure`: ✅ All side‑effects (IO, network, OpenXML, updater, DPAPI, file system).
* `App.Services`: ✅ Application logic, pipelines, composition over `App.Core` ports.
* `App.UI`: ✅ MVVM only; no OpenXML or HTTP directly.

---

## 3) Public Features (User Flows)

1. **Select files**: drag‑drop or file picker -> shows a table with per‑file status, size, last modified.
2. **Options panel** (toggles):

   * Replace double spaces+ with single.
   * Fix “Top of Document” links (internal anchor, right‑aligned, styled).
   * Standardize styles (Normal, Heading 1, Heading 2, Hyperlink).
   * Center all images.
   * “Fix only API‑eligible links” is implicit (always on, per spec).
3. **API settings**: base URL, POST path, API key, test connection button.
4. **Run**: per‑file progress, aggregate progress bar, cancel.
5. **Results**: summary (files changed, links fixed, warnings). Exportable log.
6. **Updates**: check on launch + manual “Check for updates”.

---

## 4) UI/UX & Theming

* **Shell**: Left nav (Home, Process, Settings, Logs, About). Content frame hosts pages.
* **Design**: Material 3 look, light/dark toggle, accent color selection.
* **DataGrid** for files: columns (Name, Path, Size, Modified, Status, Changes).
* **Settings** page: API (endpoint, timeout), updater channel (Stable/Preview), theme. (API Key not necessary for Power Automate HTTP Request)
* **Logs** page: tail view (live), filter by level/file.
* **Accessibility**: high‑DPI scaling, keyboard shortcuts, focus visuals.

---

## 5) Domain Rules (Hyperlinks)

**Eligible hyperlinks only** (external URLs unless otherwise noted):

* Contains `?docid=` → **Document\_ID** = everything after `?docid=` up to `#`, `&`, or end.
* Contains `CMS` or `TSRC` → **Content\_ID** pattern: `(CMS|TSRC)-[A-Za-z0-9\-]+-\d{6}`.

**Extraction**

* Scan each hyperlink target (relationship Target or field code). Build `Lookup_IDs`:

  * Add extracted **Document\_ID** values.
  * Add extracted **Content\_ID** values.
  * Deduplicate (case‑sensitive as provided), preserve original mapping to the hyperlink(s).

**API contract (high‑level)**

* HTTP POST with `Lookup_IDs` array. Response items contain: `Title`, `Status` (`Expired|Released`), `Content_ID`, `Document_ID`.

**Fix logic**

* Construct canonical URL: `http://thesource.cvshealth.com/nuxeo/thesource/#!/view?docid={Document_ID}`.
* If canonical != current URL → replace hyperlink target.
* Ensure **Text to Display** contains ` (XXXXXX)` where `XXXXXX` are last 6 digits of Content\_ID:

  * If last 6 not present, but last 5 present exactly → prefix with `0`.
  * Else: trim trailing whitespace in display text, then append ` (XXXXXX)`.
* Apply **only** to hyperlinks deemed eligible (per patterns above).

---

## 6) Formatting & Layout Rules

**Replace double spaces+**

* Collapse sequences of normal spaces across run boundaries while preserving non‑breaking spaces and punctuation spacing.

**“Top of Document” links**

* Ensure a bookmark at document start (e.g., `DocStart`).
* Locate paragraphs whose visible text equals (or starts with) “Top of Document” (case/whitespace normalized).
* Update those links to internal anchor `#DocStart`, apply **Hyperlink** character style, and right‑align the paragraph.

**Standardize styles**

* *Normal* (paragraph style): Verdana, 12pt, black, single line spacing, **6pt before**, and **do not** enable “Don’t add space between paragraphs of the same style”.
* *Heading 1* (paragraph style): Verdana, 18pt, bold, black, left‑aligned, **0pt before / 12pt after**, single spacing.
* *Heading 2* (paragraph style): Verdana, 14pt, bold, black, left‑aligned, **6pt before / 6pt after**, single spacing.
* *Hyperlink* (character style): Verdana, 12pt, blue `#0000FF`, underline, single spacing characteristics at the paragraph level remain governed by containing paragraph style.

  * Note: “Don’t add space between paragraphs of the same style” is a **paragraph** option; set it on paragraphs that are solely the hyperlink or where needed.

**Center all images**

* For paragraphs containing inline `Drawing` elements, set paragraph justification center.
* For floating images, adjust anchor alignment or wrap to center within page/column width.

---

## 7) Processing Pipeline (Per File)

```
[Load DOCX] → [Index hyperlinks] → [Extract Lookup_IDs] → [POST to API]
→ [Build id→metadata map] → [Repair eligible hyperlinks]
→ [Apply options: spaces | TopOfDoc | styles | center images]
→ [Save] → [Report]
```

* **Concurrency**: Process files in parallel (bounded, e.g., `DegreeOfParallelism = Environment.ProcessorCount - 1`).
* **Atomic writes**: Save to temp, then replace original; create `.bak` if “Keep backup” is enabled.
* **Cancellation**: Propagate `CancellationToken` through all stages.

---

## 8) OpenXML Implementation Notes (Key Helpers)

**A) Document load/save**

* Use `WordprocessingDocument.Open(path, true)`; for reading relationships, work with `MainDocumentPart.HyperlinkRelationships` and `Document.Body`.

**B) Hyperlink enumeration**

* External links: `w:hyperlink` with `r:id` → resolve via `HyperlinkRelationship` (TargetUri).
* Internal anchors: `w:hyperlink` with `w:anchor`.
* HYPERLINK field codes: detect `w:fldSimple` or `w:instrText` runs with `HYPERLINK`—normalize to `w:hyperlink` if needed.

**C) ID extraction**

* **Document\_ID**: regex `(?<=\?docid=)[^#&\s]+`.
* **Content\_ID**: regex `\b(CMS|TSRC)-[A-Za-z0-9\-]+-\d{6}\b`.

**D) API mapping**

* Request: `Lookup_IDs` (array of strings). Response: array of objects (`Title`, `Status`, `Content_ID`, `Document_ID`). Build dictionary by both keys to resolve either way.

**E) Canonical URL**

* `http://thesource.cvshealth.com/nuxeo/thesource/#!/view?docid={Document_ID}`.
* Update relationship Target for external links. For field codes, rewrite instruction.

**F) Display text update**

* Extract last 6 digits: regex `([0-9]{6})$` over `Content_ID`. If not found, pad left from last 5.
* Inspect visible text (concatenate contiguous `w:t`). If neither `(XXXXXX)` nor `(0XXXXX)` present, append ` (XXXXXX)` to trailing `w:t` while preserving run formatting.

**G) “Top of Document”**

* Ensure a bookmark at the start: insert `BookmarkStart`/`BookmarkEnd` around first paragraph’s first run. Name: `DocStart`.
* Find candidate paragraphs whose normalized visible text equals "Top of Document" (ignore case, trim). Rewrite as internal `w:hyperlink w:anchor="DocStart"`.
* Paragraph `w:jc` = `right`. Apply **Hyperlink** character style to the runs.

**H) Style standardization**

* Ensure `StylesPart` exists. Upsert paragraph styles for Normal, Heading1, Heading2 with properties:

  * Font: `w:rFonts w:ascii="Verdana" w:hAnsi="Verdana"`
  * Size: `w:sz w:val="24"` for 12pt (half‑points \* 2), `w:sz="36"` for 18pt, `w:sz="28"` for 14pt.
  * Spacing: set `w:spacing w:before/w:after` in twips (e.g., 6pt = 120, 12pt = 240).
  * Line spacing: `w:spacing w:line="240" w:lineRule="auto"` (single).
* Ensure **Hyperlink** character style exists/updated with color `w:color w:val="0000FF"` and `w:u w:val="single"`.
* Apply style mapping: Title paragraph → Heading1; section headings → Heading2; body paragraphs → Normal (only if safe—see rollout strategy below).

**I) Center images**

* For paragraphs containing only drawing/content, set `w:jc = center`. For anchored images, adjust wrapping/positioning as needed.

**J) Spaces normalization**

* Merge adjacent `w:r` with identical `rPr`. On merged text, replace regex ` {2,}` → ` `, preserving `xml:space` and not touching NBSP (`\u00A0`).

---

## 9) Services & Interfaces (App.Core)

```csharp
public interface IDocumentProcessingService {
  Task<ProcessSummary> ProcessAsync(IEnumerable<string> paths, ProcessOptions opts, CancellationToken ct);
}

public interface IHyperlinkIndexService {
  HyperlinkIndex Build(WordprocessingDocument doc);
}

public interface ILookupService { // Power Automate API
  Task<IReadOnlyList<LookupResult>> ResolveAsync(IReadOnlyCollection<string> lookupIds, CancellationToken ct);
}

public interface IHyperlinkRepairService {
  int Repair(WordprocessingDocument doc, HyperlinkIndex index, IReadOnlyDictionary<string, LookupResult> map);
}

public interface IFormattingService {
  void NormalizeSpaces(WordprocessingDocument doc);
  int FixTopOfDocumentLinks(WordprocessingDocument doc);
  StyleReport EnsureStyles(WordprocessingDocument doc);
  int CenterImages(WordprocessingDocument doc);
}

public interface ISettingsStore { AppSettings Load(); void Save(AppSettings s); }
public interface ISecretStore { void Set(string name, string value); string? Get(string name); }
public interface IUpdater { Task<UpdateCheckResult> CheckAsync(); Task<bool> ApplyAsync(UpdateInfo info); }
```

* Keep **OpenXML** touching code in `App.Infrastructure` implementations only.
* `App.Services` composes these into a deterministic pipeline.

---

## 10) Data Models (App.Domain)

```csharp
public record ProcessOptions {
  bool CollapseDoubleSpaces { get; init; }
  bool FixTopOfDocLinks { get; init; }
  bool StandardizeStyles { get; init; }
  bool CenterImages { get; init; }
}

public record LookupResult(string Title, string Status, string ContentId, string DocumentId);

public record HyperlinkRef(
  string Id,                // internal key for mapping
  string? RelationshipId,   // for external links
  string? Anchor,           // for internal links
  string Target,            // current URL or anchor
  TextRange DisplayTextRange // helper to patch text reliably
);
```

`TextRange` captures paragraph/run indices to replace or append the display text without losing formatting.

---

## 11) HTTP API Client (Power Automate)

* **Settings**: `BaseUrl`, `Path`, `ApiKey`, `TimeoutSeconds`.
* **Request**: `{ "Lookup_IDs": ["...", "..."] }` (string array).
* **Auth**: No API Key necessary
* **Resilience**: Polly policy (exponential backoff, total timeout, circuit breaker).
* **Mapping**: Build a dictionary keyed by both `Document_ID` and `Content_ID` to support either lookup path.

---

## 12) Updater (GitHub Releases)

✅ **IMPLEMENTED**
* On launch: silent check for updates. Also a "Check for updates" button.
* Use Velopack: built-in source (GitHubSource) + delta packaging across releases.
* Download to temp, verify hash/signature, replace on restart.
* Display release notes in a dialog (render Markdown to HTML control).
* **Implementation**: `VelopackUpdater` class in `App.Infrastructure/Updates/`

---

## 13) Settings & Secrets

✅ **IMPLEMENTED**
* `settings.json` stored at `%AppData%/Doc_Medic/settings.json`:

```json
{
  "Api": { "BaseUrl": "https://...", "Path": "/lookup", "TimeoutSeconds": 30 },
  "Updater": { "Channel": "Stable" },
  "UI": { "Theme": "Dark", "Accent": "Blue" },
  "Processing": { "KeepBackup": true }
}
```

* **Implementation**: `SettingsStore` class in `App.Infrastructure/Storage/`
* **Secrets**: DPAPI-encrypted storage via `SecretStore` class in `App.Infrastructure/Security/`
* API key not necessary for HTTP Request to Power Automate.

---

## 14) Logging & Telemetry

* Serilog sinks: `logs/app-.log` (daily rolling). Level: Info default, Debug when enabled.
* Correlate logs per file with `FileId` and operation span IDs.
* Emit summary object after each run: counts of links inspected, eligible, repaired; style changes; whitespace replacements; image centers.

---

## 15) Error Handling & Edge Cases

* Read‑only or locked documents → skip with warning.
* Invalid or malformed URLs → record warning; do not alter.
* Field code links not convertible → leave untouched with warning.
* Tracked changes: optionally reject or accept revisions before editing; default: leave as is and edit content nodes (document as authored view).
* Documents without StylesPart → create.
* Hyperlink display text spanning multiple runs → use `TextRange` helper to recompose safely.
* Duplicated links → de‑duplicate edits by location.
* Non‑HTTP links (mailto:, file:) → ignored unless eligible per rules (they won’t be).

---

## 16) Testing Strategy

* **Regex**

  * `?docid=` extraction: unit tests covering delimiters (&, #), URL‑encoded values.
  * `CMS/TSRC` pattern: examples including `CMS-1-123456`, `TSRC-asd2121jkla-123456`.
* **Hyperlink patcher**

  * Build doc fixtures where the display text is split across runs.
  * Verify correct `(XXXXXX)` insertion logic for 6/5 digits and zero‑pad case.
* **Styles**

  * Assert styles are created/updated with expected properties.
* **Top of Document**

  * Verify bookmark creation and internal anchor updates.
* **Integration**

  * Golden file tests: input docx → transform → compare against approved output (OpenXML PowerTools `HtmlConverter` for visual diff optional).

---

## 17) Performance

* Stream read/write; avoid loading entire package into memory unnecessarily.
* Parallel processing with bounded degree and asynchronous API calls.
* Cache API results for the duration of a run to avoid repeat lookups. (only till application restart, or 1 hour, whichever happens first)

---

## 18) Security

* API Key not needed since calling Power Automate HTTP Request and it doesn't require it; never log secrets or full URLs with tokens.
* Validate all API responses; treat missing fields as non‑actionable.
* Optional code signing of binaries.

---

## 19) Build & Release

✅ **IMPLEMENTED**
* GitHub Actions (`.github/workflows/release.yml`):

  * Build matrix (Debug/Release, x64).
  * Run tests.
  * Create artifacts (installer + portable).
  * Create GitHub Release with changelog, version tag.
  * Automated Velopack packaging and deployment.
* Updater points to latest Release.

**Release Process**:
1. Create and push version tag (e.g., `git tag v1.0.0 && git push origin v1.0.0`)
2. GitHub Actions automatically builds, tests, packages, and creates release
3. Velopack handles delta updates for existing installations

---

## 20) Rollout Strategy (Style Application)

* **Conservative first pass**

  * Only standardize styles for paragraphs already labeled `Normal`, `Heading 1`, `Heading 2`.
  * Provide toggle to re‑style unlabeled paragraphs heuristically (e.g., all‑caps lines → headings).

---

## 21) Pseudocode Highlights

**Pipeline**

```csharp
foreach file in files.ParallelForEach(opts.DOP) {
  using var doc = WordprocessingDocument.Open(file, true);
  var index = hyperlinkIndex.Build(doc);
  var lookupIds = index.ExtractLookupIds();
  var results = lookup.ResolveAsync(lookupIds).Result;
  var map = results.ToLookupMap();
  var fixedCount = hyperlinkRepair.Repair(doc, index, map);
  if (opts.CollapseDoubleSpaces) formatting.NormalizeSpaces(doc);
  if (opts.FixTopOfDocLinks) formatting.FixTopOfDocumentLinks(doc);
  if (opts.StandardizeStyles) formatting.EnsureStyles(doc);
  if (opts.CenterImages) formatting.CenterImages(doc);
  SaveAtomic(doc, file);
}
```

**Repair (per hyperlink)**

```csharp
var meta = map.ResolveByDocOrContentId(href);
if (meta == null) return; // not eligible or not found
var canonical = NuxeoBase + meta.DocumentId;
if (!UrlEquals(canonical, href.Target)) UpdateTarget(href, canonical);
EnsureDisplayTextHasContentIdSuffix(href.DisplayTextRange, meta.ContentId);
```

---

## 22) Glossary

* **Document\_ID**: Value extracted after `?docid=`; used to build canonical Nuxeo URL.
* **Content\_ID**: Pattern `(CMS|TSRC)-…-NNNNNN`; last 6 digits used in display text.
* **Eligible hyperlink**: URL containing `?docid=` or a `CMS/TSRC` Content\_ID.

---

## 23) Open Questions (for future refinement)

* Do we need a dry‑run mode to preview changes without writing files? No
* Should we provide a per‑rule toggle for style application (e.g., only fix font sizes)? Yes
* Any max parallelism or rate limits for the Power Automate API? None to worry about
* Should we keep a cache of `Lookup_IDs → LookupResult` across app runs? Only till application restart or for 1 hour, whichever comes first.
* Preferred installer type? Velopack 

---

## 24) Next Steps for Production

**Ready for deployment - complete these final steps:**

1. **Repository Setup**:
   - Update GitHub repository URL in `VelopackUpdater` (replace `owner/doc-medic` with actual repo)
   - Verify GitHub repository has Actions enabled
   
2. **Production Configuration**:
   - Configure real Power Automate API endpoint in default settings
   - Add application icon and splash screen assets
   - Test the build pipeline with version tags (`v1.0.0`)
   
3. **Quality Assurance**:
   - Run approval tests and update golden files as needed
   - Test full document processing pipeline with real .docx files
   - Verify auto-update functionality works correctly
   
4. **Optional Enhancements**:
   - Set up code signing for binaries (recommended for production)
   - Configure GitHub repository secrets if needed for signed releases

**Current Status**: ✅ All core functionality implemented and ready for production deployment.

---

*End of implementation brief.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ItMeDiaTech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
