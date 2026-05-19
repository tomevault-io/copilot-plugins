## homework-format

> Formatting rules for COSC 6373 homework notebooks


# Homework Notebook Format

When creating or formatting Jupyter Notebooks for COSC 6373, adhere to the following structure and conventions.

1. **Title Cell**: The first cell must be a Markdown cell with the following format:
   ```markdown
   # HW[XX] Part [X] :: [Assignment Title]

   COSC 6373 - Adam Nelson-Archer, 2140122
   ```

2. **File Naming**: Notebooks should be named using the format `HWXX-ICA-PartX_Adam_Nelson-Archer.ipynb` (for Part A/In-Class) or `HWXX-PartX_Adam_Nelson-Archer.ipynb` (for Part B).

3. **Acknowledgment Cell**: The final cell in the notebook must be a Markdown cell acknowledging the use of an AI assistant. Something like this, or similar:
   ```markdown
   ## Acknowledgment

   I used a GPT-5.3-Codex to help scaffold and organize this notebook.

   Gemini-3.1 was used to check the result and validate conformity with the assignment outline.

   All observations and responses were written by me, Adam Nelson-Archer.
   ```

4. **Code Structure**:
   - Organize imports at the beginning.
   - Install packages in the first or second cell if needed (e.g., `!pip install ...`).
   - Use clear markdown explanations between code blocks.

5. **Formatting Restrictions**:
   - **No bold text**: Do not use `**bold**` text in markdown cells anywhere in the notebook.
   - **No em dashes / en dashes**: Do not use em dashes (`—`), en dashes (`–`), or double hyphens (`--`) to simulate dashes. Use standard single hyphens (`-`) exclusively.

---
> Source: [adamnelsonarcher/COSC6373](https://github.com/adamnelsonarcher/COSC6373) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
