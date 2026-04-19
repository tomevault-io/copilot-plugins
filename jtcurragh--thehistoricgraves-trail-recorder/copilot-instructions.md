## thehistoricgraves-trail-recorder

> Use feature branches for changes; do not commit directly to main


# Git Feature Branch Workflow

Before making feature changes or fixes:

1. **Create a feature branch** from main:
   ```bash
   git checkout main && git pull
   git checkout -b feature/short-descriptive-name
   ```

2. **Work on the branch** – make all edits and commits on the feature branch.

3. **Commit and push by default** – after completing feature work, commit and push without being asked:
   ```bash
   git add -A && git commit -m "Short descriptive message"
   git push -u origin feature/short-descriptive-name
   ```

4. **Open a Pull Request** on GitHub from the feature branch into main.

5. **Do not commit or push directly to main.** Use PRs for merging.

Branch naming: `feature/thing-name` or `fix/thing-name`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jtcurragh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
