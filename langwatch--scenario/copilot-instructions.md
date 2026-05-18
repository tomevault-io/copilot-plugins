## code-groups

> When creating code groups


When creating code groups in Vocs markdown files, you can only use the :::code-group::: directive
when writing the code inline. If you are using imported mdx files, you must use the CustomCodeGroup
component.

Examples:

Imported mdx file:

```typescript
<CustomCodeGroup>
  <CodeTab title="TypeScript">
    <SSETestExampleTS />
  </CodeTab>
  <CodeTab title="Python">
    <SSETestExamplePy />
  </CodeTab>
</CustomCodeGroup>
```

Inline code:

:::code-group
```typescript
// Your code here
```
```python
// Your code here
```
:::code-group

---
> Source: [langwatch/scenario](https://github.com/langwatch/scenario) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
