## mcbeaddontemplate

> - **Language**: All text, including comments, documentation, commit messages, and PR descriptions, should primarily be in **Japanese**. However, code naming (variable names, function names, etc.) should be in English (camelCase, PascalCase).

## Basic Policy
<!-- 基本方針 -->

- **Language**: All text, including comments, documentation, commit messages, and PR descriptions, should primarily be in **Japanese**. However, code naming (variable names, function names, etc.) should be in English (camelCase, PascalCase).
<!-- **言語**: コメント、ドキュメント、コミットメッセージ、PR の説明など、すべてのテキストは原則として**日本語**を使用します。ただし、コードの命名（変数名、関数名など）は英語（キャメルケース、パスカルケース）とします。 -->
- **AI Operations**: AI assistants (Gemini) will not automatically commit or push changes after modifying files. All Git operations will be executed only with clear instructions and permission from the user.
<!-- **AI による操作**: AI アシスタント（Gemini）は、ファイルの変更後に自動でコミットやプッシュを行いません。すべての Git 操作は、ユーザーからの明確な指示と許可を得てから実行します。 -->
- **AI Configuration File Synchronization**: When editing `CLAUDE.md` or `GEMINI.md`, always reflect the same content in the other file.
<!-- **AI 設定ファイルの同期**: `CLAUDE.md` または `GEMINI.md` を編集する際は、必ずもう一方のファイルにも同じ内容を反映させてください。 -->
- **File Reading**: To fully understand the project context, TypeScript files (`.ts`) will always be read in their entirety, regardless of their size. Partial reading will not be performed.
<!-- **ファイル読み込み**: プロジェクトのコンテキストを完全に理解するため、TypeScript ファイル (`.ts`) は、そのサイズに関わらず、必ずファイル全体を読み込みます。部分的な読み込みは行いません。 -->

## Dependency Analysis
<!-- 依存関係の分析 -->

To understand the `npm` packages available in the project, we will not directly read the `node_modules` directory. Instead, we will analyze the contents of the following files:
<!-- プロジェクトで利用可能な `npm` パッケージを把握するために、`node_modules` ディレクトリを直接読み込むことはしません。代わりに、以下のファイルの内容を分析します。 -->

- **`package.json`**: Check the packages directly depended on by the project and their version ranges.
<!-- **`package.json`**: プロジェクトが直接依存するパッケージとそのバージョン範囲を確認します。 -->
- **`package-lock.json`**: Understand the exact version tree of all installed packages, including indirect dependencies.
<!-- **`package-lock.json`**: 間接的な依存関係を含む、実際にインストールされたすべてのパッケージの正確なバージョンツリーを把握します。 -->

This approach allows for a quick and accurate understanding of project dependencies.
<!-- このアプローチにより、迅速かつ正確にプロジェクトの依存関係を理解することができます。 -->

### Exception: Minecraft API Type Definitions
<!-- 例外: Minecraft API の型定義 -->

For official Minecraft APIs (`@minecraft/server`, `@minecraft/server-ui`, etc.), their type definition files (`.d.ts`) will be read directly. This allows for accurate understanding of available APIs, classes, methods, and event details, facilitating smooth development.
<!-- Minecraft の公式 API (`@minecraft/server`, `@minecraft/server-ui` など) に関しては、その型定義ファイル (`.d.ts`) を直接読み込みます。これにより、利用可能な API、クラス、メソッド、イベントの詳細を正確に把握し、開発を円滑に進めることができます。 -->

## Coding Conventions
<!-- コーディング規約 -->

To maintain code consistency throughout the project, please follow these conventions:
<!-- プロジェクト全体のコードの一貫性を保つため、以下の規約に従ってください。 -->

### 1. Naming Conventions
<!-- 1. 命名規則 -->

- **Classes, Interfaces, Type Aliases**: Use `PascalCase`.
<!-- **クラス、インターフェース、型エイリアス**: `PascalCase` を使用します。 -->

- **Functions, Methods, Variables**: Use `camelCase`.
<!-- **関数、メソッド、変数**: `camelCase` を使用します。 -->

- **Constants**: Use `UPPER_SNAKE_CASE`. Applies to globally shared immutable values.
<!-- **定数**: `UPPER_SNAKE_CASE` を使用します。グローバルに共有される不変の値に適用します。 -->

- **File Names**: Use `camelCase.ts`.
<!-- **ファイル名**: `camelCase.ts` を使用します。 -->

### 2. Design Patterns
<!-- 2. 設計パターン -->

- **State Management Classes (Manager)**: Classes that manage player-specific or global state will use the Singleton pattern or static maps to uniformly manage instances.
<!-- **状態管理クラス (Manager)**: プレイヤーごと、またはグローバルな状態を管理するクラスは、シングルトンパターンまたは静的マップを用いてインスタンスを統一的に管理します。 -->

