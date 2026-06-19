## physics-lab-report-skill

> Use when generating, auditing, or revising UESTC Physics Experiments I/II lab reports, prelab预习报告, postlab实验报告, or related LaTeX templates/skills for 大物实验报告.


# UESTC Physics Lab Report Generator

## Overview

Generate or audit prelab (预习报告) and postlab (实验报告) for UESTC Physics Experiments I/II using XeLaTeX. Two distinct templates share core infrastructure: cover page via `\includepdf`, score boxes, diagonal watermark, and real-data tables. The templates are designed for any UESTC student, but the rules must always be checked against the current school template, lab manual, scanned data, and reference reports.

## Work Modes

This skill supports three modes of operation:

### First-Time Setup

Run once at the start of the semester. The student provides stable information that is reused for every report:

1. Upload prelab and postlab cover template PDFs
2. Confirm student profile (name, student ID, email, college, major, class, instructor, TA) — see `config/student_profile.example.yaml`
3. Set PDF naming format for submission — see `config/naming.example.yaml`
4. Confirm course/textbook version

After setup, config files and templates are reused automatically. To change stable info later, use Update Mode.

### Regular Generation

What the student does for each report:

1. Choose **prelab** or **postlab**
2. Provide the **full experiment title** (e.g. "Polarization of Light")
3. Provide the **lab number** (for cover page, filename, and submission order only)
4. **Manually input experimental data** (recommended; see Data Input below)
5. Optionally upload a teacher-specific formula template or data-processing sheet
6. Optionally upload the completed scanned data sheet for the appendix
7. Skill generates `.tex` source and compiles to final PDF

### Update Mode

Used when stable information changes:

- Update cover template PDFs
- Update student profile (new instructor, TA, etc.)
- Update PDF naming rules
- Update course/textbook version
- Update experiment formula templates or reference files

## Report Types at a Glance

| | Prelab | Postlab |
|---|---|---|
| Cover PDF | `01-Template for Prelab work-2026*.pdf` | `02-Template for lab report-2026*.pdf` |
| Cover title | "Prelab Report" | "Lab Report" |
| Page style | `empty` | `fancy`, centered page numbers |
| Watermark | "Physics Lab 2026" (singular) | "Physics Labs 2026" (plural) |
| Sections | One: "Answers to Questions (20 points)" with overlay score box | Four: Abstract (5), Calculations/Results/Comments (15), Conclusions (10), Questions (10) |
| Multi-page | Repeat background + foreground per page; continue `enumi` counter | Natural text flow |
| Answer format | `(\arabic*)` enumeration, no left margin | `\item \textbf{Title}` with full paragraphs |
| Header on page 2+ | Absolute-position overlay with student info + score box | None (info is on cover only) |

## Compilation

**Must use XeLaTeX** (not pdfLaTeX). Two passes required. Work in a single directory containing all `.tex`, cover PDFs, data PDFs, and figure PDFs.

Use the provided build script for a one-command PDF-first workflow:

```bash
bash scripts/build_report.sh file.tex "final_submission_name.pdf"
```

Or compile manually:

```bash
xelatex -interaction=nonstopmode file.tex
xelatex -interaction=nonstopmode file.tex
```

The build script also checks XeLaTeX availability, cleans auxiliary files, and renames the output PDF. Overleaf is a fallback option if XeLaTeX is not installed locally.

## Personal Info and Inputs

After first-time setup, student identity comes from `config/student_profile.yaml`. The skill loads it automatically; the student does not need to re-enter it for each report.

For one-off use without config files, prelab pages after the cover need overlay variables:

```latex
\newcommand{\StudentNumber}{STUDENTID}
\newcommand{\StudentEmail}{student@example.com}
\newcommand{\ReportDate}{2026.X.X}        % dots format for overlay
\newcommand{\CoverPDF}{01-Template for Prelab work-2026.pdf}
```

Postlab usually gets student identity from the filled cover PDF itself. Its working variables are normally:

```latex
\newcommand{\CoverPDF}{02-Template for lab report-2026.pdf}
\newcommand{\DataPDF}{scanned-data.pdf}
% optional figure PDFs generated from real data
```

