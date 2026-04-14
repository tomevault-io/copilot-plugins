## docs-lint

> This file provides guidance to Claude Code when working with the docs-lint package.

# CLAUDE.md

This file provides guidance to Claude Code when working with the docs-lint package.

## Package Overview

`@gu-corp/docs-lint` is a documentation consistency and quality linting tool for G.U.Corp projects. It validates markdown documentation structure, terminology consistency, and adherence to organizational standards.

## Installation (for other projects)

> **重要**: このパッケージはnpm registryには公開されていません。GitHubから直接インストールしてください。

```bash
# GitHubからインストール
npm install github:gu-corp/docs-lint --save-dev

# 特定バージョンをインストール
npm install github:gu-corp/docs-lint#v2.0.0 --save-dev
```

## Development Commands

```bash
npm install          # Install dependencies
npm run build        # Build TypeScript to dist/
npm run dev          # Watch mode for development
npm run lint         # Run ESLint
npm test             # Run Vitest tests
```

## CLI Usage (v2.0)

```bash
# Initialize configuration
docs-lint init                    # Interactive setup wizard
docs-lint init -y                 # Use defaults
docs-lint init --standards        # Also generate DOCUMENT_STANDARDS.md
docs-lint init --scaffold         # Also create folder structure

# Linting
docs-lint lint                    # Run all linting rules
docs-lint lint --fix              # Auto-fix markdown formatting issues
docs-lint lint -v                 # Verbose output
docs-lint lint --json             # JSON output
docs-lint lint --ai-prompt        # Generate AI-friendly assessment

# AIへの指示: lintで警告/エラーが出たら、まず --fix を試してください
# 自動修正可能な問題は --fix で解決できます。効率的です。

# Static checks
docs-lint check code              # Code checks (test files, coverage)
docs-lint check spec              # Spec checks (structure, references)

# AI reviews (generates prompt, use --api for direct API call)
docs-lint review code             # Code review prompt
docs-lint review spec             # Spec review prompt (includes MECE check)

# Display information
docs-lint show standards          # Show current standards
docs-lint show config             # Show current configuration
docs-lint show rules              # Show available lint rules
```

## Project Architecture

### Source Structure

```
src/
├── cli.ts              # CLI entry point (Commander.js)
├── linter.ts           # Main linter class
├── types.ts            # TypeScript types and default config
├── ai-prompt.ts        # AI prompt generation for assessments
├── rules/
│   ├── structure.ts    # Folder/file structure rules
│   ├── content.ts      # Content quality rules
│   └── terminology.ts  # Terminology consistency rules
└── templates/
    ├── standards.ts    # Default DOCUMENT_STANDARDS.md template
    └── translations/   # i18n translations (ja, en)
templates/
└── 04-development/     # Development standards templates
    ├── GIT-WORKFLOW.md
    ├── CI-CD.md
    ├── CODING-STANDARDS.md (planned)
    └── TESTING.md (planned)
```

### Key Components

- **Linter**: Core linting engine that runs configurable rules
- **Rules**: Modular rule implementations (structure, content, terminology)
- **AI Prompt**: Generates structured prompts for AI-based documentation review
- **Templates**: Embeds G.U.Corp standards for synchronization

## Design Decisions and History

### Background

This package was created to standardize documentation across G.U.Corp's 90+ repositories. The goal is to:
1. Enforce consistent folder structure (numbered folders like `01-overview/`)
2. Ensure terminology consistency across documents
3. Enable AI-assisted documentation review
4. Provide a single source of truth for development standards

### ESM Module System

The package uses ES modules (`"type": "module"`). Key considerations:
- Use `import` statements, not `require()`
- Dynamic imports for config files: `await import(\`file://\${path}\`)`
- File extensions required in imports: `./linter.js`

**Issue #1 Fix**: The CLI had `require()` calls that broke ESM compatibility. Fixed by:
- Moving `readStandardsFile` and `getDefaultStandards` to static imports
- Making `loadConfig` async with dynamic `import()`

### Folder Numbering Configuration

Folder numbering strictness is configurable per path:

```typescript
folderNumbering: {
  severity: 'warn',
  strictPaths: ['', '02-spec'],  // Root and 02-spec require numbering
  checkSequence: true,           // Check for gaps (01, 02, 03...)
}
```

Default behavior:
- Root-level folders must be numbered (`01-`, `02-`, etc.)
- `02-spec/` subfolders must be numbered
- Other subfolders are flexible

### i18n Support