- **Asynchronous Processing**: Large-scale operations will be implemented as chunked asynchronous processing using `system.runJob` and generator functions to prevent server performance degradation. This splits processing into multiple ticks, avoiding script timeouts (Watchdog).
<!-- **非同期処理**: 大規模な操作は、サーバーのパフォーマンス低下を防ぐため、`system.runJob` とジェネレータ関数を用いてチャンク単位の非同期処理として実装します。これにより、処理が複数のティックに分割され、スクリプトのタイムアウト（Watchdog）を回避します。 -->

### 3. Modules and Import/Export
<!-- 3. モジュールとインポート/エクスポート -->

- **Exports**: As a general rule, use named exports (`export const ...`, `export class ...`) instead of default exports (`export default`). This prevents naming inconsistencies during import.
<!-- **エクスポート**: デフォルトエクスポート (`export default`) ではなく、名前付きエクスポート (`export const ...`, `export class ...`) を原則として使用します。これにより、インポート時の名前の揺れを防ぎます。 -->
- **Imports**: Module imports should be grouped at the top of the file.
<!-- **インポート**: モジュールのインポートは、ファイルの先頭にまとめて記述します。 -->

### 4. Comments
<!-- 4. コメント -->

- **JSDoc**: For classes, methods, and complex functions referenced from other files, it is recommended to write JSDoc-style comments (`/** ... */`) explaining their functionality, parameters, and return values.
<!-- **JSDoc**: 他のファイルから参照されるクラス、メソッド、複雑な関数には、その機能、パラメータ、返り値を説明する JSDoc 形式 (`/** ... */`) のコメントを記述することを推奨します。 -->

## About Updating README Files
<!-- README ファイルの更新について -->

When updating README files, please update both `README.md` and `README.ja.md`.
<!-- README ファイルを更新する際は、`README.md` と `README.ja.md` の両方を更新してください。 -->
For each file, write the content in the corresponding language (`README.md` in English, `README.ja.md` in Japanese).
<!-- それぞれのファイルには、対応する言語（`README.md` には英語、`README.ja.md` には日本語）で内容を記述してください。 -->
Also, `README.md` must always include links to README files in other languages.
<!-- また、`README.md` には、必ず他の言語の README ファイルへのリンクを設置してください。 -->

## GitHub Operation Rules
<!-- GitHub 運用ルール -->

### 1. Branch Strategy
<!-- 1. ブランチ戦略 -->

- **main**: Always maintain a stable, release-ready state. Direct commits are prohibited; merges are only allowed via Pull Requests.
<!-- **main**: 常に安定した、リリース可能な状態を保ちます。直接のコミットは禁止し、Pull Request を通してのみマージします。 -->
- **develop**: Manages the currently developing version. Branches with completed feature development or bug fixes are merged here.
<!-- **develop**: 現在開発中のバージョンを管理します。機能開発やバグ修正が完了したブランチは、ここにマージします。 -->
- **feature/`issue-number`-`branch-name`**: Create for individual tasks such as new feature development or bug fixes.
<!-- **feature/`issue-number`-`branch-name`**: 新機能の開発やバグ修正など、個別のタスクごとに作成します。 -->
  - Example: `feature/12-add-sphere-brush`
  <!-- 例: `feature/12-add-sphere-brush` -->

### 2. Commit Messages
<!-- 2. コミットメッセージ -->

Commit messages should clearly describe the changes and follow this format:
<!-- コミットメッセージは、変更内容が明確にわかるように、以下の形式で記述します。 -->

```
<type>(<scope>): <subject>

<body>

<footer>
```

- **type**: Represents the type of commit.
<!-- **type**: コミットの種類を表します。 -->
  - `feat`: New feature addition
  <!-- `feat`: 新機能の追加 -->
  - `fix`: Bug fix
  <!-- `fix`: バグ修正 -->
  - `docs`: Documentation changes
  <!-- `docs`: ドキュメントの変更 -->
  - `style`: Code style changes (formatting, etc.)
  <!-- `style`: コードスタイルの変更（フォーマットなど） -->
  - `refactor`: Refactoring
  <!-- `refactor`: リファクタリング -->
  - `test`: Test additions/modifications
  <!-- `test`: テストの追加・修正 -->
  - `chore`: Build process or auxiliary tool changes
  <!-- `chore`: ビルドプロセスや補助ツールの変更 -->
- **scope** (optional): Indicates the scope affected by the change. (e.g., `commands`, `history`)
<!-- **scope** (任意): 変更が影響する範囲を示します。（例: `commands`, `history`） -->
- **subject**: A concise description of the change (within 50 characters).
<!-- **subject**: 変更内容の簡潔な説明（50 文字以内）。 -->

### 3. Pull Request (PR)
<!-- 3. Pull Request (PR) -->

- Merges to the `develop` branch must always be done via Pull Requests.
<!-- `develop` ブランチへのマージは、必ず Pull Request を通して行います。 -->
- The PR description should include the implemented content and related issue numbers (e.g., `Closes #12`).
<!-- PR の説明には、実装した内容、関連する Issue 番号（例: `Closes #12`）を記載します。 -->
- PRs must be reviewed by at least one person before merging.
<!-- PR は、少なくとも 1 人以上のレビューを経てからマージします。 -->