The cover page date (filled in the template PDF itself) may use `/` format (e.g., `2026/5/12`). The overlay date on prelab page 2 uses `.` format (e.g., `2026.5.12`). These can differ by a day (submission date vs. writing date).

## Core Principles

### 1. Real Data Only

All numbers must trace to scanned data sheets, instrument specs, lab-manual values, or textbook values. Extract text with `pdftotext`/Python when possible. If a PDF is scanned and text extraction is empty, render pages with `pdftoppm` and use OCR or manual visual transcription. Never fabricate data; when data is unclear, say so or transcribe cautiously from the scan.

### 2. Cover via includepdf

```latex
\usepackage{pdfpages}
\includepdf[pages=1,pagecommand={\thispagestyle{empty}}]{\CoverPDF}
\setcounter{page}{1}
```

Never draw the cover in LaTeX.

### 3. Score Boxes

Postlab: `\labsection{Title}{(points)}` renders a left score box + heading.
Prelab: score box drawn in foreground overlay with `\rule` primitives.

### 4. Data Table Format

```latex
\datatable{DATA TABLE X-X}{(Purpose: ...)}
\begin{table}[H]
\centering
\small  % or \footnotesize or \scriptsize
\renewcommand{\arraystretch}{1.15}
\setlength{\tabcolsep}{3pt}
{\tablefont
\begin{tabular}{|...|}
\hline
... real data only ...
\hline
\end{tabular}
}
\caption*{Unit: ... $\Delta_{\mathrm{instr.}}=...$, $\Delta_{\mathrm{read}}=...$}
\end{table}
```

### 5. Uncertainty Propagation

Quantitative measurement postlabs must show the relevant uncertainty path: Type A when repeated measurements exist, Type B for instrument/reading error (rectangular distribution: divide by √3 when the course formula uses it), combined propagation, and final result as `\boxed{value ± uncertainty unit}`.

Verification or qualitative/graph-based postlabs (e.g. oscilloscope Lissajous patterns, polarization/Malus-law plots) may not have one final Type-A/Type-B result. For those, include the measured table values, fit/deviation where applicable, instrument resolution or reading uncertainty, dominant error sources, and a clear comparison with theory.

**Formula sources.** Formulas may come from three places, handled differently:

1. **Fixed textbook formulas** — Use the internal experiment knowledge base. No upload needed.
2. **Teacher-specific formula template** — User uploads it for this experiment (e.g. a special data-processing DOCX).
3. **Previously uploaded template** — Reused automatically if saved for the same experiment. Do not ask the student to upload the same file again.

### 6. Appendix with Scanned Originals

```latex
\clearpage
\appendixhead
\begin{figure}[H]
\centering
\includegraphics[page=1,width=0.90\textwidth]{\DataPDF}
\caption*{Scanned data sheet page 1}
\end{figure}
```

Multi-page: repeat `figure[H]` per page. Some reports annotate with `Scanned data sheet: filename.pdf`.

## When Materials Are Missing

The skill must never fabricate data, formulas, or questions. When materials are incomplete:

| Missing material | Behavior |
|---|---|
| Cover template PDF | Generate LaTeX source draft only; ask user to upload the official template |
| Raw experimental data | Do **not** generate a final report. Generate only a structure skeleton or a blank data-entry table for the student to fill in |
| Formula / data-processing template | Use textbook formulas from the internal knowledge base if available; warn that teacher-specific requirements may differ |
| Postlab / prelab questions | Do **not** invent questions. Leave numbered placeholders or ask the user to provide them |
| Scanned data is unclear | Extract a draft table with caveats; require user confirmation before any calculation |
| Teacher signature required | Do **not** auto-place signatures. See Teacher Signatures and Data Sheets below |

## Audit Workflow

Use this when checking whether a report or this skill matches the real course materials:

