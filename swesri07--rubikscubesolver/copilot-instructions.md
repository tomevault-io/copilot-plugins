## rubikscubesolver

> "maxLineLength": 100,

{
    "rules": {
        "typescript": {
            "indent": 4,
            "maxLineLength": 100,
            "quoteStyle": "single",
            "semicolons": true,
            "trailingComma": "es5"
        },
        "react": {
            "componentNaming": "PascalCase",
            "hookNaming": "camelCase",
            "propsNaming": "camelCase",
            "stateNaming": "camelCase"
        },
        "general": {
            "fileNaming": {
                "components": "PascalCase.tsx",
                "hooks": "use*.ts",
                "utils": "camelCase.ts",
                "types": "PascalCase.types.ts",
                "constants": "UPPER_SNAKE_CASE.ts"
            },
            "imports": {
                "order": [
                    "react",
                    "external",
                    "internal",
                    "types",
                    "styles"
                ]
            }
        },
        "testing": {
            "testFileNaming": "*.test.tsx",
            "coverage": {
                "statements": 80,
                "branches": 80,
                "functions": 80,
                "lines": 80
            }
        },
        "documentation": {
            "requireJSDoc": true,
            "requireComponentProps": true,
            "requireHookDocumentation": true
        },
        "performance": {
            "maxBundleSize": "250KB",
            "maxInitialLoadTime": "3s",
            "maxTimeToInteractive": "3.5s"
        },
        "accessibility": {
            "requireAltText": true,
            "requireAriaLabels": true,
            "requireKeyboardNavigation": true
        },
        "security": {
            "requireInputSanitization": true,
            "requireSecureCookies": true,
            "requireCSRFProtection": true
        }
    },
    "ignore": [
        "node_modules",
        "dist",
        "build",
        "coverage",
        "*.config.js",
        "*.config.ts"
    ]
} 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SweSri07) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
