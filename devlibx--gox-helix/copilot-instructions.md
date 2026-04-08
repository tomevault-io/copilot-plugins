## gox-helix

> - whn i ask for git commit.  add the chages files, generate a good commit message with details. and git commit and push

# gox-helix

A Go project.

- whn i ask for git commit.  add the chages files, generate a good commit message with details. and git commit and push
- do not git commit untill i specificially do this.
- Design documents are located in the doc/design folder.
- The implementation_details.md file contains information about the implementation of the framework.

## Worker Component Development

We are building a new, resilient `worker` component for task execution and coordination.

### High-Level Plan:
1.  **New Package Structure:** The worker component is split into an interface and a MySQL implementation:
    *   `pkg/worker/`: Defines the abstract `Worker` interface.
    *   `pkg/worker/mysql/`: Provides the concrete MySQL-based implementation.

2.  **Database Schema:** A new `helix_workers` table will be created to register and track workers. It will be partitioned by the `created_at` month to ensure long-term performance.

3.  **Documentation:**
    *   A new `pkg/worker/GEMINI.md` file will store concepts and context for the worker component.
    *   This root `GEMINI.md` file will track the high-level project status.

4.  **Examples:** A new `examples/` directory at the project root will provide runnable examples for all helix components. The worker examples will demonstrate `go-fx` dependency injection and integration with the `domain` component.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/devlibx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/devlibx)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