1. Identify the submitted file label (`labN-pre/post`) and the actual experiment number/title. Do not assume they are the same sequence.
2. Read the current school PDF/DOCX template first: section titles, point values, cover fields, page size, score boxes, and appendix expectations override older examples.
3. Read the lab-manual/data-table template for that experiment: required data tables, formulas, uncertainty steps, and postlab/prelab questions.
4. Read the student's finished PDF and TeX source: check structure, data traceability, formulas, units, significant figures, final result, questions, appendix scans, watermark, and cover.
5. Read senior/reference reports only as style and completeness examples. Older point values or section names are not authoritative if the current template differs.
6. Cross-check every reported number against scanned data sheets, generated figures, or textbook/lab-manual values.
7. Compile the TeX in a clean temporary directory with all referenced assets. Report missing dependencies, LaTeX errors, or template rules that cannot reproduce the PDF.

**The experiment title is the primary key**, not the lab number. Lab order may change every year (as seen in the mapping below). When generating a report, always identify the experiment by its full title first; the lab number is only used for the cover page, output filename, and submission order. If lab number and experiment title appear inconsistent, warn the user and treat the title as more reliable.

Known local label mapping from the completed reports:

| Submission label | Actual course experiment |
|---|---|
| lab1 | Experiment 1, Measurement of Resistance by Ammeter-Voltmeter Method |
| lab2 | Experiment 3, The Oscilloscope |
| lab3 | Experiment 6, Newton's Rings |
| lab4 | Experiment 4, Young's Modulus of Wire by Elongating |
| lab5 | Experiment 7, Polarized Light |

## Font Strategies

Two approaches evolved across reports. Both require XeLaTeX.

### Strategy A: All Times New Roman (clean, uniform)

Used in lab5 (polarized light) postlab. Best for most reports.

```latex
\usepackage{fontspec}
\setmainfont{Times New Roman}
\setsansfont{Times New Roman}
\setmonofont{Courier New}
\usepackage{newtxmath}
\newfontfamily\tablefont{Times New Roman}
\newfontfamily\titlefont{Times New Roman}
\newfontfamily\watermarkfont{Times New Roman}
```

### Strategy B: Calibri tables (matches instructor template exactly)