Translations are stored in `src/templates/translations/`:
- `ja/` - Japanese (primary)
- `en/` - English

Standards can be generated in different languages.

### Standards Drift Detection

The package includes `standardsDrift` rule that detects when project standards differ from embedded templates:

1. **Templates**: Canonical standards stored in `templates/04-development/`
2. **Drift Detection**: Lint rule alerts when project files differ from templates

This approach ensures:
- AI can read standards from project's own docs
- Engineers have local copies to reference
- Differences are visible (but not auto-synced to preserve customizations)

## Configuration

### docs-lint.config.json

```json
{
  "docsDir": "./docs",
  "include": ["**/*.md"],
  "exclude": ["node_modules/**", "drafts/**"],
  "rules": {
    "folderNumbering": {
      "severity": "warn",
      "strictPaths": ["", "02-spec"],
      "checkSequence": true
    },
    "fileNaming": "warn",
    "requiredSections": "error",
    "terminology": "warn",
    "todoComments": {
      "severity": "warn",
      "tags": {
        "TODO": { "severity": "info" },
        "FIXME": { "severity": "warn" },
        "BUG": { "severity": "error" },
        "NOTE": { "severity": "off" }
      },
      "ignoreInlineCode": true,
      "ignoreCodeBlocks": true
    }
  },
  "terminology": [
    { "preferred": "ドキュメント", "variants": ["文書"] }
  ],
  "testing": {
    "projectType": "api",
    "coreLogicPatterns": [
      "src/domain/**/*.ts",
      "src/rules/**/*.ts"
    ],
    "coverageThresholds": {
      "coreLogic": 100,
      "utilities": 90,
      "api": 80,
      "ui": 60,
      "overall": 70
    },
    "requireIntegrationTests": ["src/api/**/*.ts"],
    "requireE2ETests": false,
    "requireCITests": true
  }
}
```

### Testing Configuration

`testing` セクションで、コードレビュー時のテスト評価基準をカスタマイズできます。

| プロパティ | 説明 | デフォルト |
|-----------|------|-----------|
| `projectType` | プロジェクト種別 (`library`, `api`, `web-app`, `cli`, `critical`) | `api` |
| `coreLogicPatterns` | コアロジックのGlobパターン（100%カバレッジ目標） | `src/domain/**`, `src/lib/**` 等 |
| `coverageThresholds` | カテゴリ別カバレッジ閾値 | 下記参照 |
| `requireIntegrationTests` | Integrationテスト必須のパターン | `src/api/**` 等 |
| `requireE2ETests` | E2Eテスト必須か | `false` |
| `requireCITests` | CIでのテスト設定必須か | `true` |

**デフォルトカバレッジ閾値**:
- コアロジック: 100%
- ユーティリティ: 90%
- API/コントローラ: 80%
- UI: 60%
- 全体: 70%

## Testing

Uses Vitest for fast ESM-native testing:

```bash
npm test              # Run all tests
npm test -- --watch   # Watch mode
```

## Commit Checklist

> **AIへの指示**: コミット前に必ず [templates/04-development/GIT-WORKFLOW.md](templates/04-development/GIT-WORKFLOW.md) のセクション5「コミット前チェックリスト」に従ってください。

### 機能追加・CLI変更時の必須対応

新機能やCLIオプションを追加した場合、コミット前に以下を必ず実行:

1. **README.md を更新**
   - Features セクションに新機能を追加
   - CLI Usage に新コマンド/オプションを追加
   - CLI Options テーブルを更新
   - 必要に応じて設定例を追加

2. **CLAUDE.md を更新**
   - CLI Usage セクションを更新

3. **テストとビルド確認**
   ```bash
   npm test
   npm run build
   ./dist/cli.js --help  # 新オプションが表示されることを確認
   ```

**これらが完了していない場合、コミットしないでください。**

## Release Process

1. Update version in `package.json`
2. Commit and push to main
3. Create tag: `git tag v2.x.x`
4. Push tag: `git push origin --tags`
5. GitHub Actions automatically creates release

## Pending Features

- [ ] Additional templates (CODING-STANDARDS.md, TESTING.md)

## v2.0 Changes

- Unified `init` command with `--standards` and `--scaffold` options
- Subcommand structure: `check code/spec`, `review code/spec`, `show standards/config/rules`
- Added MECE (Mutually Exclusive, Collectively Exhaustive) check to `review spec`
- Removed `sync` command (decided not to auto-sync to preserve customizations)
- Legacy commands still work with deprecation warnings (will be removed in v3.0)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gu-corp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gu-corp)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
