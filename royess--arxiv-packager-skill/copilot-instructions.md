## arxiv-packager-skill

> Prepare a minimal arXiv submission bundle from a LaTeX project by flattening the paper into one TeX file and copying only required local assets without changing content.


# arXiv Packager

Use this skill when the user wants an arXiv-ready submission directory from an existing LaTeX project.

## Workflow

1. Require an explicit main TeX source file and an explicit output stem before packaging. If either is missing or ambiguous, ask the user for it first.
2. Resolve all local dependencies from the provided main file:
   - `\input` and `\include`
   - `\includegraphics`
   - local `.bib`, `.bst`, `.cls`, `.sty`, and `.bst` files
3. Create a new output directory for the arXiv bundle using the requested output stem. When no special instruction is given, use the convention `<stem>_arxiv` for the bundle directory name.
4. Copy only the files needed to compile the paper.
5. Ensure the submission uses a single TeX file:
   - inline any included TeX content into the main file
   - do not leave `\input` or `\include` statements in the arXiv copy
6. Keep the paper content unchanged unless the user explicitly asks for cleanup.
7. If the user asks for a camera-ready cleanup, remove full-line comments and any other standalone TeX comment lines in the packaged bundle, while leaving non-comment content untouched.
8. Build the packaged bundle with LaTeX after copying, so the bundle is verified to produce a PDF.
9. Verify the bundle contains no unnecessary files and that figure paths still match the copied layout.
10. Create a ZIP archive named `<stem>_arxiv.zip` from the bundle directory for upload to arXiv. The archive must exclude any files that arXiv does not accept or that are regenerated during compilation, specifically: `.pdf`, `.aux`, `.log`, `.out`, `.toc`, `.lof`, `.lot`, `.blg`, `.synctex.gz`, `.fdb_latexmk`, `.fls`, and any other build-only artifacts. Keep the `.bbl` file in the archive when the `.bib` source is not included, because arXiv requires a pre-built bibliography in that case; omit it when the `.bib` source is present.

## Guardrails

- Do not rewrite prose, equations, citations, or figure content unless requested.
- Do not delete user files outside the new arXiv directory.
- Prefer the smallest viable bundle for compilation.
- If the user does not provide the main file or output stem, ask before proceeding.

## Release

- Installation steps for users live in [references/install.md](references/install.md).

---
> Source: [royess/arxiv_packager_skill](https://github.com/royess/arxiv_packager_skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
