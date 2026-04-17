## curiculum-vitae-and-letter

> This repository is a structured collection of professional documents, primarily **Curriculum Vitae (CV)**, **Resumes**, and **Terms of Reference (TDR)** for **Alexandro Disla**. It highlights expertise in **Monitoring, Evaluation, Accountability, and Learning (MEAL)** and **Data Analysis**. It uses **LaTeX** (specifically the `moderncv` class) for high-quality document generation and **Pandoc** for document conversion.

# GEMINI.md - Project Context

## Project Overview
This repository is a structured collection of professional documents, primarily **Curriculum Vitae (CV)**, **Resumes**, and **Terms of Reference (TDR)** for **Alexandro Disla**. It highlights expertise in **Monitoring, Evaluation, Accountability, and Learning (MEAL)** and **Data Analysis**. It uses **LaTeX** (specifically the `moderncv` class) for high-quality document generation and **Pandoc** for document conversion.

The project is organized to manage multiple versions (English/French), specific job-related projects, and historical versions of professional applications.

## Directory Overview
- **Root Directory:** Contains the main LaTeX source files for the English and French resumes, along with the primary build script.
- **`appels_doffres/`:** Contains job descriptions and tender documents (e.g., `4056.pdf` for MEAL Specialist roles).
- **`jobs/`:** Contains specific documents related to particular job applications or projects (e.g., `tekkod_modernization_job.tex`).
- **`tdr/`:** Focuses on "Terms of Reference" (TDR) documents, including a specialized build script for DOCX conversion.
- **`old_version/`:** A comprehensive archive of previous versions of cover letters, recommendation letters, and older CV templates.
- **`specific_situation/`:** Placeholder for documents tailored to unique professional scenarios.

## Key Files
- `alexandro.disla_cv.tex`: Main French CV source (tailored for MEAL roles).
- `alexandrodislaResume.tex`: Main English Resume source (tailored for MEAL roles).
- `build_cv.sh`: Bash script to compile the LaTeX sources into PDFs using `latexmk`.
- `tdr/tdr_analyst_data_dev.tex`: Source for Data/Dev Analyst Terms of Reference.
- `tdr/build_tdr.sh`: Script to convert TDR LaTeX files to DOCX using `pandoc`.

## Usage & Building

### Building Resumes (PDF)
The main resumes can be built using the provided shell script. It requires `latexmk` and a TeX distribution (like TeX Live).
```bash
./build_cv.sh
```
This script generates `Alexandro_Disla_Resume_English.pdf` and `Alexandro_Disla_Resume.pdf`, then cleans up auxiliary LaTeX files.

### Building TDRs (DOCX)
Terms of Reference documents are intended for word processing format and are converted using `pandoc`.
```bash
cd tdr
./build_tdr.sh
```
This requires `pandoc` to be installed.

## Agent Expertise
When working in this repository, the agent should act with the following expertise:
- **MEAL Specialist:** Expert in Monitoring, Evaluation, Accountability, and Learning frameworks, Logframes, and ITTs.
- **Data Analyst:** Mastery of data automation, visualization (Power BI, Looker Studio), and reporting (RMarkdown/Quarto).
- **LaTeX Expert:** Deep understanding of `moderncv`, preamble configuration, and document layout.
- **Professional CV & Cover Letter Builder:** Ability to craft compelling professional narratives.
- **Professional CV & Cover Letter Reviewer:** Critical eye for formatting, grammar, and impactful content.
- **Pandoc Expert:** Mastery of document conversion between LaTeX, Markdown, and DOCX.
- **Bash Script Expert:** Proficiency in maintaining and optimizing build automation scripts.

## Development Conventions
- **LaTeX Class:** Most documents use the `moderncv` class with the `classic` style and `blue` color scheme.
- **Formatting:** Use standard LaTeX command `\textbf{...}` for bold (avoid markdown `**...**`) and always escape ampersands (`\&`).
- **Encoding:** Files are encoded in **UTF-8** (`\usepackage[utf8]{inputenc}`).
- **Automation:** Prefers automated build scripts (`.sh`) over manual compilation to ensure consistency and clean-up of temporary LaTeX artifacts.
- **Version Control:** Historical versions are maintained in the `old_version/` directory rather than relying solely on Git history for easy side-by-side comparison of templates.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AD0791) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
