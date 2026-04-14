## obsidian-encryptall-plugin

> "formatOnSave": true,

{
    "formatOnSave": true,
    "formatOnPaste": true,
    "defaultFormatter": "prettier",
    "typescript": {
        "preferences": {
            "importModuleSpecifier": "non-relative",
            "quoteStyle": "single"
        }
    },
    "javascript": {
        "preferences": {
            "importModuleSpecifier": "non-relative",
            "quoteStyle": "single"
        }
    },
    "search": {
        "exclude": [
            "**/node_modules/**",
            "**/dist/**",
            "**/.git/**",
            "**/build/**"
        ]
    },
    "editor": {
        "tabSize": 4,
        "insertSpaces": true,
        "detectIndentation": true,
        "trimTrailingWhitespace": true,
        "insertFinalNewline": true,
        "rulers": [100]
    },
    "files": {
        "exclude": {
            "**/.git": true,
            "**/node_modules": true,
            "**/dist": true,
            "**/build": true
        },
        "watcherExclude": {
            "**/.git/objects/**": true,
            "**/node_modules/**": true,
            "**/dist/**": true,
            "**/build/**": true
        }
    },
    "tasks": {
        "onSave": {
            "run": "npm run build",
            "exclude": [
                "**/node_modules/**",
                "**/dist/**",
                "**/.git/**",
                "**/build/**"
            ]
        }
    }
} 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shensmart) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
