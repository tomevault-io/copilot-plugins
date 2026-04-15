## capital-craft

> - Use Prettier for all code formatting

# Cursor Rules for Capital Craft Project

## Code Formatting
- Use Prettier for all code formatting
- Format LESS files automatically on save
- Use 2 spaces for indentation
- Maximum line length: 80 characters

## File Associations
- *.less files should be treated as LESS
- Use Prettier as default formatter for LESS files

## Editor Settings
- Enable format on save
- Use Prettier as default formatter
- Auto-format LESS files when saving

[project]
name = "Capital Craft"
main_path = "/templates/capitalcraft"
readme = "Full project structure and workflow documented in README.md in repository root."
stack = "Joomla 5.3.2, PHP 8.3+, Less, native JavaScript, esbuild build scripts"
build_commands = [
  "npm run less:all  # compile all Less to CSS",
  "npm run js:build  # rebuild global JavaScript bundle",
  "npm run build     # full production build (less + js)",
  "npm run watch:all # watch for changes during development"
]

[priority_rules]
1. Work slower but with maximum quality and full project context awareness.
2. If information is missing — stop and report inconsistencies, do not assume.
3. Propose fixes only after listing all detected issues.
4. Always start with a step-by-step plan of what will be done and why.
5. For general tasks — consider all related files in the project.
6. For file-specific tasks — work only with the provided file.

[workflow]
- Never give quick, partial, or raw answers.
- Before producing any code or making changes:
  1. Perform deep analysis of the task, including all relevant files in /templates/capitalcraft.
  2. List all detected issues with file references.
  3. Create a detailed, step-by-step plan for solving them.
  4. Only after plan confirmation, deliver final code/patch.
- If any data is missing, stop and request it — no assumptions allowed.

[style]
- Use only existing variables from less/_variables.less.
- Nesting only via '&'.
- Place media queries immediately after the class using `.media(@breakpoint, @rules)`.
- Partial Less files start with `_` and are imported into the main page .less file.

[js]
- Modular structure: js/global for global modules, js/pages/<page>/ for page-specific scripts.
- Rebuild the bundle after global changes with `npm run js:build`.

[images]
- Store images in images/<page>/.
- Avoid creating new folders unless absolutely necessary.

[php]
- Page blocks: /pages/<page>/<block>.php
- Common parts: /partials/
- Index: /index.php connects blocks/styles/scripts via $isHome, $isFaq, etc.

[sensitive_files]
- telegram_config.php (server-side, not overwritten during deployment)
- htaccess can be analyzed but must not be modified unless explicitly requested.

[recommendations]
- Suggest broad improvements separately from the main task.
- Always check compatibility with Joomla 5.x, Less, and esbuild.
- Cross-check PHP, Less, and JS for consistency when needed.

[git]
- When performing any git reset, rollback, or branch switch, always preserve the file `.cursorrules`.
- If rolling back to a previous commit, first stash or save `.cursorrules`, then restore it after the reset.
- Never delete or overwrite `.cursorrules` unless explicitly instructed.

[dev_workflow]
# Agreed default flow for tasks executed by Codex
mode = "task-end-push"          # every task ends with push
branch = "dev"                  # target branch
watcher = false                  # do not use watch-all.sh by default
auto_commit = true               # Codex commits changes itself
auto_push = true                 # Codex pushes to dev when task is done
commit_style = "conventional"   # e.g. fix(styles): message
commit_scopes = ["styles","js","php","build","docs"]

# Build behavior
build_strategy = "targeted"     # compile only what changed
less_targets = ["base","critical","home","faq"]
js_bundle = "templates/capitalcraft/js/global/bundle.js"
css_outputs = [
  "templates/capitalcraft/css/base.css",
  "templates/capitalcraft/css/critical.css",
  "templates/capitalcraft/css/home.css",
  "templates/capitalcraft/css/faq.css"
]

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Web-A1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
