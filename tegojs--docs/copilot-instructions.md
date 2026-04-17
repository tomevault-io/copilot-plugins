## docs

> **CRITICAL REQUIREMENT:** All documentation changes MUST maintain Chinese-English consistency.

# Tachybase Documentation Translation Rules

## Primary Rule: Bilingual Consistency

**CRITICAL REQUIREMENT:** All documentation changes MUST maintain Chinese-English consistency.

### When Adding or Modifying Chinese Documentation:

1. **Mandatory English Translation:**
   - When creating a new file in `docs/zh/`, you MUST create the corresponding file in `docs/en/`
   - When modifying content in `docs/zh/`, you MUST update the corresponding file in `docs/en/`
   - The English translation should be created/updated in the SAME commit or pull request

2. **File Path Mapping:**
   ```
   docs/zh/guides/example.md  →  docs/en/guides/example.md
   docs/zh/api/example.md     →  docs/en/api/example.md
   ```
   The directory structure MUST be identical between `zh` and `en` folders.

3. **Translation Quality Standards:**
   - Use professional technical terminology
   - Maintain original formatting (headings, lists, code blocks, tables, links)
   - Preserve code examples as-is (do not translate code or variable names)
   - Keep image references and links functional
   - Maintain markdown syntax consistency

4. **Key Terminology Mapping:**
   - 灵矶 → Lingji (or Tachybase when referring to the platform)
   - 数据表 → Data table / Collection
   - 字段 → Field
   - 卡片 → Block
   - 工作流 → Workflow
   - 触发器 → Trigger
   - 节点 → Node
   - 关系字段 → Relationship field / Association field
   - 数据范围 → Data scope
   - 联动规则 → Linkage rules
   - 操作 → Action / Operation
   - 弹窗 → Modal / Popup / Drawer

### When Adding or Modifying English Documentation:

1. **Verify Chinese Source:**
   - If modifying `docs/en/`, ensure the Chinese version in `docs/zh/` is up-to-date
   - The Chinese version is considered the source of truth
   - If discrepancies exist, align with the Chinese version

### File Operations Workflow:

**Creating New Documentation:**
```bash
# Step 1: Create Chinese version
docs/zh/guides/new-feature.md

# Step 2: IMMEDIATELY create English version
docs/en/guides/new-feature.md
```

**Modifying Existing Documentation:**
```bash
# Step 1: Modify Chinese version
docs/zh/guides/existing-feature.md

# Step 2: IMMEDIATELY update English version
docs/en/guides/existing-feature.md
```

### Validation Checklist:

Before completing any documentation work, verify:

- [ ] Chinese file exists in `docs/zh/`
- [ ] English file exists in `docs/en/` with same relative path
- [ ] Both files have equivalent content (translated appropriately)
- [ ] All code blocks are preserved exactly
- [ ] All images, links, and references are functional in both versions
- [ ] Markdown formatting is consistent

### Exceptions:

The following can be language-specific:
- Configuration files (`_meta.json`, `_nav.json`)
- Build scripts and tooling
- README.md in repository root (can be English-only or bilingual)

### AI Assistant Instructions:

When working on documentation in this repository:

1. **Always check both language versions** before making changes
2. **Never modify only one language** - always update both `zh` and `en`
3. **Use the Chinese version as the source of truth** for content
4. **Translate technical content accurately** while maintaining readability
5. **Preserve all technical details, code examples, and formatting**
6. **Alert the user** if you find orphaned files (exists in only one language folder)

### Error Prevention:

**❌ WRONG:**
```bash
# Only creating/modifying Chinese version
docs/zh/guides/new-feature.md  ← Created
docs/en/guides/new-feature.md  ← Missing! This is an ERROR
```

**✅ CORRECT:**
```bash
# Creating/modifying both versions
docs/zh/guides/new-feature.md  ← Created/Modified
docs/en/guides/new-feature.md  ← Created/Modified (translated)
```

---

## Additional Documentation Standards:

### Markdown Best Practices:
- Use proper heading hierarchy (H1 → H2 → H3)
- Include code language tags in code blocks
- Use relative paths for internal links
- Optimize images before committing
- Follow the style guide in `contribution/markdown.md`

### Code Examples:
- Keep code blocks untranslated
- Translate code comments only if they contain important explanations
- Preserve variable names, function names, and API calls
- Include language tags (```typescript, ```bash, etc.)

### Images and Assets:
- Store in `docs/public/` directory
- Use descriptive file names
- Reference consistently in both language versions
- Include alt text for accessibility

---

**This rule is MANDATORY and non-negotiable for maintaining documentation quality and consistency across languages.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tegojs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