## Main Workflow
<!-- 主要なワークフロー -->

1.  **Entry Point**: `src/main.ts` is the main entry point for the add-on. Here, Minecraft event listeners are registered, and the add-on's initialization process takes place.
    <!-- **エントリポイント**: `src/main.ts` がアドオンの主要なエントリポイントです。ここで、Minecraft のイベントリスナーが登録され、アドオンの初期化処理が行われます。 -->
2.  **Event Handling**: Event listeners registered in `main.ts` capture player operations and in-game events (e.g., block destruction, item use, chat command execution).
    <!-- **イベント処理**: `main.ts` で登録されたイベントリスナーが、プレイヤーの操作やゲーム内のイベント（例: ブロックの破壊、アイテムの使用、チャットコマンドの実行など）を捕捉します。 -->
3.  **Feature Modules**: Captured events are dispatched to modules corresponding to their respective features (e.g., command processing, custom item logic), where specific processing is executed.
    <!-- **機能モジュール**: 捕捉されたイベントは、それぞれの機能に対応するモジュール（例: コマンド処理、カスタムアイテムのロジックなど）にディスパッチされ、具体的な処理が実行されます。 -->

## Build and Deployment
<!-- ビルドとデプロイ -->

Use the scripts defined in `package.json`:
<!-- `package.json` に定義されたスクリプトを使用します。 -->

- **`npm run build`**: Builds the TypeScript code and prepares it for the game.
<!-- **`npm run build`**: プロジェクトをビルドします。 -->

- **`npm run local-deploy`**: Builds the project and automatically deploys it to your Minecraft `development_packs` folder. This script watches for file changes and redeploys automatically.
<!-- **`npm run local-deploy`**: プロジェクトをビルドし、Minecraft の `development_packs` フォルダに自動的にデプロイします。このスクリプトはファイルの変更を監視し、自動的に再デプロイします。 -->

- **`npm run watch`**: An alias for `npm run local-deploy`.
<!-- **`npm run watch`**: `npm run local-deploy` のエイリアスです。 -->

- **`npm run mcaddon`**: Builds the project and creates a `.mcaddon` file in the `dist/packages` directory, making add-on installation and distribution easy.
<!-- **`npm run mcaddon`**: プロジェクトをビルドし、`dist/packages` ディレクトリに `.mcaddon` ファイルを作成します。これにより、アドオンのインストールや配布が容易になります。 -->

- **`npm run lint`**: Checks TypeScript code for style and quality issues.
<!-- **`npm run lint`**: コードの静的解析を実行します。 -->

- **`npm run clean`**: Removes temporary build files and directories.
<!-- **`npm run clean`**: 一時的なビルドファイルとディレクトリを削除します。 -->

## Differences in Commands Between OS
<!-- OS間のコマンド差異について -->

The Gemini CLI needs to use different shell commands depending on the user's OS environment (Windows, macOS, Linux). Particular attention should be paid to file operations and path handling, as problems are likely to occur here.
<!-- Gemini CLI は、ユーザーの OS 環境 (Windows, macOS, Linux) に応じて、実行するシェルコマンドを使い分ける必要があります。特に、ファイル操作やパスの扱いで問題が発生しやすいため、注意が必要です。 -->

### Problems Encountered and Solutions
<!-- 発生した問題と解決策 -->

1.  **Problem**: Japanese messages enclosed in double quotes, such as `git commit -m "message"`, may be garbled or not correctly recognized as arguments.
    <!-- **問題**: `git commit -m "メッセージ"` のように、ダブルクォートで囲んだ日本語メッセージが文字化けしたり、引数として正しく認識されなかったりする。 -->
    *   **Solution**: Write the commit message to a temporary file and use `git commit -F <file>`.
    <!-- **解決策**: コミットメッセージを一時ファイルに書き出し、`git commit -F <file>` を使用する。 -->

2.  **Problem**: The `pwd` command (display current directory) does not exist in Windows.
    <!-- **問題**: `pwd` コマンド (現在のディレクトリ表示) が Windows に存在しない。 -->
    *   **Solution**: In Windows, use the `cd` command (without arguments) to display the current directory.
    <!-- **解決策**: Windows では `cd` コマンド (引数なし) を使用して現在のディレクトリを表示する。 -->

3.  **Problem**: The `rm` command (file deletion) does not exist in Windows.
    <!-- **問題**: `rm` コマンド (ファイル削除) が Windows に存在しない。 -->
    *   **Solution**: In Windows, use the `del` command.
    <!-- **解決策**: Windows では `del` コマンドを使用する。 -->

We aim to accumulate this knowledge and achieve stable operations independent of the OS.
<!-- これらの知見を蓄積し、OS に依存しない安定した操作を目指します。 -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kaaariyaaa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kaaariyaaa)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
