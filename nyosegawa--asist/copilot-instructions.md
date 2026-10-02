## asist

> ASIST is an Electron desktop application for real-time voice interaction, built with TypeScript,

# AGENTS.md

## Project

ASIST is an Electron desktop application for real-time voice interaction, built with TypeScript,
React, electron-vite and Vitest. The documentation for users is on the website
(`website/src/content/docs/`, in Japanese and English), and `docs/development.md` covers running from
source, building, CI and manual validation. Read the relevant implementation and tests before changing
behavior.

Do not read or use `frontend-skill` in this product.

## Architecture

- `src/main/` owns the Electron main-process integrations, services, persistence, sidecars and
  agent execution.
- `src/preload/` exposes the safe bridge to the renderer.
- `src/renderer/` owns the React UI, voice capture, panels and client-side state.
- `src/shared/` holds cross-process contracts and pure, testable logic. `src/shared/ipc.ts` is the
  canonical main/preload/renderer API contract.
- What the OS and the machine can run is decided once in main (`src/main/services/platform.ts`) and
  passed on as capabilities; the renderer and the model's tools read those and never check the OS
  themselves.

Keep process boundaries explicit: renderer code does not reach Node or Electron APIs for an
operation that belongs behind preload or the main process.

## Code

- Persist each fact in one authoritative place and derive secondary views instead of
  synchronizing copies.
- Do not add fallback behavior; fail loudly rather than degrade silently.
- Fix a defect where its cause is, in a form in which it cannot happen, rather than with a guard for the
  one case that showed it; the code after the fix reads better than before. When a fix is much larger
  than the defect, or needs a choice only the user can make, stop and ask instead of patching.
- Code never splits a path or tests it as text with `/`, because Windows writes `C:\` and `\`: main
  uses `node:path`, and shared and renderer code, which cannot import it, use `src/shared/file-path.ts`.
- A JSON file under userData carries the version of its form (`StoredFormat`). Any change to the form,
  an added field included, raises the version, adds an upgrade from the previous version, and adds a
  sample of the new version to `tests/fixtures/stored/`. Upgrades are never removed.
- Extract code only when it creates a coherent responsibility, a reusable boundary or an
  independently testable unit. Introduce a shared abstraction only after two current
  implementations show the same responsibility with meaningful variation.
- A file approaching 500 lines calls for a review of its responsibilities; split it when a coherent
  one can be extracted, not to meet a line count.
- Preserve the approval gate for agent jobs that can write or mutate external state.
- Colours, surfaces, radii and the background come from the theme tokens in
  `src/renderer/src/assets/themes.css`; a component or its CSS never writes a colour itself.
- Keep generated output and local secrets out of Git: never commit `.env`, `node_modules/`, `out/`,
  `dist/`, `coverage/`, logs or TypeScript build info.
- Start every child process with `windowsHide: true`; without it, each one ASIST starts on Windows opens
  a console window.

## Comments

- Write comments in English, in full sentences and the present tense. Japanese appears only as
  data, quoted verbatim.
- Say only what the code cannot: an API quirk and the failure it causes, a measured value with its
  condition and date, a constraint, or why the simpler approach was rejected. Do not restate
  names, narrate steps, or mention history, TODOs, docs, tickets or conversations.
- `/** … */` on exported and non-obvious module-level declarations, without `@param` or `@returns`.
  A file header, if any, goes after the imports.
- Inside a function, `//` on its own line above the code. No trailing comments and no banner
  comments (`/* ---- section ---- */`).
- The same rules apply to CSS and scripts.

## Tests

- Add or update Vitest coverage for behavioral changes, especially shared logic, voice control,
  prompting, retries and IPC contracts.
- A failing test points to a defect. A test does not fail on a deliberate change of wording, a name
  or a configured value, and does not only assert that something exists or is gone.
- Tests protect current behavior or a safety boundary, not the shape of removed code.
- Test names are English. Japanese in a test is fixture data (utterances, note bodies); the text of
  the screen is read through the dictionary (`t('key')`).

