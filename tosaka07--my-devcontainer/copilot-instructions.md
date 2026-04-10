## my-devcontainer

> - Respond in Japanese.

# Repository Guidelines

## Project Rules

- Respond in Japanese.

## Project Structure & Module Organization

- Root: `README.md`, `LICENSE`, `mise.toml` (tool versions), `.devcontainer/` (container setup).
- Create `src/` for application code and `tests/` for automated tests when adding functionality.
- Keep non-code assets in `assets/` and docs in `docs/` (create as needed). Co-locate small module-specific fixtures under `tests/fixtures/<module>/`.

## Build, Test, and Development Commands

- Dev container (recommended): open in VS Code and run “Reopen in Container”. Ownership and `mise install -y` run automatically.
- Tooling: `mise install -y` installs versions from `mise.toml` (currently Node). Example: `node --version` to verify.
- Node projects: use `npm ci` for clean installs, `npm test` to run tests, `npm run build` for builds (add scripts in `package.json`).

## Coding Style & Naming Conventions

- Indentation: 2 spaces for JS/TS/JSON/YAML; 4 spaces for shell scripts.
- Naming: `camelCase` for variables/functions, `PascalCase` for classes, `kebab-case` for filenames and CLI commands.
- Formatting: prefer Prettier and ESLint if present; otherwise run `npx prettier -w .` before PRs.
- Imports: use relative paths within a package; avoid deep imports across modules without an index barrel.

## Testing Guidelines

- Framework: Jest or Vitest for Node projects (recommended). Keep unit tests close to code in `tests/` and mirror directory names.
- Naming: test files as `<name>.test.ts|js`. Aim for clear, isolated tests with minimal mocks.
- Running: `npm test -- --watch` locally; add coverage with `npm test -- --coverage` and keep deltas green.

## Commit & Pull Request Guidelines

- Commits: follow Conventional Commits, e.g., `feat: add user session store`, `fix: handle empty input`.
- Scope small and focused. Reference issues with `Closes #123` when applicable.
- PRs: include summary, rationale, screenshots (if UI), and testing notes. Link related issues/discussions. Keep PRs under ~300 lines of diff when possible.

## Security & Configuration Tips

- Do not commit secrets. Use `.env.local` and provide `.env.example` when config is needed.
- Devcontainer: environment sets `XDG_STATE_HOME` and fixes permissions so `mise` can write state. If tools change, update `mise.toml` and run `mise install -y`.

## プロジェクト構成とモジュール

- `.devcontainer/`: 開発コンテナ設定一式。`devcontainer.json`（Dev Containers 設定）、`Dockerfile`（Ubuntu ベース、TZ/locale/ユーザー設定）。
- `mise.toml`: プロジェクト固有のツール管理（例: Node のバージョン指定）。
- `.devcontainer/mise/config.toml`: ユーザー向けのグローバル設定（XDG）。コンテナ内 `~/.config/mise/config.toml` に bind。
- ルートには `README.md`, `LICENSE`。現時点でアプリ用の `src/` や `test/` は未配置（必要に応じて追加）。

## ビルド・実行・開発コマンド

- Dev Containers（推奨）
  - `devcontainer build --workspace-folder .`: 開発コンテナのイメージをビルド。
  - `devcontainer up --workspace-folder .`: 開発コンテナを起動（VS Code からも同等操作可）。
- Docker 直接利用（参考）
  - `docker run --rm -it -v "$PWD":/workspaces -w /workspaces mcr.microsoft.com/devcontainers/base:ubuntu zsh`
- mise（ツール管理）
  - `mise install`: `mise.toml` に基づきツールを取得。
  - コンテナ内 `PATH` は `/mnt/mise-data/shims` を含むため追加設定は不要。

## コーディングスタイルと命名

- JSON: 2 スペースインデント、末尾カンマなし、キーはローワーケース。
- Dockerfile: 1 命令 1 目的、キャッシュを意識して順序最適化、不要なレイヤーを避ける。
- ファイル/ディレクトリ命名: ローワーケース + ハイフン（例: `sample-script.sh`）。
- ブランチ命名: `feat/...`, `fix/...`, `docs/...`。

## テスト方針

- 本リポジトリは開発環境（Dev Container）提供が主目的のためテストは未提供。
- 将来的にテンプレートやスクリプトを追加する場合は `test/` を作成し、スモークテスト + `make test` などで統一実行を推奨。

## コミットとプルリクエスト

- コミット: Conventional Commits を推奨（例: `docs: add README`）。種別: `feat`, `fix`, `docs`, `chore`, `refactor`, `ci`。短く現在形で要点を記述。
- PR: 目的/背景/変更点/動作確認手順を明記。必要に応じスクリーンショットやログを添付。関連 Issue をリンク。小さく分割し、Rebase で履歴を整える。

## セキュリティと設定のヒント

- 機密情報はコミットしない（環境変数やトークンはローカル/CI の安全な保管に）。
- `devcontainer.json` のボリューム `/mnt/mise-data` はツール用。権限は `postCreateCommand` で設定されるため手動変更不要。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tosaka07)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tosaka07)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
