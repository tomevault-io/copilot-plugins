## xuanji

> "name": "玄机(XuanJi) - AI算命系统",

{
    "name": "玄机(XuanJi) - AI算命系统",
    "description": "基于Gemini的智能算命系统",
    "version": "1.0.0",
    "rules": {
        "references": {
            "code": {
                "style": ".rules/code/style.json",
                "structure": ".rules/code/structure.json"
            },
            "git": ".rules/git/commit.json",
            "testing": ".rules/testing/rules.json",
            "documentation": ".rules/docs/standards.json"
        },
        "fileNaming": {
            "components": "PascalCase.tsx",
            "pages": "kebab-case.tsx",
            "lib": "camelCase.ts",
            "types": "PascalCase.types.ts",
            "tests": "*.test.ts",
            "styles": "*.module.css"
        },
        "directories": {
            "src": {
                "description": "主要源代码目录",
                "subdirectories": {
                    "app": "Next.js应用主目录",
                    "components": "React组件目录",
                    "lib": "工具函数和API调用",
                    "styles": "样式文件",
                    "types": "TypeScript类型定义"
                }
            },
            "tests": "测试文件目录",
            "docs": "文档目录"
        },
        "files": {
            "src/pages/*.tsx": "Next.js页面组件",
            "src/pages/api/*.ts": "API路由处理函数",
            "src/components/*.tsx": "可复用的React组件",
            "src/lib/*.ts": "工具函数和服务",
            "src/styles/*.css": "样式文件",
            "*.test.ts": "测试文件",
            "*.md": "文档文件"
        },
        "codeStyle": {
            "typescript": {
                "strict": true,
                "indent": 2,
                "quotes": "single",
                "semicolons": true
            },
            "react": {
                "componentStyle": "functional",
                "propsInterface": true,
                "stateManagement": "hooks"
            },
            "imports": {
                "order": [
                    "react",
                    "next",
                    "external",
                    "internal",
                    "styles"
                ]
            }
        },
        "documentation": {
            "components": "必须包含Props接口定义和使用示例",
            "functions": "必须包含参数类型、返回值和用途说明",
            "api": "必须包含请求/响应类型和错误处理说明"
        }
    }
} 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zh4men9) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
