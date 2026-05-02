## latex

> Instructions for AI agents working with this repository.

# AGENTS.md

Instructions for AI agents working with this repository.

## Repository Overview

**LaTeX Toolkit** - LaTeX templates, build scripts, and PDF tools for producing professional documents in secure, compliance-aware environments.

### Philosophy

This toolkit follows the [SpeakUp project](https://github.com/brucedombrowski/SpeakUp) philosophy:
- **Structured systems** - All work performed in version-controlled, reproducible workflows
- **Traceability** - Source-to-output chains via digital signatures with embedded hashes
- **Automation** - Cross-platform build scripts, AI agent workflows, PDF manipulation
- **Auditability** - Formal decision documentation, compliance markings, signing infrastructure

### Capabilities

| Capability | Component | Status |
|------------|-----------|--------|
| **Decision Memorandums** | Documentation-Generation/DecisionMemorandum/ | Production |
| **Decision Documents** | Documentation-Generation/DecisionDocument/ | Production |
| **Slide Decks** | Documentation-Generation/SlideDecks/ | Planned |
| **Meeting Agendas** | Documentation-Generation/MeetingAgenda/ | Production |
| **CUI Cover Sheets** | Compliance-Marking/CUI/ | Production |
| **Requirements/V&V** | Compliance-Marking/Requirements/ | Production |
| **Export Markings** | Compliance-Marking/Export/ | Planned |
| **PDF Merging** | .scripts/merge-pdf.* | Production |
| **Digital Signatures** | Documentation-Generation/DecisionDocument/ | Production |
| **Software Attestations** | Documentation-Generation/Attestations/ | Production |

### Key Features

**Document Generation:**
- Decision Memorandums (single-page) and Program Decision Documents (multi-page)
- Beamer presentation slide decks
- Structured meeting agendas with timed items
- Software attestations documenting versions, checksums, and download URLs

**Compliance & Security:**
- SF901 CUI cover sheets (32 CFR Part 2002 compliant)
- Export control markings (ITAR/EAR) - planned
- Digital signatures with PIV/CAC smart card support
- Self-signed certificates with source hash traceability

**PDF Operations:**
- Merge multiple PDFs with user-defined ordering
- LaTeX-based (pdfpages) - no external dependencies
- Works on airgapped systems

**Cross-Platform:**
- Windows: `.bat` (double-click), `.ps1` (PowerShell)
- macOS/Linux: `.sh` (bash)
- All tools work offline after LaTeX installation

## Target Environment

**Primary:** Airgapped Windows 11 with security hardening
- CIS Windows 11 Enterprise baseline
- DISA STIG Windows 11 baseline
- Microsoft Security Baseline

**Secondary:** macOS/Linux for development

## Components

| Component | Description | AGENTS.md |
|-----------|-------------|-----------|
| [.scripts/](.scripts/) | Build tools, release scripts, PDF merge utilities | — |
| [.assets/](.assets/) | Shared images and logos | — |
| [.bin/](.bin/) | Executable binaries (PdfSigner.exe) | — |
| [.dist/](.dist/) | Build output (tracked for examples) | — |
| [Documentation-Generation/](Documentation-Generation/) | Document templates (decisions, slides, agendas, attestations) | [Documentation-Generation/AGENTS.md](Documentation-Generation/AGENTS.md) |
| [Decisions/](Decisions/) | Formal Decision Memorandums archive | — |
| [Attestations/](Attestations/) | Generated software attestation PDFs | — |
| [Analysis/](Analysis/) | Technical assessments, DRY analysis, mitigation plans | — |
| [Compliance-Marking/](Compliance-Marking/) | CUI cover pages, export markings, security compliance | [Compliance-Marking/AGENTS.md](Compliance-Marking/AGENTS.md) |

## Documentation-Generation

Document templates following the [SpeakUp project](https://github.com/brucedombrowski/SpeakUp) workflow.

### Workflow Model

```
Mobile Ideation (AI agent, text/voice)
    ↓
IDE-Integrated Execution (LaTeX source → PDF)
    ↓
Verification (build, compliance checks)
    ↓
Distributable Output (PDF)
```

### Documentation Structure

All components follow a consistent `templates/` and `examples/` structure:

```
Documentation-Generation/
├── AGENTS.md                 # Documentation-Generation instructions
│
├── DecisionMemorandum/       # Single-page decision memos
│   ├── templates/
│   │   └── decision_memo.tex
│   ├── examples/
│   └── sign.* -> ../DecisionDocument/sign.*  # Symlinks to signing tools
│
├── DecisionDocument/         # Multi-page program decisions
│   ├── AGENTS.md             # Signing-specific instructions
│   ├── README.md
│   ├── templates/
│   │   └── decision_document.tex
│   ├── examples/
│   ├── sign.sh / sign.ps1 / sign.bat
│   └── PdfSigner.exe         # From github.com/brucedombrowski/PDFSigner
│
├── SlideDecks/               # Presentation slide decks (Beamer)
│   ├── templates/
│   │   └── standard_brief.tex
│   └── examples/
│
├── MeetingAgenda/            # Meeting agenda documents
│   ├── templates/
│   │   └── meeting_agenda.tex
│   └── examples/
│       ├── project_kickoff.tex
│       └── requirements_review.tex
│
├── TechnicalReport/          # Technical reports and assessments
│   ├── templates/
│   │   └── technical_report_base.tex
│   └── examples/
│       └── DRY_Assessment_Report.tex
│
└── Attestations/             # Software attestation documents
    ├── templates/
    │   └── attestation-template.tex  # Shared template (like SF901-template)
    └── examples/
        └── software_attestation.tex  # PdfSigner version attestation
```

### DecisionMemorandum - Brief Decisions

**Purpose:** Single-page formal records of program decisions

Template: `templates/decision_memo.tex`

### DecisionDocument - Comprehensive Decisions

**Purpose:** Multi-page program decision documentation with full traceability

**AGENTS.md:** [Documentation-Generation/DecisionDocument/AGENTS.md](Documentation-Generation/DecisionDocument/AGENTS.md) - signing workflows

Template: `templates/decision_document.tex`

**Note:** Signing scripts live here; PdfSigner.exe from [PDFSigner repo](https://github.com/brucedombrowski/PDFSigner).

### SlideDecks - Beamer Presentations

**Purpose:** Formal briefings, status updates, technical presentations

**Conventions:**
1. **Document class:** Use `beamer` class
2. **Theme:** Prefer minimal themes (e.g., `default`, `metropolis`)
3. **Frames:** One concept per frame

Example minimal slide deck:

```latex
\documentclass{beamer}
\usetheme{default}
\title{Brief Title}
\author{Author Name}
\date{\today}

\begin{document}
\frame{\titlepage}

\begin{frame}{Agenda}
\begin{itemize}
    \item Topic 1
    \item Topic 2
    \item Topic 3
\end{itemize}
\end{frame}

\begin{frame}{Key Point}
Content goes here.
\end{frame}
\end{document}
```

### MeetingAgenda - Structured Agendas

**Purpose:** Meeting preparation, action item tracking, time allocation

**Conventions:**
1. **Document class:** Use `article` class with custom formatting
2. **Structure:** Header with meeting metadata, timed agenda items, action items section
3. **Tables:** Use `tabularx` for flexible column widths

Example minimal agenda:

```latex
\documentclass[11pt]{article}
\usepackage[margin=1in]{geometry}
\usepackage{tabularx}
\usepackage{booktabs}

\begin{document}

\begin{center}
\Large\textbf{Meeting Agenda}\\[0.5em]
\normalsize Project Status Review\\
January 21, 2026 | 10:00 AM | Conference Room A
\end{center}

\vspace{1em}

\begin{tabularx}{\textwidth}{@{}lXr@{}}
\toprule
\textbf{Time} & \textbf{Topic} & \textbf{Lead} \\
\midrule
10:00 & Welcome and introductions & Chair \\
10:05 & Review previous action items & All \\
10:15 & Status update: Component A & J. Smith \\
10:30 & Status update: Component B & M. Jones \\
10:45 & Open discussion & All \\
10:55 & Next steps and action items & Chair \\
\bottomrule
\end{tabularx}

\vspace{1em}
\textbf{Attendees:} Name 1, Name 2, Name 3

\end{document}
```

### TechnicalReport - Technical Reports and Assessments

**Purpose:** Formal technical reports for assessments, analysis results, and project documentation.

**Structure:**
- `templates/technical_report_base.tex` - Base template with title page, TOC, headers/footers
- `examples/` - Report implementations using the template

**Required Variables:**
```latex
\newcommand{\ReportTitle}{Report Title}
\newcommand{\ReportSubtitle}{Subtitle}
\newcommand{\ReportID}{ASSESS-2026-001}
\newcommand{\ReportDate}{January 21, 2026}
\newcommand{\ReportVersion}{1.0}
\newcommand{\ReportAuthor}{Author Name}
\newcommand{\HeaderLeft}{Header Text}
\newcommand{\FooterLeft}{Footer Text}
```

**Content Commands:**
- `\ReportContent` - Main body of the report (required)
- `\TitlePageExtras` - Additional title page content (optional)
- `\ReportAppendix` - Appendix content (optional)

**Example:** See `examples/DRY_Assessment_Report.tex` for a complete implementation.

### Attestations - Software Version Documentation

**Purpose:** Document software versions, SHA256 checksums, and download URLs for external binaries used by the toolkit. Provides traceability and verification capability for security-conscious environments.

**Structure:**
- `Documentation-Generation/Attestations/templates/` - Shared attestation template
- `Documentation-Generation/Attestations/examples/` - Attestation wrappers (like `software_attestation.tex`)
- `Attestations/` - Generated attestation PDFs (root level, like `Decisions/`)
- `dist/attestations/` - Attestations included in release builds

**Generation:**
```bash
# Generate attestation manually
./scripts/generate-attestation.sh

# Attestations are also generated automatically during release builds
./.scripts/release.sh
```

**DRY Pattern:**
Attestations follow the same DRY pattern as SF901 cover sheets:
1. `attestation-template.tex` - Shared template with header/footer, colors, document structure
2. Thin wrappers (e.g., `software_attestation.tex`) define variables and `\input` the template
3. Generation scripts perform sed substitution on placeholders

**Output:**
- `Attestations/software-attestation-YYYYMMDD.pdf` - Dated attestation
- `Attestations/software-attestation-latest.pdf` - Symlink to most recent
- `dist/attestations/` - Same files for release distribution

## DRY (Don't Repeat Yourself) Patterns

This repository uses template wrapper patterns to minimize code duplication and ensure consistency across documents.

### Template Wrapper Pattern

The template wrapper pattern separates shared structure from document-specific content:

```
Template (shared)                Wrapper (document-specific)
┌─────────────────────────┐     ┌─────────────────────────┐
│ \documentclass{...}     │     │ \newcommand{\var1}{...} │
│ \usepackage{...}        │     │ \newcommand{\var2}{...} │
│ Header/footer config    │ <── │ \newcommand{\content}   │
│ Common styling          │     │   {...}                 │
│ \begin{document}        │     │ \input{_template.tex}   │
│ \content                │     └─────────────────────────┘
│ \end{document}          │
└─────────────────────────┘
```

### Pattern Implementations

| Component | Template | Wrapper Example |
|-----------|----------|-----------------|
| SF901 Cover Sheets | `SF901-template.tex` | `Examples/SF901_BASIC.tex` |
| Decision Memoranda | `Decisions/_template.tex` | `DM-2026-001_sf901_decision.tex` |
| Meeting Agendas | `templates/meeting_agenda_base.tex` | `examples/project_kickoff.tex` |
| Attestations | `templates/attestation-template.tex` | `examples/software_attestation.tex` |

### Creating New Documents Using Templates

**Decision Memorandum:**
```latex
% DM-2026-004_example.tex
\newcommand{\UniqueID}{DM-2026-004}
\newcommand{\DocumentDate}{January 21, 2026}
\newcommand{\AuthorName}{Your Name}
% ... other variables ...
\newcommand{\dmContent}{%
\begin{enumerate}
\dmsection{Purpose}
Your purpose text here.
% ... more sections ...
\end{enumerate}
}
\input{_template.tex}
```

**Meeting Agenda:**
```latex
% new_meeting.tex
\newcommand{\MeetingTitle}{Technical Review}
\newcommand{\MeetingSubtitle}{Phase 2 Checkpoint}
\newcommand{\MeetingDate}{January 21, 2026}
% ... other variables ...
\newcommand{\MeetingAgendaItems}{%
    0:00 & Welcome & Facilitator \\
    0:05 & Status updates & All \\
    % ... more items ...
}
% ... other content commands ...
\input{../templates/meeting_agenda_base.tex}
```

### Shell Script Common Utilities

Build scripts share common utilities via `.scripts/lib/common.sh`:

```bash
# Source common utilities
source "$SCRIPT_DIR/lib/common.sh"

# Use shared functions
COMPILER=$(determine_compiler "$tex_file")  # Returns "pdflatex" or "xelatex"
cleanup_aux_files "."                        # Removes .aux, .log, etc.
print_success "Build complete"               # Green output
print_error "Build failed"                   # Red output
print_warning "Optional step skipped"        # Yellow output
require_command pdflatex "Install TeX Live"  # Check command exists
```

### Benefits

1. **Consistency:** All documents of the same type share formatting
2. **Maintainability:** Fix bugs or update styles in one place
3. **Simplicity:** New documents are small, focused on content
4. **Reduced Errors:** Less code = fewer opportunities for mistakes

### Assessment Reference

See `Analysis/DRY-Assessment-2026-01-21.md` for the complete DRY compliance analysis and `Analysis/DRY-Mitigation-Plan.md` for implementation details.

## Repository-Wide Conventions

### LaTeX Style

1. **Section separators:** Use comment blocks
   ```latex
   % ============================================================================
   % SECTION NAME
   % ============================================================================
   ```

2. **Document variables:** Define as `\newcommand` at top of file under `DOCUMENT VARIABLES - EDIT THESE`

3. **Indentation:** 4 spaces for nested environments

4. **Placeholders:** Use plain text, NOT brackets `[like this]` (causes LaTeX issues)

### Date Format Placeholders

| Placeholder | Meaning | Example |
|-------------|---------|---------|
| `YYYY` | 4-digit year | 2026 |
| `MM` | 2-digit month | 01 |
| `DD` | 2-digit day | 15 |
| `MMMM` | Full month name | January |

### Build Scripts

Centralized build tools in `.scripts/`:

| Script | Purpose |
|--------|---------|
| `.scripts/build-tex.sh` | Build any single .tex file |
| `.scripts/release.sh` | Build all documents to `.dist/` |
| `.scripts/generate-attestation.sh` | Generate software attestation PDF |
| `.scripts/merge-pdf.sh` | Merge multiple PDFs (interactive) |
| `.scripts/merge-pdf.ps1` | Merge multiple PDFs (Windows PowerShell) |
| `.scripts/sign-pdf.sh` | Sign PDFs with digital certificates |
| `.scripts/sign-pdf.ps1` | Sign PDFs (Windows PowerShell) |
| `.scripts/sign-pdf.bat` | Sign PDFs (Windows batch) |
| `.scripts/lib/common.sh` | Shared utilities (colors, compiler detection) |
| `.scripts/lib/external-deps.sh` | External dependency management |

**Usage:**

```bash
# Build a single document
./.scripts/build-tex.sh path/to/document.tex

# Build with Word output (requires pandoc)
./.scripts/build-tex.sh path/to/document.tex --docx

# Build all documents for release
./.scripts/release.sh

# Clean .dist/ directory
./.scripts/release.sh --clean

# Merge PDFs (interactive - run from folder containing PDFs)
./.scripts/merge-pdf.sh

# Download missing external dependencies
source .scripts/lib/common.sh && source .scripts/lib/external-deps.sh
download_dependency "PdfSigner"
```

### External Dependencies

External binaries are configured in `.config/external-deps.json` and downloaded from GitHub releases:

| Dependency | Source | Install Path |
|------------|--------|--------------|
| PdfSigner.exe | [brucedombrowski/PDFSigner](https://github.com/brucedombrowski/PDFSigner/releases) | `.bin/PdfSigner.exe` |

**Automatic checks:** `release.sh` checks for missing/outdated dependencies and reports status.

**Manual download:**
```bash
source .scripts/lib/common.sh && source .scripts/lib/external-deps.sh
download_dependency "PdfSigner"
```

**Output:**
- Single builds: PDF in same directory as source
- Release builds: All PDFs in `.dist/` (organized by type)
- Merge: Creates `merged.pdf` in current directory

### Digital Signatures

Use the signing infrastructure in `.scripts/`:
- `sign-pdf.sh` / `sign-pdf.bat` / `sign-pdf.ps1`
- `.bin/PdfSigner.exe` for Windows (download from [PDFSigner releases](https://github.com/brucedombrowski/PDFSigner/releases))
- PIV/CAC smart card support
- Software certificate support

Symlinks exist in DecisionDocument/ and DecisionMemorandum/ for convenience.

### AI Agent Build Workflow

When building documents as an AI agent:

```bash
# Build a single document
./.scripts/build-tex.sh Documentation-Generation/DecisionDocument/decision_document.tex

# Build all documents for release
./.scripts/release.sh

# Optionally sign with source traceability
cd Documentation-Generation/DecisionDocument/
TEX_HASH=$(shasum -a 256 decision_document.tex | cut -c1-12)
./sign.sh decision_document.pdf
# ... (see Documentation-Generation/DecisionDocument/AGENTS.md for full workflow)
```

### Decision Documentation

All significant design decisions must be documented as formal Decision Memorandums:
- Format: LaTeX-generated PDF
- Location: `Decisions/`
- ID format: `DM-YYYY-NNN`
- See [Compliance-Marking/AGENTS.md](Compliance-Marking/AGENTS.md) for requirements and the Decision Memorandum Index

## Dependencies

### Required LaTeX Packages

Core packages used across components:
- `geometry`, `graphicx`, `fancyhdr`, `lastpage`
- `titlesec`, `enumitem`, `booktabs`, `longtable`
- `xcolor`, `hyperref`, `datetime2`, `tabularx`
- `pdfpages` (PDF merging)
- `tikz` (CUI layout)
- `beamer` (presentations)

### Installation

**Windows (MiKTeX):**
- Download from https://miktex.org/download
- 250MB basic installer, packages install on-demand

**macOS:**
```bash
brew install --cask mactex
```

**Linux:**
```bash
# Debian/Ubuntu
sudo apt install texlive-latex-base texlive-latex-extra texlive-fonts-recommended

# Fedora
sudo dnf install texlive-scheme-medium
```

## File Structure

```
LaTeX/
├── AGENTS.md                 # This file - repository-wide instructions
├── README.md                 # User documentation
├── .gitignore                # Git ignore rules
│
├── .config/                  # Configuration files
│   └── external-deps.json    # External dependency definitions (GitHub sources)
│
├── .scripts/                 # Centralized build and utility scripts
│   ├── lib/
│   │   ├── common.sh         # Shared utilities (colors, compiler detection)
│   │   └── external-deps.sh  # External dependency management
│   ├── build-tex.sh          # Build any single .tex file
│   ├── release.sh            # Build all documents to .dist/
│   ├── generate-attestation.sh # Generate software attestation
│   ├── merge-pdf.sh          # PDF merge utility (macOS/Linux)
│   ├── merge-pdf.ps1         # PDF merge utility (Windows)
│   ├── sign-pdf.sh           # PDF signing (macOS/Linux)
│   ├── sign-pdf.ps1          # PDF signing (Windows PowerShell)
│   └── sign-pdf.bat          # PDF signing (Windows batch)
│
├── .bin/                     # Executable binaries
│   └── PdfSigner.exe         # PDF signing tool for Windows
│
├── .assets/                  # Shared images, logos (symlinked from subfolders)
│   ├── logo.png
│   └── logo.svg
│
├── .dist/                    # Build output (tracked for examples)
│   ├── decisions/
│   ├── meetings/
│   ├── compliance/
│   └── attestations/
│
├── Documentation-Generation/ # All document generation
│   ├── DecisionMemorandum/   # Single-page decision memos
│   ├── DecisionDocument/     # Multi-page decisions (with signing tools)
│   ├── SlideDecks/           # Beamer presentations
│   ├── MeetingAgenda/        # Meeting agendas
│   └── Attestations/         # Attestation templates and examples
│
├── Decisions/                # Formal Decision Memorandums (cross-cutting)
│
├── Attestations/             # Generated attestation PDFs
│
├── Analysis/                 # Technical assessments, DRY analysis, mitigation plans
│
└── Compliance-Marking/       # Compliance templates
    ├── AGENTS.md
    ├── CUI/                  # SF901 cover sheets
    ├── Requirements/         # Formal REQ docs (JSON → PDF)
    └── Verifications/        # VER document index (docs live in impl repos)
```

## Common Tasks

### Creating a New Component

1. Create directory with descriptive name
2. Add `AGENTS.md` with component-specific instructions
3. Add `README.md` with user documentation
4. Add build scripts following existing patterns
5. Update this root `AGENTS.md` with component entry

### Creating a New Decision Memorandum

1. Copy template from `Documentation-Generation/DecisionMemorandum/templates/decision_memo.tex`
2. Edit document variables at top of `.tex` file
3. Build: `./.scripts/build-tex.sh path/to/your_memo.tex`
4. Sign with `./sign.sh` for distribution (from DecisionMemorandum/)
5. Move final PDF to `Decisions/`

### Creating a New Decision Document

1. Copy template from `Documentation-Generation/DecisionDocument/templates/decision_document.tex`
2. Edit document variables at top of `.tex` file
3. Build: `./.scripts/build-tex.sh path/to/your_document.tex`
4. Sign with `./sign.sh` for distribution (from DecisionDocument/)
5. Move final PDF to `Decisions/`

### Creating a New Slide Deck

1. Copy template from `Documentation-Generation/SlideDecks/templates/`
2. Edit content in `.tex` file
3. Build: `./.scripts/build-tex.sh path/to/your_slides.tex`
4. Review generated PDF
5. Optionally sign for distribution

### Creating a New Meeting Agenda

1. Copy template from `Documentation-Generation/MeetingAgenda/templates/meeting_agenda.tex`
2. Edit meeting metadata (date, time, location, attendees)
3. Fill in timed agenda items
4. Build: `./.scripts/build-tex.sh path/to/your_agenda.tex`
5. Review generated PDF

### Building All Documents

```bash
# Build everything and output to .dist/
./.scripts/release.sh

# Clean the .dist/ directory
./.scripts/release.sh --clean
```

### Testing Changes

After modifying any component:
1. Build with `./.scripts/build-tex.sh path/to/file.tex`
2. Verify PDF generates without errors
3. Check formatting and layout
4. Verify cross-references and page numbers
5. Run full release build: `./.scripts/release.sh`
6. Test on target platform (Windows if possible)

---
> Source: [brucedombrowski/LaTeX](https://github.com/brucedombrowski/LaTeX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
