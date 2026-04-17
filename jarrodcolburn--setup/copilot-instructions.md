## setup

> This project provides a declarative and procedural framework for agent-led system configuration. It separates **intent** (what to install) from **protocol** (how to install), allowing for robust, self-correcting automation of Linux environment setups.

# System Provisioning Framework

## Project Mission
This project provides a declarative and procedural framework for agent-led system configuration. It separates **intent** (what to install) from **protocol** (how to install), allowing for robust, self-correcting automation of Linux environment setups.

## Core Components
- **`apps.yaml` (State Registry)**: The single source of truth for software packages. It categorizes tools into logical groups and defines the metadata (verification commands, install refs) needed for automation.
- **`setup.md` (Operational Protocol)**: Defines the execution logic. It serves as the "brain" for the agent, containing the decision tree for package manager selection (e.g., prioritizing `apt` over `deb-get`), dependency handling, and interactive user loops.
- **`artifacts/`**: Persistent logs and reports generated during execution.

## Maintenance & Evolution
When interacting with this project, your role often shifts between **executor** and **architect**:

### 1. Registry Maintenance (`apps.yaml`)
- **Extending**: Add new applications following existing schemas. Ensure `verify_cmd` is robust (preferring `command -v` or `which`).
- **Refining**: Update `install_ref` or `method` as tools evolve.
- **Learning**: Capture failures and workarounds in the `install_notes` section to build "project memory."

### 2. Protocol Refinement (`setup.md`)
- **Rule Optimization**: Improve the "Installation Strategy" to handle edge cases or new package managers (e.g., `uv`, `cargo`).
- **Safety**: Enhance the "Pre-flight Check" and "Error Handling" sections to minimize system state corruption.
- **Interactivity**: Refine how the agent prompts the user to ensure a balance between autonomy and control.

### 3. Best Practices
- **Minimalism**: Prefer native package managers (`apt`) or well-regarded community tools (`deb-get`) over containerized formats (`snap`, `flatpak`) to keep the system lightweight.
- **Idempotency**: All installation checks should be idempotent; running the setup multiple times should never cause duplicate installs or errors.
- **Path Awareness**: Always account for non-standard binary locations (e.g., `~/.local/bin`, `~/.cargo/bin`).

---
*This context is intended for agents tasked with either executing the setup or evolving the provisioning system itself.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jarrodcolburn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
