## math-project-client

> 使用MCP README服务器获取相关文档

       在处理需求前，我将先查询 MCP README 服务获取相关文档内容:
       
       文件路径: {{currentFilePath}}
       模块名: {{fileBasename}}
       
       请先阅读以下相关文档，再开始编写代码:
       {{fetch('http://localhost:8374/api/find-readmes', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: { filePath: '{{currentFilePath}}', moduleName: '{{fileBasename}}' } })}}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iamtouchskyer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/iamtouchskyer)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
