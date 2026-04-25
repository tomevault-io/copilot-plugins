## phys349-github-io

> This is PHYS 349, an undergraduate quantum computing course taught by Jonathan Hood (quantum optics / AMO physics). Students have had a Griffiths-level QM course. The course website is built with Jupyter Book 2 and hosted on GitHub Pages.

# CLAUDE.md — PHYS 349: Introduction to Quantum Computing

## Course Overview

This is PHYS 349, an undergraduate quantum computing course taught by Jonathan Hood (quantum optics / AMO physics). Students have had a Griffiths-level QM course. The course website is built with Jupyter Book 2 and hosted on GitHub Pages.

## Repository Layout

- `lectures/` — Lecture files named `{ch}_{sec}_lecture.md` in MyST markdown
- `lectures/claude_summary.md` — Summaries of all lectures with key equations. **Read this before writing any new lecture.**
- `lecture_resources/` — Qiskit tutorial notebooks (.ipynb), course outline, lecture notes, reference materials
- `lecture_resources/outline.md` — Semester plan for weeks 10–14 (remaining lectures)
- `homework/` — Jupyter notebook assignments
- `figures/` — Lecture figures
- `exams/` — Exam materials
- `myst.yml` — Jupyter Book configuration

## Lecture Writing Conventions

### Format
- Jupyter Book 2 / MyST markdown
- YAML frontmatter with kernelspec (python3)
- LaTeX math: display with `$$...$$`, inline with `$...$`
- Horizontal rules (`------------------------------------------------------------------------`) between major Parts

### Structure
Each lecture follows this template:
1. **Title:** `# Lecture X.Y — Topic Name`
2. **"Where We Left Off"** section connecting to the previous lecture
3. **Numbered Parts** with `## Part N:` headings and `###` subheadings
4. **Summary** section with bullet points
5. **"What's Next"** teaser for the following lecture
6. Optional: embedded homework at the end

### Admonitions
- `{admonition}` with classes: `tip`, `warning`, `note`, `important`, `question`
- iClicker questions: `{admonition} iClicker` with `:class: question` — used for active learning throughout
- `{dropdown}` for optional/advanced content
- `{admonition} Check Yourself` with `:class: tip` for self-assessment

### Code Cells
- Python/Qiskit code in `{code-cell} python` blocks
- Use `:tags: [hide-input]` for setup/visualization code students don't need to read
- Import patterns: `from qiskit import QuantumCircuit`, `from qiskit.quantum_info import Statevector, Operator`, `from qiskit_aer import AerSimulator`
- Always verify results computationally after deriving them analytically

### Qiskit Convention Note
- Qiskit qubit ordering: top wire = qubit 0 = rightmost position in ket
- This reverses the tensor product order from most textbooks
- Note this explicitly when it matters

## Pedagogical Style

- **Physical intuition over formalism** — always connect math to physical systems
- **Real hardware connections** — ground abstract concepts in superconducting qubits, trapped ions, photonics, neutral atoms
- **Interferometer analogy** — Mach-Zehnder interferometer as the bridge between optics and quantum computing (recurring theme)
- **Build up from what students know** — connect new concepts to prior lectures explicitly
- **Verify analytically then computationally** — derive by hand, then confirm with Qiskit
- **Active learning** — iClicker questions throughout, in-class activities where appropriate

## Current Status (March 2026)

### Completed Lectures
- Chapter 1 (01_01–01_04): Motivation, computational hardness, hardware survey
- Chapter 2 (02_01–02_07): Single-qubit physics (waves, interferometers, Bloch sphere, group theory, Pauli matrices, Rabi/Ramsey, decoherence)
- Chapter 3 (03_01–03_04): Two qubits (tensor products, entanglement, Bell states, EPR, Bell's theorem)
- 04_00: Quantum circuits + no-cloning theorem
- 04_01: CHSH game, GHZ states, quantum half-adder
- 03_07: QKD (BB84)

### Remaining Lectures (7 total)
- 03_08: Teleportation + superdense coding
- 03_09: Quantum hardware platforms
- 03_10: Quantum sensing + squeezing
- 04_01 (new numbering): Deutsch-Jozsa algorithm
- 04_02 (new numbering): Grover's algorithm
- 04_03: QFT + phase estimation
- 04_04: Shor's algorithm

### Renumbering
Lecture numbering is being reorganized. The old 03_xx series is shifting into Chapter 4. Lecture 03_07 (BB84) will become 4.2 in the new scheme.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PHYS349) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