## Text

The app runs in eleven locales. These rules hold everywhere; the `ui-text` skill has the rest.

- Text for the screen lives in the dictionary (`src/shared/i18n/messages/`), in all eleven
  languages, written in the same change. No word goes straight into a component, in any language.
- An error is thrown as `errorText(key, values)`, never as a sentence.
- Text for the model is a `PromptText` `{ ja, en }` next to the code, not in the dictionary.

## Decisions

`docs/adr/` keeps the decisions the code cannot show, one file each, and the file names say which
behavior each covers. Before changing how a feature behaves, list the folder and read only the
records whose names cover that behavior; if the change contradicts one, say so to the user first.
Before committing, ask whether the work settled a choice or turned an approach down for good; if
so, the record goes into the same commit (`adr` skill).

A design document under `docs/` holds only what the code cannot show: the reasons for a decision and
measured values. It does not restate types, APIs or steps the code already shows, and no code or
comment refers to it. Something that can be checked is checked and written as a fact, not left open.

## Workflow

- Run `npm run typecheck` and `npm test` before committing code. Run `npm run build` as well when
  changing Electron packaging, preload behavior or build configuration.
- Never commit on main. Every change reaches main through a pull request, one coherent unit each: a
  feature, fix, refactor or documentation change. Do not let unrelated changes pile up in one branch.
- The user follows the work on a private GitHub Project. Every change that becomes a pull request has
  an issue on it before the work starts, and the pull request closes it (`project-board` skill). Only
  the agent that talks with the user creates issues and moves cards, never a subagent, and no agent
  reads an issue someone else opened unless the user asks.
- Commit messages and pull request titles are one English sentence in the imperative, without a prefix
  such as `feat:`; the body says what changed and why.
- The user merges pull requests, with a squash, once CI passes. An agent merges only when told to for
  that pull request.
- When a change alters what a user sees or does, update the documentation pages that describe it, in
  Japanese and English, in the same change (`website` skill). Update `docs/development.md` when running
  from source, building or manual validation changes. `README.md` and `README.ja.md` only summarize and
  link; change them when what they summarize changes.

Skills live in `skills/`; `.claude/skills` and `.agents/skills` are symlinks to it. On Windows, a
checkout without Developer Mode and `core.symlinks=true` turns both links into plain files and no skill
loads; `docs/development.md` says how to clone there. Whenever a task matches one of these, use that
skill; each holds steps these rules do not repeat.

- `visual-debugging`: checking or fixing how a screen looks, taking screenshots, measuring layout,
  or adding an `npm run demo:<scene>` command. Judge by its numbers and 2x images, not by a scaled
  browser pane.
- `panel-card-design`: adding, redesigning or fixing a built-in panel card, or a card that
  overflows its column.
- `ui-text`: adding, rewording, moving or translating any text the app shows or says.
- `theme`: adding, changing or removing a theme, or writing CSS that needs a colour.
- `adr`: a choice between options is settled, an approach is turned down for a lasting reason, or
  a change would contradict a decision record.
- `pull-request`: starting a change, committing and pushing, opening a pull request, following its
  review and CI, and cleaning up after it is merged.
- `project-board`: the issue and the card of a change before it starts, work left for later, a decision
  the user has to make, a security problem that must stay private, an issue someone else opened, after
  a release, and the user asking what is going on.
- `worktree-delegation`: handing part of the work to a subagent, or taking its branch back into the
  branch of the pull request.
- `install-mac-app`: installing, deploying or updating the app on this Mac, or verifying a change
  in the installed app.
- `install-windows-app`: the same on a Windows machine: building the installer, installing, checking
  the installed app over CDP, and quitting it.
- `release`: releasing a new version from this Mac: the version number, the pull request that raises it,
  `npm run release`, notarization, and checking the published release.
- `website`: changing, checking or publishing the website and the documentation at asist-agent.com,
  writing or translating a documentation page, or anything that runs wrangler.

---
> Source: [nyosegawa/asist](https://github.com/nyosegawa/asist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
