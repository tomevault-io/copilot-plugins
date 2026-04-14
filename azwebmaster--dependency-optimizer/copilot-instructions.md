## dependency-optimizer

> API design patterns and public interface guidelines


# API Design Patterns

## Public API Structure

The main API is exported from [src/index.ts](mdc:src/index.ts) with these patterns:

### Class Exports
```typescript
export { DependencyScanner } from './scanner.js';
export { NodeModulesAnalyzer } from './analyzer.js';
export { PackageSizeAnalyzer } from './sizeAnalyzer.js';
export { DuplicateDetector } from './duplicateDetector.js';
export { LockFileParser } from './lockFileParser.js';
export { DependencyTreeBuilder } from './dependencyTreeBuilder.js';
```

### Type Exports
```typescript
export type {
  ScanOptions,
  AnalyzeOptions,
  ScanResult,
  AnalyzeResult,
  UnusedDependency,
  PackageAnalysis,
  PackageJson,
  WorkspaceConfig
} from './types.js';
```

### Convenience Functions
```typescript
export async function scanDependencies(projectPath?: string, options?: ScanOptions)
export async function analyzeNodeModules(projectPath?: string, options?: AnalyzeOptions)
export async function analyzePackageSizes(projectPath?: string, packageName?: string, options?: SizeAnalyzerOptions)
export async function detectDuplicates(projectPath?: string, options?: DuplicateDetectorOptions)
```

## Interface Design Principles

### Consistency
- **Use consistent naming** across all interfaces
- **Group related properties** logically
- **Provide optional parameters** with sensible defaults
- **Use descriptive property names**

### Error Handling
- **Return null/undefined** for missing data rather than throwing
- **Use try-catch** for recoverable errors
- **Provide meaningful error messages** with context
- **Log errors with debug** for troubleshooting

### Data Structures
- **Use Maps for lookups** by key (e.g., `Map<string, DependencyTreeNode[]>`)
- **Use arrays for ordered data** (e.g., dependency paths)
- **Provide both flat and tree structures** for different use cases
- **Include metadata** like depth, parent paths, etc.

## CLI Interface

The CLI in [src/cli.ts](mdc:src/cli.ts) follows these patterns:

### Command Structure
- **Use Commander.js** for argument parsing
- **Provide clear descriptions** for all commands and options
- **Support both positional and optional arguments**
- **Include help text** and examples

### Output Formatting
- **Use consistent formatting** across all commands
- **Provide verbose and quiet modes**
- **Support JSON output** for programmatic use
- **Include progress indicators** for long operations

## Parser Integration

### Unified Interface
- **Single entry point** through `LockFileParser`
- **Consistent return types** regardless of lock file format
- **Automatic format detection** based on available files
- **Graceful fallback** when parsing fails

### Data Conversion
- **Normalize to common format** (`DependencyInfo`)
- **Preserve original metadata** when possible
- **Handle version differences** in lock file formats
- **Calculate derived properties** (parent paths, depths)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/azwebmaster)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/azwebmaster)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
