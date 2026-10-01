## dms-plugins

> This repository is a personal collection of custom and community plugins for [DankMaterialShell (DMS)](https://github.com/AvengeMedia/DankMaterialShell), maintained by [@hthienloc](https://github.com/hthienloc). These guidelines help human contributors and AI agents build, maintain, and contribute plugins safely and consistently.

# Agent Guidelines — dms-plugins

This repository is a personal collection of custom and community plugins for [DankMaterialShell (DMS)](https://github.com/AvengeMedia/DankMaterialShell), maintained by [@hthienloc](https://github.com/hthienloc). These guidelines help human contributors and AI agents build, maintain, and contribute plugins safely and consistently.

---

## 1. Project Overview & Architecture

### Monorepo Structure
Each subdirectory in this repository represents an independent plugin:
```
dms-plugins/
├── <pluginName>/             # Individual self-contained plugin
│   ├── plugin.json           # Required: Plugin manifest & metadata
│   ├── <MainComponent>.qml   # Required: Plugin entrypoint
│   ├── <Plugin>Settings.qml  # Optional: Settings card in DMS Settings
│   ├── shared/               # Optional: Shared DMS utility components
│   ├── docs/                 # Optional: Specifications & architecture notes
│   ├── translations/         # Optional: Localized UI strings (.json)
│   └── README.md             # Required: Plugin documentation & shortcuts
├── shared/                   # Canonical source for shared UI components
├── scripts/                  # Repository maintenance & tooling scripts
└── README.md                 # Root catalog of all plugins
```

### Plugin Types & Type Selection Criteria
Every plugin declares its surface type in `plugin.json`. Choose the simplest, most appropriate type based strictly on the plugin's primary responsibility:

| Type | Base Component | Role & Primary Surface | When to Use | Examples in repo |
| :--- | :--- | :--- | :--- | :--- |
| `widget` | `PluginComponent` | DankBar pill + popout card / Control Center | Persistent bar presence needed for quick status readout, toggling state, or interacting via popout | `caffeine`, `timer`, `ipIndicator`, `hydrate`, `breathing`, `lutrisLauncher`, `bongoCat`, `ambientSound`, `handMirror`, `hiddenBar`, `mediaDownloader`, `ocrScanner` |
| `daemon` | `PluginComponent` | Background service or standalone on-demand modal/overlay | Headless background event/timer listener, or on-demand modal/overlay triggered via shortcut/IPC without cluttering the bar | `emojiPicker` |
| `launcher` | `Item` | DMS Launcher search result provider | Actionable search results, conversions, or query integrations inside DMS Launcher (`trigger` based) | `kaomojiPicker` |
| `desktop` | `DesktopPluginComponent` | Floating desktop widget | Persistent or draggable widget placed directly onto the desktop workspace | `activateLinux` |
| `composite` | Multiple entry components | Multi-surface integration | Complex plugins where distinct surfaces (e.g. daemon + Control Center toggle / bar widget) are strictly interdependent | `screenkey`, `typingSounds`, `takeABreak`, `quickCapture`, `desktopWidgetToggle`, `niriDS` |

### Type Selection Principles: Avoid Over-Engineering
- **Single Responsibility First:** Match the type strictly to what the plugin actually does. If a plugin is an on-demand modal picker triggered by hotkey (like `emojiPicker`), declare it as `daemon`; do not add a bar widget or launcher search unless the core UX demands it.
- **Do Not Default to `composite`:** `composite` introduces multiple entry points and significantly complicates state management, testing, and memory footprint. Never use `composite` when a single surface (`daemon` or `widget`) suffices.
- **Keep the Shell Clean:** Avoid adding bar widgets for tools used occasionally. Reserve the bar for real-time monitoring and frequent toggles. Prefer `daemon` with IPC toggle shortcuts for on-demand tools.

---

## 2. Agent Scope & Blast Radius Rules

- **Plugin Isolation:**
  - Every pull request, commit, and change must target exactly one plugin folder (`<pluginName>/`).
  - Do not make cross-cutting edits across multiple plugins in the same change unless explicitly instructed (e.g., repository-wide migration or dependency upgrade).
- **Protected Files & Scope Boundaries:**
  - **Translations (`<pluginName>/translations/`):** Do not modify translation JSON files manually unless explicitly instructed. Keep strings in `I18n.trFor()` calls stable.
  - **Manifests (`plugin.json`):** Preserve required fields (`id`, `name`, `version`, `entryPoint`, `type`, `capabilities`). Version bumps should match the nature of changes.
  - **Symlink Awareness:** Testing is conducted against local DMS symlinks (`~/.config/DankMaterialShell/plugins/<pluginName>`). Never create circular symlinks or alter the symlink targets outside this repository.

---

## 3. QML & DMS Code Standards

### Pragmas & Modern QML
- Use `pragma ComponentBehavior: Bound` in modern QML components when appropriate for component binding safety.
- Import standard DMS modules:
  - `qs.Common` / `qs.Widgets` for standard UI widgets and styling.
  - `qs.Services` for compositor, clipboard, toast, and system services (`CompositorService`, `ToastService`, `DMSService`).
  - `qs.Modules.Plugins` for `PluginComponent` and `PluginService`.

### Theme Tokens & Visual Consistency

DMS uses an opinionated Material Design 3 (M3) design system exposed globally via the `Theme` singleton (`qs.Common`). Every plugin must adhere strictly to these tokens for seamless visual integration with the desktop shell.

#### 1. Surface Hierarchy & Layering
DMS builds depth through tonal surface elevation rather than heavy drop shadows:

| Surface Token | Purpose & Typical Use | Matching Text / Content Token |
| :--- | :--- | :--- |
| `Theme.hostSurface` / `Theme.surface` | Root modal, window, or popout background layer | `Theme.surfaceText` / `Theme.onSurface` |
| `Theme.cardSurface` / `Theme.surfaceContainer` | Cards, panels, or distinct content groupings | `Theme.surfaceText` |
| `Theme.chipSurface` / `Theme.surfaceContainerHigh` | Interactive elements, search bars, buttons, chips | `Theme.surfaceText` |
| `Theme.chipSurfaceNested` / `Theme.surfaceContainerHighest` | Sub-chips, active selection items, elevated hover rows | `Theme.surfaceText` |
| `Theme.surfaceVariant` | Subdued containers, dividers, or tertiary backgrounds | `Theme.surfaceVariantText` / `Theme.onSurfaceVariant` |

#### 2. Color Roles & States
- **Primary / Accents:** Use `Theme.primary` for key actions, focus outlines, and active state highlights; pair with `Theme.onPrimary` for text/icons on primary backgrounds.
- **Selection:** Use `Theme.selectedContainer` for highlighted list/grid delegates and `Theme.onSelectedContainer` for its foreground.
- **Destructive Actions:** Use `Theme.error` and `Theme.onError` for clear, delete, or warning actions.
- **Outlines & Borders:** Use `Theme.outline` for prominent borders, and `Theme.outlineVariant` for subtle separators and field outlines.
- **Alpha & States via `Theme.withAlpha()`:** Never construct raw rgba strings. Always use `Theme.withAlpha(color, opacity)`:
  - Hover states: `Theme.withAlpha(Theme.surfaceText, 0.08)` or `Theme.withAlpha(Theme.primary, 0.12)`
  - Subdued / Secondary text: `Theme.surfaceVariantText` or `Theme.withAlpha(Theme.surfaceText, 0.6)`
  - Popup transparency: Respect `Theme.popupTransparency` and `Theme.blurLayersActive` when creating overlay backdrops.

#### 3. Spacing & Metric Scale
Never use arbitrary pixel margins or paddings ("magic numbers"). Always use the `Theme.spacing*` scale:

| Token | Pixels | Standard Usage |
| :--- | :--- | :--- |
| `Theme.spacingXXS` | `2px` | Micro-gaps between keycaps, badge padding, thin dividers |
| `Theme.spacingXS` | `4px` | Compact button padding, tight element gaps |
| `Theme.spacingS` | `8px` | Space between related controls, inside cards |
| `Theme.spacingM` | `12px` | Standard modal/panel padding, row layouts, container margins |
| `Theme.spacingL` | `16px` | Section margins, card boundaries, outer content gutters |
| `Theme.spacingXL` | `24px` | Major section separators, prominent dialog margins |

#### 4. Corner Radius System
Maintain the shape hierarchy across different component scales:

| Token | Typical Component Application |
| :--- | :--- |
| `Theme.cornerRadiusXS` | Small badges, sub-chips, tooltip indicators |
| `Theme.cornerRadiusSmall` (`cornerRadiusS`) | Buttons, individual list/grid item delegate hover boxes |
| `Theme.cornerRadius` (`cornerRadiusM`) | Standard cards, dropdown menus, popover containers |
| `Theme.cornerRadiusLarge` (`cornerRadiusL`) | Main plugin modals (`DankModal`), dialog hosts |
| `Theme.cornerRadiusFull` (`9999px`) | Pills, circular action buttons, search field pill styling |

#### 5. Typography & Font Hierarchy
Use `StyledText` (not raw `Text`) for proper font rendering and theme integration:

| Token | Base Size | Typical Use |
| :--- | :--- | :--- |
| `Theme.fontSizeSmall` | ~`12px` | Footers, keyboard hints, badges, secondary captions |
| `Theme.fontSizeMedium` | ~`14px` | Standard body text, button labels, grid item descriptions |
| `Theme.fontSizeLarge` | ~`16px` | Search bar input text, section headers, modal subtitles |
| `Theme.fontSizeXLarge` | ~`20px` | Dialog titles, prominent summary indicators |
| `Theme.fontSizeXXLarge` | ~`28px` | Large numbers, countdown clocks, metrics display |

- **Font Family:** `Theme.defaultFontFamily` ("Google Sans Flex") for UI text; `Theme.defaultMonoFontFamily` ("Fira Code") for keystrokes, shortcuts, and code blocks.
- **Font Weights:** Use `Font.Normal` (400), `Font.Medium` (500), or `Font.DemiBold` (600) sparingly for emphasis.

#### 6. Component Reuse & Iconography
Always prefer built-in DMS primitives over custom implementations:
- **Icons:** Use `DankIcon` with standard Google Material Symbols icon names (e.g. `"search"`, `"delete_sweep"`, `"settings"`). Browse and find suitable icon names at [Google Fonts Icons](https://fonts.google.com/icons).
- **Buttons:** Use `DankButton` for text/icon buttons, `DankActionButton` for compact circular or square bar actions.
- **Keycaps:** Use `DankKeycap` for rendering keyboard shortcuts (`"↵"`, `"Ctrl ↵"`, `"Esc"`).
- **Search & Fields:** Use `DankSearchField` for unified search styling with clear buttons and focus rings.
- **Dropdowns & Popups:** Use `DankDropdown` for unified menu placement, focus trapping, and border radius.
- **Notifications & Feedback:** Use `ToastService.showInfo(...)` and `ToastService.showError(...)` for user feedback instead of custom inline alerts.

### UX/UI Philosophy: Minimalism, Clear Guidance & Dogfooding

- **Minimalism over Cognitive Overload:**
  - Prioritize clean, distraction-free interfaces over cramming excessive buttons, secondary toggles, or visual noise into the UI.
  - Every UI element must earn its place. If an action can be handled gracefully via a sensible default, an intuitive shortcut, or a context menu, do not clutter the primary surface with extra buttons.
  - Keep modals and popouts compact, focused strictly on their primary task.
- **Clear & Unobtrusive Visual Guidance:**
  - Make available interactions self-explanatory without adding visual clutter.
  - Use subtle, standardized footer hint bars with `DankKeycap` (e.g. `↑ ↓ nav`, `↵ select`, `Esc close`) to make keyboard navigation immediately discoverable.
  - Provide descriptive placeholder text in inputs and explicit empty-state messaging (e.g. "No matches found" rather than an ambiguous empty screen).
- **Dogfooding & Design Deliberation (Test Your Own Design):**
  - Never push UI changes immediately after writing code without live testing. Contributors and agents must actively dogfood and test their own interface in realistic desktop conditions over time.
  - Critically evaluate your own design choices:
    - *Does this layout feel cramped, noisy, or overwhelming after repeated daily use?*
    - *Can the user complete their task with minimal keystrokes and zero cognitive friction?*
    - *Are active focus rings and hover states immediately obvious without guessing?*
  - Iterate and deliberately simplify the design based on real hands-on interaction before declaring the work ready.

### Keyboard Navigation & Event Bubbling
- Modals and popouts must maintain reliable focus management using `FocusScope`.
- Intercept common shortcuts (`Ctrl+F`, `Ctrl+C`, `Ctrl+V`, `Escape`, `Enter`) explicitly and mark `event.accepted = true` to prevent unhandled bubbling or conflicts with desktop compositor shortcuts.
- Support keyboard accessibility for grid and list views (arrow navigation, Escape to close or clear state).

### The DRY Principle
- Avoid duplicate action implementations. Common shortcuts that trigger identical behavior (such as `Enter` and `Ctrl+C` for copying, or `Ctrl+Enter` and `Ctrl+V` for pasting) must route through shared helper functions (e.g., `commitCurrent(...)`).
- In documentation and UI hints, group shared shortcuts together (e.g., `Enter / Ctrl+C`).

### Internationalization (i18n)
- Wrap all user-facing strings with `I18n.trFor("<pluginId>", "English text")`.
- Dynamic formatting should use `.arg(...)` rather than string concatenation.
- Do not manually modify existing translation JSON files in `translations/` unless explicitly requested.

### Security & Process Execution Safety

Plugins frequently interact with external CLI utilities (e.g. via Quickshell's `Process` or IPC). Follow strict security conventions to prevent command injection and unsafe operations:

- **No Shell String Concatenation:** Never use `["sh", "-c", "... " + userInput]` with concatenated strings. Pass commands as discrete argument arrays (`[bin, arg1, arg2, ...]`) so parameters bypass shell evaluation entirely:
  ```qml
  // SECURE: Executed directly via execve/execvp
  command: ["yt-dlp", "--no-warnings", url]

  // DANGEROUS: Susceptible to shell injection
  command: ["sh", "-c", "yt-dlp " + url]
  ```
- **Positional Parameters for Shell Commands:** If shell features (pipes, redirections) are unavoidable, never interpolate variables directly into the shell string. Pass them as positional arguments:
  ```qml
  // SECURE: Parameter $1 is safely isolated
  command: ["sh", "-c", 'command "$1"', "sh", userInput]
  ```
- **IPC & Input Sanitization:** Never assume arguments passed to `IpcHandler` methods are trusted. Always type-check, clamp numeric bounds, and check file paths for directory traversal (`../`) before using them.
- **Principle of Least Privilege:** Only request permissions in `plugin.json` (`permissions: [...]`) that the plugin actively requires. Never request unneeded capabilities.
- **Process Cleanup:** Ensure long-running child processes are terminated cleanly on plugin component destruction (`Component.onDestruction`).

---

## 4. Verification & Testing Workflow

### Symlink Setup
Plugins are developed in-tree and tested via symlinks inside the local DMS config directory:
```bash
ln -sfn "$PWD/<pluginName>" "$HOME/.config/DankMaterialShell/plugins/<pluginName>"
```

### Hot Reloading via IPC
Do not restart DMS or Quickshell to test plugin changes. Always trigger hot-reloading via IPC:
```bash
dms ipc call plugins reload <pluginName>
```
Verify the output returns `PLUGIN_RELOAD_SUCCESS: <pluginName>`.

### Functional Verification via IPC
Exercise plugin methods directly from the terminal:
```bash
# General lifecycle
dms ipc call <pluginName> open
dms ipc call <pluginName> close
dms ipc call <pluginName> toggle
```

### Verification Scripts
- **Unused Imports:** Run `python3 scripts/check_unused_imports.py` before committing.
- **Backward Compatibility:** Only required when adopting new/cutting-edge DMS components or modifying `shared/`:
  ```bash
  python3 scripts/check_compatibility.py
  ```
  Ensures components not yet released in stable DMS (1.6.2) are properly bundled in `shared/` and imported correctly.

---

## 5. Documentation Standards

- **Plugin README (`<pluginName>/README.md`):**
  - Keep the **Controls** table updated whenever keybindings or interaction modes change.
  - Apply the DRY rule: group keys that share identical actions (e.g. `Enter / Ctrl+C`, `Ctrl+Enter / Ctrl+V`).
  - Document all supported IPC methods and default configuration options.
- **Specification Files (`<pluginName>/docs/specs/`):**
  - If a feature has an associated spec (e.g. `emoji-queue.md`), update the interaction contract tables to reflect any modified shortcuts or behaviors.
- **Root Catalog (`README.md`):**
  - When adding a new plugin, add an entry to the root table with plugin name, type, and a one-line description.

---

## 6. Git & Contribution Hygiene

### Conventional Commits
All commits must follow the **Conventional Commits** specification with the target plugin name as the scope:
```
<type>(<pluginName>): <short description>
```
Common examples:
- `feat(emojiPicker): add Ctrl+F, Ctrl+C and Ctrl+V shortcuts`
- `fix(screenkey): handle click-through overlay mask`
- `docs(quickCapture): document backdrop toggle shortcut`
- `refactor(caffeine): simplify state timer handling`

### Issue & Pull Request Communication

- **Conciseness over Walls of Text:** Humans review every issue and pull request. Keep all descriptions brief, condensed, and high-signal. Do not output walls of text, chatty preambles, or redundant AI-generated boilerplate.
- **Ask Before Guessing:** Never make speculative assumptions about underspecified requirements, plugin UX, or edge cases. Ask the maintainer directly for clarification instead of guessing or over-engineering.
- **Language:** Write all issues, PR titles, and descriptions in English.
- **PR Description Structure:** Keep PR descriptions strictly focused on three concise items:
  - **What:** One or two sentences summarizing the change.
  - **Why:** The concrete problem or motivation.
  - **How tested:** Specific IPC commands run and verification results.
- **Maintainer Edits:** Always enable `maintainer_can_modify=true` when creating pull requests.

### Safety & Remote Etiquette
- Do not trigger remote bot reviews or automated comments (e.g. `gh pr comment`, `/claude review`) unless explicitly requested by the maintainer.
- Rebase over merge when syncing local feature branches with upstream.

---
> Source: [hthienloc/dms-plugins](https://github.com/hthienloc/dms-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
