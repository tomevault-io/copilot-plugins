## canai-cursor-codex-v6-1-4

> "trustScoreThreshold": 4.2,

{
  "trustScoreThreshold": 4.2,
  "aiProvider": "openai",
  "aiProviderConfig": {
    "apiKey": "${OPENAI_API_KEY}",
    "model": "gpt-4o"
  },
  "githubConfig": {
    "token": "${GITHUB_PAT}",
    "owner": "CanAISolutions",
    "repo": "canai-cursor-codex-v6.1.4",
    "baseBranch": "preboot"
  },
  "eslintConfig": {
    "enabled": true,
    "configFile": "eslint.config.js",
    "maxErrors": 5,
    "errorPenalty": 0.5
  },
  "rules": {
    "promptFormat": "markdown-commented",
    "fileNaming": "kebab-case",
    "tone": "codex-aligned, emotionally intelligent, operator voice",
    "commenting": "every function, every logic block, and every public constant",
    "linting": true,
    "branding": {
      "primaryColor": "#3A69E0",
      "accentColor": "#26D9C1",
      "font": "Inter",
      "tagline": "Empowerment Through Ease.",
      "logo": "canai-logo.svg"
    },
    "enforcement": {
      "rejectEmptyFiles": true,
      "requirePromptExamples": true,
      "requirePromptVersioning": true,
      "blockConsoleLogs": true,
      "warnOnUncommentedFunctions": true
    },
    "defaults": {
      "temperature": 0.7,
      "maxTokens": 1200,
      "model": "gpt-4o",
      "fallbackTone": "clear, strategic, emotionally intelligent"
    },
    "folderPreferences": {
      "prompts": {
        "requireNamedInputs": true,
        "requireMarkdownStructure": true
      },
      "scripts/tools": {
        "mustBeCLIReady": true,
        "requireHelpFlag": true
      },
      "gpt-templates": {
        "requireVersionSuffix": true
      }
    }
  }
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CanAISolutions) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
