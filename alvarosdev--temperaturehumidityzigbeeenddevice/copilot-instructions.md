## temperaturehumidityzigbeeenddevice

> * **Use English only** for all documentation, comments, commit messages, and code. This ensures consistency and accessibility for all contributors.

### Documentation

* **Use English only** for all documentation, comments, commit messages, and code. This ensures consistency and accessibility for all contributors.
* Maintain a **single `README.md` file** located at the root of the project.

  * Do **not** create additional `README.md` files in subdirectories.
  * If extra explanation is needed for a submodule or folder, consider linking a separate `.md` file from the root `README.md` instead.
* Prefer concise, **actionable documentation** with examples over long descriptions.

### Code Organization

* Follow a **clear and consistent folder structure**. Group related modules logically.
* Avoid deeply nested folders. Keep the file hierarchy as flat as reasonably possible.
* Each source file should have a **clear and descriptive name**.

### Commits and Branches

* Use **descriptive commit messages** (e.g., `fix: correct Zigbee config`, `feat: add ESP32 sensor handler`).
* Add an emoji following this guide from this repo -> https://github.com/carloscuesta/gitmoji
* Use [Conventional Commits](mdc:https:/www.conventionalcommits.org) style where possible.
* Use feature branches (`feature/sensor-support`) and submit Pull Requests with a clear summary ong

* All critical logic should have **unit tests or integration tests** where applicable.
* Test files should mirror the source structure, using a `__tests__` or `tests/` directory convention.

### Builds and Output

* Do not commit compiled, built, or temporary files (`build/`, `.DS_Store`, `.env`, etc.).
* Use `.gitignore` to keep the repository clean from unwanted files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alvarosdev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alvarosdev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
