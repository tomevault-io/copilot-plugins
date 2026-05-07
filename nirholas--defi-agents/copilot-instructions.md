## logging-system

> This project uses a custom logging system located in [scripts/utils/logger.ts](mdc:scripts/utils/logger.ts) for consistent, structured output across all scripts.

# Custom Logger System Guide

This project uses a custom logging system located in [scripts/utils/logger.ts](mdc:scripts/utils/logger.ts) for consistent, structured output across all scripts.

## 🚨 Critical Rule
**NEVER use `console.log`, `console.warn`, `console.error` directly in any scripts.** Always use the custom Logger.

## 📝 Logger Usage Patterns

### Basic Import
```typescript
import { Logger } from '../utils/logger';
// OR import specific functions
import { info, warn, error, success, start } from '../utils/logger';
```

### Common Logging Methods

#### Information and Status
```typescript
Logger.info('处理文件', '文件名或详情');
Logger.success('操作完成', '目标', '详细信息');
Logger.warn('警告信息', '原因或详情');
Logger.error('错误信息', '错误对象或详情');
```

#### Process Flow
```typescript
Logger.start('开始操作', '目标', '详细信息');
Logger.split('分隔线标题', '='); // 创建分隔线
```

#### File Operations
```typescript
Logger.file('read', 'path/to/file.json', 'success');
Logger.file('write', 'path/to/file.json', 'start');
Logger.file('delete', 'path/to/file.json', 'error');
```

#### Translation Process
```typescript
Logger.translate(agentId, '源语言', '目标语言', 'start');
Logger.translate(agentId, '源语言', '目标语言', 'success');
Logger.translate(agentId, '源语言', '目标语言', 'skip');
Logger.translate(agentId, '源语言', '目标语言', 'error');
```

#### Progress and Statistics
```typescript
Logger.progress(current, total, '操作描述');
Logger.stats({
  '总文件数': totalCount,
  '成功数量': successCount,
  '失败数量': failureCount
});
```

## 🎨 Logger Features

### Colored Output
- Uses `consola` with colored icons and formatting
- Different colors for different log levels
- Consistent visual hierarchy

### Structured Information
- Progress bars for long operations
- Statistical summaries
- File operation tracking
- Translation workflow logging

### Debugging Support
```typescript
Logger.debug('调试信息', debugData);
```

## 📋 Migration from console.*

### Old Pattern ❌
```typescript
console.log('Processing file:', fileName);
console.warn('File not found:', filePath);
console.error('Error occurred:', error);
```

### New Pattern ✅
```typescript
Logger.info('处理文件', fileName);
Logger.warn('文件未找到', filePath);
Logger.error('发生错误', error);
```

## 🔧 Command Line Tools

Commands like [scripts/commands/validate-language.ts](mdc:scripts/commands/validate-language.ts) demonstrate proper Logger usage:
- Use `Logger.split()` for section headers
- Use appropriate log levels for different message types
- Provide structured output with `Logger.stats()`
- Use `Logger.file()` for file operations

---
> Source: [nirholas/defi-agents](https://github.com/nirholas/defi-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