Used in lab4 (Young's modulus) postlab. Tables match the original Word template font. On macOS, fall back to Arial since Calibri may not be installed.

```latex
\IfFontExistsTF{Calibri}{\newfontfamily\tablefont{Calibri}}{\newfontfamily\tablefont{Arial}}
\IfFontExistsTF{Calibri Light}{\newfontfamily\watermarkfont{Calibri Light}}{\newfontfamily\watermarkfont{Arial}}
```

### Prelab Fonts

```latex
\newfontfamily\calibrifont{Arial}      % Calibri fallback for macOS
\newfontfamily\calibrilight{Arial}
```

For prelab header info, font size evolved from `\fontsize{10pt}{10pt}` to `\fontsize{8.5pt}{8.5pt}` for better fit.

## Spacing Settings

| Setting | Standard (lab1, lab5) | Compact (lab4, long reports) |
|---|---|---|
| `\setstretch` | 1.04 | 1.02 |
| `\parskip` | 1pt | 0.5pt |
| Display skips | 5pt/4pt (above/below) | 4pt/3pt |
| `\captionsetup{skip}` | 4pt | 2pt |
| `\labsection` vspace before | 0.5em | 0.35em |
| `\labsection` vspace after | 0.55em | 0.4em |
| Extra float spacing | — | `\textfloatsep=6pt`, `\floatsep=5pt`, `\intextsep=5pt` |

Standard geometry for all reports:
```latex
\usepackage[a4paper,top=1in,bottom=1in,left=1.25in,right=1.25in,headsep=10pt]{geometry}
\setlength{\parindent}{2em}
```

## Prelab Template

See `templates/prelab_template.tex`. Page structure:

1. **Page 1**: Cover PDF inserted via `\includepdf`
2. **Page 2+**: Background watermark + foreground overlay (student info, score box, section title, page number)

### Two Structural Approaches

**Approach A: `\PrelabAnswerBody` command** — All answers in one `\newcommand`. Best for 1-2 page prelabs. Used in lab1, lab2, lab3, lab5.

**Approach B: `{prelabanswers}` environment** — Answers in a reusable environment with overlays split into `\PrelabPageBackground`, `\PrelabPageForeground{page#}`, `\PrelabTitleForeground`. Multi-page via `\clearpage` + `\setcounter{enumi}{N}`. Used in lab4.

### Prelab Header Evolution

Header line on page 2 evolved across 5 labs:

1. **lab1**: cramped, typo `edu,cn` instead of `edu.cn`, no spaces between fields
2. **lab2-4**: cramped but typo fixed, still no spaces between fields
3. **lab5** (recommended): spaces added between fields for readability: `UESTCStudentNumber: XXXXX Email: XXX@xxx Date:2026.X.X`

The current template uses the clean lab5 spacing.

### Overlay Coordinates (absolute, unit = 1pt)

| Element | Position |
|---|---|
| "UESTCStudentNumber:" label | (75.864, 785.568) |
| Student number value | (~190, 785.568) |
| "Email:" label | (~270, 785.568) |
| Email value | (~302, 784.0) |
| "Date:" label | (~450, 785.568) |
| Separator rule | (74.34, 783.585), width 441.766pt |
| Score box | (99.984, 756.46) to (135.86, 724.18) |
| Section title "Answers to Questions" | (149.8, 723.0) |
| "(20 points)" | (~305, 723.45) |
| Footer page number | (294.4, 60.2) |
| First answer page vspace | 2.15cm |
| Continuation page vspace | 0.45cm |

## Postlab Template

See `templates/postlab_template.tex`. Four mandatory sections in fixed order:

### Section 1: Abstract (5 points, ~100 words)

Concise summary: what was measured, method, key results with final values and uncertainties. Bold key physics terms with `\textbf{}`.

### Section 2: Calculations, Results, and Comments (15 points)

The section title is usually `Calculations, Results, and Comments` but lab3 used the shorter `Calculations and Results`. Either is acceptable.

Structure pattern (varies by lab):

**Pattern A** (lab2, lab3): Data tables first → then calculations
**Pattern B** (lab4, lab5): Sample calculations first → then data tables → then comments/error analysis

Sample calculations should:
- Box key formulas with `\boxed{}`
- Show one complete worked example per formula
- Include substitution of real measured values
- State the final propagated uncertainty when the experiment requires a final uncertainty result

Data tables:
- Start with `\small`, drop to `\footnotesize` or `\scriptsize` for wide tables
- Reduce `\tabcolsep` (to 2-3pt) before resorting to smaller font
- Wide tables (9+ columns) need `\scriptsize` + minimal `\tabcolsep`

Figures (when applicable):
- Generate from real data with Python/matplotlib, export PDF for vector quality
- Width: 0.88-1.03 `\textwidth`
- Caption should describe both what is plotted and the key finding

Error analysis:
- List sources with `\textbf{bold labels}`
- Discuss which source dominates (e.g., wire diameter appears as d², doubles its relative error)
- Mention: reading error, instrument calibration, alignment, environmental factors

### Section 3: Conclusions (10 points, ~100 words)

State what was verified/measured, restate final results with uncertainties, note main error sources. Use `\textbf{}` for key findings. In lab4 and lab5, conclusions are structured with bold sub-headings.

### Section 4: Answers to Postlab Questions (10 points)

```latex
\begin{enumerate}[leftmargin=2em,itemsep=0.25em,topsep=0.15em]
\item \textbf{Question title here}
% Full paragraph answer with equations, reasoning, and experimental references
\end{enumerate}
```

Lab5 established the best pattern: bold question summary as first sentence, then full explanation with equations, then connection to experimental observations. Earlier labs (lab1) had shorter 1-line answers.

### Appendix

Multi-page data sheets: one `figure[H]` per page. Some reports include a small signature/joke image at the bottom right: `\vfill\hfill\includegraphics[width=2.8cm]{signature.jpg}` (optional, personal preference).

### Teacher Signatures and Data Sheets

The recommended workflow for official data sheets:

1. The student fills in the official data sheet manually (raw data + required signatures).
2. The student obtains teacher/TA signatures on the physical data sheet if required.
3. The student scans or photographs the completed and signed data sheet.
4. The skill attaches this scanned data sheet directly in the appendix.

**Do not** attempt to automatically place the teacher's signature onto the data sheet. AI cannot reliably position signatures or preserve the official layout. Only attempt auto-placement if the user explicitly requests it, and warn them about the risk of misalignment.

## Data Input

### Recommended: Manual Input

For reliability, the student should **manually type in** raw experimental data. AI vision and OCR can easily misread handwritten numbers — especially decimals, units, table rows, angles, and small values. The skill can generate a blank data-entry table based on the experiment type for the student to fill in.

### Fallback: OCR / Scanned Data Extraction

Only use when the student cannot type the data manually. All extracted values **must be confirmed by the user** before any calculation.

Try direct text extraction first:

```bash
pdftotext -layout data.pdf data.txt
```

For text-based PDFs, Python/pypdf can also work:

```python
from pypdf import PdfReader
reader = PdfReader("data.pdf")
for i, page in enumerate(reader.pages):
    text = page.extract_text()
```

For scanned PDFs where extracted text is empty, render and OCR:

```bash
pdftoppm -png -r 180 -f 1 -l 3 data.pdf page
tesseract page-1.png page-1 -l eng
```

Scripts may live alongside the working directory.

## Figure Generation

```python
import matplotlib.pyplot as plt
# Use consistent styling: fontsize 12-14, grid, labeled axes
plt.rcParams.update({'font.size': 12})
# plot from real data arrays
plt.savefig("fig_name.pdf", bbox_inches="tight")
```

Only lab5-post included generated figures. Earlier labs used placeholder boxes or no figures.

## File Naming Convention

```
Source:  ExperimentName_Prelab.tex  /  ExperimentName_Postlab.tex
Output:  大物实验I-学院-学号-姓名-labN-pre.pdf
         大物实验I-学院-学号-姓名-labN-post.pdf
```

Adapt the naming pattern to your school's requirements.

## Common Mistakes

| Mistake | Fix |
|---|---|
| pdfLaTeX instead of XeLaTeX | XeLaTeX required for fontspec/Calibri/Times New Roman |
| Missing watermark on postlab | `\AddToShipoutPictureBG` after `\includepdf`, before first section |
| Single XeLaTeX pass | Two passes for cross-refs and page numbers |
| Fabricating data | Every value must be on a scanned data sheet or instrument spec |
| Assuming lab file number equals experiment number | Check the actual title and DATA TABLE numbers |
| Treating old senior reports as current rules | Current school templates override older point values |
| Wrong cover PDF filename | Match `\CoverPDF` exactly to file in working directory |
| Missing TeX dependencies during rebuild | Copy cover PDFs, data PDFs, figures, and signature images into the compile directory |
| Table exceeds margin | Reduce `\tabcolsep`, use smaller font, or rotate: `\scriptsize` + `\tabcolsep=2pt` |
| Prelab overlay misalignment | Don't change `\setlength{\unitlength}{1pt}` or geometry |
| Missing `\setcounter{page}{1}` after cover | Page numbering starts from cover |
| Prelab header `edu,cn` typo | Always check: it's `edu.cn` with a period |
| Cover date vs. overlay date mismatch | Cover uses `/`, overlay uses `.` — both fine, small differences OK |
| Forgetting `\tablefont` wrapper | Table text renders in wrong font without it |

## Experiment Knowledge Base

The `references/experiments/` directory is an extension point for per-experiment structured knowledge. Each `.md` file can include: experiment title (English + Chinese aliases), objectives, theory summary, required formulas, data table format, uncertainty calculation steps, prelab and postlab questions, common error analysis, and teacher/PPT notes.

Experiments are organized by **title** (not lab number) because lab order may change yearly. When the course version or teacher requirements change, update the corresponding experiment file.

For copyright reasons, do not commit full textbook PDFs to the public repository. Keep original textbook/PPT files in a local `private_assets/` folder (gitignored). The public repository contains only structured summaries and templates.

---
> Source: [ydh0411/physics-lab-report-skill](https://github.com/ydh0411/physics-lab-report-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
