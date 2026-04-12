## superadmin

> For public code, ALWAYS write jsdoc comments.

For public code, ALWAYS write jsdoc comments. 
To determine public code consider following AST selectors:
- ClassDeclaration
- ClassProperty
- FunctionDeclaration
- MethodDefinition
- ExportNamedDeclaration > VariableDeclaration
- TSDeclareFunction
- TSEnumDeclaration
- TSInterfaceDeclaration
- TSMethodSignature
- :not(TSTypeParameterDeclaration) TSPropertySignature
- TSTypeAliasDeclaration
Do not add unnecessary type declarations for jsdoc.
Do not remove existing eslind disable comments (like `// eslint-disable-next-line @typescript-eslint/no-explicit-any`), leave them under jsdoc as is.
Do not document internal functions defined in a service, command or component scope.

If function has multiple overloads, document each of them.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/superadmin-js)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/superadmin-js)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
